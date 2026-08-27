# Examples for carp-collections

This file contains usage examples for all sub-modules in the library.

## Deque

## Basic FIFO Queue

```clojure
(use Deque)

(defn main []
  (let [q (Deque.new)]
    (do
      (push-back! &q 1)
      (push-back! &q 2)
      (push-back! &q 3)
      
      (while (not (empty? &q))
        (IO.println &(str (pop-front! &q)))) ; 1, 2, 3
    )))
```

## Sliding Window Pattern

Deques are excellent for maintaining a fixed-size window of recent values.

```clojure
(use Deque)

(defn process-stream [val]
  (let [window (Deque.new)
        max-size 5]
    (do
      (push-back! &window val)
      (while (> (length &window) max-size)
        (ignore (pop-front! &window)))
      (IO.println &(str* "Average: " (str (/ (reduce &(fn [acc x] (+ acc @x)) 0 &window) 
                                             (length &window))))))))
```

## Deque as a Stack (LIFO)

```clojure
(use Deque)

(let [dq (Deque.new)]
  (do
    (push-front! &dq 1)
    (push-front! &dq 2)
    (IO.println &(str (pop-front! &dq))) ; Just 2
  ))
```

## Random Access & Iteration

```clojure
(let [dq (Deque.from-array [1 2 3 4 5])]
  (do
    (IO.println &(str (nth &dq 2))) ; Just 3
    (doall &(fn [x] (IO.print &(str* (str @x) " "))) &dq) ; 1 2 3 4 5 
  ))
```


---

## Priority-Queue

## Custom Structs

You can use the `PriorityQueue` with custom types by providing a comparison function.

```carp
(deftype Task [priority Int, name String])

(defmodule Task
  (defn compare [a b]
    (Int.> @(Task.priority a) @(Task.priority b)))
)

(defn main []
  (let [pq (PriorityQueue.new)]
    (do
      (push &pq (Task.init 1 @"low-priority") &Task.compare)
      (push &pq (Task.init 10 @"high-priority") &Task.compare)
      
      (match (pop &pq &Task.compare)
        (Maybe.Just t) (IO.println (Task.name &t)) ;; high-priority
        (Maybe.Nothing) ()))))
```

## Building from an Array (Heapify)

If you already have a collection of data, `from-array` is the fastest way to turn it into a heap ($O(n)$).

```carp
(let [items [10 50 20 40 30]
      pq (from-array items &IntRef.>)]
  (peek &pq)) ;; Returns (Maybe.Just 50)
```

## Top-K Selection

A common use for `push-pop` is maintaining a fixed-size heap of the "top K" elements.

```carp
(let [pq (PriorityQueue.new)
      limit 3]
  (do
    (foreach [x [10 50 20 40 30]]
      (if (< (length &pq) limit)
        (push &pq @x &IntRef.<)
        (ignore (push-pop &pq @x &IntRef.<))))
    ;; pq now contains the 3 largest elements: 30, 40, 50
    ))
```


---

## Ringbuf

## Basic Usage and Overwriting

```clojure
(use RingBuf)

(defn main []
  (let [rb (RingBuf.new 5 0)]
    (do
      ; push! will silently discard oldest elements
      (for [i 0 10]
        (RingBuf.push! &rb i))
      (println* "Buffer content (last 5): " (str &(RingBuf.to-array &rb)))
      (println* "Current length: " (RingBuf.length &rb)))))
```

## Safe Pushing

```clojure
(let [rb (RingBuf.new 2 0)]
  (do
    (if (RingBuf.try-push! &rb 1)
        (println* "Pushed 1")
        (println* "Buffer full!"))
    (if (RingBuf.try-push! &rb 2)
        (println* "Pushed 2")
        (println* "Buffer full!"))
    (if (RingBuf.try-push! &rb 3)
        (println* "Pushed 3")
        (println* "Buffer full!")))) ; prints "Buffer full!"
```

## Relative Indexing (get)

```clojure
(let [rb (RingBuf.new 10 0)]
  (do
    (RingBuf.push! &rb 100)
    (RingBuf.push! &rb 200)
    (println* "Oldest: " (str &(RingBuf.get &rb 0))) ; Just 100
    (println* "Newest: " (str &(RingBuf.get &rb 1))))) ; Just 200
```

## Deque-like behavior

```clojure
(let [rb (RingBuf.new 5 0)]
  (do
    (RingBuf.push! &rb 1)
    (RingBuf.push! &rb 2)
    (println* (str &(RingBuf.pop-back! &rb))) ; Just 2
    (println* (str &(RingBuf.pop! &rb))))) ; Just 1
```

## Efficient Traversal

