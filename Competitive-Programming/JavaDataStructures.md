# Java Containers Cheat Sheet
*Java Data Structures and Algorithm - Competitive Programming Refresher*

- [Constants & Type Limits](#0-constants--type-limits)
- [Primitive Arrays - (contiguous)](#1-raw--primitive-arrays)
- [Dynamic Arrays - (contiguous references, scattered boxed elements)](#2-dynamic-arrays-arraylist)
- [Binary Search On Sorted Asc Array](#25-binary-search-lower-bound-upper-bound-equal-range)
- [Linked List](#3-linked-list)
- [Stacks](#4-stack-arraydeque--preferred-over-legacy-stack)
- [Queues](#5-queue-fifo--arraydeque)
- [Pairs & Tuples](#6-pair--tuple)
- [Custom Pair/Tuple as Map & Set Keys](#65-custom-pair--tuple-as-map--set-keys)
- [Utilities for Arrays](#7-arrays-utilities)
- [Utilities for Collections](#8-collections-utilities)
- [Frequency Counting With Hashmaps - (Multimap like)](#9-frequency-counting-with-hashmap)
- [String Builder](#10-stringbuilder)
- [Ordered & Unordered Sets](#11-set-variants)
- [Ordered & Unordered Maps](#12-map-variants)
- [Min Heaps](#13-min-heap-priorityqueue)
- [Max Heaps](#14-max-heap-priorityqueue-reversed)
- [Min/Max Heaps With Pairs](#15-priorityqueue-with-pairs--custom-objects)
- [Monotonic Deque](#16-monotonic-deque-sliding-window-maxmin)
- [Bitsets](#17-bitset)
- [Big Integers](#18-biginteger)

---

## CP Quick Notes
- Prefer `offer/poll/peek` over `add/remove/element` in queues/heaps - they return `null`/`false` instead of throwing exceptions.
- `int[]` pairs beat `Map.Entry` or custom classes for speed and simplicity in CP.
- All sections use **JDK-only** classes - no external libraries required.

---

## 0. Constants & Type Limits

```java
// Safe infinity values (avoids overflow on INF + INF)
int    INF  = (int) 1e9;         // use instead of Integer.MAX_VALUE in graphs
long   LINF = (long) 1e18;       // use instead of Long.MAX_VALUE in graphs
double EPS  = 1e-9;              // epsilon for floating-point comparison

// Absolute limits (for reference)
int    INT_MAX  = Integer.MAX_VALUE; // 2^31 - 1 = 2,147,483,647
int    INT_MIN  = Integer.MIN_VALUE;
long   LONG_MAX = Long.MAX_VALUE;    // ~9.2 * 10^18
```

---

## 1. Raw / Primitive Arrays

> The closest Java gets to a C++ `std::vector<int>` in terms of memory layout.
> Elements are stored **inline and contiguously** - no boxing, no heap pointers per element.
> Fixed size: you must know the length upfront, or resize manually (copy to a new array).

### Memory layout vs. ArrayList

```
int[] a = {1, 2, 3, 4, 5};

Memory (contiguous ints, no indirection):
[ 1 | 2 | 3 | 4 | 5 ]
  ^   ^   ^   ^   ^
  each slot IS the int value - no pointer, no Integer object

ArrayList<Integer> list = new ArrayList<>();
list.add(1); list.add(2); ...

Memory (contiguous references, objects scattered on heap):
[ ref0 | ref1 | ref2 | ref3 | ref4 ]   <- backing Object[] (contiguous)
    |      |      |      |      |
  Int(1) Int(2) Int(3) Int(4) Int(5)   <- boxed Integer objects (scattered)
```

This matters for cache performance on large arrays: iterating `int[]` is cache-friendly
because every element is adjacent. Iterating `ArrayList<Integer>` chases a pointer per
element, causing cache misses proportional to how scattered the Integer objects are.

### Declaration and initialization

```java
int[]    a = new int[5];         // zero-initialized               - O(n)
int[]    b = {3, 1, 4, 1, 5};    // inline literal                 - O(n)
int[]    c = new int[]{3, 1, 4}; // explicit literal (works anywhere) - O(n)
long[]   d = new long[5];        // zero-initialized               - O(n)
double[] e = new double[5];      // zero-initialized to 0.0        - O(n)
boolean[]f = new boolean[5];     // zero-initialized to false      - O(n)
char[]   g = new char[5];        // zero-initialized to '\0'       - O(n)

int n = b.length;                // fixed length field             - O(1)
```

### Access and modification

```java
int[] a = {3, 1, 4, 1, 5};

int x    = a[2];     // read by index  - O(1)
a[2]     = 99;       // write by index - O(1)
int len  = a.length; // length field   - O(1)
```

### Sorting

```java
int[] a = {3, 1, 4, 1, 5};

Arrays.sort(a);              // sort asc in-place (dual-pivot quicksort) - O(n log n) average
Arrays.sort(a, 1, 4);        // sort subarray [1, 4) in-place           - O(k log k), k = range size

// PITFALL (CP): before JDK 14, dual-pivot quicksort on primitives has an O(n^2) worst case,
// and some judges (e.g. Codeforces hacks) include "anti-quicksort" tests that trigger it.
// Fix: shuffle the array randomly before sorting, or sort a boxed Integer[] / List
// (objects are sorted with TimSort, which is always O(n log n)).

// PITFALL: Arrays.sort on primitives has NO overload that accepts a Comparator.
// You CANNOT do: Arrays.sort(a, Comparator.reverseOrder()); - compile error.
// To sort descending you have two options:

// Option A: sort asc then reverse - O(n log n) + O(n)
Arrays.sort(a);
for (int i = 0, j = a.length - 1; i < j; i++, j--) {
    int tmp = a[i]; a[i] = a[j]; a[j] = tmp;
}

// Option B: box to Integer[], sort with Comparator, then unbox if needed
// This defeats the purpose of using primitives - avoid in hot paths.
Integer[] boxed = new Integer[a.length];
for (int i = 0; i < a.length; i++) boxed[i] = a[i];   // O(n) - boxing cost
Arrays.sort(boxed, Comparator.reverseOrder());          // O(n log n)
// Unbox back if you need int[]:
for (int i = 0; i < a.length; i++) a[i] = boxed[i];   // O(n) - unboxing cost
```

### Copying and filling

```java
int[] a = {3, 1, 4, 1, 5};

int[] copy  = Arrays.copyOf(a, a.length);     // full copy                        - O(n)
int[] grown = Arrays.copyOf(a, a.length * 2); // copy + grow (zero-pads new slots) - O(n)
int[] slice = Arrays.copyOfRange(a, 1, 4);    // partial copy [1, 4)              - O(k), k = range size

Arrays.fill(a, 0);                            // fill entire array                - O(n)
Arrays.fill(a, 1, 4, -1);                     // fill subarray [1, 4) with -1     - O(k)
```

### 2D arrays

```java
// 2D array - array of row arrays; rows are contiguous, but row objects are scattered
int[][] grid = new int[3][4];                           // 3 rows, 4 cols, zero-init - O(n*m)
for (int[] row : grid) Arrays.fill(row, -1);           // fill all cells with -1    - O(n*m)

int val    = grid[1][2];     // read cell (row 1, col 2)  - O(1)
grid[0][0] = 7;              // write cell                - O(1)
int rows   = grid.length;    // number of rows            - O(1)
int cols   = grid[0].length; // number of cols            - O(1)

// PITFALL: rows in a 2D array are separate heap objects.
// grid[0], grid[1], grid[2] are pointers to three separate int[] arrays.
// Accessing grid[r][c] = two pointer hops: one to get row r, one to get element c.
// This is fine for most CP problems but means 2D arrays are NOT a flat contiguous block.

// Flat 1D array as a 2D grid - single contiguous block, better cache behavior:
int[] flat = new int[3 * 4];              // O(n*m)
flat[1 * 4 + 2] = 7;                     // write grid[1][2]  - O(1)
int v = flat[1 * 4 + 2];                 // read grid[1][2]   - O(1)
// Helper: inline index formula -> row * numCols + col
```

### Printing for debugging

```java
int[]   a      = {1, 2, 3};
int[][] matrix = {{1, 2}, {3, 4}};

System.out.println(Arrays.toString(a));           // "[1, 2, 3]"         - O(n)
System.out.println(Arrays.deepToString(matrix));  // "[[1, 2], [3, 4]]"  - O(n*m)

// PITFALL: do NOT use a.toString() or System.out.println(a) -
// these print the identity hashcode (e.g. "[I@6d06d69c"), not the contents.
```

### CP best practices summary

```
DO:
  - Use int[] / long[] for all fixed-size numeric data - no boxing overhead, cache-friendly.
  - Pre-allocate at maximum possible size (e.g. int[200_005]) rather than resizing.
  - Use a manual `size` variable to track the logical end if you need dynamic behavior.
  - Use Arrays.sort(a) for ascending sort - it's fast (dual-pivot quicksort on primitives),
    but shuffle first if anti-quicksort tests are possible (older JDKs).
  - Use the reverse-in-place trick for descending sort instead of boxing.
  - Use flat 1D arrays (row * cols + col) for 2D grids in performance-critical code.

DON'T:
  - Use Integer[] when int[] works - boxing/unboxing adds GC pressure and cache misses.
  - Call a.toString() or print an array directly - use Arrays.toString().
  - Sort a primitive array with a Comparator - it won't compile; box first or sort then reverse.
  - Assume 2D arrays (int[][]) are a flat memory block - each row is a separate heap object.
```

---

## 2. Dynamic Arrays (`ArrayList`)

> **Autoboxing and cache-miss warning for CP**
>
> `ArrayList` requires a type parameter, and Java generics do not support primitives.
> This means `ArrayList<Integer>` stores **boxed `Integer` objects**, not raw `int` values.
> Every `add` autoboxes the `int` to an `Integer` heap object; every `get` may unbox it back.
>
> Two costs you pay that `int[]` does not:
> - **Autoboxing overhead**: each element addition allocates an `Integer` object on the heap.
> - **Cache misses on iteration**: the backing `Object[]` holds *references* (pointers) to
>   `Integer` objects scattered across the heap. Iterating a large `ArrayList<Integer>` chases
>   one pointer per element, destroying cache locality. An `int[]` of the same data is a single
>   contiguous block with zero indirection.
>
> **CP rule of thumb**: if the size is bounded and known (or has a safe upper bound),
> prefer a pre-allocated `int[]` with a manual size counter. Use `ArrayList` when you
> genuinely need dynamic resizing and the boxing cost is acceptable (e.g. storing objects
> that are already on the heap, like `int[]` pairs or `String`s).

```java
import java.util.ArrayList;
import java.util.List;

List<Integer> a = new ArrayList<>();    // init
a.add(10);                              // add to end              - O(1) amortized (autoboxes int -> Integer)
a.add(20);                             // add to end              - O(1) amortized
a.add(1, 15);                           // insert at index         - O(n) (shifts elements right)

int x = a.get(0);                       // get by index            - O(1) (unboxes Integer -> int)
boolean has20 = a.contains(20);         // search                  - O(n)
int idx = a.indexOf(15);               // first index of value    - O(n)

a.sort(Comparator.naturalOrder());      // sort asc                - O(n log n)
a.sort(Comparator.reverseOrder());      // sort desc               - O(n log n)

a.remove(Integer.valueOf(10));          // remove by object        - O(n) (search + shift)
a.remove(0);                            // remove by index         - O(n) (shifts elements left)

int size = a.size();                    // size                    - O(1)
a.clear();                              // clear all elements      - O(n)
```

## 2.5 Binary Search, Lower Bound, Upper Bound, Equal Range
> List **must be sorted** before calling any of these.

### Binary search on sorted primitive arrays

```java
int[] a = {1, 3, 3, 5, 7, 9}; // must be sorted first

int idx = Arrays.binarySearch(a, 5); // O(log n) - exact index, or -(insertionPoint+1) if absent
// Same caveat as Collections.binarySearch: no guarantee which duplicate is returned.
// Use the custom lowerBound / upperBound helpers below for precise control.
```

### Binary Search On ArrayList

```java
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

List<Integer> a = new ArrayList<>(Arrays.asList(1, 3, 3, 5, 7, 9));

// --- Binary Search ---
// Returns index of any match, or -(insertionPoint + 1) if not found.
// If duplicates exist, no guarantee which index is returned.
int idx = Collections.binarySearch(a, 3);  // O(log n) - some index where a.get(idx) == 3
if (idx < 0) {
    // not found; insertion point = -(idx + 1)
    int insertAt = -(idx + 1);
}
```

### Lower Bound Binary Search - like `std::lower_bound`

```java
// First index of an element >= target.
// Returns `n` if all elements are < target.
static int lowerBound(long[] a, int n, long target) {
    int l = 0, r = n;
    while (l < r) {
        // int mid = (l + r) / 2;
        int mid = l + (r - l) / 2; // much less prone to overflows
        if (a[mid] < target) {
            l = mid + 1;
        } else {
            r = mid;
        }
    }
    return l;
}
```

### Upper Bound Binary Search - like `std::upper_bound`

```java
// First index of an element > target.
// Returns `n` if all elements are <= target.
static int upperBound(long[] a, int n, long target) {
    int l = 0, r = n;
    while (l < r) {
        // int mid = (l + r) / 2;
        int mid = l + (r - l) / 2; // much less prone to overflows
        if (a[mid] <= target) {
            l = mid + 1;
        } else {
            r = mid;
        }
    }
    return l;
}
```

### Equal Range Binary Search - like `std::equal_range`

```java
// Returns int[]{lo, hi} - the half-open range [lo, hi) where a[i] == target.
// Range is empty (lo == hi) if target is not found.
// Slightly faster than calling lowerBound + upperBound independently,
// because the second search starts from the lower bound instead of 0:
static int[] equalRange(long[] a, int n, long target) {
    int l = 0, r = n;
    while (l < r) {
        int mid = (l + r) / 2;
        if (a[mid] < target)
            l = mid + 1;
        else
            r = mid;
    }
    int left = l;

    r = n;
    while (l < r) {
        int mid = (l + r) / 2;
        if (a[mid] <= target)
            l = mid + 1;
        else
            r = mid;
    }
    int right = l;

    return new int[] { left, right };
}
```

### Usage Example

```java
// arr = [1, 3, 3, 5, 7, 9]
long[] arr = {1, 3, 3, 5, 7, 9};
int n = arr.length;

lowerBound(arr, n, 3);   // O(log n) -> 1  (first index where value >= 3)
upperBound(arr, n, 3);   // O(log n) -> 3  (first index where value >  3)
lowerBound(arr, n, 4);   // O(log n) -> 3  (first index where value >= 4, lands on 5)
upperBound(arr, n, 4);   // O(log n) -> 3  (first index where value >  4, lands on 5)

int[] range = equalRange(arr, n, 3); // O(log n) -> [1, 3)  i.e. range[0]=1, range[1]=3
int[] empty = equalRange(arr, n, 4); // O(log n) -> [3, 3)  i.e. range[0]==range[1] -> not found

// Count occurrences of target:
int count = range[1] - range[0];        // O(1) -> 2 (two 3s)

// Check if target exists:
boolean exists = range[0] < range[1];   // O(1) -> true

// Iterate over all matching elements:
for (int i = range[0]; i < range[1]; i++) {
    // arr[i] == target                  // O(1) per access
}

// Check if target exists (using lowerBound directly):
int lb = lowerBound(arr, n, 3);            // O(log n)
boolean found = lb < n && arr[lb] == 3;   // O(1) -> true
```

---

## 3. Linked List

```java
import java.util.LinkedList;
import java.util.Deque;

LinkedList<String> ll = new LinkedList<>();
ll.add("a");                                // add to end              - O(1)
ll.addFirst("head");                        // add to front            - O(1)
ll.addLast("tail");                         // add to end              - O(1)
ll.add(1, "middle");                        // insert at index         - O(n)

String first = ll.getFirst();               // peek front              - O(1)
String last  = ll.getLast();                // peek last               - O(1)
boolean has  = ll.contains("a");            // search                  - O(n)

ll.sort(Comparator.naturalOrder());         // sort asc                - O(n log n)
ll.sort(Comparator.reverseOrder());         // sort desc               - O(n log n)

ll.removeFirst();                           // remove front            - O(1)
ll.removeLast();                            // remove back             - O(1)
ll.remove("middle");                        // remove by object        - O(n) (scan + unlink)
ll.remove(1);                               // remove by index         - O(n) (scan to index + unlink)

// Use as Deque (stack-like or queue-like)
Deque<String> dq = ll;
dq.push("stackTop");       // add to front            - O(1)
String popped = dq.pop();  // remove from front       - O(1)
```

---

## 4. Stack (`ArrayDeque` - preferred over legacy `Stack`)

> `ArrayDeque` is faster and not synchronized. Prefer it over `java.util.Stack`.

```java
import java.util.ArrayDeque;
import java.util.Deque;

Deque<Integer> st = new ArrayDeque<>();
st.push(10);                      // push to top             - O(1)
st.push(20);                      // push to top             - O(1)
Integer top    = st.peek();       // peek top (20)           - O(1), returns null if empty
Integer popped = st.pop();        // pop top (20)            - O(1), throws NoSuchElementException if empty
boolean empty  = st.isEmpty();    // check empty             - O(1)
int size       = st.size();       // size                    - O(1)
```

---

## 5. Queue (FIFO - `ArrayDeque`)

```java
import java.util.ArrayDeque;
import java.util.Queue;

Queue<Integer> q = new ArrayDeque<>();
q.offer(1);                   // enqueue                 - O(1), returns false if fails (prefer over add())
q.offer(2);                   // enqueue                 - O(1)
Integer head   = q.peek();    // peek front              - O(1), returns null if empty
Integer polled = q.poll();    // dequeue                 - O(1), returns null if empty (prefer over remove())
boolean empty  = q.isEmpty(); // check empty             - O(1)
int size       = q.size();    // size                    - O(1)
```

---

## 6. Pair / Tuple
> Java has no built-in `Pair`. These are your options, ranked for CP use.

---

### Option 1: `int[]` - fastest, zero overhead, best for CP

```java
// --- Create ---
int[] pair = new int[]{3, 7};          // allocate pair           - O(1)
List<int[]> pairs = new ArrayList<>();
pairs.add(new int[]{3, 7});            // add to list             - O(1) amortized
pairs.add(new int[]{1, 5});            // add to list             - O(1) amortized
pairs.add(new int[]{3, 2});            // add to list             - O(1) amortized

// --- Access ---
int first  = pair[0];   // read first element      - O(1)
int second = pair[1];   // read second element     - O(1)

// --- Sort ---
pairs.sort((a, b) -> a[0] - b[0]);                                     // by first asc              - O(n log n)
pairs.sort((a, b) -> a[0] != b[0] ? a[0] - b[0] : a[1] - b[1]);      // by first, then second     - O(n log n)
pairs.sort((a, b) -> Integer.compare(b[0], a[0]));                     // by first desc             - O(n log n)
// Use Integer.compare() / Long.compare() when values may overflow int subtraction
```

---

### Option 2: `record` (Java 16+) - clean and type-safe

```java
// --- Define (once, outside method) ---
record Pair<A, B>(A first, B second) {}

// --- Create ---
Pair<Integer, Integer> pair = new Pair<>(3, 7);  // allocate pair          - O(1)
List<Pair<Integer, Integer>> pairs = new ArrayList<>();
pairs.add(new Pair<>(3, 7));                      // add to list            - O(1) amortized
pairs.add(new Pair<>(1, 5));                      // add to list            - O(1) amortized
pairs.add(new Pair<>(3, 2));                      // add to list            - O(1) amortized

// --- Access ---
int first  = pair.first();   // read first component   - O(1)
int second = pair.second();  // read second component  - O(1)

// --- Sort ---
pairs.sort(Comparator.comparingInt(Pair::first));                                           // by first asc                - O(n log n)

// PITFALL: when chaining (.reversed(), .thenComparing...), Java can no longer infer the type
// from a method reference like `Pair::first` on a generic record (compile error),
// so give the lambda parameter an explicit type:
pairs.sort(Comparator.comparingInt((Pair<Integer, Integer> p) -> p.first()).reversed());                                   // by first desc               - O(n log n)
pairs.sort(Comparator.comparingInt((Pair<Integer, Integer> p) -> p.first()).thenComparingInt(p -> p.second()));            // by first asc, then sec asc  - O(n log n)
pairs.sort(Comparator.comparingInt((Pair<Integer, Integer> p) -> p.first()).thenComparingInt(p -> p.second()).reversed()); // both desc                   - O(n log n)
```

---

### Option 3: `Map.Entry` - no class needed, but verbose

```java
// --- Create ---
Map.Entry<Integer, Integer> pair = Map.entry(3, 7);  // allocate entry        - O(1)
List<Map.Entry<Integer, Integer>> pairs = new ArrayList<>();
pairs.add(Map.entry(3, 7));                           // add to list           - O(1) amortized
pairs.add(Map.entry(1, 5));                           // add to list           - O(1) amortized
pairs.add(Map.entry(3, 2));                           // add to list           - O(1) amortized

// --- Access ---
int first  = pair.getKey();    // read key             - O(1)
int second = pair.getValue();  // read value           - O(1)

// --- Sort ---
pairs.sort(Map.Entry.comparingByKey());                                              // by key asc                - O(n log n)
pairs.sort(Map.Entry.<Integer, Integer>comparingByKey().reversed());                 // by key desc               - O(n log n)
pairs.sort(Map.Entry.comparingByKey(Comparator.comparingInt(k -> k)));               // by key asc (explicit)     - O(n log n)
pairs.sort(Map.Entry.<Integer, Integer>comparingByKey()
           .thenComparing(Map.Entry.comparingByValue()));                             // by key, then value        - O(n log n)
```

---

## 6.5 Custom Pair / Tuple as Map & Set Keys

> Depending on which container you target, you need different things:
> - **`HashMap` / `HashSet`** → `equals()` + `hashCode()` (so the hash table can bucket and compare)
> - **`TreeMap` / `TreeSet`** → a consistent ordering via `Comparable` (or a `Comparator`)
> - **All four at once** → implement both `Comparable` AND override `equals()`/`hashCode()`

---

### Option A: `record` — best default (Java 16+)

> `record` auto-generates `equals()` and `hashCode()` from its components — works in `HashMap`/`HashSet` out of the box.
> Add `implements Comparable` to also support `TreeMap`/`TreeSet`.

```java
// --- Define once, outside your method ---

// For HashMap / HashSet only (equals + hashCode come free with record)
record Pair(int x, int y) {}

// For ALL four containers: add Comparable
record PairC(int x, int y) implements Comparable<PairC> {
    @Override
    public int compareTo(PairC o) {
        if (this.x != o.x) return Integer.compare(this.x, o.x); // sort by x asc
        return Integer.compare(this.y, o.y);                     // then by y asc
    }
}
```

```java
// --- HashMap: O(1) average put/get ---
Map<Pair, String> hashMap = new HashMap<>();
hashMap.put(new Pair(1, 2), "a");          // insert              - O(1) average
hashMap.put(new Pair(3, 4), "b");          // insert              - O(1) average
String val = hashMap.get(new Pair(1, 2));  // lookup "a"          - O(1) average
boolean has = hashMap.containsKey(new Pair(3, 4)); // true        - O(1) average

// --- HashSet: O(1) average add/contains ---
Set<Pair> hashSet = new HashSet<>();
hashSet.add(new Pair(1, 2));               // insert              - O(1) average
hashSet.add(new Pair(1, 2));               // duplicate - ignored - O(1) average
boolean inSet = hashSet.contains(new Pair(1, 2)); // true         - O(1) average
int size = hashSet.size();                 // 1                   - O(1)

// --- TreeMap: sorted by PairC.compareTo - O(log n) per op ---
TreeMap<PairC, String> treeMap = new TreeMap<>();
treeMap.put(new PairC(3, 4), "b");        // insert              - O(log n)
treeMap.put(new PairC(1, 2), "a");        // insert              - O(log n)
treeMap.put(new PairC(1, 9), "c");        // insert              - O(log n)
PairC firstKey = treeMap.firstKey();      // PairC(1, 2) - min   - O(log n)
PairC lastKey  = treeMap.lastKey();       // PairC(3, 4) - max   - O(log n)

// Closest-key lookups
PairC floor   = treeMap.floorKey(new PairC(2, 0));   // largest key <= (2,0) -> (1,9)  - O(log n)
PairC ceiling = treeMap.ceilingKey(new PairC(2, 0)); // smallest key >= (2,0) -> (3,4) - O(log n)

// --- TreeSet: sorted by PairC.compareTo - O(log n) per op ---
TreeSet<PairC> treeSet = new TreeSet<>();
treeSet.add(new PairC(3, 4));             // insert              - O(log n)
treeSet.add(new PairC(1, 2));             // insert              - O(log n)
treeSet.add(new PairC(1, 2));             // duplicate - ignored - O(log n)
PairC smallest = treeSet.first();         // PairC(1, 2)         - O(log n)
PairC largest  = treeSet.last();          // PairC(3, 4)         - O(log n)
PairC higher   = treeSet.higher(new PairC(1, 2)); // PairC(3, 4) - O(log n)
```

---

### Option B: Manual class — full control, works on all Java versions

> Override `equals()` + `hashCode()` manually for `HashMap`/`HashSet`.
> Implement `Comparable` for `TreeMap`/`TreeSet`.
>
> **Pitfall**: if you implement `Comparable`, its ordering should be consistent with `equals()`:
> `compareTo(o) == 0` ⟺ `equals(o) == true`. `TreeMap`/`TreeSet` only use `compareTo` to decide
> equality, so if they disagree, Tree containers and Hash containers will behave differently
> (e.g. a `TreeSet` may reject an element that a `HashSet` would accept).

```java
// --- Define once, outside your method ---
static class Pair implements Comparable<Pair> {
    int x, y;
    Pair(int x, int y) { this.x = x; this.y = y; }

    // Required for HashMap / HashSet
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Pair)) return false;
        Pair p = (Pair) o;
        return this.x == p.x && this.y == p.y;
    }

    // Required for HashMap / HashSet
    // Rule: if a.equals(b), then a.hashCode() == b.hashCode() MUST hold.
    @Override
    public int hashCode() {
        return 31 * x + y;                 // simple polynomial hash - fast, good spread for small values
        // Alternatively, use Objects.hash(x, y) for cleaner code (slight boxing overhead):
        // return Objects.hash(x, y);
    }

    // Required for TreeMap / TreeSet (must be consistent with equals)
    @Override
    public int compareTo(Pair o) {
        if (this.x != o.x) return Integer.compare(this.x, o.x);
        return Integer.compare(this.y, o.y);
    }

    @Override
    public String toString() { return "(" + x + ", " + y + ")"; }
}
```

```java
// --- HashMap ---
Map<Pair, Integer> hashMap = new HashMap<>();
hashMap.put(new Pair(1, 2), 10);                      // insert              - O(1) average
hashMap.put(new Pair(3, 4), 20);                      // insert              - O(1) average
int v = hashMap.getOrDefault(new Pair(1, 2), -1);     // 10                 - O(1) average
hashMap.remove(new Pair(3, 4));                        // delete             - O(1) average

// --- HashSet ---
Set<Pair> hashSet = new HashSet<>();
hashSet.add(new Pair(1, 2));                           // insert             - O(1) average
hashSet.add(new Pair(1, 2));                           // duplicate - ignored- O(1) average
boolean inSet = hashSet.contains(new Pair(1, 2));      // true              - O(1) average
int size = hashSet.size();                             // 1                  - O(1)

// --- TreeMap ---
TreeMap<Pair, Integer> treeMap = new TreeMap<>();
treeMap.put(new Pair(1, 9), 30);                      // insert             - O(log n)
treeMap.put(new Pair(1, 2), 10);                      // insert             - O(log n)
treeMap.put(new Pair(3, 4), 20);                      // insert             - O(log n)

// Iteration is in sorted order: (1,2) -> (1,9) -> (3,4)
for (Map.Entry<Pair, Integer> e : treeMap.entrySet()) {
    System.out.println(e.getKey() + " -> " + e.getValue()); // O(n) total
}

Pair firstKey = treeMap.firstKey();                   // (1, 2) - minimum   - O(log n)
Pair lastKey  = treeMap.lastKey();                    // (3, 4) - maximum   - O(log n)

// --- TreeSet ---
TreeSet<Pair> treeSet = new TreeSet<>();
treeSet.add(new Pair(3, 4));                          // insert             - O(log n)
treeSet.add(new Pair(1, 2));                          // insert             - O(log n)
treeSet.add(new Pair(1, 9));                          // insert             - O(log n)

// Iteration is in sorted order: (1,2) -> (1,9) -> (3,4)
for (Pair p : treeSet) System.out.println(p);         // O(n) total

Pair smallest = treeSet.first();                      // (1, 2)            - O(log n)
Pair floor    = treeSet.floor(new Pair(2, 0));        // (1, 9)            - O(log n)
Pair ceiling  = treeSet.ceiling(new Pair(2, 0));      // (3, 4)            - O(log n)
```

---

### Option C: External `Comparator` (no `Comparable` needed for Tree containers)

> Use when you can't or don't want to modify the class itself,
> or when you need a different sort order than `compareTo` provides.
> **Cannot** replace `equals()`/`hashCode()` — those are still needed for `HashMap`/`HashSet`.

```java
// Sort by y desc, then x asc - without touching the Pair class
Comparator<Pair> byYDescThenXAsc =
    Comparator.comparingInt((Pair p) -> p.y).reversed()
              .thenComparingInt(p -> p.x);

TreeMap<Pair, Integer> treeMap = new TreeMap<>(byYDescThenXAsc);
treeMap.put(new Pair(1, 9), 1);    // O(log n)
treeMap.put(new Pair(3, 4), 2);    // O(log n)
treeMap.put(new Pair(1, 2), 3);    // O(log n)
// Iteration order (by y desc): (1,9) -> (3,4) -> (1,2)

TreeSet<Pair> treeSet = new TreeSet<>(byYDescThenXAsc);
treeSet.add(new Pair(1, 9));       // O(log n)
treeSet.add(new Pair(3, 4));       // O(log n)
treeSet.add(new Pair(1, 2));       // O(log n)
// Iteration order: (1,9) -> (3,4) -> (1,2)

// PITFALL: TreeMap/TreeSet use comparator.compare(a, b) == 0 to decide equality,
// NOT equals(). So if your comparator says two objects are equal but equals() disagrees,
// the second one will silently overwrite / be rejected. Keep them consistent.
```

---

### Quick Reference — What Each Container Needs

```
Container   | Needs                           | Purpose
------------|---------------------------------|-------------------------------------
HashMap     | equals() + hashCode()           | hash bucket placement & key equality
HashSet     | equals() + hashCode()           | same as HashMap (backed by one)
TreeMap     | Comparable OR Comparator        | determines key ordering
TreeSet     | Comparable OR Comparator        | same as TreeMap (backed by one)
All four    | equals()+hashCode()+Comparable  | safest: consistent across all containers
```

```
DO:
  - Use `record` when on Java 16+ - equals/hashCode are generated for free.
  - Always make equals() and compareTo() consistent: a.compareTo(b)==0 iff a.equals(b).
  - Use Objects.hash(x, y) for cleaner hashCode() when boxing overhead is acceptable.
  - Use an external Comparator on TreeMap/TreeSet when you need ad-hoc ordering.

DON'T:
  - Use `int[]` as a HashMap/HashSet key - arrays use identity hashCode (memory address),
    so new int[]{1,2} and new int[]{1,2} are NEVER equal as keys.
  - Forget hashCode() when overriding equals() - HashMap/HashSet will silently break
    (two "equal" keys hash to different buckets and both get stored).
  - Use a Comparator that is inconsistent with equals() in Tree containers -
    duplicate logical keys will appear or entries will silently overwrite.
```

---

## 7. Arrays Utilities

```java
import java.util.Arrays;

int[] a = {3, 1, 4, 1, 5};

// Sorting
Arrays.sort(a);                             // sort primitive array asc      - O(n log n)
Arrays.sort(a, 0, 3);                       // sort subarray [0, 3)          - O(k log k), k = range size

// Reverse sort requires boxed Integer[], NOT int[]
Integer[] b = {3, 1, 4};
Arrays.sort(b, Comparator.reverseOrder());  // sort desc                     - O(n log n)
Arrays.sort(b, (x, y) -> y - x);           // sort desc (same effect, but y - x can overflow for large values; prefer Integer.compare(y, x)) - O(n log n)

// Copying
int[] copy  = Arrays.copyOf(a, a.length);      // full copy                 - O(n)
int[] range = Arrays.copyOfRange(a, 1, 4);     // partial copy [1, 4)       - O(k), k = range size

// Fill
Arrays.fill(a, 0);                              // fill entire array         - O(n)
int[][] matrix = new int[3][4];
for (int[] row : matrix) Arrays.fill(row, -1); // fill 2D array             - O(n*m)

// Binary search (array must be sorted first)
int idx = Arrays.binarySearch(a, 4);        // binary search                - O(log n), returns negative if not found

// Debug printing
System.out.println(Arrays.toString(a));          // print 1D array           - O(n)
System.out.println(Arrays.deepToString(matrix)); // print 2D array           - O(n*m)
```

---

## 8. Collections Utilities

```java
import java.util.Collections;

List<Integer> list = new ArrayList<>(Arrays.asList(3, 1, 2));

Collections.sort(list);                            // sort asc                - O(n log n)
Collections.sort(list, Comparator.reverseOrder()); // sort desc               - O(n log n)
Collections.reverse(list);                         // reverse in-place        - O(n)
Collections.shuffle(list);                         // random shuffle          - O(n)
Collections.swap(list, 0, 1);                      // swap two elements       - O(1)
Collections.fill(list, 0);                         // fill with value         - O(n)

int min  = Collections.min(list);                  // find minimum            - O(n)
int max  = Collections.max(list);                  // find maximum            - O(n)
int freq = Collections.frequency(list, 3);         // count occurrences of 3  - O(n)

// Binary search (list must be sorted first)
int idx = Collections.binarySearch(list, 2);       // binary search           - O(log n), returns negative if not found

// Immutable list of n copies (useful for initialization)
List<Integer> zeros = new ArrayList<>(Collections.nCopies(5, 0)); // O(n)
```

---

## 9. Frequency Counting with `HashMap`

```java
import java.util.HashMap;
import java.util.Map;

Map<String, Integer> freq = new HashMap<>();

// Increment count
freq.merge("apple", 1, Integer::sum);                  // increment by 1          - O(1) average
freq.put("apple", freq.getOrDefault("apple", 0) + 1); // increment by 1          - O(1) average (explicit form)

// Read count
int c = freq.getOrDefault("apple", 0);                 // get count (0 if absent) - O(1) average

// Decrement / remove
freq.merge("apple", -1, Integer::sum);                 // decrement by 1 (key stays with value 0) - O(1) average
freq.remove("apple");                                   // remove key entirely     - O(1) average

// Decrement and auto-remove the key when the count reaches 0
// (merge removes the key if the remapping function returns null)
freq.merge("apple", -1, (a, b) -> a + b == 0 ? null : a + b);   // O(1) average

```

---

## 10. StringBuilder

```java
StringBuilder sb = new StringBuilder();
sb.append("hello");            // append string          - O(k) amortized, k = length of appended string
sb.append(' ');                // append char            - O(1) amortized
sb.insert(0, "prefix");        // insert at index        - O(n) (shifts all chars right)
sb.deleteCharAt(0);            // delete one char        - O(n) (shifts chars left)
sb.delete(0, 3);               // delete range [0, 3)    - O(n) (shifts chars left)
sb.reverse();                  // reverse in-place       - O(n)
sb.setCharAt(1, 'z');          // overwrite one char     - O(1)
char c   = sb.charAt(1);       // read one char          - O(1)
int len  = sb.length();        // current length         - O(1)
String result = sb.toString(); // convert to String      - O(n)

```

---

## 11. Set Variants

```java
import java.util.HashSet;
import java.util.TreeSet;
import java.util.LinkedHashSet;
import java.util.Iterator;

// HashSet - unordered, backed by hash table
Set<String> hs = new HashSet<>();
hs.add("a");                    // insert               - O(1) average
boolean has = hs.contains("a"); // membership test      - O(1) average
hs.remove("a");                 // delete               - O(1) average
int size = hs.size();           // size                 - O(1)

// TreeSet - sorted (red-black tree)
TreeSet<Integer> ts = new TreeSet<>();
ts.add(5);                               // insert               - O(log n)
ts.add(2);                               // insert               - O(log n)
ts.add(9);                               // insert               - O(log n)
boolean hasFive = ts.contains(5);        // membership test      - O(log n)
Integer smallest  = ts.first();          // smallest element     - O(log n)
Integer largest   = ts.last();           // largest element      - O(log n)
Integer floor     = ts.floor(6);         // largest element <= 6 - O(log n)
Integer ceiling   = ts.ceiling(6);       // smallest element >= 6- O(log n)
Integer lower     = ts.lower(5);         // largest element < 5  - O(log n)
Integer higher    = ts.higher(5);        // smallest element > 5 - O(log n)
ts.remove(5);                            // delete               - O(log n)
int tsSize = ts.size();                  // size                 - O(1)

// Iterator Navigation vs. NavigableSet Methods
Iterator<Integer> tsIt = ts.iterator();  // get iterator         - O(1)
tsIt.hasNext();                          // check if next exists - O(1)
Integer nextVal = tsIt.next();           // advance to next      - O(1) amortized (O(log n) worst-case)
// Note: higher()/lower()/ceiling()/floor() search from the tree root -> strict O(log n)
// iterator.next() moves sequentially from the current node pointer  -> amortized O(1)

// LinkedHashSet - insertion-ordered, backed by hash table + linked list
Set<String> ins = new LinkedHashSet<>();
ins.add("b");                   // insert (preserves order) - O(1) average
ins.add("a");                   // insert (preserves order) - O(1) average
ins.contains("a");              // membership test          - O(1) average
ins.remove("a");                // delete                   - O(1) average
int insSize = ins.size();       // size                     - O(1)

```

---

## 12. Map Variants

### Unordered (`HashMap`)

```java
import java.util.HashMap;
import java.util.Map;

Map<String, Integer> hm = new HashMap<>();
hm.put("x", 1);                        // insert / overwrite      - O(1) average
int val  = hm.getOrDefault("x", 0);    // get (with default)      - O(1) average
boolean has = hm.containsKey("y");     // key existence test      - O(1) average
hm.remove("x");                        // delete by key           - O(1) average
hm.putIfAbsent("z", 3);               // insert only if absent   - O(1) average
hm.computeIfAbsent("z", k -> 42);     // compute value if absent - O(1) average
int hmSize = hm.size();                // size                    - O(1)

```

### Sorted (`TreeMap`)

```java
import java.util.TreeMap;
import java.util.Map;
import java.util.Iterator;

TreeMap<Integer, Integer> tm = new TreeMap<>();
tm.put(1, 10);                       // insert / overwrite        - O(log n)
tm.put(3, 30);                       // insert / overwrite        - O(log n)
tm.put(5, 50);                       // insert / overwrite        - O(log n)

int v = tm.getOrDefault(1, 0);       // get (with default)        - O(log n) (get() can return null -> NPE)
tm.putIfAbsent(2, 20);               // insert only if absent     - O(log n)
boolean tmHas = tm.containsKey(3);   // key existence test        - O(log n)

// Closest-key operations - all O(log n)
Integer floorKey   = tm.floorKey(4);      // largest key <= 4        - O(log n) -> 3
Integer ceilingKey = tm.ceilingKey(4);    // smallest key >= 4       - O(log n) -> 5
Integer lowerKey   = tm.lowerKey(3);      // largest key < 3         - O(log n) -> 1
Integer higherKey  = tm.higherKey(3);     // smallest key > 3        - O(log n) -> 5

Map.Entry<Integer, Integer> floorEntry   = tm.floorEntry(4);   // entry with largest key <= 4    - O(log n)
Map.Entry<Integer, Integer> ceilingEntry = tm.ceilingEntry(4); // entry with smallest key >= 4   - O(log n)

Integer firstKey = tm.firstKey();    // smallest key              - O(log n)
Integer lastKey  = tm.lastKey();     // largest key               - O(log n)
tm.remove(1);                        // delete by key             - O(log n)
int tmSize = tm.size();              // size                      - O(1)

// Iterator Navigation vs. NavigableMap Methods
Iterator<Map.Entry<Integer, Integer>> tmIt = tm.entrySet().iterator(); 
tmIt.hasNext();                      // check if next exists - O(1)
Map.Entry<Integer, Integer> nxt = tmIt.next(); // advance to next entry - O(1) amortized (O(log n) worst-case)
// Note: higherKey()/lowerKey()/ceilingKey()/floorKey() search from the tree root -> strict O(log n)
// iterator.next() moves sequentially from the current node pointer              -> amortized O(1)

```

### Multimap (JDK-only - Map to List)

```java
Map<String, List<String>> mm = new HashMap<>();
mm.computeIfAbsent("k1", k -> new ArrayList<>()).add("v1"); // computeIfAbsent: O(1) average; ArrayList.add: O(1) amortized
mm.computeIfAbsent("k1", k -> new ArrayList<>()).add("v2"); // computeIfAbsent: O(1) average; ArrayList.add: O(1) amortized

List<String> vals = mm.getOrDefault("k1", List.of()); // HashMap.getOrDefault: O(1) average
mm.get("k1").remove("v1");                            // HashMap.get: O(1) average; ArrayList.remove by object: O(n)

```

---

## 13. Min-Heap (`PriorityQueue`)

```java
import java.util.PriorityQueue;

PriorityQueue<Integer> minHeap = new PriorityQueue<>(); // min-heap by default
minHeap.offer(5);  // insert                  - O(log n)
minHeap.offer(1);  // insert                  - O(log n)
minHeap.offer(3);  // insert                  - O(log n)

Integer top    = minHeap.peek();    // read minimum (no remove)  - O(1),     returns null if empty
Integer polled = minHeap.poll();    // remove and return minimum - O(log n), returns null if empty
int size       = minHeap.size();    // size                      - O(1)
boolean empty  = minHeap.isEmpty(); // check empty               - O(1)
```

---

## 14. Max-Heap (`PriorityQueue` reversed)

```java
import java.util.PriorityQueue;
import java.util.Comparator;

PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.offer(5);  // insert                  - O(log n)
maxHeap.offer(1);  // insert                  - O(log n)
maxHeap.offer(3);  // insert                  - O(log n)

Integer max     = maxHeap.peek();    // read maximum (no remove)  - O(1),     returns null if empty
Integer removed = maxHeap.poll();    // remove and return maximum - O(log n), returns null if empty
int size        = maxHeap.size();    // size                      - O(1)
boolean empty   = maxHeap.isEmpty(); // check empty               - O(1)
```

---

## 15. PriorityQueue with Pairs / Custom Objects

```java
// Min-heap by first element of int[2] - classic Dijkstra pattern
// (dist and node are placeholder variable names)
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
pq.offer(new int[]{dist, node}); // insert {dist, node}       - O(log n)
int[] top = pq.poll();           // remove min-dist entry     - O(log n) - top[0] = dist, top[1] = node

// If values can overflow int (e.g. long distances), use Long.compare:
PriorityQueue<long[]> pq2 = new PriorityQueue<>((a, b) -> Long.compare(a[0], b[0]));
pq2.offer(new long[]{dist, node}); // insert {dist, node}     - O(log n) - same placeholder names, now long

// Custom class comparator
// For reference types, prefer Comparator.comparingInt (avoids boxing):
PriorityQueue<Person> pq3 = new PriorityQueue<>(Comparator.comparingInt(p -> p.age));
PriorityQueue<Person> pq4 = new PriorityQueue<>(Comparator.comparingInt((Person p) -> p.age).reversed());

Person person = new Person("Alice", 30); // placeholder - replace with your actual object
pq3.offer(person);               // insert by age asc         - O(log n)
pq4.offer(person);               // insert by age desc        - O(log n)
```

---

## 16. Monotonic Deque (Sliding Window Max/Min)

```java
import java.util.ArrayDeque;
import java.util.Deque;

int[] a = {1, 3, -1, -3, 5, 3, 6, 7};
int k = 3; // window size
int n = a.length;
int[] windowMax = new int[n - k + 1];

Deque<Integer> dq = new ArrayDeque<>(); // stores indices

// Overall algorithm: O(n) - each index is added and removed at most once
for (int i = 0; i < n; i++) {
    while (!dq.isEmpty() && dq.peekFirst() < i - k + 1) dq.pollFirst();  // remove out-of-window front - O(1) amortized
    while (!dq.isEmpty() && a[dq.peekLast()] <= a[i])   dq.pollLast();   // maintain decreasing order  - O(1) amortized (flip <= to >= for min)
    dq.offerLast(i);                                                       // append new index           - O(1)
    if (i >= k - 1) windowMax[i - k + 1] = a[dq.peekFirst()];            // read window max from front - O(1)
}
```

---

## 17. Bitset

> Java's `BitSet` is dynamic (grows as needed), unlike C++'s fixed-size `std::bitset<N>`.

```java
import java.util.BitSet;

BitSet bs = new BitSet(128);    // allocate with capacity hint (grows if needed)

bs.set(2);                      // set single bit              - O(1)
bs.set(10, 20);                 // set bits [10, 20)           - O(k), k = range size
boolean bit2 = bs.get(2);       // read single bit             - O(1)
bs.clear(2);                    // clear single bit            - O(1)
bs.flip(5);                     // toggle single bit           - O(1)

int count = bs.cardinality();   // popcount across all words   - O(n/64), n = highest set bit

// Iterate set bits
for (int i = bs.nextSetBit(0); i >= 0; i = bs.nextSetBit(i + 1)) {
    // process bit i                                           // O(n/64) per nextSetBit call in worst case
}

// Bitwise ops (in-place, modify bs)
BitSet other = new BitSet();
other.set(10);                  // set single bit              - O(1)
bs.and(other);                  // bs &= other                 - O(max(bs.size(), other.size()) / 64)
bs.or(other);                   // bs |= other                 - O(max(bs.size(), other.size()) / 64)
bs.xor(other);                  // bs ^= other                 - O(max(bs.size(), other.size()) / 64)

int highestPlusOne = bs.length(); // index of highest set bit + 1 (0 if empty) - O(1)
bs.clear();                       // clear all bits              - O(n/64)
```

---

## 18. BigInteger

> **Immutable** - all operations return new objects. Significantly slower than `long`; use only when precision is required.

```java
import java.math.BigInteger;
import java.security.SecureRandom;

BigInteger a = BigInteger.valueOf(123456789L);          // from long literal      - O(1)
BigInteger b = new BigInteger("9876543210123456789");   // from decimal string    - O(n), n = digits
BigInteger c = new BigInteger("deadbeef", 16);          // from hex string        - O(n), n = digits

// n = number of bits in the operands
// Arithmetic
BigInteger sum  = a.add(b);             // addition                - O(n)
BigInteger diff = b.subtract(a);        // subtraction             - O(n)
BigInteger prod = a.multiply(b);        // multiplication          - O(n^2) schoolbook for small numbers; Karatsuba O(n^1.585) / Toom-Cook 3 for very large ones
BigInteger[] dr = b.divideAndRemainder(a);  // quotient + remainder- O(n^2) (Burnikel-Ziegler for very large) - dr[0]=quotient, dr[1]=remainder
BigInteger mod  = b.mod(a);             // b % a (a must be > 0)  - O(n^2)
BigInteger modPow = a.modPow(BigInteger.valueOf(7), BigInteger.valueOf(1000)); // a^7 % 1000 - O(log(exp) * n^2)

// Comparison & GCD
BigInteger g   = a.gcd(b);             // GCD                     - O(n^2)
int        cmp = a.compareTo(b);       // compare (-1, 0, or 1)   - O(n)
boolean    eq  = a.equals(b);          // equality check           - O(n)

// Bit operations
BigInteger sl   = a.shiftLeft(10);     // a * 2^10                - O(n)
BigInteger sr   = a.shiftRight(3);     // a / 2^3 (floor)         - O(n)
boolean    bit2 = a.testBit(2);        // read single bit          - O(1)
BigInteger set5 = a.setBit(5);         // set bit 5               - O(n) (returns new object)

// Conversion
String decimal = b.toString();         // to decimal string        - O(n^2) (base conversion is expensive; JDK uses divide-and-conquer for very large numbers)
String hex     = b.toString(16);       // to hex string            - also uses the general base-conversion algorithm (not a cheap O(n) path)
byte[] bytes   = a.toByteArray();      // to byte array            - O(n)

// Primality
BigInteger prime = BigInteger.probablePrime(128, new SecureRandom()); // O(k * n^2), k = certainty rounds
boolean isPrime  = prime.isProbablePrime(100); // Miller-Rabin        - O(k * n^2), k = 100 rounds here

// Constants
BigInteger ZERO = BigInteger.ZERO;
BigInteger ONE  = BigInteger.ONE;
BigInteger TWO  = BigInteger.TWO;
```
