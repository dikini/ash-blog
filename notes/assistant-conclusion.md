That gives us a useful architectural commitment:

Ordinary Ash code can execute during compilation to construct, discover, manipulate, and submit evidence for propositions. Its execution is sandboxed, and its results are subject to Ash's verification rules.

This separates three things without requiring three languages:

1. Expression: Ordinary Ash syntax and semantics, reused at compile time.
2. Execution: A compile-time environment with restricted capabilities.
3. Certification: A validation process that determines whether the produced evidence establishes the proposition.

The key distinction is that executing proof-producing code successfully is not the same as proving its target proposition.

For example, conceptually:

```
property associative (op : a -> a -> a) =
    forall x y z .
        op (op x y) z == op x (op y z)

proof associative =
    -- Ordinary Ash code, executed at compile time
    -- Produces evidence for the proposition
    ...
```

This also preserves an important freedom: proof-producing code can use algorithms, abstractions, recursion, and potentially permitted effects without forcing the underlying proposition or its accepted proof certificate to have the same computational semantics.

I would therefore provisionally distinguish:

- Proposition language: The logical interpretation of Ash expressions.
- Proof construction: Ordinary Ash computation at compile time.
- Proof validation: The trusted judgment over the resulting evidence.

The first two may share almost all their syntax and much of their implementation. The third establishes the trust boundary.

One consequence worth preserving is that `proof` need not denote a special kind of function. It could instead be a declaration that establishes a compile-time obligation and associates an Ash computation with its discharge.

That fits our emerging model of propositions, obligations, and evidence while leaving the other complications aside.

I think we now have a coherent starting point for examining the universe and kind structure without prematurely committing to full propositions-as-types.