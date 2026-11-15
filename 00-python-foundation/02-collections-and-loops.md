# 02 — Collections and Pythonic Loops

> Phase 0 · Python foundation. Collections are how you hold tickets, chunks, embeddings and metadata. Choosing the right one decides speed and clarity.
> Running example: **support tickets**.
> `🎯 Interview:` marks a point worth remembering for interviews. `⚠️ Common confusion:` marks a place where I got stuck, with the reason explained.

---

## 1. The four collections

### 1.1 Definition
**Syntax to create each one:**
```
my_list  = [item1, item2, item3]          # or list(iterable) ; empty: []
my_tuple = (item1, item2)                 # one item needs a comma: (item1,) ; empty: ()
my_dict  = {key1: value1, key2: value2}   # or dict(key1=value1) ; empty: {}
my_set   = {item1, item2}                 # or set(iterable) ; empty: set()  (NOT {})
```
A **collection** holds many objects in one place. Python has four main ones: **list**, **tuple**, **dict**, **set**.

### 1.2 Which one to use

| Type | What it is | Ordered | Changeable | Duplicates |
|---|---|---|---|---|
| `list` | A resizable array of items. | Yes | Yes | Allowed |
| `tuple` | A fixed record that cannot change. | Yes | No | Allowed |
| `dict` | A lookup table of `key → value`. | Yes (insertion order) | Yes | Keys are unique |
| `set` | A bag of unique items. | No | Yes | Not allowed |

### 1.3 Important points
- **list** = dynamic array. Index and `append` are fast (O(1)). Searching with `in` is slow (O(n)).
- **dict and set** are **hash tables**: lookup, insert and `in` are fast (O(1) on average).
- Dict keys and set items must be **hashable**.

### 1.4 ⚠️ Common confusion: what does "hashable" mean?
Python finds a dict key by computing its **hash** (a number from its content) and jumping straight to that slot. That only works if the hash **never changes**. A list can change, so its hash could change after you store it, and the dict would never find it again.
```python
d = {[1, 2]: "x"}          # TypeError: unhashable type: 'list'
d = {(1, 2): "x"}          # works: a tuple of numbers can't change
```
**Rule:** keys must be immutable: `str`, `int`, `tuple` of immutables.

### 1.5 ⚠️ Common confusion: why is `in` faster on a set?
Question: you have 1 million ticket ids and must check 10,000 others against them.
- **list:** `x in my_list` checks items one by one. Up to 1,000,000 checks **per lookup**, about 10 billion in total.
- **set:** hash `x`, jump to its slot. About 10,000 steps in total.
```python
valid_ids = set(big_list)                       # build once
matches = [i for i in other_ids if i in valid_ids]
```
🎯 **Interview:** list vs set membership: O(n) vs O(1). Also `list.insert(0, x)` and `list.pop(0)` are O(n) because every item shifts. Use `collections.deque` for queues.

---

## 2. List, dict, set: the methods you need

### 2.1 List
| Method | What it does | Time |
|---|---|---|
| `lst.append(x)` | Add **one** item at the end. | O(1) |
| `lst.extend(items)` | Add **each** item of another iterable. | O(k) |
| `lst.insert(i, x)` | Insert before index `i`. | O(n) |
| `lst.remove(x)` | Remove the first `x` (error if missing). | O(n) |
| `lst.pop()` / `lst.pop(i)` | Remove and return the last item / item at `i`. | O(1) / O(n) |
| `lst.sort()` | Sort **in place**, returns `None`. | O(n log n) |
| `sorted(lst)` | Return a **new** sorted list. | O(n log n) |
| `lst.index(x)`, `lst.count(x)` | Position of `x` / how many `x`. | O(n) |
| `lst.copy()`, `lst[:]` | Shallow copy. | O(n) |

```python
ids = [3, 1, 2]
ids.append([7, 8])        # [3, 1, 2, [7, 8]]   nested, not merged
ids = [3, 1, 2]
ids.extend([7, 8])        # [3, 1, 2, 7, 8]
x = ids.sort()            # x is None   (common bug: ids = ids.sort())
```
🎯 **Interview:** `append` adds one item (a list becomes nested). `extend` adds each item. `sort()` returns `None`, `sorted()` returns a new list. Both sorts are stable.

