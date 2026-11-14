# Collections and Pythonic Loops

## 1. Definition
Collections hold multiple objects: list, tuple, dict, set. "Pythonic" code uses comprehensions, unpacking, enumerate and zip instead of index-based loops.

## 2. Why it exists
Picking the right collection decides speed and readability. Python's built-in iteration tools remove index bookkeeping and reduce bugs.

## 3. Mental model
| Type | Model | Ordered | Mutable | Duplicates |
|---|---|---|---|---|
| list | Resizable array | Yes | Yes | Yes |
| tuple | Fixed record | Yes | No | Yes |
| dict | Key → value table | Yes (insertion) | Yes | Unique keys |
| set | Bag of unique items | No | Yes | No |

## 4. How it works internally
- list = dynamic array: index/append O(1) avg; `in`, `insert(0, x)` O(n).
- dict/set = hash tables: lookup/insert/`in` O(1) avg.
- Dict keys and set items must be hashable (immutable): str, int, tuple of hashables. Not list or dict.
- A mutable key could change its hash after insertion and the dict could never find it again. So `{[1, 2]: "x"}` raises `TypeError: unhashable type: 'list'`.

## 5. Important terminology
- **Hashable:** has a stable hash, so it can be a dict key or set member.
- **Comprehension:** compact expression that builds a list, dict or set.
- **Unpacking:** splitting a collection into variables (`a, *rest = seq`).
- **Iterable:** anything you can loop over.
- **View:** a live window onto a dict's keys, values or items (updates when the dict changes).

## 6. Example
```python
squares = [x * x for x in range(1, 10) if x % 2 == 1]   # [1, 9, 25, 49, 81]
lengths = {w: len(w) for w in ["ann", "bob"]}
first, *middle, last = [10, 20, 30, 40]   # middle is always a list
```
Comprehension shape: `[what_to_store for item in iterable if which_to_keep]`

## 7. Code example
```python
from collections import defaultdict, Counter

txns = [("food", 120), ("travel", 300), ("food", 80)]

totals = defaultdict(int)
for cat, amt in txns:
    totals[cat] += amt

counts = Counter(cat for cat, _ in txns)

# manual counting with .get()
words = ["apple", "bob", "apple"]
d = {}
for w in words:
    d[w] = d.get(w, 0) + 1
c = Counter(words)             # same result, takes any iterable directly

for i, (cat, amt) in enumerate(txns, start=1):
    print(i, cat, amt)

# zip stops at the shortest input
names, scores = ["a", "b", "c"], [90, 80]
list(zip(names, scores))                 # [('a', 90), ('b', 80)]  -> "c" silently dropped
list(zip(names, scores, strict=True))    # ValueError (Python 3.10+)

from itertools import zip_longest
list(zip_longest(names, scores, fillvalue=0))   # [('a',90), ('b',80), ('c',0)]
```

## 7A. Built-in methods cheat sheet

### List methods
| Method | What it does | Time | Notes |
|---|---|---|---|
| `lst.append(x)` | Adds one item at the end. | O(1) | Adds `x` as a single element, even if `x` is a list. |
| `lst.extend(it)` | Adds every item of an iterable at the end. | O(k) | `[1].extend([2,3])` gives `[1,2,3]`. |
| `lst.insert(i, x)` | Inserts `x` before index `i`. | O(n) | Shifts all later items right. |
| `lst.remove(x)` | Removes the first item equal to `x`. | O(n) | Raises `ValueError` if missing. |
| `lst.pop()` | Removes and returns the last item. | O(1) | Raises `IndexError` on an empty list. |
| `lst.pop(i)` | Removes and returns the item at index `i`. | O(n) | Shifts later items left. |
| `lst.index(x)` | Returns the index of the first `x`. | O(n) | Raises `ValueError` if missing. |
| `lst.count(x)` | Counts how many times `x` appears. | O(n) | Scans the whole list. |
| `lst.sort(key=, reverse=)` | Sorts the list **in place**. | O(n log n) | **Returns `None`**. Stable sort. |
| `sorted(lst)` | Returns a **new** sorted list. | O(n log n) | Works on any iterable. |
| `lst.reverse()` | Reverses the list in place. | O(n) | Returns `None`. `reversed(lst)` gives an iterator instead. |
| `lst.copy()` | Makes a shallow copy. | O(n) | Same as `lst[:]`. |
| `lst.clear()` | Removes all items. | O(n) | The list object stays the same. |

```python
nums = [3, 1, 2]
nums.append(4)          # [3, 1, 2, 4]
nums.extend([5, 6])     # [3, 1, 2, 4, 5, 6]
nums.append([7, 8])     # [3, 1, 2, 4, 5, 6, [7, 8]]  <- nested, not merged
x = nums.sort()         # x is None (common bug: nums = nums.sort())
people = [("bob", 30), ("ann", 25)]
people.sort(key=lambda p: p[1])     # sort by age
```

