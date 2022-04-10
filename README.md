# ixnay
nix in python proof-of-concept - calculate, invalidate, repeat

## Some thoughts so far

### Notes from Feb

git-graph-branch doesn't seem immediately compatible with Dataflow and reactive.

Key things that's different/difficult about git-graph-branch:

* DAG changes over time
* Invalidation signals are separate from data signals
* Shape of code is very sequential, not dataflow
* I think data can recombine? Not 100% sure

Want to:

1. Catch up on incoming invalidation data and determine everything that needs recalculating
2. From the top of the graph, rerun all the computations, reusing anything that has not been invalidated
3. Garbage-collect anything that wasn't reused
4. Discard and restart some nodes if the incoming invalidation data is swamping us
5. Asynchronously handle some data sources (network calls!), running them concurrently and merging in the results later
6. Auto-batch some data sources? e.g. one call to GitHub rather than multiple

1-3 is super similar to build systems!

This is not a simple async-and-forget system, because we want a more phased approach to scheduling the work. Potentially leverage async scheduler under the hood? And we probably need async functions to describe code doing 5 + 6.

Two kinds of live data source:

* Updated — new data is pushed in
* Invalidated — invalidation is pushed in, updates are explicit on-demand

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

`git graph-branch` is a good testbed, as it has a very complex and wide dependency DAG. Perhaps the best approach would be to spike a clean, non-nix, async, one-shot version, including hitting the github API, then try different ways of implementing `--watch` cleanly.