**Syntax:**
```
lst.append(item)              lst.extend(iterable)         lst.insert(index, item)
lst.remove(item)              lst.pop()  /  lst.pop(index)
lst.index(item)               lst.count(item)              item in lst
lst.sort(key=None, reverse=False)           sorted(iterable, key=None, reverse=False)
lst.reverse()                 lst.copy()                   lst.clear()
len(lst)                      lst1 + lst2                  lst * n
```
**Every list method with an example** (each block starts from the list shown):
```python
l = [10, 20, 30]
l.append(40)           # [10, 20, 30, 40]       add one at the end
l.insert(1, 15)        # [10, 15, 20, 30, 40]   insert BEFORE index 1
l.remove(20)           # [10, 15, 30, 40]       remove the first 20
l.remove(99)           # ValueError: list.remove(x): x not in list
```
```python
l = [10, 15, 30, 40]
last = l.pop()         # last = 40, l = [10, 15, 30]      remove and return the last
first = l.pop(0)       # first = 10, l = [15, 30]         remove and return index 0
```
```python
l = [3, 1, 2, 3]
l.index(3)             # 0        position of the first 3
l.count(3)             # 2        how many 3s
3 in l                 # True     membership
len(l)                 # 4
```
```python
l = [3, 1, 2, 3]
sorted(l)                       # [1, 2, 3, 3]   new list, l unchanged
sorted(l, reverse=True)         # [3, 3, 2, 1]
l.sort()                        # l is now [1, 2, 3, 3]   in place, returns None
l.reverse()                     # l is now [3, 3, 2, 1]   in place
l[::-1]                         # [1, 2, 3, 3]   reversed copy
list(reversed(l))               # [1, 2, 3, 3]   another reversed copy
```
```python
c = l.copy()           # shallow copy
c.clear()              # c = []     remove everything (l is untouched)
[1, 2] + [3]           # [1, 2, 3]  join two lists into a new one
[0] * 3                # [0, 0, 0]
```
```python
x = [1, 2, 3]
x.extend("ab")         # [1, 2, 3, 'a', 'b']   extend takes ANY iterable (a string gives letters)
```

### 2.2 Dict
| Method | What it does |
|---|---|
| `d.get(k, default)` | Value or `default`. Never raises `KeyError`. |
| `d.setdefault(k, v)` | Return the value; if the key is missing, insert `k: v` first. |
| `d.update(other)` | Merge another dict in; new values overwrite. |
| `d.pop(k, default)` | Remove the key and return its value. |
| `d.keys()`, `d.values()`, `d.items()` | Live **views** of keys, values, pairs. |
| `d1 \| d2` | Merged dict (Python 3.9+). |

```python
ticket = {"id": 101, "text": "Refund not received"}
ticket.get("priority", "normal")        # 'normal'  (no KeyError)
for key, value in ticket.items():
    print(key, value)
```
🎯 **Interview:** `d[key]` raises `KeyError` if missing. `d.get(key, default)` doesn't, and doesn't insert anything.

