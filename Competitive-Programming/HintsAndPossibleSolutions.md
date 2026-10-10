# Hints & Possible Solutions To The Problem

| Signal in the problem                                                                                      | Likely technique                                      | Difficulty        |
|:-----------------------------------------------------------------------------------------------------------|:------------------------------------------------------|:------------------|
| Story-based problem, no math trick visible, just apply rules step by step                                  | Simulation                                            | Beginner          |
| Count how many times each value appears, then use those counts to answer queries                           | Hash map / frequency array                            | Beginner          |
| Given an array, answer multiple "sum of elements from index l to r" queries                                | Prefix sums                                           | Beginner          |
| Find a contiguous subarray whose sum equals k, or maximize/minimize a subarray sum                        | Prefix sums + hash map, sliding window (only if all values are non-negative), Kadane's (max/min sum) | Beginner          |
| Check if a string reads the same forwards and backwards, or find palindromic substrings                   | Two pointers, hashing                                 | Beginner          |
| Input is already sorted, or sorting it first makes the problem obviously easier                            | Binary search, two pointers                           | Beginner          |
| "Rearrange/reorder elements to satisfy a condition" — order doesn't matter, just pick the best arrangement | Sorting + greedy                                      | Beginner          |
| Among all valid answers, return the one that comes first alphabetically or has the smallest digits         | Greedy, always make the locally best character choice | Beginner          |
| n is up to 10^9 or 10^18, no loop over n possible, must derive answer from a formula or pattern           | Formula-based, math, binary search                    | Beginner          |
| n is up to 10^5 or 10^6, nested loops will TLE, need an efficient single-pass or log-factor solution       | Must be O(n log n) or O(n)                            | Beginner          |
| "Find the minimum cost/steps/moves to achieve X" or "maximize the total value"                            | Greedy or DP                                          | Easy              |
| Nodes connected by edges, traverse or find relationships between them                                     | BFS, DFS, shortest path                               | Easy              |
| Find shortest path in a graph where all edges have equal weight (or weight = 1)                           | BFS always                                            | Easy              |
| Input is a 2D grid, find paths, count regions, or check reachability between cells                        | BFS/DFS for traversal, DP for optimization            | Easy              |
| Add a constant value to every element in range [l, r] across multiple operations, then print final array  | Difference array                                      | Easy              |
| Q range-update operations (add X to [l,r]), no queries during updates, only read results at the end       | Difference array                                      | Easy              |
| Problem involves divisibility, finding GCD/LCM of numbers, or checking if a divides b                    | Number theory, Euclidean algorithm                    | Easy              |
| Check if a number is prime, find all primes up to n, or count divisors of many numbers                   | Sieve of Eratosthenes, prime factorization            | Easy              |
| Array is circular (index wraps around), or you need to handle wrap-around indexing                        | Double the array, or use modular indexing             | Easy              |
| Operations on bits directly: toggle, check, combine values using XOR/AND/OR across elements               | Bit manipulation                                      | Easy              |
| Schedule jobs/meetings with start and end times, maximize number of non-overlapping intervals              | Greedy (sort by end time), sweep line                 | Easy-Medium       |
| Count the number of distinct ways to do something, or count valid subsets/sequences                       | DP or combinatorics                                   | Easy-Medium       |
| Compute a^b where b is huge (up to 10^18), especially under a modulo                                      | Fast power (binary exponentiation)                    | Easy-Medium       |
| Answer involves very large numbers and problem says to output result mod 10^9+7                            | DP + combinatorics, modular inverse                   | Medium            |
| Find the longest/shortest subsequence (elements don't need to be adjacent) satisfying some condition      | DP (LCS-style)                                        | Medium            |
| Given two strings/sequences, find how similar they are or the minimum edits to transform one to another   | DP (edit distance, LCS)                               | Medium            |
| Find all groups of connected nodes, check if two nodes are in the same component, or merge groups         | Union-Find (DSU) or BFS/DFS                           | Medium            |
| For each element, find the nearest element to its left or right that is greater or smaller than it        | Monotonic stack                                       | Medium            |
| Count ways to choose k items from n, or compute combinations C(n,k) possibly under modulo                | Pascal's triangle, combinatorics with modular inverse | Medium            |
| Tasks have prerequisites, find a valid order to complete all tasks without violating dependencies          | Topological sort (Kahn's / DFS)                       | Medium            |
| Edges have different weights, find shortest path from source to all nodes (all weights non-negative)       | Dijkstra (non-negative weights), BFS (unweighted)     | Medium            |
| Values can be up to 10^9 but only a few thousand distinct values exist — need to index or map them        | Coordinate compression                                | Medium            |
| Same range query (sum, min, max) asked many times on a fixed unchanging array                             | Prefix sums, sparse table, segment tree               | Medium-Hard       |
| Array changes (single element updates) AND range queries both happen in the same problem                  | Fenwick tree (BIT), segment tree                      | Medium-Hard       |
| Problem asks for expected value of some outcome, or probability that something happens                    | DP on probabilities                                   | Medium-Hard       |
| Graph has edges with negative weights, find shortest path (Dijkstra won't work here)                      | Bellman-Ford                                          | Medium-Hard       |
| Need shortest path between ALL pairs of nodes, graph is small (n ≤ 500)                                   | Floyd-Warshall                                        | Medium-Hard       |
| Find if a pattern string exists in a text string, possibly many times, efficiently                        | KMP, Z-algorithm, hashing                             | Medium-Hard       |
| Given a set of strings, answer queries about common prefixes or check if a word exists in a dictionary    | Trie                                                  | Medium-Hard       |
| n ≤ 20 and problem needs you to try all combinations of elements (include/exclude each)                   | Bitmask DP or brute force                             | Hard              |
| n ≤ 40, brute force is too slow but splitting into two halves and combining results is feasible            | Meet in the middle                                    | Hard              |
| Array has both range updates (add to [l,r]) AND range queries (sum/min/max of [l,r]) interleaved          | Segment tree with lazy propagation, or two BITs (range add + range sum only, not min/max) | Hard              |
| Tree problem with queries asking about sums, max, or distances along paths or within subtrees             | DFS, tree DP, LCA                                     | Hard              |
| Two players play a game optimally, determine who wins — problem has clear states and move rules           | Game theory, Sprague-Grundy, Nim                      | Hard              |
| DP has a recurrence like f(n) = f(n-1) + f(n-2) but n is up to 10^18, too large to iterate              | Matrix exponentiation                                 | Very Hard         |
