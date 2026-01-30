# Ch4 — Data Structures

Two angles: (1) When to use which DS. (2) How to implement common ones when asked.

## Heap

- **When:** Top K, kth largest, merge K sorted
- **Implement:** Array-based; parent `(i-1)/2`, left `2i+1`, right `2i+2`; siftUp, siftDown; (template to fill)

## Monotonic stack

- **When:** Next greater/smaller element; span; one-pass
- **Implement:** Usually inline in problem; maintain stack invariant; (to fill)

## Trie

- **When:** Prefix search; "starts with"; word search
- **Implement:** Node with `children` map/array + `isEnd`; insert, search, startsWith; (template to fill)

## LRU Cache

- **When:** "LRU cache", "get/put with capacity", evict least recently used
- **Implement:** Hash map (key → node) + doubly linked list (order by recency); get → move to front; put → add/move to front, evict tail if over capacity; (template to fill)

## Other (queue with stacks, etc.)

- (To fill when needed)

## Failure modes

- (To fill per DS)