**Syntax:**
```
d[key]                        d[key] = value               del d[key]
d.get(key)  /  d.get(key, default)
d.setdefault(key, default)    d.update(other_dict)  /  d.update(key=value)
d.pop(key)  /  d.pop(key, default)       d.popitem()
d.keys()      d.values()      d.items()
key in d                      len(d)
d1 | d2                       {**d1, **d2}
dict(zip(keys, values))       dict.fromkeys(keys, value)
{key_expr: value_expr for item in iterable if condition}
```
**Every dict operation with an example:**
```python
d = {"id": 101, "text": "Refund"}
d["id"]                    # 101
d["x"]                     # KeyError: 'x'
d.get("x")                 # None
d.get("x", "normal")       # 'normal'
```
```python
d["priority"] = "high"     # add a new key
d["id"] = 102              # change an existing key
# d is now {'id': 102, 'text': 'Refund', 'priority': 'high'}
```
```python
d.setdefault("status", "open")   # 'open'  key was missing, so it is inserted
d.setdefault("id", 0)            # 102     key exists, so nothing changes
# d now also has 'status': 'open'
```
```python
d.update({"id": 103, "agent": "Ann"})   # merge: 'id' is overwritten, 'agent' is added
d.update(owner="Bob")                   # keyword form also works
d.pop("owner")                          # 'Bob'    removed and returned
d.pop("nope", "none")                   # 'none'   default avoids KeyError
del d["agent"]                          # remove a key (KeyError if missing)
d.popitem()                             # removes and returns the LAST pair
```
```python
d = {"id": 103, "text": "Refund", "status": "open"}
list(d.keys())       # ['id', 'text', 'status']
list(d.values())     # [103, 'Refund', 'open']
list(d.items())      # [('id', 103), ('text', 'Refund'), ('status', 'open')]
"id" in d            # True    `in` checks KEYS
"Refund" in d        # False   not the values
len(d)               # 3
```
Merging and building:
```python
a = {"x": 1, "y": 2}
b = {"y": 9, "z": 3}
a | b                        # {'x': 1, 'y': 9, 'z': 3}   the right side wins on clashes
{**a, **b}                   # {'x': 1, 'y': 9, 'z': 3}   same result (works in older Python too)
b | a                        # {'y': 2, 'z': 3, 'x': 1}   order matters
dict(zip(["a", "b"], [1, 2]))    # {'a': 1, 'b': 2}   build from two lists
dict(a=1, b=2)                   # {'a': 1, 'b': 2}
{k: v * 2 for k, v in a.items()} # {'x': 2, 'y': 4}  dict comprehension
dict.fromkeys(["a", "b"], 0)     # {'a': 0, 'b': 0}  same starting value for every key
sorted(a.items(), key=lambda kv: kv[1], reverse=True)   # [('y', 2), ('x', 1)]  sort by value
```

### 2.3 Set
| Method / operator | What it does |
|---|---|
| `s.add(x)` | Add an item (no effect if present). |
| `s.remove(x)` / `s.discard(x)` | Remove; `remove` raises `KeyError` if missing, `discard` doesn't. |
| `a \| b`, `a & b`, `a - b`, `a ^ b` | Union, intersection, difference, in-exactly-one. |

```python
today = {"billing", "login"}
yesterday = {"login", "shipping"}
print(today & yesterday)       # {'login'}
print(today - yesterday)       # {'billing'}
```
**Real use:** remove duplicate tickets with `set(ids)`, check "have I seen this chunk before?".

**Syntax:**
```
s.add(item)                   s.update(iterable)
s.remove(item)                s.discard(item)
a | b   a & b   a - b   a ^ b             # union, intersection, difference, in-exactly-one
a <= b  (or a.issubset(b))    a.issuperset(b)      a.isdisjoint(b)
item in s                     len(s)               set(iterable)
```
**Every set operation with an example:**
```python
s = {1, 2, 2, 3}          # {1, 2, 3}      duplicates vanish
set([1, 1, 2])            # {1, 2}         make a set from a list
set()                     # an EMPTY set. Careful: {} is an empty DICT, not a set
s.add(4)                  # {1, 2, 3, 4}
s.add(4)                  # no change (already there)
s.update([7, 8])          # add many at once
s.remove(4)               # removes 4
s.remove(99)              # KeyError: 99
s.discard(99)             # no error if missing
```
```python
a = {1, 2, 3}
b = {3, 4}
a | b                     # {1, 2, 3, 4}   union: in either
a & b                     # {3}            intersection: in both
a - b                     # {1, 2}         difference: in a but not b
b - a                     # {4}
a ^ b                     # {1, 2, 4}      in exactly one of them
{1, 2} <= a               # True           is it a subset?
a.issuperset({1})         # True
a.isdisjoint({9})         # True           nothing in common
2 in a                    # True
len(a)                    # 3
```
```python
list(set([3, 1, 3, 2]))        # [1, 2, 3]   remove duplicates (order NOT guaranteed)
sorted(set([3, 1, 3, 2]))      # [1, 2, 3]   duplicates removed AND sorted
```

