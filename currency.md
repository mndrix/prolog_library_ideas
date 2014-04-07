# Currency

Make `library(currency)` (from PriceCharting) available as open source.  Teach it how to support different currencies based on their symbol (leading or trailing), separator (comma or full stop), and the number of decimals (Dollars, Yen and Bitcoin differ).  Perhaps these currency details should be configured via a hook.  So you'd end up with something like this:

```prolog
% define(Iso4217, Symbol, SymbolPosition, Thousands, Fraction, FractionSize)
:- multifile currency:define/6.
currency:define(usd, '$', leading, ',', '.', 2).
currency:define(jpy, '¥', leading, ',', '', 0).
currency:define(btc, 'BTC', trailing, ',', '.', 4).

?- atom_currency('$12.34', U).
U = usd(1234).

?- atom_currency(Y, jpy(1234)).
Y = '¥1234'.

?- atom_currency(B, btc(1234)).
B = '0.1234 BTC'.
```

There should be variants for other text types: `string_currency/2` and `codes_currency/2`.  There should also be a DCG equivalent: `dcg_currency//1`.

Should we also specify a pip size?  That might be helpful for currency traders.
