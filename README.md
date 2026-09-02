# carp-collections

A consolidated Collections Library suite for the Carp programming language.

## Modules Included
- **[Bitflags](#bitflags)**: See module documentation below.
- **[Bitset](#bitset)**: See module documentation below.
- **[Deque](#deque)**: See module documentation below.
- **[Handle](#handle)**: See module documentation below.
- **[Hashset](#hashset)**: See module documentation below.
- **[Priority-Queue](#priority-queue)**: See module documentation below.
- **[Ringbuf](#ringbuf)**: See module documentation below.
- **[Sparseset](#sparseset)**: See module documentation below.
- **[Stablepool](#stablepool)**: See module documentation below.
- **[Stack](#stack)**: See module documentation below.

## Installation

```
(load "git@github.com:carpentry-org/carp-collections@master")
```

That pulls in every module. A single one can be loaded on its own:

```
(load "git@github.com:carpentry-org/carp-collections@master" "deque.carp")
```

## Examples

See [examples.md](examples.md) for module usage examples, and the
[API documentation](https://carpentry.dev/carp-collections) for the full
reference.

---

## Deque

A robust, generic double-ended queue (deque) for the [Carp language](https://github.com/carp-lang/Carp).

Implemented as a **Dynamic Ring Buffer**, this module provides $O(1)$ amortized performance for both front and back operations.

## Features

- **Generic Type Support:** Works with any Carp type.
- **Double-Ended:** Efficient `push-front!`, `pop-front!`, `push-back!`, and `pop-back!`.
- **Dynamic Growth:** Automatically resizes and unrolls when the capacity is reached.
- **Random Access:** $O(1)$ `nth` and `unsafe-nth` access.
- **Tiered API:** Safe (`Maybe`), Enforced (`panic!`), and Fast-Path (`unsafe-`) variants.
- **Functional Utilities:** Built-in support for `map`, `filter`, `reduce`, `any?`, `all?`, and `doall`.
- **Standard Interfaces:** Implements `str`, `=`, `zero`, and `count`.

## Design Philosophy & Safety

The Deque module is designed for performance-critical systems:

1. **Amortized Efficiency:** Both FIFO (queue) and LIFO (stack) patterns run in amortized $O(1)$ time.
2. **Safety Layering:** Query functions use `Maybe` to avoid crashes. Mutation primitives enforce invariants via `panic!`.
3. **The Unsafe Path:** `unsafe-` variants are provided for maximum performance in trusted loops.
4. **Memory Management:** Minimizes reallocations and uses an efficient "unrolling" strategy during growth.

## Installation

```clojure
(load "git@github.com:carpentry-org/carp-collections@master" "deque.carp")
(use Deque)
```

## Usage

### Basic Queue (FIFO)

```clojure
(use Deque)

(let [q (Deque.new)]
  (do
    (push-back! &q 1)
    (push-back! &q 2)
    (IO.println &(peek-front &q)) ; Just 1
    (pop-front! &q)               ; Just 1
    (IO.println &(length &q))))    ; 1
```

### Random Access

```clojure
(let [dq (Deque.from-array [10 20 30])]
  (do
    (IO.println &(nth &dq 1)) ; Just 20
    (push-front! &dq 5)
    (IO.println &(nth &dq 1)))) ; Just 10
```

## Running Tests

```bash
carp -x test/deque_test.carp
```

## Examples

See [examples.md](examples.md) for usage examples.

## License

MIT


---

## Priority-Queue

A high-performance, generic Priority Queue (Binary Heap) for the [Carp](https://github.com/carp-lang/Carp) programming language.

## Features

- **Generic**: Works with any type that can be compared.
- **Flexible**: Can be used as a Max-Heap or Min-Heap by providing the appropriate comparison function.
- **Efficient**: $O(\log n)$ insertion and extraction.
- **$O(n)$ Heapify**: Build a heap from an existing array in linear time.
- **Iterative Implementation**: High-performance bubbling logic without recursion overhead.
- **Zero Allocations in Hot Paths**: Reuses the underlying array.


## Examples

See [examples.md](examples.md) for usage examples.
### Min-Heap

To use as a Min-Heap, just use the `<` comparison:

```carp
(push &pq 10 &IntRef.<)
(push &pq 30 &IntRef.<)
(pop &pq &IntRef.<) ;; Returns 10
```

### Efficient Replacement

If you need to replace the top element (e.g., in some pathfinding algorithms), use `pop-push`:

```carp
(pop-push &pq new-val &IntRef.>) ;; Faster than a pop followed by a push
```

## Testing

Run the test suite with:

```bash
carp -x test/pq_test.carp
```

## License

MIT


---

## Ringbuf

A generic fixed-capacity circular buffer for the [Carp language](https://github.com/carp-lang/Carp).

## Features

- **Generic Support:** Store any Carp type.
- **Fixed Capacity:** Pre-allocate memory for performance and predictability.
- **Multiple Push Modes:**
    - `push!`: Automatically overwrites the oldest element when full (standard ring buffer behavior).
    - `push-strict!`: Asserts that the buffer is not full.
    - `try-push!`: Returns `true` if successful, `false` if full (no overwrite).
- **Deque-like Operations:** `pop!` (front) and `pop-back!`.
- **Random Access:** `get` for relative indexing (0 is oldest).
- **Order Preserving:** Access elements in FIFO order.
- **Conversion:** Convert the buffer to a standard Carp Array with `to-array`.
- **Iteration:** Efficiently traverse elements with `foreach`.

## Installation

```clojure
(load "git@github.com:carpentry-org/carp-collections@master" "ringbuf.carp")
(use RingBuf)
```


## Examples

See [examples.md](examples.md) for usage examples.
## Running Tests

```bash
carp -x test/ringbuf_test.carp
```

## License

MIT


---

## Sparseset

A high-performance Sparse Set implementation for the [Carp language](https://github.com/carp-lang/Carp).

Sparse sets are a vital data structure for Entity Component Systems (ECS) and other high-performance systems. They provide $O(1)$ lookup, insertion, and deletion while maintaining contiguous storage for $O(n)$ cache-friendly iteration.

## Features

- **Contiguous Iteration**: Elements are stored packed in a "dense" array for maximum cache locality.
- **Fast Lookup**: $O(1)$ check for membership and retrieval via a "sparse" index array.
- **Constant-Time Clear**: Reset the entire set in $O(1)$ by just zeroing the length.
- **Zero Overhead**: Minimal memory footprint and no hidden allocations during operation.
- **Foundation Grade**: Designed for use in massive-scale simulations and game engines.

## Design Philosophy

The SparseSet module follows the "Golden Standard" for ECS storage:

1. **Memory Discipline**: Pre-allocates or grows the sparse and dense arrays as needed.
2. **Stable Iteration**: Deleting an element moves the last element into its place, preserving the contiguous property while potentially changing order.
3. **Safety Layering**: Provides checked random access and unmasked performance paths.

## Installation

```clojure
(load "git@github.com:carpentry-org/carp-collections@master" "sparse_set.carp")
(use SparseSet)
```


## Examples

See [examples.md](examples.md) for usage examples.
## Running Tests

```bash
carp -x test/sparse_set_test.carp
```

## License

MIT


---

## Stablepool

A high-performance, generational object pool (arena) for the [Carp language](https://github.com/carp-lang/Carp).

This module provides $O(1)$ allocation and deallocation of objects while providing safe, stable references via generational handles.

## Features

- **Generational Safety**: Uses `Handle` with `Uint32` generation counters to ensure stale references are caught with zero ambiguity.
- **High Performance**: $O(1)$ constant-time allocation and deallocation using an internal free-list.
- **Memory Efficient**: Backed by a contiguous array to minimize fragmentation, with a tiered slot model that separates occupancy from generation state.
- **Robust Invariants**: Strictly maintains length and free-list state, with program aborts for critical internal violations (e.g. double-free).
- **Type-Safe**: Generic implementation allows pooling any Carp type.
- **Zero Dependencies**: Pure Carp implementation requiring only `handle.carp`.

## Design Philosophy

The StablePool is designed for foundation-grade resource management:

1. **Stable Indices**: Unlike a regular array, removing an element does not shift others. Handles remain valid until the object is explicitly freed.
2. **Generational Hardening**: Every slot tracks its generation using an unsigned 32-bit counter. Comparisons are exact and wraparound is documented.
3. **Safety Layering**: Isoles `Array.unsafe-nth` behind internal checked boundaries. Random access via handles is fully validated.
4. **Pragmatic Utilities**: Provides O(n) iteration and reduction, and a debug-only `contains?` helper.

## Installation

```clojure
(load "git@github.com:carpentry-org/carp-collections@master" "stable_pool.carp")
(use StablePool)
```


## Examples

See [examples.md](examples.md) for usage examples.
## Running Tests

```bash
carp -x test/stable_pool_test.carp
```

## License

MIT


---

## Stack

A robust, generic stack data structure for the [Carp language](https://github.com/carp-lang/Carp).

## Features

- **Generic Type Support:** Works with any Carp type.
- **In-Place Mutation:** Memory-efficient `push!`, `pop!`, `swap!`, `reverse!`, and `remove-at!`.
- **Forth-style Primitives:** Advanced stack manipulation with `over!`, `rot!`, `nip!`, and `tuck!`.
- **High Performance:** Pointer-based inspection with `peek-ptr` to avoid unnecessary copies, plus `unsafe-` variants for zero-overhead primitives.
- **Functional Utilities:** Built-in support for `map`, `filter`, `reduce`, `any?`, `all?`, `find`, and `find-index`.
- **Safe API:** Operations return `Maybe` types to handle empty stacks gracefully.
- **Collection Standards:** Implements `str`, `=`, `zero`, and `doall`.

## Design Philosophy & Safety

The Stack module is designed for systems programming and VM implementations:

1. **Inspection/Query:** Functions that check state (like `peek`, `find`, `top-index`) return `Maybe` types. This ensures you can't accidentally access data in an empty stack without explicit handling.
2. **Mutation with Invariants:** Mutating primitives (like `dup!`, `swap!`, `over!`) enforce depth requirements. If these invariants are violated, the module will call `panic!` (aborting the program). This prevents silent stack corruption in performance-critical code.
3. **The Unsafe Path:** Every panicking primitive has an `unsafe-` counterpart (e.g., `unsafe-swap!`). These skip all checks and provide maximum performance when you can guarantee stack depth.
4. **Pointer Lifetime:** `peek-ptr` provides direct access to stack elements. Be aware that the returned pointer may become invalid if the stack is mutated in a way that triggers a reallocation of the underlying array.

## Installation

```clojure
(load "git@github.com:carpentry-org/carp-collections@master" "stack.carp")
(use Stack)
```

## Usage

### Basic Operations

```clojure
(use Stack)

(let [s (Stack.new)]
  (do
    (push! &s 1)
    (push! &s 2)
    (IO.println &(peek &s))  ; Just 2
    (pop! &s)                ; Just 2
    (IO.println &(length &s)))) ; 1
```

### Forth-style Manipulation

```clojure
(let [s (Stack.from-array [1 2])]
  (do
    (over! &s) ; [1 2 1]
    (swap! &s) ; [1 1 2]
    (drop! &s) ; [1 1]
  ))
```

### Functional Patterns

```clojure
(let [s (Stack.from-array [1 2 3])
      doubled (Stack.map &(fn [x] (* @x 2)) &s)]
  (IO.println &(Stack.str &doubled))) ; Stack([2 4 6])
```

## Running Tests

```bash
carp -x test/stack_test.carp
```

## Examples

See [examples.md](examples.md) for usage examples.

## License

MIT


---

## Hashset

A robust, generic hash set library for the [Carp language](https://github.com/carp-lang/Carp).

This module provides an idiomatic set abstraction implemented as a high-performance wrapper around Carp's built-in `Map`.

## Features

- **Generic Type Support**: Works with any type that implements `hash` and `=`.
- **Algebraic Operations**: optimized `union`, `intersection`, `difference`, and `symmetric-difference` using `kv-reduce`.
- **Tiered API**: Safe inspection, checked mutation, and functional composition.
- **Zero-Value Integration**: Works with `zero` and `empty?` interfaces.
- **Efficient**: Thin wrapper over the optimized `Map` core, avoiding unnecessary allocations in algebraic operations.

## Design Philosophy

The HashSet module is designed to feel like a first-class collection:

1. **Pragmatic Wrapping**: Uses unit values `()` in the underlying `Map` to minimize memory overhead.
2. **Set Semantics**: Focuses on uniqueness and membership.
3. **High Performance**: Uses `kv-reduce` for set operations to minimize temporary allocations.

## Installation

```clojure
(load "git@github.com:carpentry-org/carp-collections@master" "hash_set.carp")
(use HashSet)
```

## Examples

See [examples.md](examples.md) for usage examples.

## Running Tests

```bash
carp -x test/hash_set_test.carp
```

## License

MIT

---

**Note**: This module currently relies on a bugfix in `Map.put!` to correctly handle length increments during updates. If you encounter incorrect lengths when using duplicate keys, please ensure you are using a version of Carp with the fix applied (see [PR #1557](https://github.com/carp-lang/Carp/pull/1557)).


---

## Handle

A foundational generational handle library for the [Carp language](https://github.com/carp-lang/Carp).

This module provides a type-safe way to reference resources in a pool or array without using raw pointers.

## Features

- **Generational Safety**: Detect if a handle is stale because the underlying resource was deleted or replaced.
- **Generic Type-Tagging**: Handles are tagged with the type they reference to prevent mixing handles of different pools.
- **Zero Overhead**: Minimal memory footprint (typically two integers).
- **Foundation Grade**: Designed for use in high-performance engines and ECS architectures.

## Design Philosophy

The Handle module solves the "Dangling Pointer" problem in a systems environment:

1. **Uniqueness**: A handle is a combination of an `index` and a `generation`.
2. **Detection**: When a resource is replaced, its generation is incremented. Old handles will fail a generation check.
3. **Ergonomics**: Provides standard comparison and string conversion logic.

## Installation

```clojure
(load "git@github.com:carpentry-org/carp-collections@master" "handle.carp")
(use Handle)
```

## Examples

See [examples.md](examples.md) for usage examples.

## Running Tests

```bash
carp -x test/handle_test.carp
```

## License

MIT


---

## Bitflags

A robust, type-safe bitflags library for the [Carp language](https://github.com/carp-lang/Carp).

This module provides an idiomatic way to manage sets of bitmask flags using integer wrappers.

## Features

- **Type-Safe Wrappers**: Encapsulate raw integers into meaningful flag types.
- **Precise Semantics**: Distinguishes between `contains?` (all) and `intersects?` (any).
- **Full Set Logic**: Support for `union`, `intersection`, `difference`, `symmetric-difference`, and `subset?`.
- **Validation Helpers**: Includes `power-of-two?` and `single-bit?` for mask integrity.
- **Fluent API**: Set, unset, toggle, and check flags with ease.
- **Functional Support**: `with` and `without` for non-mutable compositions.
- **Zero Overhead**: Compiles down to standard C bitwise operations.
- **Hybrid Power-Macro**: Define flags with auto-incrementing powers of two or explicit `(Flag Value)` overrides.

## Design Philosophy

The BitFlags module is designed for performance and clarity:

1. **Inspection/Query**: Functions like `contains?`, `intersects?`, and `empty?` return `Bool`.
2. **Mutation**: `set!`, `unset!`, and `toggle!` perform in-place bitwise modifications.
3. **Functional Composition**: `with`, `union`, etc., return new `BitFlags` instances.
4. **Macro Integrity**: The `bitflags` macro uses a pure syntax-transformation approach for maximum compiler compatibility.

## Installation

```clojure
(load "git@github.com:carpentry-org/carp-collections@master" "bitflags.carp")
(use BitFlags)
```

## Examples

See [examples.md](examples.md) for usage examples.

## Running Tests

```bash
carp -x test/bitflags_test.carp
```

## License

MIT


---

## Bitset

A high-performance dynamic bitset for the [Carp language](https://github.com/carp-lang/Carp).

Bitsets provide extremely space-efficient storage for boolean flags and enable fast set-algebraic operations via bitwise logic.

## Features

- **Dynamic Growth**: Automatically expands to accommodate any bit index.
- **Set Algebra**: High-speed `union`, `intersection`, `difference`, and `symmetric-difference`.
- **Efficient Membership**: $O(1)$ check, set, and clear.
- **Optimized Iteration**: Fast iteration over set bits using bit manipulation.
- **Memory Efficient**: Backed by `Uint64` blocks to minimize overhead.

## Installation

```clojure
(load "git@github.com:carpentry-org/carp-collections@master" "bitset.carp")
(use BitSet)
```

## Examples

See [examples.md](examples.md) for usage examples.

## Running Tests

```bash
carp -x test/bitset_test.carp
```

## License

MIT


## License

MIT
