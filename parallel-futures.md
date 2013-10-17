# Parallel Futures

I’m leaning towards calling this `library(concur)`.  The name is short, it suggests concurrent computation and suggests a more limited scope than the name “futures” does.

In Alice ML `val x = spawn f` creates a spark (evaluating `f` in parallel) and returns immediately.  Subsequent code needing `x` blocks until the spark is finished.  This is a helpful abstraction for parallel computations.

Using macros and threads, it should be possible to support a similar abstraction in Prolog.  The goal is that to parallelize a working piece of code, one must only wrap a goal in `spawn/1` and all other code remains the same.  For example, macros rewrite this

```prolog
main(Url) :-
    spawn(http_get(Url, Html)),
    do_some_stuff,
    use_it(Html).
```

into this

```prolog
main(Url) :-
    queue_work( http_get(Url, Html), Future ),
    do_some_stuff,
    await_future( Future, http_get(Url, Html) ),
    use_it(Html).
```

`queue_work/2` creates a new thread pool (if necessary), creates a new queue on which to receive answers, places the goal into that pool’s work queue and creates Future which remembers the answer queue.  The macro places `await_future/2` as late as possible while still happening before the first use of a variable from the goal.

When the thread finds a solution, it wraps it in `fail/0`, `solution/1` or `final/1` to indicate failure, one solution out of several to come, or the final solution (respectively).  This allows `await_future/2` to create choicepoints so it can iterate all solutions.  If there are multiple solutions, the thread begins seeking them immediately after delivering the previous solution.

A main goal of parallel futures is to allow futures to behave exactly like normal variables so that parallel computation is transparent.  Unfortunately, the macro approach described above doesn’t allow a predicate to return a future.  In that case, we’d have to insert `await_future/2` as the last goal in a clause to convert a future back into a regular variable before returning.  That misses opportunities for parallelism since the caller may not need those results immediately.

Coroutines via `freeze/2` are almost ideal to work around this, but they only work when the future faces unification.  Arithmetic, output and many other predicates fail when working with attributed variables because they look like variables until after unification.  For example, we’d like `X is 1 + Future` to block until the value of Future is available, but it doesn’t.  It only behaves correctly for things like `Future=stuff`.  Consider an option (see below) that makes `await_future/2` create a coroutine instead of blocking.  When creating coroutines, be careful to handle backtracking and multiple solutions correctly.  Coroutines are clearly a niche optimization which should only be implemented later.

## Thread Policy

Different kinds of computation warrant different kinds of thread pools to perform the parallel computation.  The thread policy should be configurable based on the goal being spawned.  Here are some policies and examples where they’d be useful:

  * none
    * for: debugging
    * creates no threads. queuing work actually calculates all results.  await iterates them
  * lazy
    * for: goals whose value might not be needed at all
    * create no threads. the future stores the original goal and calls it during await
  * ephemeral threads
    * for: one-off computations where resource consumption needs no bounds
seeking a goal creates a single, new thread which finds all solutions then exits
dedicated thread pool with N threads
    * for: HTTP requests to limit concurrent network activity; disk IO
creates a pool of N threads dedicated to goals of a specific type. each thread stays alive to process many goals
  * shared, global thread pool with one thread per core
    * for: CPU bound computations
    * creates a pool of N threads which process goals of many different types. each thread stays alive to process many goals

By default, the shared pool is used.  However, there needs to be some mechanism by which a user can set policy on a per-goal basis.  `queue_work/2` consults this policy when deciding how to create a future for a given goal.  One approach would be to create a multifile hook `concur:thread_policy(+Module:Goal, -Policy)`.  If that predicate fails, use the shared thread pool; otherwise, use the first `Policy` returned.

This thread policy hook gives substantial flexibility.  For example, there could be a dedicated 2-thread pool for most HTTP requests, but a dedicated 4-thread pool for requests to google.com which can handle the extra load.  Because the hook receives both module and goal as input, one has fairly precise control over which goals are addressed.

It might be worth supporting `lazy/1` as an alias for `spawn/1` which mandates a lazy thread policy for this goal.

Don’t build on top of `library(thread_pool)` because it requires creating a new thread for each goal.  In most cases, I want the pool to hold persistent worker threads.