### 2.4 Tuple
**Syntax:**
```
t = (a, b, c)          t = (a,)          a, b = t
t[index]               t[start:stop]     t.count(item)      t.index(item)
```
Only two methods: `count` and `index`. Use a tuple for a **fixed record** (`(lat, lng)`, one database row). A tuple is also hashable if all its items are, so it can be a dict key.
```python
t = (1, 2, 2, 3)
t.count(2)        # 2
t.index(3)        # 3       position of 3
len(t)            # 4
t[0], t[-1]       # (1, 3)
t[1:3]            # (2, 2)  slicing gives a tuple
t[0] = 5          # TypeError: 'tuple' object does not support item assignment
```
The one-item trap:
```python
type((1,))        # <class 'tuple'>   the comma makes the tuple
type((1))         # <class 'int'>     brackets alone do nothing
x, y = (5, 6)     # unpacking: x = 5, y = 6
```

### 2.5 Built-in functions that work on collections
**Syntax:**
```
len(x)    sum(iterable, start=0)    min(iterable, key=None, default=...)    max(iterable, key=None, default=...)
sorted(iterable, key=None, reverse=False)    reversed(sequence)
any(iterable)    all(iterable)    range(start, stop, step)
```
```python
nums = [3, 9, 2]
len(nums)                    # 3
sum(nums)                    # 14
min(nums), max(nums)         # (2, 9)
sorted(nums)                 # [2, 3, 9]
max(["bb", "a"], key=len)    # 'bb'      key= works here too
max([], default=0)           # 0         without default, an empty list raises ValueError
any([0, 0, 5])               # True      is at least one item truthy?
all([1, 2, 0])               # False     are ALL items truthy?
any([]), all([])             # (False, True)   empty: any is False, all is True
list(range(1, 10, 3))        # [1, 4, 7]
```
`any` and `all` are great for rules: `all(len(t) > 0 for t in texts)` ("is every text non-empty?").

---

## 3. Slicing and unpacking

### 3.1 Definition
- **Slicing** takes part of a sequence: `lst[start:stop:step]` (stop is not included). It makes a **new** list.
- **Unpacking** splits a collection into separate variables.

### 3.2 Example
```python
nums = [10, 20, 30, 40]
print(nums[1:3])               # [20, 30]
print(nums[::-1])              # [40, 30, 20, 10]

first, *middle, last = nums
print(first, middle, last)     # 10 [20, 30] 40
a, b = 1, 2
a, b = b, a                    # swap
```
**Important point:** the starred variable (`*middle`) always becomes a **list**, even if it is empty.

**Syntax:**
```
seq[start:stop:step]           # start included, stop NOT included, step optional
seq[:stop]   seq[start:]   seq[::step]   seq[::-1]   seq[:]
first, *middle, last = seq     # * collects the rest into a list
a, b = b, a                    # swap
func(*list_or_tuple)           # unpack into arguments
[*list1, *list2]               # join lists
```
**More slicing examples** (`n = [10, 20, 30, 40, 50]`):
```python
n[1:3]       # [20, 30]
n[:2]        # [10, 20]          from the start
n[3:]        # [40, 50]          to the end
n[-2:]       # [40, 50]          last two
n[:-1]       # [10, 20, 30, 40]  everything except the last
n[::2]       # [10, 30, 50]      every second item
n[1:4:2]     # [20, 40]
n[::-1]      # [50, 40, 30, 20, 10]
n[10:]       # []                out-of-range slices are safe (no error)
n[10]        # IndexError        but a single bad index is an error
cp = n[:]    # a shallow copy
"hello"[1:3] # 'el'              strings slice the same way
```
**More unpacking examples:**
```python
f, *m, l = [10, 20, 30, 40, 50]    # f=10, m=[20, 30, 40], l=50
f, *rest = [10, 20, 30]            # f=10, rest=[20, 30]
*head, l = [10, 20, 30]            # head=[10, 20], l=30
a, _, c = (1, 2, 3)                # a=1, c=3     _ means "I don't need this one"
(a, b), c = (1, 2), 3              # nested unpacking: a=1, b=2, c=3
first, *m, last = [1, 2]           # m=[]         the star can be empty
[*[1, 2], *[3, 4]]                 # [1, 2, 3, 4]  join lists with *
print(*[1, 2, 3])                  # 1 2 3        * spreads items into arguments
```

