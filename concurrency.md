I’m leaning towards calling this `library(concur)`.  The name is short, it suggests concurrent computation and suggests a more limited scope than the name “futures” does.  Although "concur" means agree, which could be confusing.  Maybe choose a random noun as the name and be done with it.  Dr. Suess is always good for inspiration.

# Explicit Concurrency

SWI Prolog [message queues](http://www.swi-prolog.org/pldoc/man?section=threadcom) give us a good concurrency primitive.  They're very similar to Go's channels.  Working with them, as with Go channels, becomes a little too verbose in the common case.  I'd like a higher-level interface.

```prolog
main(Url) :-
    async(http_get(Url,Html), Ch),
    do_some_stuff,
    await(Ch),  % binds Html when it's ready
    use_it(Html).
```

Calling `async(:Goal,-Token)` creates a message queue, captures all unbound variables in `Goal` and starts seeking `Goal` in a separate thread (or thread pool).  `await(+Token)` blocks until `Goal` has produces a solution; backtracking iterates solutions.  `await/1` communicates solutions to its caller by binding the formerly unbound variables in `Goal`.

When the thread finds a solution, it wraps it in `fail/0`, `solution/1` or `final/1` to indicate failure, one solution out of several to come, or the final solution (respectively).  This allows `await/1` to create choicepoints so it can iterate all solutions.

# Futures

In Alice ML `val x = spawn f` creates a spark (evaluating `f` in parallel) and returns immediately.  Subsequent code needing `x` automatically blocks until the spark is finished.  This is a helpful abstraction for parallel computations.

Using macros, `async/2` and `wait/1`, I should be possible to support a similar abstraction in Prolog.  A working piece of code is made concurrency by wrapping a goal in `spawn/1`.  All other code remains the same.  For example, macros rewrite this

```prolog
main(Url) :-
    spawn(http_get(Url, Html)),
    do_some_stuff,
    use_it(Html).
```

into this

```prolog
main(Url) :-
    async( http_get(Url, Html), Ch ),
    do_some_stuff,
    await(Ch),
    use_it(Html).
```

The macro places `await/1` as late as possible while still happening before the first use of a variable from the goal.

A main goal of parallel futures is to allow futures to behave exactly like normal variables so that parallel computation is transparent.  Unfortunately, the macro approach described above doesn’t allow a predicate to return a future.  In that case, we’d have to insert `await/1` as the last goal in a clause to convert a future back into a regular value before returning.  That misses opportunities for parallelism since the caller may not need those results immediately.

Coroutines via `freeze/2` are almost ideal to work around this, but they only work when the future faces unification.  Arithmetic, output and many other predicates fail when working with attributed variables because they look like variables until after unification.  For example, we’d like `X is 1 + Future` to block until the value of Future is available, but it doesn’t.  It only behaves correctly for things like `Future=stuff`.  Consider an option (see below) that makes `await/1` create a coroutine instead of blocking.  When creating coroutines, be careful to handle backtracking and multiple solutions correctly.  Coroutines are clearly a niche optimization which should only be implemented later.

## Thread Policy

Different kinds of computation warrant different kinds of thread pools to perform the parallel computation.  The thread policy should be configurable based on the goal being spawned.  Here are some policies and examples where they’d be useful:

  * none
    * for: debugging
    * creates no threads. queuing work actually calculates all results.  wait iterates them
  * lazy
    * for: goals whose value might not be needed at all
    * create no threads. the future stores the original goal and calls it during wait
  * ephemeral threads
    * for: one-off computations where resource consumption needs no bounds
seeking a goal creates a single, new thread which finds all solutions then exits
dedicated thread pool with N threads
    * for: HTTP requests to limit concurrent network activity; disk IO
creates a pool of N threads dedicated to goals of a specific type. each thread stays alive to process many goals
  * shared, global thread pool with one thread per core
    * for: CPU bound computations
    * creates a pool of N threads which process goals of many different types. each thread stays alive to process many goals
    * ideally, performing a blocking operation on a thread adds a new thread to the pool so we still use max CPU possible

By default, the shared pool is used.  However, there needs to be some mechanism by which a user can set policy on a per-goal basis.  `async/2` consults this policy when deciding how to create a future for a given goal.  One approach would be to create a multifile hook `concur:thread_policy(+Module:Goal, -Policy)`.  If that predicate fails, use the shared thread pool; otherwise, use the first `Policy` solution.

This thread policy hook gives substantial flexibility.  For example, there could be a dedicated 2-thread pool for most HTTP requests, but a dedicated 4-thread pool for requests to google.com which can handle the extra load.  Because the hook receives both module and goal as input, one has fairly precise control over which goals are addressed.

It might be worth supporting `lazy/1` as an alias for `spawn/1` which mandates a lazy thread policy for this goal.  Maybe both are aliases for `async/3` which accepts an options list.  One of the options specifies thread policy.

Don’t build on top of `library(thread_pool)` because it requires creating a new thread for each goal.  In most cases, I want the pool to hold persistent worker threads.
