# 01 — Objects, Types and Mutability

> Phase 0 · Python foundation. This is the idea behind most "weird" Python bugs, in Pandas, NumPy, FastAPI and RAG code too.
> Running example: **support tickets** (the kind of data a customer-support RAG project cleans, chunks and embeds).
> `🎯 Interview:` marks a point worth remembering for interviews. `⚠️ Common confusion:` marks a place where beginners (including me) got stuck, with the reason explained.

---

## 1. Names and objects

### 1.1 Definition
In Python **everything is an object**. A variable is **not a box that holds a value**. It is a **name (label) that points to an object**.

### 1.2 Important points
- Every object has three things: **identity** (where it lives, `id()`), **type** (`type()`), **value** (its content).
- `b = a` does **not copy**. It sticks a second label on the **same** object.
- `==` asks "is the **value** equal?". `is` asks "is it the **same object**?".
- Python counts how many names point to an object (**reference count**). At 0, the object is freed.

### 1.3 Example
```python
a = [101, 102, 103]        # ticket ids
b = a                      # second label, same list
b.append(104)
print(a)                   # what do you expect?
print(a is b)
```
**Result:**
```
[101, 102, 103, 104]
True
```
`a` changed because `a` and `b` are two names for **one** list.

```python
x = [1, 2]
y = [1, 2]
print(x == y, x is y)      # True False   (same value, different objects)
```

🎯 **Interview:** `is` vs `==`. Use `is` only for `None` (`if x is None`). Never use `is` to compare numbers or strings (small ints and some strings are cached, so it can mislead).

### 1.4 The tools: `type()`, `id()`, `isinstance()`, `is`, `==`, `del`
**Syntax:**
```
type(object)                         # the type of an object
id(object)                           # the identity number of an object
isinstance(object, type)             # True/False. Also: isinstance(object, (type1, type2))
a is b       /   a is not b          # same object? / not the same object?
a == b       /   a != b             # same value? / different value?
del name                             # remove a name
del collection[index_or_key]         # remove one item from a list or dict
```
Each one with an example:
```python
a = [1, 2]
b = a              # same object
c = [1, 2]         # equal value, different object
```
```python
type(a)                  # <class 'list'>      what kind of object?
type(5)                  # <class 'int'>
type(None)               # <class 'NoneType'>
```
```python
id(a) == id(b)           # True    same identity (same place in memory)
id(a) == id(c)           # False
```
```python
isinstance(a, list)            # True
isinstance(5, (int, float))    # True    a tuple means "any of these"
isinstance(True, int)          # True    bool is a kind of int
```
```python
a == c                   # True    same value
a is c                   # False   different objects
a is b                   # True
a is not c               # True
```
```python
x = None
x is None                # True    the correct way to check for None
```
```python
nums = [1, 2, 3]
del nums[0]              # remove an item
print(nums)              # [2, 3]

name = "x"
del name                 # remove the NAME (the label), not necessarily the object
print(name)              # NameError: name 'name' is not defined
```
**Important points:**
- `type(x)` tells the exact type. `isinstance(x, T)` is better for checks because it also accepts subclasses.
- `del name` only removes the label. The object is freed when **no label** points to it.

---

## 2. Mutable vs immutable

### 2.1 Definition
- **Mutable** = can be changed **in place** (the same object changes).
- **Immutable** = cannot be changed in place. Any "change" creates a **new** object.

| Mutable | Immutable |
|---|---|
| `list`, `dict`, `set` and your own class objects can be changed in place, so every name pointing to them sees the change. | `int`, `float`, `bool`, `str`, `tuple`, `frozenset` and `bytes` cannot be changed in place, so a "change" makes a new object and other names keep the old one. |

### 2.2 Example
```python
s = "refund"
s2 = s
s += " pending"            # makes a NEW string
print(s2)                  # refund   (unchanged)

l = [1]
l2 = l
l += [2]                   # changes the SAME list
print(l2)                  # [1, 2]
```
**Result:** `s2` stays `refund`, but `l2` becomes `[1, 2]`.

### 2.3 Important points
- Only **immutable** (hashable) objects can be dict keys or set members (see Topic 02).
- Strings are immutable, so `+=` in a loop builds a new string every time. For many pieces use `"".join(parts)`. It is much faster.

### 2.4 One example for every type (does the change reach the other name?)
```python
# immutable: int  (a "change" makes a new object)
n = 5
m = n
n += 1
print(n, m)              # 6 5      m kept the old object
```
```python
# immutable: str
s = "ab"
s2 = s
s += "c"
print(s, s2)             # abc ab
```
```python
# immutable: tuple
t = (1, 2)
t2 = t
t += (3,)                # builds a NEW tuple
print(t, t2)             # (1, 2, 3) (1, 2)
```
```python
# immutable: frozenset (a set that cannot change)
fs = frozenset([1, 2])
fs.add(3)                # AttributeError: 'frozenset' object has no attribute 'add'
{fs: "ok"}               # works: it is hashable, so it can be a dict key
```
```python
# mutable: dict
d = {"a": 1}
d2 = d
d["b"] = 2
print(d2)                # {'a': 1, 'b': 2}    same dict
```
```python
# mutable: set
st = {1}
st2 = st
st.add(2)
print(st2)               # {1, 2}
```
**Memory trick:** a "change" on an immutable type makes a **new** object, so the other name keeps the old one. A change on a mutable type changes the **one shared** object.