Slicing: `lst[start:stop:step]` makes a new list. `lst[::-1]` reverses it. `lst[1:3]` gives items at index 1 and 2.

### Dict methods
| Method | What it does | Notes |
|---|---|---|
| `d.get(k, default)` | Returns the value, or `default` if the key is missing. | Never raises `KeyError`. |
| `d.setdefault(k, v)` | Returns the value; inserts `k: v` first if the key is missing. | Handy for grouping. |
| `d.update(other)` | Merges another dict in, overwriting existing keys. | Also accepts keyword arguments. |
| `d.pop(k, default)` | Removes the key and returns its value. | Raises `KeyError` if missing and no default is given. |
| `d.popitem()` | Removes and returns the **last inserted** pair. | Raises `KeyError` on an empty dict. |
| `d.keys()` / `d.values()` / `d.items()` | Return live **views** of the dict. | Views update when the dict changes. |
| `d.copy()` | Makes a shallow copy. | Same shallow-copy caveat as lists. |
| `d.clear()` | Removes all pairs. | |
| `dict.fromkeys(keys, v)` | Builds a dict with every key set to `v`. | Don't use a mutable `v`; all keys share it. |
| `d1 \| d2` | Returns a merged dict (Python 3.9+). | Right-hand values win. |
| `del d[k]` | Deletes the key. | Raises `KeyError` if missing. |

All lookup, insert and delete operations are **O(1)** on average.

```python
d = {"a": 1}
d.setdefault("b", []).append(5)     # {'a': 1, 'b': [5]}
for k, v in d.items():              # loop over key and value together
    print(k, v)
```

### Set methods
| Method / operator | What it does | Notes |
|---|---|---|
| `s.add(x)` | Adds one item. | Does nothing if already present. |
| `s.remove(x)` | Removes `x`. | Raises `KeyError` if missing. |
| `s.discard(x)` | Removes `x` if present. | Never raises an error. |
| `s.pop()` | Removes and returns an **arbitrary** item. | Don't rely on which one. |
| `a \| b` or `a.union(b)` | Items in either set. | |
| `a & b` or `a.intersection(b)` | Items in both sets. | |
| `a - b` or `a.difference(b)` | Items in `a` but not in `b`. | |
| `a ^ b` | Items in exactly one of the two sets. | Symmetric difference. |
| `a.issubset(b)` / `a <= b` | True if every item of `a` is in `b`. | |
| `a.isdisjoint(b)` | True if the sets share nothing. | |

`frozenset` is an immutable set, so it can be a dict key or live inside another set.

### Tuple methods
Tuples have only two: `t.count(x)` and `t.index(x)`. Everything else works through unpacking, slicing and `len()`. Use `collections.namedtuple` or a `dataclass` when you want named fields.

### Handy built-ins for any collection
`len(x)`, `sum(x)`, `min(x)`, `max(x)`, `any(x)`, `all(x)`, `sorted(x)`, `enumerate(x)`, `zip(a, b)`, `reversed(x)`, `x in c`.

## 7B. The `collections` module: defaultdict, Counter, deque, namedtuple
`collections` is part of the standard library. It gives specialised versions of the built-in containers.

### defaultdict
**What it is:** a dict that never raises `KeyError` for a missing key. You give it a **factory** (a function that takes no arguments). When you access a missing key, it calls the factory, **stores** the result under that key, and returns it.

```python
from collections import defaultdict

# Without defaultdict: you must check the key first
totals = {}
for cat, amt in [("food", 120), ("travel", 300), ("food", 80)]:
    if cat not in totals:
        totals[cat] = 0
    totals[cat] += amt

# With defaultdict(int): the missing key starts at int() == 0
totals = defaultdict(int)
for cat, amt in [("food", 120), ("travel", 300), ("food", 80)]:
    totals[cat] += amt            # {'food': 200, 'travel': 300}

# Grouping: defaultdict(list) starts each missing key as []
groups = defaultdict(list)
for cat, amt in [("food", 120), ("travel", 300), ("food", 80)]:
    groups[cat].append(amt)       # {'food': [120, 80], 'travel': [300]}
```

| Factory | Missing key starts as | Typical use |
|---|---|---|
| `int` | `0` | Counting and summing. |
| `list` | `[]` | Grouping items by a key. |
| `set` | `set()` | Grouping unique items by a key. |
| `lambda: "N/A"` | `"N/A"` | Any custom default value. |

**How it works internally:** `defaultdict` is a dict subclass. When `d[key]` fails, Python calls the special method `__missing__`, which runs your factory, inserts the result, and returns it.