---

## 4. Comprehensions

### 4.1 Definition
A **comprehension** builds a list, dict or set in one line.

```
[ what_to_store   for item in iterable   if which_to_keep ]
```

### 4.2 ⚠️ Common confusion: the filter is **not** the result
Task: squares of the odd numbers from 1 to 9.
```python
[x for x in range(1, 10) if x % 2 == 1]       # [1, 3, 5, 7, 9]   odd numbers only (wrong task)
[x * x for x in range(1, 10) if x % 2 == 1]   # [1, 9, 25, 49, 81]   squares of odd numbers
```
The `if` decides **which items to keep**. The part **before** `for` decides **what to store**. Change that part to change the result.

### 4.3 More examples
```python
texts = ["  Refund  ", "", "Login issue"]
cleaned = [t.strip().lower() for t in texts if t.strip()]    # ['refund', 'login issue']
lengths = {t: len(t) for t in cleaned}                        # dict comprehension
unique_first = {t[0] for t in cleaned}                        # set comprehension
total = sum(len(t) for t in cleaned)                          # generator expression (no list built)
```
**Important points:**
- A **generator expression** (round brackets) produces items one at a time and saves memory.
- If a comprehension becomes long or nested, use a normal loop instead.

**Syntax:**
```
[expression for item in iterable]                       # list
[expression for item in iterable if condition]          # list, with filter
[a if condition else b for item in iterable]            # list, choose a value
{key_expr: value_expr for item in iterable}             # dict
{expression for item in iterable}                       # set
(expression for item in iterable)                       # generator (lazy, one use)
[expression for outer in outers for inner in outer]     # nested: outer loop first
```
**Five more comprehension shapes:**
```python
[x * 2 for x in range(4)]                         # [0, 2, 4, 6]
[x for x in range(10) if x % 3 == 0]              # [0, 3, 6, 9]
["even" if x % 2 == 0 else "odd" for x in range(3)]   # ['even', 'odd', 'even']
[(i, j) for i in range(2) for j in range(2)]      # [(0, 0), (0, 1), (1, 0), (1, 1)]
[x for row in [[1, 2], [3, 4]] for x in row]      # [1, 2, 3, 4]   flatten a list of lists
```
⚠️ Note the two different `if` positions. `... for x in items if cond` **filters** (drops items). `value_a if cond else value_b for x in items` **chooses** a value for every item.

Dict and set comprehensions:
```python
{w: len(w) for w in ["a", "bb"]}     # {'a': 1, 'bb': 2}
{x % 3 for x in range(10)}           # {0, 1, 2}
```
A generator is used **once**:
```python
g = (x * x for x in range(3))
list(g)       # [0, 1, 4]
list(g)       # []        already used up
sum(x for x in range(5))        # 10    no list built in memory
any(x > 3 for x in range(5))    # True  stops as soon as it finds one
```

---

## 5. enumerate and zip

### 5.1 Definition
- `enumerate(items, start=0)` gives `(index, item)` pairs.
- `zip(a, b)` walks two (or more) sequences together.

### 5.2 Example
```python
ids = [101, 102, 103]
scores = [0.91, 0.75]

for i, tid in enumerate(ids, start=1):
    print(i, tid)                          # 1 101 / 2 102 / 3 103

print(list(zip(ids, scores)))              # [(101, 0.91), (102, 0.75)]
```

### 5.3 ⚠️ Common confusion: `zip` silently drops items
`ids` has 3 items and `scores` has 2, but there is **no error**. The third id (`103`) simply disappears. In a data pipeline this loses data quietly.
```python
list(zip(ids, scores, strict=True))        # ValueError: lengths differ (Python 3.10+)

from itertools import zip_longest
list(zip_longest(ids, scores, fillvalue=0))   # [(101, 0.91), (102, 0.75), (103, 0)]
```
🎯 **Interview:** `zip` stops at the shortest input. Use `strict=True` to catch mismatches, or `zip_longest` to keep everything.