---

## 3. Alias, shallow copy, deep copy

### 3.1 Definition
- **Alias:** `b = a`. No copy at all.
- **Shallow copy:** a new outer container, but the **inner objects are shared**. (`a.copy()`, `a[:]`, `list(a)`, `dict(a)`)
- **Deep copy:** everything is copied, fully independent. (`copy.deepcopy(a)`)

### 3.2 Example (tickets with tags)
```python
import copy

tickets = [{"id": 1, "tags": ["billing"]}, {"id": 2, "tags": []}]
shallow = tickets.copy()
deep = copy.deepcopy(tickets)

tickets[0]["tags"].append("urgent")
print(shallow[0]["tags"])
print(deep[0]["tags"])
```
**Result:**
```
['billing', 'urgent']      <- shallow copy still shares the inner dict
['billing']                <- deep copy is independent
```
🎯 **Interview:** shallow vs deep copy. A shallow copy of a nested structure still shares the inner objects.

### 3.4 Every way to copy, with a test
**Syntax:**
```
new = old.copy()               # shallow (list, dict, set)
new = old[:]                   # shallow (list)
new = list(old)  /  dict(old)  # shallow
import copy
new = copy.copy(old)           # shallow
new = copy.deepcopy(old)       # deep
```
```python
import copy
q = [1, [2, 3]]

cp1 = q.copy()               # shallow
cp2 = q[:]                   # shallow (slice)
cp3 = list(q)                # shallow
cp4 = copy.copy(q)           # shallow
cp5 = copy.deepcopy(q)       # deep

q[1].append(9)               # change the INNER list
q.append(7)                  # change the OUTER list

print(cp1)    # [1, [2, 3, 9]]   inner changed (shared), 7 not added (outer is separate)
print(cp2)    # [1, [2, 3, 9]]
print(cp3)    # [1, [2, 3, 9]]
print(cp4)    # [1, [2, 3, 9]]
print(cp5)    # [1, [2, 3]]      deep copy: nothing changed
```
Same for dicts:
```python
d = {"tags": ["a"]}
d3 = dict(d)                 # shallow
d4 = d.copy()                # shallow
d["tags"].append("z")
print(d3, d4)                # {'tags': ['a', 'z']} {'tags': ['a', 'z']}
```
**Rule:** `.copy()`, `[:]`, `list()`, `dict()` and `copy.copy()` are all **shallow**. Only `copy.deepcopy()` is fully independent. A shallow copy is enough when the inner items are immutable (numbers, strings).

### 3.3 ⚠️ Common confusion: the grid trap `[[0] * 3] * 3`
```python
grid = [[0] * 3] * 3
grid[0][0] = 9
print(grid)
print(grid[0] is grid[1])
```
**Result:**
```
[[9, 0, 0], [9, 0, 0], [9, 0, 0]]     <- ALL rows changed, not one
True
```
**Why:** `[0] * 3` creates **one** inner list. The outer `* 3` repeats the **reference** to it three times, so all three rows are the same list:
```
grid ──> [ ●,  ●,  ● ]
           │   │   │
           └───┴───┴──> [9, 0, 0]     (one shared list)
```
**Fix:** make three separate lists:
```python
grid = [[0] * 3 for _ in range(3)]
```
The comprehension runs `[0] * 3` three separate times. (The inner `[0] * 3` is fine because ints are immutable.)

---

## 4. Mutable default argument

### 4.1 Definition
**Syntax (the safe pattern):**
```
def function_name(param=None):
    if param is None:
        param = []            # or {} or set(): a NEW object on every call
    ...
    return param
```
A default value in `def f(x, bucket=[])` is created **once, when the function is defined**, not on every call. So a mutable default is **shared by all calls**.

### 4.2 Example (a real bug in a ticket cleaner)
```python
def add_tag(tag, tags=[]):         # BUG
    tags.append(tag)
    return tags

print(add_tag("billing"))
print(add_tag("urgent"))
```
**Result:**
```
['billing']
['billing', 'urgent']      <- tags leaked from the first ticket into the second
```

### 4.3 The fix, step by step
```python
def add_tag(tag, tags=None):
    if tags is None:           # nothing was passed in
        tags = []              # make a NEW list on every call
    tags.append(tag)
    return tags

print(add_tag("billing"))      # ['billing']
print(add_tag("urgent"))       # ['urgent']
```
**⚠️ Common confusion:** just writing `tags=None` is **not** enough. If you forget the `if tags is None: tags = []` part, `None.append(...)` fails with `AttributeError: 'NoneType' object has no attribute 'append'`. `None` only means "the caller passed nothing". **You** must create the new list inside the function.

🎯 **Interview:** "What is the mutable default argument problem?" The default is evaluated once at definition time. Fix: default to `None` and create the object inside. This is the one place where `is None` is the right check.