```clojure
(let [rb (RingBuf.new 3 0)]
  (do
    (RingBuf.push! &rb 10)
    (RingBuf.push! &rb 20)
    (RingBuf.foreach &rb &(fn [x] (println* "Element: " (str x))))))
```


---

## Sparseset

## 1. Basic Initialization and Insertion
A SparseSet maps `Uint32` IDs to values of any type. It is perfect for tracking components in an ECS or managing indices in a pool.

```clojure
(load "sparse_set.carp")
(use SparseSet)

(defn main []
  (let [set (SparseSet.new)]
    (do
      ;; Insert some values at specific IDs
      (SparseSet.insert! &set 10u32 @"PositionComponent")
      (SparseSet.insert! &set 42u32 @"VelocityComponent")
      (SparseSet.insert! &set 7u32  @"HealthComponent")
      
      (println* "Set length: " (SparseSet.length &set)) ;; 3
      (println* "Contains ID 42? " (SparseSet.contains? &set 42u32)))))
```

## 2. Fast Lookups and Updates
SparseSet provides $O(1)$ lookups and updates.

```clojure
(load "sparse_set.carp")
(use SparseSet)

(defn main []
  (let [set (SparseSet.new)]
    (do
      (SparseSet.insert! &set 1u32 100)
      
      ;; Get a value (returns Maybe)
      (match (SparseSet.get &set 1u32)
        (Maybe.Just val) (println* "Value: " val)
        (Maybe.Nothing) (println* "Not found"))

      ;; Update a value directly via pointer
      (SparseSet.with-ptr &set 1u32 
        &(fn [p] (Pointer.set p (+ @p 50))))
      
      (println* "New Value: " (Maybe.unsafe-from (SparseSet.get &set 1u32))))))
```

## 3. Cache-Friendly Iteration
SparseSet maintains data in a contiguous "dense" array, making iteration extremely fast (cache-friendly).

```clojure
(load "sparse_set.carp")
(use SparseSet)

(defn main []
  (let [set (SparseSet.new)]
    (do
      (SparseSet.insert! &set 0u32 1.5)
      (SparseSet.insert! &set 100u32 2.5)
      (SparseSet.insert! &set 50u32 3.5)
      
      ;; Iterate over all values
      (SparseSet.for-each &(fn [v] (println* "Iterating value: " @v)) &set)
      
      ;; Use functional helpers
      (let [total (SparseSet.reduce &(fn [acc v] (+ acc @v)) 0.0 &set)]
        (println* "Total sum: " total)))))
```

## 4. Efficient Removal
Removal is $O(1)$ and uses the "Swap-with-last" pattern to keep the dense array contiguous.

```clojure
(load "sparse_set.carp")
(use SparseSet)

(defn main []
  (let [set (SparseSet.new)]
    (do
      (SparseSet.insert! &set 1u32 @"Alice")
      (SparseSet.insert! &set 2u32 @"Bob")
      (SparseSet.insert! &set 3u32 @"Charlie")
      
      ;; Remove 'Bob' - This swaps 'Charlie' into Bob's slot to keep the array dense
      (SparseSet.remove! &set 2u32)
      
      (println* "Length after removal: " (SparseSet.length &set)) ;; 2
      (println* "Remaining: " (str &set)))))
```


---

## Stablepool

## Basic Object Management

```clojure
(use StablePool)

(defn main []
  (let [pool (StablePool.new)]
    (do
      ;; Allocate objects
      (let [h1 (StablePool.alloc! &pool 100)
            h2 (StablePool.alloc! &pool 200)]
        (do
          (IO.println &(str (StablePool.get &pool &h1))) ; (Just 100)
          
          ;; Free an object
          (StablePool.free! &pool &h1)
          
          ;; h1 is now stale
          (IO.println &(str (StablePool.get &pool &h1))) ; Nothing
          
          ;; Next allocation will likely reuse h1's slot but with a new generation
          (let [h3 (StablePool.alloc! &pool 300)]
             (IO.println &(str (StablePool.get &pool &h3)))) ; (Just 300)
        )))))
```

## Entity System Integration

StablePool is ideal for ECS-like architectures where entities need stable references.

```clojure
(use StablePool)
(use Handle)

(deftype Entity [
  name String
  hp Int
])

(defn main []
  (let [entities (StablePool.new)]
    (do
      (let [player (StablePool.alloc! &entities (Entity.init @"Player" 100))
            enemy  (StablePool.alloc! &entities (Entity.init @"Goblin" 20))]
        (do
          ;; Process active entities
          (StablePool.doall &(fn [e] (IO.println (Entity.name e))) &entities)
          
          ;; Safe deletion
          (StablePool.free! &entities &enemy)
          
          ;; Other systems holding the 'enemy' handle can safely detect it's gone
          (match (StablePool.get &entities &enemy)
            (Maybe.Just e) (IO.println "Still here")
            (Maybe.Nothing) (IO.println "Enemy defeated!"))
        )))))
```