**Syntax:**
```
for index, item in enumerate(iterable, start=0):
for a, b in zip(list1, list2):                zip(list1, list2, strict=True)
dict(zip(keys, values))                       a_tuple, b_tuple = zip(*list_of_pairs)
from itertools import zip_longest
zip_longest(list1, list2, fillvalue=0)
```
**More enumerate and zip examples:**
```python
ids = [101, 102, 103]
names = ["a", "b", "c"]

list(enumerate(ids))              # [(0, 101), (1, 102), (2, 103)]
list(enumerate(ids, start=1))     # [(1, 101), (2, 102), (3, 103)]

list(zip(ids, names))             # [(101, 'a'), (102, 'b'), (103, 'c')]
dict(zip(ids, names))             # {101: 'a', 102: 'b', 103: 'c'}   id -> name lookup

for tid, nm in zip(ids, names):
    print(tid, nm)                # 101 a / 102 b / 103 c

list(zip([1, 2], [3, 4], [5, 6])) # [(1, 3, 5), (2, 4, 6)]   zip takes 3 or more sequences

pairs = [(1, "a"), (2, "b")]
nums, letters = zip(*pairs)       # "unzip": nums = (1, 2), letters = ('a', 'b')
```

---

## 6. The `collections` module

It is part of Python's standard library. It has special containers that save you code.

### 6.1 defaultdict: a dict that never raises `KeyError`
**Definition:** you give it a **factory** (a function with no arguments). When you read a missing key, it calls the factory, **stores** the result under that key, and returns it.

| Factory | A missing key starts as | Use |
|---|---|---|
| `int` | `0` | counting, summing |
| `list` | `[]` | grouping items by key |
| `set` | `set()` | grouping unique items |

```python
from collections import defaultdict

tickets = [(101, "billing"), (102, "login"), (103, "billing")]
by_category = defaultdict(list)
for ticket_id, category in tickets:
    by_category[category].append(ticket_id)

print(dict(by_category))        # {'billing': [101, 103], 'login': [102]}
```
**Important points:**
- Write `defaultdict(list)`, not `defaultdict([])`. The argument is a function, not a value.
- **Just reading a missing key creates it.** Use `in` or `.get()` if you don't want that.
- Without it you would write `if category not in d: d[category] = []` every time.