---

## 5. A tuple that holds a list

### 5.1 Definition
A tuple's immutability means it cannot change **which objects it points to**. It says nothing about whether those objects can change **themselves**.

### 5.2 Example
```python
t = (1, [2, 3])
t[1].append(4)             # works: the tuple still points to the same list
print(t)                   # (1, [2, 3, 4])
```

### 5.3 ⚠️ Common confusion: `t[1] += [5]` raises an error **and** changes the list
```python
t = (1, [2, 3])
try:
    t[1] += [5]
except TypeError as e:
    print("Error:", e)
print(t)
```
**Result:**
```
Error: 'tuple' object does not support item assignment
(1, [2, 3, 5])
```
**Why:** `t[1] += [5]` happens in **two steps**:
1. `t[1].__iadd__([5])` changes the **list in place**. This works, because lists are mutable.
2. `t[1] = <that list>` tries to **assign back** into the tuple. This fails, because tuples don't allow item assignment.

Step 1 already happened before step 2 failed.

🎯 **Interview:** "Why does `t[1] += [5]` raise an error but still change the list?" Answer: in-place add first, then assignment back into the tuple fails.

---

## 6. Function arguments: pass by object reference

### 6.1 Definition
A function parameter becomes **another label on the same object** you passed in.

### 6.2 Example
```python
def add_one(lst):
    lst.append(1)          # mutates the caller's list

def rebind(lst):
    lst = [99]             # only moves the local label

data = [0]
add_one(data); print(data)     # [0, 1]
rebind(data);  print(data)     # [0, 1]   unchanged
```
**Rule:** **mutating** an argument changes the caller's object. **Re-assigning** the parameter does not.

---

## 7. Where this shows up in AI / data projects

- **RAG chunk metadata:** one shared dict gets reused by every chunk.
  ```python
  meta = {"source": "faq.pdf"}
  chunks = [{"text": t, "meta": meta} for t in ["a", "b"]]      # BUG: all chunks share ONE dict
  chunks = [{"text": t, "meta": dict(meta)} for t in ["a", "b"]]  # FIX: each gets its own copy
  ```
- **NumPy / Pandas:** slicing can give a **view** (shared memory) instead of a copy, so changing it can change the original. Use `.copy()` when you need independence.
- **FastAPI:** a mutable default or a shared global list can leak one user's data into another user's request.

---

## 8. Patterns to remember

**Pattern 1: "Is this shared?" check.** Before you reuse a list or dict in more than one place, ask: *will two places change it?* If yes, make a copy.

| Situation | Use |
|---|---|
| Flat list or dict (items are numbers, strings) | `new = old.copy()` |
| Nested (a list of dicts, a dict holding lists) | `new = copy.deepcopy(old)` |
| You want the same object on purpose | `b = a` (and you know it) |

**Pattern 2: safe default.**
```
def f(x, items=None):
    if items is None:
        items = []
```
**Pattern 3: build text with `join`.**
```
parts = []
for ...: parts.append(piece)
text = "".join(parts)
```
**Pattern 4: debug a "my data changed by itself" bug.** Print `a is b` or `id(a), id(b)`. If they are the same, you have one object with two names.

---

## 9. Use this when building (project guide)

**RAG chunk builder:** every chunk needs its own metadata dict. If all chunks share one dict, setting `page` on one chunk changes all of them.
```python
meta = {"source": "faq.pdf"}

def make_chunks(texts, meta):
    return [{"text": t, "meta": dict(meta)} for t in texts]   # dict(meta) = a fresh copy per chunk

chunks = make_chunks(["a", "b"], meta)
chunks[0]["meta"]["page"] = 1
print(chunks)
# [{'text': 'a', 'meta': {'source': 'faq.pdf', 'page': 1}}, {'text': 'b', 'meta': {'source': 'faq.pdf'}}]
print(chunks[0]["meta"] is chunks[1]["meta"])    # False   they are separate dicts
```
**Checklist when you write a function or loop:**
- Does it **change its argument**? If yes, say so in the docstring, or work on a copy.
- Does any parameter have `=[]` or `={}`? Replace it with `None`.
- Am I copying a **nested** structure? Use `deepcopy`.
- Am I comparing with `is`? Only for `None`.
- Building a big string? Use `join`.

---

## 🧠 Quick Revision

- Variables are **names pointing to objects**. `b = a` never copies.
- `==` compares value, `is` compares identity. Use `is` only for `None`.
- Mutable: list, dict, set, objects. Immutable: int, float, str, tuple, bool, frozenset.
- Copies: alias (none), shallow (`.copy()`, inner objects shared), deep (`copy.deepcopy`, fully separate).
- `[[0]*3]*3` makes 3 references to **one** row. Use `[[0]*3 for _ in range(3)]`.
- Mutable default arguments are created once and shared. Use `None`, then create the object **inside** the function.
- A tuple can't change which objects it holds, but a mutable object inside it can still change. `t[1] += [...]` errors **after** changing the list.
- Mutating an argument affects the caller; re-assigning the parameter does not.
- Build strings from many pieces with `"".join(parts)`, not repeated `+=`.