---

## Stack

## Basic Operations

```clojure
(use Stack)

(defn main []
  (let [s (Stack.new)]
    (do
      (Stack.push! &s 10)
      (Stack.push! &s 20)
      (Stack.push! &s 30)
      
      (IO.println &(str (Stack.length &s))) ; 3
      (IO.println &(str (Stack.peek &s)))   ; (Just 30)
      
      (while (not (Stack.empty? &s))
        (IO.println &(str (Stack.pop! &s)))) ; 30, 20, 10
    )))
```

## Advanced Operations

```clojure
(use Stack)

(defn main []
  (let [s (Stack.from-array [1 2 3])]
    (do
      (Stack.swap! &s) ; [1 3 2]
      (Stack.dup! &s)  ; [1 3 2 2]
      (Stack.drop! &s) ; [1 3 2]
      (let [doubled (Stack.map &(fn [x] (* @x 2)) &s)]
        (IO.println &(Stack.str &doubled))) ; Stack([2 6 4])
    )))
```

## Forth-style Primitives

```clojure
(use Stack)

(defn main []
  (let [s (Stack.from-array [1 2 3])]
    (do
      (Stack.over! &s) ; [1 2 3 2]
      (Stack.rot! &s)  ; [1 3 2 2]
      (Stack.tuck! &s) ; [1 3 2 2 2]
      (Stack.nip! &s)  ; [1 3 2 2]
    )))
```

## Performance & Pointers

```clojure
(use Stack)

(defn main []
  (let [s (Stack.from-array [10 20 30])]
    (match (Stack.peek-ptr &s)
      (Maybe.Just p) (IO.println &(str @p)) ; 30 (no copy of the actual element)
      (Maybe.Nothing) (IO.println "Stack is empty")
    )))
```

## Using with Different Types

```clojure
(use Stack)

(defn main []
  (let [s (Stack.new)]
    (do
      (Stack.push! &s @"Hello")
      (Stack.push! &s @"World")
      (IO.println &(str (Stack.pop! &s))) ; (Just "World")
    )))
```


---

## Hashset

## Basic Usage

Creating a hash set, inserting elements, checking membership, removing elements, and inspecting size:

```clojure
(use HashSet)

(defn main []
  (let [s (HashSet.new)]
    (do
      (HashSet.insert! &s &@"apple")
      (HashSet.insert! &s &@"banana")
      (IO.println &(str (HashSet.contains? &s &@"apple"))) ; true
      (HashSet.remove! &s &@"apple")
      (IO.println &(str (HashSet.length &s)))))) ; 1
```


---

## Handle

## Basic Usage

Initializing handles, inspecting the index and generation, and performing null checks:

```clojure
(use Handle)

(deftype Entity [])

(defn main []
  (let [h (Handle.init 10un 1un)]
    (do
      (IO.println &(str (Handle.index &h)))      ; 10
      (IO.println &(str (Handle.generation &h))) ; 1
      (IO.println &(str (Handle.not-null? &h))))))  ; true
```


---

## Bitflags

## Basic Usage

Defining flags with auto-incrementing powers of two or explicit overrides, and manipulating them using `set!`, `contains?`, and `toggle!`:

```clojure
(use BitFlags)

; Auto-incrementing flags: Read (1), Write (2), Execute (4)
; Explicit override flag: Admin (32)
(bitflags [Read Write (Admin 32) Execute])

(defn main []
  (let [f (BitFlags.new 0)]
    (do
      (BitFlags.set! &f Read)
      (BitFlags.set! &f Write)
      (IO.println &(str (BitFlags.contains? &f Read))) ; true
      (BitFlags.toggle! &f Admin)
      (IO.println &(str (BitFlags.to-int &f)))))) ; 35
```


---

## Bitset

## Basic Usage

Adding indices, checking membership, counting set bits, and iterating over set bits using `BitSet`:

```clojure
(use BitSet)

(defn main []
  (let [bs (BitSet.new)]
    (do
      (BitSet.add! &bs 10)
      (BitSet.add! &bs 100)
      
      (IO.println &(str (BitSet.contains? &bs 10))) ; true
      (IO.println &(str (BitSet.count &bs)))        ; 2
      
      ;; Efficiently iterate over set indices
      (BitSet.for-each-set &(fn [idx] (IO.println &(str @idx))) &bs))))
```