### 6.2 Counter: count things
**Definition:** a dict built for counting. Keys are items, values are counts.
```python
from collections import Counter

categories = ["billing", "login", "billing", "shipping", "login", "billing"]
c = Counter(categories)
print(c)                      # Counter({'billing': 3, 'login': 2, 'shipping': 1})
print(c.most_common(2))       # [('billing', 3), ('login', 2)]
print(c["refund"])            # 0   (a missing key gives 0 and is NOT inserted)
```
| Counter feature | What it does |
|---|---|
| `Counter(iterable)` | Counts everything in one pass. |
| `c.most_common(n)` | The `n` most frequent items, highest first. |
| `c.update(iterable)` | **Adds** counts (a normal dict's `update` overwrites). |

🎯 **Interview:** `Counter` vs `defaultdict(int)`. Both count. `Counter` is built for counting (`most_common`, returns 0 for missing keys). `defaultdict(int)` is more general, for example summing amounts per key.

### 6.3 deque: fast at both ends
```python
from collections import deque

recent = deque(maxlen=3)        # keeps only the last 3 items
for tid in [1, 2, 3, 4, 5]:
    recent.append(tid)
print(recent)                    # deque([3, 4, 5], maxlen=3)
recent.popleft()                 # remove from the left in O(1) (a list's pop(0) is O(n))
```
**Use for:** queues and "last N items" windows. Indexing in the middle is slow (O(n)).

### 6.4 namedtuple: a tuple with names
```python
from collections import namedtuple

Ticket = namedtuple("Ticket", ["id", "category"])
t = Ticket(101, "billing")
print(t.category, t[0])          # billing 101
```
Use it for small read-only records. For records with defaults, hints and validation, use a `dataclass` (Topic 08).

### 6.5 More examples for each container
**Syntax:**
```
from collections import defaultdict, Counter, deque, namedtuple

defaultdict(factory)                  # factory = int, list, set (a function, no brackets)
Counter(iterable)                     c.most_common(n)       c.update(iterable)
deque(iterable, maxlen=None)          q.append(x)  q.appendleft(x)  q.pop()  q.popleft()  q.rotate(n)
Name = namedtuple("Name", ["field1", "field2"])      t = Name(value1, value2)      t.field1
```
**defaultdict**
```python
from collections import defaultdict

counts = defaultdict(int)
for w in "a b a".split():
    counts[w] += 1
dict(counts)                 # {'a': 2, 'b': 1}      counting

groups = defaultdict(set)
groups["x"].add(1)
groups["x"].add(1)
dict(groups)                 # {'x': {1}}            a set ignores the repeat

dl = defaultdict(list)
dl["miss"]                   # []    just reading a missing key CREATES it
dict(dl)                     # {'miss': []}
```
**Counter**
```python
from collections import Counter

c = Counter("banana")        # Counter({'a': 3, 'n': 2, 'b': 1})
c["a"]                       # 3
c["z"]                       # 0        missing key gives 0, and is not inserted
c.most_common(1)             # [('a', 3)]
c.update("aa")               # adds counts: now a = 5
Counter(a=3, b=1) + Counter(a=1)    # Counter({'a': 4, 'b': 1})
Counter(a=3, b=1) - Counter(a=1)    # Counter({'a': 2, 'b': 1})
sum(Counter(a=3, b=1).values())     # 4      total count
```
**deque**
```python
from collections import deque

q = deque([1, 2, 3])
q.append(4)                  # deque([1, 2, 3, 4])      add right
q.appendleft(0)              # deque([0, 1, 2, 3, 4])   add left
q.pop()                      # 4                        remove right
q.popleft()                  # 0                        remove left
q.rotate(1)                  # deque([3, 1, 2])         move items one step right
q.extend([7, 8])             # deque([3, 1, 2, 7, 8])
len(q)                       # 5
```
**namedtuple**
```python
from collections import namedtuple

T = namedtuple("T", ["id", "cat"])
t = T(1, "billing")          # T(id=1, cat='billing')
t.id, t[1]                   # (1, 'billing')       by name or by position
t._asdict()                  # {'id': 1, 'cat': 'billing'}
t._replace(cat="login")      # T(id=1, cat='login')   a NEW tuple (the old one is unchanged)
T._fields                    # ('id', 'cat')
idd, cat = t                 # unpacking works like a normal tuple
t.id = 5                     # AttributeError: can't set attribute   (it is read-only)
```

---

## 7. Real use: top-K results (sorting with `key=`)
```python
results = [{"id": 1, "score": 0.41}, {"id": 2, "score": 0.93}, {"id": 3, "score": 0.77}]
top2 = sorted(results, key=lambda r: r["score"], reverse=True)[:2]
print([r["id"] for r in top2])   # [2, 3]
```
`key` is a function called **once per item** to get the value to sort by. This is exactly how a search engine returns the top-K chunks.

---

## 8. Gotchas
- Don't modify a list or dict **while looping over it**. Loop over a copy: `for x in list(items):`.
- `d[key]` on a missing key raises `KeyError`. Use `.get()` or `defaultdict`.
- A set has **no order**. Don't rely on it.
- `nums = nums.sort()` sets `nums` to `None`.

---

## 9. Patterns to remember

Sample data used below (support tickets):
```python
tickets = [
    {"id": 101, "category": "billing",  "text": "Refund not received",   "score": 0.91},
    {"id": 102, "category": "login",    "text": "Cannot reset password", "score": 0.75},
    {"id": 103, "category": "billing",  "text": "Charged twice",         "score": 0.88},
    {"id": 101, "category": "billing",  "text": "Refund not received",   "score": 0.91},
    {"id": 104, "category": "shipping", "text": "Late delivery",         "score": 0.40},
]
```
| Pattern | Template | Result on the tickets |
|---|---|---|
| **Count** | `Counter(x["key"] for x in items)` | `Counter({'billing': 3, 'login': 1, 'shipping': 1})` |
| **Group** | `d = defaultdict(list)` then `d[x["key"]].append(x["id"])` | `{'billing': [101, 103, 101], 'login': [102], 'shipping': [104]}` |
| **Remove duplicates, keep order** | `list(dict.fromkeys(items))` | `[101, 102, 103, 104]` (from the ids) |
| **Lookup by id** | `{x["id"]: x for x in items}` | `idx[103]["text"]` gives `'Charged twice'` |
| **Top-K** | `sorted(items, key=lambda x: x["score"], reverse=True)[:k]` | ids `[101, 101, 103]` |
| **Find duplicates** | keep a `seen = set()`; if `x in seen`, it is a duplicate | `[101]` |
| **Batch (chunk a list)** | `[items[i:i+n] for i in range(0, len(items), n)]` | `[[0,1,2],[3,4,5],[6]]` for `range(7)`, n=3 |
| **Sliding window** | `w = deque(maxlen=3)` then `w.append(x)` | keeps only the last 3 values |

```python
from collections import Counter, defaultdict, deque

Counter(t["category"] for t in tickets)           # Counter({'billing': 3, 'login': 1, 'shipping': 1})

by = defaultdict(list)
for t in tickets:
    by[t["category"]].append(t["id"])
dict(by)                                          # {'billing': [101, 103, 101], 'login': [102], 'shipping': [104]}

list(dict.fromkeys(t["id"] for t in tickets))     # [101, 102, 103, 104]

idx = {t["id"]: t for t in tickets}
idx[103]["text"]                                  # 'Charged twice'

seen, dups = set(), []
for t in tickets:
    if t["id"] in seen:
        dups.append(t["id"])
    seen.add(t["id"])
dups                                              # [101]

items = list(range(7))
[items[i:i + 3] for i in range(0, len(items), 3)] # [[0, 1, 2], [3, 4, 5], [6]]

w = deque(maxlen=3)
for x in [1, 2, 3, 4]:
    w.append(x)
list(w), sum(w) / len(w)                          # ([2, 3, 4], 3.0)
```
`dict.fromkeys(items)` works as "remove duplicates and keep order" because dict keys are unique and a dict remembers insertion order. (`set(items)` removes duplicates but loses the order.)

**Choosing the container (decision list):**
1. Need order and changes? **list**.
2. Fixed record, or a dict key? **tuple**.
3. Look up by a key? **dict**.
4. Need "unique" or fast "is it in here?" **set**.
5. Counting? **Counter**. Grouping? **defaultdict(list)**. Queue or last-N? **deque**.

---

## 10. Use this when building (project guide)

- **Ticket and chunk data** is almost always a **list of dicts**. The patterns above (count, group, dedupe, lookup, top-K, batch) cover most cleaning and analysis steps.
- **RAG retrieval:** the result list from a vector search is `[{"id":..., "score":...}, ...]`. Top-K is `sorted(..., key=..., reverse=True)[:k]`. A `{id: chunk}` lookup dict turns ids back into text instantly.
- **Embedding in batches:** APIs take limited input per call. Use the **batch** pattern to send 100 texts at a time.
- **Dedupe before embedding** saves cost: embed each unique text once.
- **Pandas later:** `df["category"].value_counts()` is `Counter`, `df.groupby("category")` is `defaultdict(list)`. You are learning the same ideas first.

**Checklist:** choose the container by the job (decision list). Never modify a collection while looping over it. Convert a list to a set before repeated `in` checks. Use `strict=True` in `zip` when the lengths must match.

---

## 🧠 Quick Revision

- list = ordered, changeable. tuple = fixed record. dict = key→value, O(1). set = unique items, O(1).
- Dict keys and set items must be **hashable** (immutable). A list can't be a key.
- `x in set` is O(1); `x in list` is O(n). Convert to a set for repeated checks.
- `append` adds one item, `extend` adds many. `sort()` returns `None`, `sorted()` returns a new list.
- Comprehension: `[what_to_store for item in iterable if which_to_keep]`.
- `zip` drops extras silently. Use `strict=True` or `zip_longest`.
- `*middle` in unpacking is always a list.
- `defaultdict(list)` to group, `defaultdict(int)` to sum, `Counter` to count (and `most_common`).
- `deque` for queues and last-N windows.
- `any(...)` = at least one true, `all(...)` = every one true. `{**a, **b}` or `a | b` merges dicts. `set()` is the empty set (`{}` is a dict).
- `sorted(items, key=..., reverse=True)[:k]` gives top-K.