**Gotchas:**
- **Just reading a missing key creates it.** `d["x"]` inserts `"x"` with the default value. `key in d` and `d.get(key)` do **not** insert, so use those to check without side effects.
- The factory must be a **callable**, not a value: write `defaultdict(list)`, not `defaultdict([])`.
- Convert back with `dict(d)` if you want a plain dict (for example, before returning JSON).

**vs `setdefault`:** `d.setdefault(k, []).append(x)` does the same job on a plain dict, but `defaultdict(list)` is cleaner when you do it many times.

### Counter
**What it is:** a dict subclass built for **counting hashable items**. Keys are the items and values are their counts.

```python
from collections import Counter

words = ["apple", "bob", "cat", "apple", "bob", "apple"]
c = Counter(words)
print(c)                    # Counter({'apple': 3, 'bob': 2, 'cat': 1})

c["apple"]                  # 3
c["zebra"]                  # 0   (missing key gives 0, no KeyError, and is NOT inserted)

c.most_common(2)            # [('apple', 3), ('bob', 2)]  top-N by count
c.update(["cat", "cat"])    # adds counts: cat becomes 3
c.total()                   # total of all counts (Python 3.10+)
list(c.elements())          # expands back: ['apple','apple','apple','bob','bob','cat',...]

Counter("hello")            # counts characters: {'l': 2, 'h': 1, 'e': 1, 'o': 1}
```

| Operation | What it does |
|---|---|
| `Counter(iterable)` | Counts every item in the iterable in one pass. |
| `c[item]` | Returns the count, or 0 if the item was never seen. |
| `c.most_common(n)` | Returns the `n` most frequent items as `(item, count)` pairs, highest first. |
| `c.update(iterable)` | **Adds** counts from another iterable (unlike `dict.update`, which overwrites). |
| `c.subtract(iterable)` | Subtracts counts, and allows zero or negative results. |
| `c1 + c2`, `c1 - c2` | Adds or subtracts counts; the result keeps only positive counts. |
| `c1 & c2`, `c1 \| c2` | Takes the minimum or maximum of each item's count. |

**Counter vs defaultdict(int):** both can count. Use `Counter` when you only need counts (it gives `most_common`). Use `defaultdict(int)` when you add up other values (like amounts) per key.

### deque (double-ended queue)
**What it is:** a list-like container optimised for adding and removing at **both ends**.

```python
from collections import deque

q = deque([1, 2, 3])
q.append(4)         # add right:  [1, 2, 3, 4]
q.appendleft(0)     # add left:   [0, 1, 2, 3, 4]
q.pop()             # remove right (returns 4)
q.popleft()         # remove left  (returns 0), O(1) (a list's pop(0) is O(n))

recent = deque(maxlen=3)   # fixed size: old items fall off automatically
for x in range(5):
    recent.append(x)       # ends as deque([2, 3, 4], maxlen=3)
```
**Use it for:** queues, "last N items" windows, and breadth-first traversal. **Trade-off:** indexing in the middle (`q[500]`) is O(n), unlike a list.

### namedtuple
**What it is:** a tuple whose positions also have **names**. It is still immutable and memory-light.

```python
from collections import namedtuple

Txn = namedtuple("Txn", ["category", "amount"])
t = Txn("food", 120)
t.category          # 'food'  (readable access by name)
t[1]                # 120     (still works by index)
cat, amt = t        # unpacking still works
```
Use it for small read-only records. Later, when you need defaults, type hints or validation, use a `dataclass` (Topic 8).

## 8. Real-world usage
- Grouping and counting records (Transaction Processor, analytics).
- Fast membership checks with sets.
- Chunk metadata as dicts in RAG; groupby ideas carry into Pandas.

## 9. Common mistakes
- `x in list` inside a loop (use a set).
- Modifying a list/dict while iterating over it.
- `d[key]` on a missing key (use `.get()` or `defaultdict`).
- `range(len(x))` instead of `enumerate` / direct iteration.
- Over-nested comprehensions that are unreadable.
- Expecting set order.
- `zip` silently drops extra items when lengths differ. Use `strict=True` or `zip_longest`.
- Writing the filter but forgetting the expression (odd numbers vs squares of odd numbers).
- `nums = nums.sort()` sets `nums` to `None` because `sort()` returns `None`.
- `append` of a list creates a nested list; use `extend` to merge.

## 10. Performance considerations
- Convert to `set` once for repeated membership checks: 10,000 lookups against 1 million IDs is ~10,000 steps with a set, versus up to ~10 billion with a list.
- `list.insert(0, x)` / `pop(0)` are O(n); use `collections.deque` for queues.
- Comprehensions are usually faster than equivalent append loops.
- Generator expressions avoid building large lists in memory.

