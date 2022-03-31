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
