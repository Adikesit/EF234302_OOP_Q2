# Q054 — Your answer

## (a) The eight lines

Hierarchy:
`Garment ← Shirt`, `Garment ← Towel ← BathTowel`, `Object` is supertype of all.
Helper signatures:
- `static double totalKg(List<? extends Garment> items)`
- `static void addSpareTowels(List<? super Towel> dst, int n)`
- `static <T> int transfer(Collection<? extends T> src, Collection<? super T> dst)`

Given:
`List<Garment> basket; List<Shirt> shirts; List<Towel> towels; List<BathTowel> bathTowels; List<Object> anything; List<? extends Garment> view = towels;`

1. **`totalKg(shirts)`**
   - **Compiles**: **Yes**.
   - **Why**: `totalKg` expects `List<? extends Garment>`. Since `Shirt` is a subtype of `Garment`, `List<Shirt>` is compatible with covariant wildcard `List<? extends Garment>`.
2. **`totalKg(anything)`**
   - **Compiles**: **No**.
   - **Why**: `anything` is `List<Object>`. `Object` is a supertype of `Garment`, not a subtype (`Object` does not extend `Garment`), so it violates the upper bound `? extends Garment`.
3. **`addSpareTowels(basket, 2)`**
   - **Compiles**: **Yes**.
   - **Why**: `addSpareTowels` requires `List<? super Towel>`. `basket` is `List<Garment>`, and `Garment` is a superclass of `Towel`, which satisfies the lower bound `? super Towel`.
4. **`addSpareTowels(bathTowels, 1)`**
   - **Compiles**: **No**.
   - **Why**: `bathTowels` is `List<BathTowel>`. `BathTowel` is a subtype of `Towel`, not a supertype. `List<? super Towel>` only accepts `Towel` or its supertypes (`Garment`, `Object`).
5. **`transfer(bathTowels, towels)`**
   - **Compiles**: **Yes**.
   - **Why**: Choosing type argument `T = Towel`: `src` (`List<BathTowel>`) satisfies `Collection<? extends Towel>` because `BathTowel` extends `Towel`; `dst` (`List<Towel>`) satisfies `Collection<? super Towel>` because `Towel` is super `Towel`.
6. **`transfer(towels, shirts)`**
   - **Compiles**: **No**.
   - **Why**: For `transfer` to compile, there must exist a type `T` such that `Towel` is a subtype of `T` (`Collection<? extends T>`) AND `Shirt` is a supertype of `T` (`Collection<? super T>`). That requires `Towel` to be a subtype of `Shirt`, which is false (they are sibling branches under `Garment`).
7. **`List<Garment> g = shirts;`**
   - **Compiles**: **No**.
   - **Why**: Java generics are invariant. Even though `Shirt` extends `Garment`, `List<Shirt>` is not a subtype of `List<Garment>`. If this were allowed, one could insert a `Towel` into `g` and corrupt `shirts`.
8. **`view.add(new Shirt("x"));`**
   - **Compiles**: **No**.
   - **Why**: `view` is typed as `List<? extends Garment>`. With an upper-bounded wildcard (`? extends Garment`), the compiler cannot know which specific subtype of `Garment` the underlying list actually holds (here it is `towels`). To maintain type safety, the compiler forbids adding any object (except literal `null`) to a collection with `? extends`.

---

## (c) The rule

### The PECS Rule:
**"Producer Extends, Consumer Super"** (from Joshua Bloch, *Effective Java*):
- If a parameterized type represents a **producer** (data is read from it), use `? extends T` (covariance).
- If a parameterized type represents a **consumer** (data is written into it), use `? super T` (contravariance).
- If a collection is both read from and written to, do not use wildcards (keep it invariant).

### Application to `transfer`:
`static <T> int transfer(Collection<? extends T> src, Collection<? super T> dst)`
- `src` is the **producer**: items are extracted (read) from `src` to be transferred. Therefore, it is declared with `Collection<? extends T>`, allowing any collection of `T` or subtypes of `T` to serve as the source.
- `dst` is the **consumer**: items are inserted (written) into `dst`. Therefore, it is declared with `Collection<? super T>`, allowing any collection capable of holding `T` (i.e. of `T` or supertypes of `T`) to serve as the destination.

---

## (d) Python

### Which calls does `mypy` reject:
In Python:
- `List[T]` is invariant in `mypy`.
  - Passing `List[Shirt]` to a function expecting `List[Garment]` is rejected by `mypy` because `List` is mutable and invariant.
  - Calling `add_spare_towels(basket, 2)` where `dst: MutableSequence[Towel]` is rejected by `mypy` when `basket: List[Garment]`, because `MutableSequence` is invariant.
  - Calling `transfer(towels, shirts)` is rejected by `mypy` because `Towel` is incompatible with `Shirt`.
  - Conversely, read-only interfaces such as `Sequence[T]` and `Iterable[T]` are defined as covariant (`Sequence[+T_co]`, `Iterable[+T_co]`), so `total_kg(shirts)` with `Sequence[Garment]` is accepted by `mypy`.

### How to express `? super Towel` in Python:
Python does not have use-site wildcard syntax like Java's `? super T`. Instead, Python uses **declaration-site variance**:
1. Define a contravariant type variable:
   ```python
   T_contra = TypeVar("T_contra", contravariant=True)
   ```
2. Create a generic consumer protocol (`Protocol`) that takes `T_contra` only as an input argument (write-only / sink):
   ```python
   class Sink(Protocol[T_contra]):
       def append(self, item: T_contra, /) -> None: ...
   ```
3. Use `Sink[Towel]` as the type annotation. Because `T_contra` is contravariant, `Sink[Garment]` and `Sink[object]` are valid subtypes of `Sink[Towel]`. Thus, passing a `List[Garment]` (which implements `append(Garment)`) to `Sink[Towel]` is accepted by `mypy`, achieving the exact semantics of Java's `List<? super Towel>`.