## 11. Security considerations
- Dicts built from untrusted input can hold unexpected keys; validate keys and values.
- Never use unvalidated user input as a key that later triggers logic.

## 12. Interview questions and answers

**1. List vs tuple: when to use each?**
A list is a mutable, ordered sequence, so use it for a collection that grows or changes, such as a queue of tasks. A tuple is immutable, so use it for a fixed record such as `(lat, lng)` or a database row. Because a tuple is immutable and (if its items are) hashable, it can be a dict key or set member. A tuple is also slightly smaller and faster to create.

**2. Why is `in` faster on a set than on a list?**
A list checks items one by one, which is O(n). A set stores items in a hash table: Python computes the item's hash, jumps directly to that slot and checks there, which is O(1) on average. Convert to a set once if you check membership many times.

**3. What makes an object hashable? Why can't a list be a dict key?**
An object is hashable if it has a hash value that never changes during its lifetime, and it can be compared with `==`. Immutable types (`str`, `int`, `tuple` of hashables) qualify. A list is mutable, so its contents and therefore its hash could change after insertion, and the dict would never find the entry again. So Python raises `TypeError: unhashable type: 'list'`.

**4. `.get()` vs `d[key]` vs `defaultdict`?**
`d[key]` raises `KeyError` if the key is missing. `d.get(key, default)` returns the default and does **not** insert anything. `defaultdict(factory)` creates and **stores** a default value automatically on first access, which is ideal for grouping and counting (`defaultdict(list)`, `defaultdict(int)`).

**5. How does `zip` handle unequal lengths? How do you prevent silent data loss?**
`zip` stops at the shortest input and silently drops the rest. Use `zip(a, b, strict=True)` (Python 3.10+) to raise a `ValueError` on a mismatch, or `itertools.zip_longest(a, b, fillvalue=...)` to keep everything.

**6. Complexity of `list.insert(0, x)`?**
O(n), because every existing item must shift one position right. Appending at the end is O(1) on average. If you need fast inserts or pops at both ends, use `collections.deque`, where both are O(1).

**7. `append` vs `extend`?**
`append(x)` adds `x` as **one** element, so appending a list creates a nested list. `extend(iterable)` adds each element of the iterable individually.

**8. `list.sort()` vs `sorted()`?**
`sort()` sorts in place and returns `None`. `sorted()` returns a new list and works on any iterable. Both are stable and take `key=` and `reverse=`.

**9. `remove` vs `pop` vs `del` on a list?**
`remove(x)` deletes by **value** (first match). `pop(i)` deletes by **index** and returns the item. `del lst[i]` deletes by index and returns nothing. `remove` and `pop(i)` are O(n).

**10. `remove` vs `discard` on a set?**
`remove` raises `KeyError` if the item is missing, and `discard` does nothing in that case.

**11. What is `defaultdict` and how is it different from a normal dict?**
A `defaultdict` takes a factory such as `int`, `list` or `set`. When you access a missing key, it calls the factory, stores the result under that key, and returns it, so you never get a `KeyError`. A normal dict raises `KeyError`. The catch is that merely reading a missing key inserts it, so use `in` or `.get()` when you don't want that.

**12. `Counter` vs `defaultdict(int)`?**
Both count items. `Counter` is built for counting: it counts a whole iterable in one call, returns 0 for missing keys without inserting them, and provides `most_common(n)`. `defaultdict(int)` is more general and fits better when you sum values (like transaction amounts) per key.

**13. When would you use `deque` instead of a list?**
When you add or remove at the **front**. `deque.popleft()` and `appendleft()` are O(1), while `list.pop(0)` and `list.insert(0, x)` are O(n). It also supports `maxlen` for "last N items" windows. Indexing in the middle is slower than a list.

## 13. Things to remember
- Need fast lookup or uniqueness → set or dict.
- Need ordered, changeable sequence → list. Fixed record → tuple.
- Use `enumerate` for index + item, `zip` for parallel iteration.
- `defaultdict(list)` for grouping, `defaultdict(int)` for summing, `Counter` for counting (and `most_common`).
- Reading a missing key on a `defaultdict` creates it; a `Counter` returns 0 without creating it.
- `deque` for fast operations at both ends; `namedtuple` for small read-only records.
- `*middle` in unpacking always becomes a list.
- `sort()` returns `None`; `sorted()` returns a new list.

## 14. One-page revision summary
list = ordered array, O(n) search. tuple = immutable record. dict/set = hash tables, O(1) lookup, hashable keys only. Prefer comprehensions, `enumerate`, `zip`, unpacking. Group with `defaultdict`, count with `Counter`. Never mutate while iterating. Convert to set for repeated `in` checks. `zip` drops extras silently. `append` adds one item, `extend` adds many. `sort()` is in place and returns `None`.