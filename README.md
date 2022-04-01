# ixnay
nix in python proof-of-concept - calculate, invalidate, repeat

## Some thoughts so far

### Transitioning between sync and async

async (one-shot + generator) -> sync

* initial invocation throws `ixnay.Pending`
* provision of an async result nixes the outer computation
* caching needed to avoid losing the result(s)
* convenience alternative: return a placeholder (default None) instead of throwing
  * useful for e.g. UI code that will render a placeholder visual (skeleton, spinner, etc.)

sync -> async one-shot

* future blocks until a (non-`ixnay.Pending`) result and/or exception occurs
* all thrown `ixnay.Pending` thrown while awaiting will be silently dropped
* any subsequent `ixnay.Pending` or new result nixes the outer computation

sync -> async generator

* generator blocks until a (non-`ixnay.Pending`) result and/or exception occurs
* all thrown `ixnay.Pending` thrown while awaiting will be silently dropped
* the generator can be repeatedly awaited for fresh results
* if the outer computation discards the generator, any subsequent `ixnay.Pending` or new result nixes it

### Aggregation

Aggregate data structures, where a lot of different, independent calculations are pulled together, and then transformed (e.g. into a UI) *could* be redone on every nix, but it would be better to be incremental. React and its ilk are a great place to look for techniques.

* components nest, inner components can change without altering outer components
* shadow DOM rebuilds the UI from scratch each time, but then uses hashes to decide what has actually changed

`git graph-branch` is a good testbed, as it has a very complex and wide dependency DAG. Perhaps the best options would be to spike a non-nix version, then try different approaches to implementing `--watch` cleanly.
