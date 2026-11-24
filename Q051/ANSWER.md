# Q051 — Your answer

## (d) HashMap order vs Python dict order

### Why `map.keySet()` from a `HashMap` may come out in a different order:
1. **Hash Table Bucket Placement:** A `HashMap` in Java organizes entries into buckets based on the hash codes of keys (`hashCode()`), followed by an internal bit-masking/hash-spread calculation (`(n - 1) & hash`). 
2. **Implementation & Machine Differences:**
   - The exact bucket array capacity and rehashing thresholds can lead to different iteration sequences across different JVM versions or implementations.
   - `String.hashCode()` itself is fixed by the spec, but how `HashMap` spreads hashes, sizes its table and treeifies collisions is an implementation detail that has changed between versions (e.g. Java 7 → 8). Keys using the default `Object.hashCode()` can even get different hashes on every run. Either way the bucket order, and so the iteration order, can differ.
   - Crucially, `HashMap` makes **no guarantee** regarding iteration order; it does not guarantee that the order remains constant over time or across different platforms/JVM runs.
   - To guarantee sorted order, `TreeMap` should be used. To preserve insertion order, `LinkedHashMap` should be used.

### What Python `dict` guarantees instead:
- Since **Python 3.7+** (and CPython 3.6+ as an implementation detail), Python's standard `dict` **strictly guarantees insertion order**. Iterating over `dict` or `dict.keys()` will always yield the keys in the exact order in which they were first inserted.
- Note that this guarantee is **not alphabetical**; it reflects insertion sequence only. If keys need to be in alphabetical order, one must explicitly sort them (e.g., using `sorted(d.keys())`).
