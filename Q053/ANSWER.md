# Q053 — Your answer

## (a) Underlying Collection Choice & Justification
- **Underlying Collection:** `Map<T, Integer>` implemented as `LinkedHashMap<T, Integer>` in Java (and standard `dict` in Python 3.7+).
- **Justification:**
  - A multiset requires mapping each unique element to its positive multiplicity count.
  - Using a map gives $O(1)$ average time complexity for lookup (`count`), insertion (`add`), and deletion (`remove`).
  - `LinkedHashMap` preserves insertion order, which is directly required by `mostCommon()` to break ties in favor of the element added first.
  - `counts.size()` gives `distinct()` in $O(1)$ time, avoiding duplicate keys.

## (d) Type checking in Java vs Python
- **In Java:**
  - `Bag<String> g = new Bag<>(); g.add(42);`
  - The Java compiler (`javac`) rejects this code at compile time with a compilation error: `incompatible types: int cannot be converted to java.lang.String`.
  - Java enforces strict static type checking before generating bytecode. Due to type erasure, generics do not exist at runtime, so the check happens entirely during compilation.
- **In Python:**
  - In Python, type annotations (e.g. `Bag[str]`) are purely informational at runtime and are ignored by the standard Python runtime interpreter.
  - Therefore, `g: Bag[str] = Bag(); g.add(42)` executes without any error at runtime in standard Python, and `42` is simply inserted into the dictionary.
  - The mistake is only caught if a static type checker like `mypy` or `pyright` is run separately before execution, where `mypy` will report an argument type error (e.g., `Argument 1 to "add" of "Bag" has incompatible type "int"; expected "str"`).
