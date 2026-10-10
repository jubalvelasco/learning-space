# STL Types and Data Structures

## Sequence / array-like containers

* **`std::vector<T>`** — contiguous dynamic array.
  Use: default container for lists, adjacency lists, DP arrays.
  Key ops: `push_back`, `operator[]`, `size`, `reserve`, `begin`/`end`.
  Tip: `reserve(n)` to avoid reallocations; best default choice.

* **`std::string`** — dynamic sequence of characters.
  Use: input, string algorithms, parsing.
  Key ops: `substr`, `find`, `size`, `compare`, `push_back`.
  Tip: use `string_view` to avoid copies.

* **`std::string_view` (C++17)** — non-owning view into a string.
  Use: fast parsing / slices.
  Tip: be careful about lifetime of the referenced string.

* **`std::array<T,N>`** — fixed-size array (contiguous).
  Use: small fixed-dimension arrays, stack allocation, structured bindings.
  Key ops: `operator[]`, `fill`, `begin`/`end`.

* **`std::deque<T>`** — double-ended queue (blocks of arrays).
  Use: sliding-window, BFS with push_front/pop_front.
  Key ops: `push_front`, `push_back`, `pop_front`, `operator[]`.
  Tip: good when you need efficient both-ends ops.

* **`std::list<T>` / `std::forward_list<T>`** — (double/singly) linked lists.
  Use: rare in CP; useful for splicing or O(1) erase with iterator.
  Tip: usually avoid—pointer-heavy and slower than vector/deque.

* **`std::bitset<N>`** — fixed-size bitset with bitwise ops.
  Use: subset DP, masks, fast bit operations.
  Key ops: `count()`, `to_ulong()`, bitwise ops.
  Tip: often faster and safer than `vector<bool>`.

---

## Associative (ordered) containers

* **`std::set<T>`** — ordered unique keys (node-based; commonly red–black tree).
  Use: sorted unique keys, predecessor/successor queries.
  Key ops: `insert`, `erase`, `find`, `lower_bound`, `upper_bound`.
  Tip: use `lower_bound` + `--it` to get predecessor (if any).

* **`std::multiset<T>`** — like `set` but allows duplicates.
  Use: multisets of values (e.g., free times), erase by iterator.
  Tip: `erase(value)` removes *all* equal elements — use `find`/`erase(it)` to remove one.

* **`std::map<K,V>`** — ordered associative map (key→value).
  Use: ordered dictionaries, coordinate compression with order.
  Key ops: `operator[]` (inserts), `find`, `lower_bound`.
  Tip: prefer `find` over `operator[]` when not inserting.

* **`std::multimap<K,V>`** — ordered map allowing duplicate keys.
  Use: grouping values by key when duplicates matter.

---

## Associative (hash-based) containers

* **`std::unordered_map<K,V>`** — hash table (average O(1)).
  Use: frequency counting, memoization.
  Tip: call `reserve(n)` and consider custom hash (splitmix64) for `long long`/pair to avoid hacks.

* **`std::unordered_set<T>`** — hash set (average O(1)).
  Use: fast membership/dedup.
  Tip: tune `reserve` and `max_load_factor` for performance.

* **`std::unordered_multimap` / `std::unordered_multiset`** — hash containers allowing duplicates.

---

## Priority / queue adapters & heap algorithms

* **`std::priority_queue<T,Container,Compare>`** — binary heap adapter (max-heap default).
  Use: Dijkstra, greedy selection, k-largest.
  Key ops: `top`, `push`, `pop`.
  Tip: `priority_queue<T, vector<T>, greater<T>>` for min-heap.

* **`std::queue<T>`** — FIFO queue (BFS).

* **`std::stack<T>`** — LIFO stack (DFS iterative).
  (Both are container adapters backed by `std::deque` by default; `stack` can also use `vector`.)

* **Heap algorithms**: `std::make_heap`, `std::push_heap`, `std::pop_heap` — alternative manual heap ops on a `vector`.

---

## Tuples / pairs / small aggregate types

* **`std::pair<T1,T2>`** — simple 2-tuple; sorts lexicographically.
  Use: (value, index), intervals `(start,end)`.
  Tip: structured bindings `auto [a, b] = p;` in C++17.

* **`std::tuple<Ts...>`** — general fixed-size heterogeneous tuple.
  Use: multi-key sorts, returning multiple values.
  Key ops: `get<i>`, `tie`.

---

## Optional / variant / function wrappers

* **`std::optional<T>` (C++17)** — value or none.
  Use: optional return, sentinel.
  Ops: `has_value()`, `value()`, `emplace`.

* **`std::variant<Ts...>` (C++17)** — type-safe union.
  Use: heterogeneous states (rare in CP).

* **`std::any` (C++17)** — type-erased single value (rare in CP).

* **`std::function<R(Args...)>`** — type-erased callable.
  Use: recursive lambdas (but slower — prefer templates for hot code).

---

## Misc useful types & utilities

* **`std::pair`, `std::tuple`** covered above (used heavily for sorting keys).
* **`std::complex<T>`** — complex numbers; used in FFT.
* **`std::chrono::steady_clock` / durations** — microbenchmarking local runs.
* **I/O streams**: `std::iostream`, `std::ostringstream`, `std::stringstream` — for formatting (rare on judge).
* **`std::reference_wrapper<T>`** — store references in containers (rare).
* **`std::function_ref`** — only added in C++26, not available on most judges (skip).

---

## Random & distributions (common in testers / randomized algorithms)

* **`std::mt19937`, `std::mt19937_64`** — high-quality RNG engines.
  Use: randomized tests, randomized algorithms.
  Tip: seed with `std::random_device{}()` (or `chrono::steady_clock::now().time_since_epoch().count()`) or a fixed seed for repeatability.

* **`std::uniform_int_distribution`, `std::uniform_real_distribution`** — mapping RNG to ranges.

---

## Algorithms & function objects (not types but essential tools)

(These are functions/algorithms you’ll rely on heavily — listed because they frequently pair with types above.)

* `std::sort`, `std::stable_sort`, `std::nth_element`
* `std::lower_bound`, `std::upper_bound`, `std::binary_search`
* `std::next_permutation`, `std::prev_permutation`
* `std::iota`, `std::accumulate`, `std::partial_sum`
* `std::unique`, `std::erase` (C++20), `std::rotate`, `std::reverse`
* `std::make_pair`, `std::make_tuple`, `std::tie`
* `std::move`, `std::forward` (move/forward semantics)
* `std::hash<T>` (hash functor used by unordered containers)

---

## Quick picks & tips (cheat-sheet)

* Default container: **`vector`**.
* Need sorted order + predecessor/successor: **`set` / `map` / `multiset`**.
* Need fastest average lookups: **`unordered_map` / `unordered_set`** (remember `reserve`).
* Need min-heap: **`priority_queue<..., greater<...>>`** or `make_heap` on vector.
* Need bitwise DP/subset ops: **`bitset`**.
* Want optional return: **`optional`**.
* For random tests: **`mt19937`** + distributions.
* For interval problems: store as `vector<pair<int,int>>`, `sort`, and use `lower_bound`/`priority_queue`/`multiset` idioms.
