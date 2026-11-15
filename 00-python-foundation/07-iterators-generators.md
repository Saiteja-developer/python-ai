# 07 — Iterators and Generators

> Phase 0 · Python foundation. Embeddings of a million chunks do not fit in a list in memory, and an LLM sends its answer **piece by piece** (streaming). Both use iterators and generators. This file also explains what a `for` loop really does, which makes every later topic (files, Pandas, FastAPI streaming) easier.
> Running example: **support tickets, log lines, chunks and embedding batches**.
> `🎯 Interview:` marks a point worth remembering for interviews. `⚠️ Common confusion:` marks a place where people get stuck, with the reason explained.
> Each section follows: Definition → Important points → Syntax → Examples (output in comments). All code in this file was run (Python 3.13); `itertools.batched` needs Python 3.12 or newer, and you have 3.15.
> The assignment with full answers is at the very end.

Java word map (you know Java): `Iterable<T>` with `iterator()` is `__iter__`. `Iterator<T>` with `hasNext()` / `next()` is `__next__`, which raises `StopIteration` instead of returning `false` from `hasNext()`. The enhanced `for (x : list)` is `for x in list`. A generator is like a lazy `Stream<T>` that you write as an ordinary function.

---

## 1. Iterable and iterator

### 1.1 Definition
| Term | Meaning |
|---|---|
| **Iterable** | Anything you can loop over with `for`. It has the method `__iter__`. Examples: list, tuple, str, dict, set, file, `range`. |
| **Iterator** | An object that hands out items one at a time, remembers its position, and can be used only once. It has `__next__` (and `__iter__` that returns itself). |

`iter(iterable)` gives you an iterator. `next(iterator)` gives the next item. When nothing is left, Python raises `StopIteration`.

### 1.2 Syntax
```
it = iter(iterable)          # get an iterator
next(it)                     # next item (raises StopIteration when finished)
next(it, default)            # next item, or `default` instead of the error
```

### 1.3 Example
```python
nums = [10, 20, 30]
it = iter(nums)
type(it)          # <class 'list_iterator'>
next(it)          # 10
next(it)          # 20
next(it)          # 30
next(it)          # StopIteration        nothing left

next(iter([]), "empty")     # 'empty'    the default avoids the error
it = iter(nums)
next(it, "done")            # 10
```

---

## 2. What a `for` loop really does

### 2.1 Definition
`for x in items:` means: get an iterator with `iter(items)`, then call `next` again and again until `StopIteration`, and stop quietly.
```python
items = ["a", "b", "c"]

for x in items:                  # the normal way
    print(x, end=" ")            # a b c

it = iter(items)                 # the same thing written out
while True:
    try:
        x = next(it)
    except StopIteration:        # this is how the loop knows it is finished
        break
    print(x, end=" ")            # a b c
```
### 2.2 Important points
- Every `for`, comprehension, `list(...)`, `sum(...)`, `max(...)`, `in` and unpacking uses this same protocol.
- `StopIteration` is not an error you see in a `for` loop. It is the **signal** that the loop is finished.
- Java comparison: it is the same as `while (it.hasNext()) { x = it.next(); ... }`, except Python signals the end with an exception.
🎯 **Interview:** explain how a `for` loop works internally (`iter()`, `next()`, `StopIteration`).

---

## 3. The one-use rule: iterable vs iterator

### 3.1 Definition and important points
A list is an **iterable**: every `iter(lst)` creates a **fresh** iterator, so you can loop again and again. An **iterator** has **one position that only moves forward**, so after one pass it is used up.
```python
lst = [1, 2, 3]
list(lst), list(lst)             # ([1, 2, 3], [1, 2, 3])    loop many times

it = iter([1, 2, 3])
list(it), list(it)               # ([1, 2, 3], [])           second time: nothing left
```
⚠️ **Common confusion: "why is my data gone?"** The second `list(it)` is empty because the first one already read every item. Nobody "deleted" anything: the position is at the end. To use the data again, keep the **list**, or create a new iterator.

### 3.2 How to tell them apart
```python
from collections.abc import Iterable, Iterator
isinstance([1], Iterable)         # True     a list is iterable
isinstance([1], Iterator)         # False    but a list is not an iterator
isinstance(iter([1]), Iterator)   # True
isinstance(5, Iterable)           # False    an int cannot be looped
hasattr([1], "__iter__")          # True
hasattr([1], "__next__")          # False    only iterators have __next__
iter(it) is it                    # True     iter() on an iterator returns itself
```
⚠️ **Common confusion: `range` is iterable but NOT an iterator.**
```python
r = range(3)
next(r)               # TypeError: 'range' object is not an iterator
next(iter(r))         # 0
list(r), list(r)      # ([0, 1, 2], [0, 1, 2])    a range can be looped many times
```

---

## 4. Built-in things that are lazy
`map`, `filter`, `zip`, `enumerate`, `reversed` and open files give items **on demand** (they are iterators), so you can read them **once**. This is why `list(map(...))` is needed (Topic 03).
```python
type(map(str, [1])).__name__         # 'map'
type(zip([1], [2])).__name__         # 'zip'
type(enumerate([1])).__name__        # 'enumerate'
type(reversed([1])).__name__         # 'list_reverseiterator'
type(filter(None, [1])).__name__     # 'filter'
```
What different iterables give you when looped:
```python
d = {"a": 1, "b": 2}
next(iter(d))              # 'a'                  a dict gives its KEYS
list(d.items())            # [('a', 1), ('b', 2)]
s = iter("hi")
next(s), next(s)           # ('h', 'i')           a string gives characters
```

---

## 5. What consumes an iterator
Anything that reads items **moves the position forward**. Remember these five:
```python
it = iter([1, 2, 3])
sum(it), sum(it)             # (6, 0)       the second sum has nothing left

it = iter([1, 2, 3, 4])
next(it)                     # 1
list(it)                     # [2, 3, 4]    continues from where it stopped

it = iter([1, 2, 3])
2 in it                      # True         `in` reads items until it finds 2
list(it)                     # [3]          2 and everything before it are gone

a, b = iter([1, 2])          # unpacking also consumes the iterator
a, b                         # (1, 2)
```
Changing a dict while looping over it fails:
```python
d = {"a": 1}
for k in d:
    d["b"] = 2               # RuntimeError: dictionary changed size during iteration
```
Rule: **never change the size of a collection while looping over it.** Loop over a copy (`list(d)`) or build a new one.

---

## 6. Your own iterator and iterable classes (uses Topic 06)

### 6.1 Syntax
```
class Name:
    def __iter__(self):
        return self                    # an iterator returns itself
    def __next__(self):
        if no more items:
            raise StopIteration
        ...return the next item...
```
### 6.2 Example: an iterator class (one use)
```python
class Countdown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.current <= 0:
            raise StopIteration       # tells the for loop: "no more"
        value = self.current
        self.current -= 1
        return value

for n in Countdown(3):
    print(n, end=" ")                 # 3 2 1

c = Countdown(3)
list(c), list(c)                      # ([3, 2, 1], [])    iterator: one use
```
### 6.3 Example: an iterable class (reusable)
Let `__iter__` return a **fresh** iterator each time (Java: `Iterable.iterator()`).
```python
class Tickets:
    def __init__(self, items):
        self.items = items

    def __iter__(self):
        return iter(self.items)        # a new iterator every time

t = Tickets(["t1", "t2"])
list(t), list(t)                       # (['t1', 't2'], ['t1', 't2'])    loopable many times
```
**Rule:** an **iterator class** has `__iter__` (returns `self`) and `__next__` (one use). An **iterable class** has only `__iter__`, returning a new iterator (reusable). Writing these by hand is rare, because generators (section 8) do the same in a few lines.

---

## 7. Lazy vs eager, and memory

### 7.1 Definition
- **Eager:** make everything now and keep it in memory (a list).
- **Lazy:** make each item only when asked (an iterator or generator), so memory stays small.
```python
import sys
sys.getsizeof(range(10**9))             # 48      a range of a billion numbers is tiny
sys.getsizeof(list(range(1000)))        # 8056    a list of only 1,000 numbers is already 8 KB
```
(The exact bytes depend on your Python version and computer; the idea matters.) A list of 10**9 numbers would need many gigabytes.
**AI example:** `[embed(c) for c in chunks]` builds all vectors in memory first. A lazy loop embeds and saves a few at a time.
🎯 **Interview:** lazy vs eager evaluation, and why generators save memory.

---

## 8. Generator functions: `yield`

### 8.1 Definition
A **generator function** is a function that uses `yield` instead of `return`. Calling it does **not** run the body. It gives you a **generator object** (an iterator). Each `next()` runs the body until the next `yield`, hands out that value, and **pauses**, remembering all its local variables.

### 8.2 Syntax
```
def name(parameters):
    ...
    yield value          # hand out a value and pause here
    ...

gen = name(args)         # nothing runs yet
next(gen)                # runs until the first yield
for x in gen: ...        # runs it step by step
```

### 8.3 Example
```python
def count_up(limit):
    n = 1
    while n <= limit:
        yield n              # give n out, then pause here
        n += 1               # resumes here on the next call

g = count_up(3)
type(g)                      # <class 'generator'>
next(g)                      # 1
next(g)                      # 2
next(g)                      # 3
next(g)                      # StopIteration       the function body finished

list(count_up(3))            # [1, 2, 3]
for x in count_up(3):
    print(x, end=" ")        # 1 2 3
```
Compare with Section 6.2: the same Countdown/CountUp job took a whole class there, and here it is a function with `yield`. Python writes `__iter__`, `__next__` and `StopIteration` for you.

### 8.4 See the pause: a trace
```python
def demo():
    print("start")
    yield 1
    print("middle")
    yield 2
    print("end")

g = demo()          # prints nothing: the body has NOT started
print("created")    # created
next(g)             # prints: start        and returns 1
next(g)             # prints: middle       and returns 2
next(g)             # prints: end          then raises StopIteration
```
Output in order: `created`, `start`, `1`, `middle`, `2`, `end`, `StopIteration`.
⚠️ **Common confusion:** calling `demo()` does not print "start". Nothing runs until the first `next()` (or the first loop step).

### 8.5 A real generator: clean lines lazily
```python
def clean_lines(lines):
    for line in lines:
        text = " ".join(line.lower().split())
        if text:
            yield text               # only non-empty lines come out

list(clean_lines(["  Hello   World ", "", "   ", "BYE"]))     # ['hello world', 'bye']
```

### 8.6 `return` vs `yield`: how they differ
| Function with `return` | Generator function with `yield` |
|---|---|
| It runs completely, builds the whole result, and gives it back in one go. | It gives back one item at a time, pausing after each, so the whole result never has to exist at once. |
| The result (for example a list) takes memory for all items. | Only one item (plus the local variables) is in memory at a time. |
| You can loop over the result many times. | The generator is used up after one pass. |
```python
def squares_list(n):
    return [i * i for i in range(n)]

def squares_gen(n):
    for i in range(n):
        yield i * i

squares_list(4)                # [0, 1, 4, 9]
list(squares_gen(4))           # [0, 1, 4, 9]
import sys
sys.getsizeof(squares_list(100000))    # 800984     bytes (about 0.8 MB)
sys.getsizeof(squares_gen(100000))     # 200        bytes, however large n is
```

### 8.7 Generators are used once
```python
g = count_up(3)
list(g), list(g)             # ([1, 2, 3], [])        the second list is empty
```
⚠️ **Common confusion:** just like iterators (section 3), a generator is used up after one pass. To loop again, **call the function again** (`count_up(3)`) to get a fresh generator.

### 8.8 `return` and errors inside a generator
```python
def early(n):
    for i in range(n):
        if i == 2:
            return               # ends the generator (no value is handed out)
        yield i

list(early(10))                  # [0, 1]

def boom():
    yield 1
    raise ValueError("bad row")  # an error inside a generator travels to the code that reads it
    yield 2

try:
    for x in boom():
        print(x)                 # 1
except ValueError as e:
    print("ValueError", e)       # ValueError bad row
```
🎯 **Interview:** what does `yield` do? Difference between `return` and `yield`; why is a generator used once; what does a generator function return when you call it?

---

## 9. Generator expressions

### 9.1 Definition
A generator expression is a list comprehension with **round brackets**. It makes a lazy generator instead of a list.

### 9.2 Syntax
```
[expression for item in iterable]       # list: built now, in memory
(expression for item in iterable)       # generator: made item by item, on demand
sum(expression for item in iterable)    # inside a call, the extra brackets can be left out
```
### 9.3 Examples
```python
g = (x * x for x in range(5))
g.__class__.__name__                    # 'generator'
list(g)                                 # [0, 1, 4, 9, 16]

sum(x * x for x in range(5))            # 30
any(x > 3 for x in range(5))            # True     stops as soon as it finds one
max(len(w) for w in ["a", "bbb", "cc"]) # 3
```
Memory:
```python
import sys
sys.getsizeof([x for x in range(10**6)])      # 8448728   a list: about 8 MB
sys.getsizeof((x for x in range(10**6)))      # 192       a generator: tiny
```
Reading a generator expression lazily:
```python
data = [1, 2, 3]
g = (x * 2 for x in data)
data.append(4)                          # changed AFTER the generator was made
list(g)                                 # [2, 4, 6, 8]     the generator read the list only when used
```
### 9.4 Important points
| List comprehension `[...]` | Generator expression `(...)` |
|---|---|
| It builds the whole list immediately, so you can index it, measure it with `len`, and loop it many times. | It produces items one at a time, so it uses almost no memory, but it can be read only once and has no `len`. |
Use `[...]` when you need the list again. Use `(...)` when you only pass the items on once (to `sum`, `max`, `any`, `all`, `join`, or to a loop).

---

## 10. `yield from`

### 10.1 Definition
`yield from other` hands out every item from another iterable, one by one. It is shorter than writing a `for` loop with `yield`.

### 10.2 Syntax and examples
```
yield from iterable
```
```python
def chain_two(a, b):
    yield from a
    yield from b

list(chain_two([1, 2], "ab"))           # [1, 2, 'a', 'b']

def flatten(nested):                    # a recursive generator (Topic 03 recursion)
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)    # hand out everything the inner call gives
        else:
            yield item

list(flatten([1, [2, [3, 4]], 5]))      # [1, 2, 3, 4, 5]
```

---

## 11. `send()` and `close()` (know they exist)
A generator can also **receive** values. This is rare in everyday AI code (it matters for `async` later), so know the idea.
```python
def running_total():
    total = 0
    while True:
        x = yield total          # hand out total, then wait for a value from outside
        total += x

g = running_total()
next(g)                          # 0     "prime" the generator: run to the first yield
g.send(5)                        # 5     x = 5, total = 5
g.send(10)                       # 15
g.close()                        # stop the generator for good
g.send(1)                        # StopIteration     it is closed
```

---

## 12. Infinite generators and `islice`
A generator can run forever, because it never builds the whole result. You take only what you need with `itertools.islice`.
```python
from itertools import islice

def ids(start=1):
    n = start
    while True:               # never ends: that is fine, it is lazy
        yield n
        n += 1

list(islice(ids(), 3))        # [1, 2, 3]
list(islice(ids(100), 2))     # [100, 101]
```
⚠️ Never call `list(ids())`: it would try to collect infinite items and freeze your computer. Always limit with `islice`, `break` or a condition.

---

## 13. Generator pipelines (chain lazy steps)

### 13.1 Definition
Connect generators like pipes: each step takes the previous one as input. Items flow through **one at a time**, so the whole data never sits in memory.

### 13.2 See the order (a trace)
```python
def src():
    for i in range(3):
        print("make", i)
        yield i

def double(it):
    for x in it:
        print("double", x)
        yield x * 2

for v in double(src()):
    print("got", v)
# make 0
# double 0
# got 0
# make 1
# double 1
# got 2
# make 2
# double 2
# got 4
```
Each item travels through **every** step before the next item starts. A list pipeline would first make all items, then double all, then print all.

### 13.3 Pattern: text pipeline
```
raw lines → clean_lines → classify_lines → batches → use
```
Each arrow is a generator. A full version is in the assignment (section 19, Task 5).

---

## 14. `itertools`: ready-made lazy tools

### 14.1 Definition
`itertools` is a standard-library module of fast, lazy building blocks for iterators. All of them return iterators, so wrap with `list(...)` to see the result.

### 14.2 Syntax and examples
**Taking and slicing**
```
islice(iterable, stop)    islice(iterable, start, stop, step)
```
```python
from itertools import *
list(islice(range(10), 3))            # [0, 1, 2]
list(islice(range(10), 2, 5))         # [2, 3, 4]
list(islice(range(10), 0, 10, 4))     # [0, 4, 8]
```
**Endless counters**
```
count(start, step)    cycle(iterable)    repeat(value, times)
```
```python
list(islice(count(5), 3))             # [5, 6, 7]
list(islice(count(0, 10), 3))         # [0, 10, 20]
list(islice(cycle("ab"), 5))          # ['a', 'b', 'a', 'b', 'a']
list(repeat("x", 3))                  # ['x', 'x', 'x']
```
**Joining**
```
chain(a, b, ...)    chain.from_iterable(list_of_iterables)    zip_longest(a, b, fillvalue=...)
```
```python
list(chain([1, 2], [3], "ab"))                  # [1, 2, 3, 'a', 'b']
list(chain.from_iterable([[1, 2], [3, 4]]))     # [1, 2, 3, 4]      flatten one level
list(zip_longest([1, 2, 3], ["a"], fillvalue="-"))   # [(1, 'a'), (2, '-'), (3, '-')]
```
**Running results and combinations**
```
accumulate(iterable, func)    product(a, b, repeat=n)    combinations(iterable, r)    permutations(iterable, r)
```
```python
from operator import mul
list(accumulate([1, 2, 3, 4]))               # [1, 3, 6, 10]     running sum
list(accumulate([1, 2, 3, 4], max))          # [1, 2, 3, 4]      running max
list(accumulate([1, 2, 3, 4], mul))          # [1, 2, 6, 24]     running product
list(product([1, 2], "ab"))                  # [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]
list(product([0, 1], repeat=2))              # [(0, 0), (0, 1), (1, 0), (1, 1)]
list(combinations([1, 2, 3], 2))             # [(1, 2), (1, 3), (2, 3)]
list(permutations([1, 2, 3], 2))             # [(1, 2), (1, 3), (2, 1), (2, 3), (3, 1), (3, 2)]
```
(`product` is handy for hyper-parameter grids in ML: every combination of learning rate and batch size.)

**Conditions**
```
takewhile(predicate, iterable)    dropwhile(predicate, iterable)    compress(data, selectors)    starmap(func, iterable)    pairwise(iterable)
```
```python
list(takewhile(lambda x: x < 3, [1, 2, 3, 1]))   # [1, 2]      keep while true, stop at the first false
list(dropwhile(lambda x: x < 3, [1, 2, 3, 1]))   # [3, 1]      skip while true, then keep everything
list(compress("abcd", [1, 0, 1, 0]))             # ['a', 'c']  keep where the selector is true
list(starmap(pow, [(2, 3), (3, 2)]))             # [8, 9]      pow(2,3), pow(3,2)
list(pairwise([1, 2, 3, 4]))                     # [(1, 2), (2, 3), (3, 4)]   neighbours
```
**Batching (Python 3.12+)**
```
batched(iterable, n)
```
```python
list(batched(range(7), 3))      # [(0, 1, 2), (3, 4, 5), (6,)]     the last batch can be smaller
```
**Grouping**
```
groupby(iterable, key=None)       # groups only NEIGHBOURING equal items
```
```python
for k, grp in groupby(sorted(["billing", "login", "billing", "shipping", "login"])):
    print(k, list(grp))
# billing ['billing', 'billing']
# login ['login', 'login']
# shipping ['shipping']

words = ["apple", "avocado", "banana", "blueberry", "cherry"]
for k, grp in groupby(words, key=lambda w: w[0]):       # group by first letter
    print(k, list(grp))
# a ['apple', 'avocado']
# b ['banana', 'blueberry']
# c ['cherry']

[(k, len(list(g))) for k, g in groupby("aabbb")]        # [('a', 2), ('b', 3)]     counting runs
```
⚠️ **Common confusion:** `groupby` groups only **neighbours**. If the equal items are not next to each other, you get several groups for the same key. **Sort first** (as the first example does) or use `defaultdict(list)` (Topic 02).

**Copying an iterator**
```python
a, b = tee([1, 2, 3])         # two independent iterators over the same data
list(a), list(b)              # ([1, 2, 3], [1, 2, 3])
```
Why: an iterator can be read once, so `tee` makes two copies if you need to read twice (but if you can, keep a list).

---

## 15. Streaming patterns for AI projects

### 15.1 Reading a big file line by line
An open file is an iterator over its lines, so it never loads the whole file. (`with open(...)` closes the file automatically; files are explained fully in Topic 08.)
```python
def read_clean(path):
    with open(path, encoding="utf-8") as f:
        for line in f:                       # one line at a time
            text = " ".join(line.split())
            if text:
                yield text

# file content:  "line one\n\nline  two \nline three\n"
list(read_clean("log.txt"))                  # ['line one', 'line two', 'line three']
```
A 10 GB log file works the same way as a 10-line one.

### 15.2 Batching for embeddings
APIs and models take a limited number of texts per call. Send them in **batches**.
```python
from itertools import batched

def fake_embed(batch):                       # stands in for a real embedding call
    return [[float(len(t))] for t in batch]

texts = [f"chunk{i}" for i in range(7)]
total = 0
for n, batch in enumerate(batched(texts, 3), start=1):
    vecs = fake_embed(batch)
    total += len(vecs)
    print("batch", n, "size", len(batch), vecs[0])
print(total)
# batch 1 size 3 [6.0]
# batch 2 size 3 [6.0]
# batch 3 size 1 [6.0]
# 7
```
Your own batching generator (works on any Python version and shows how `batched` works inside):
```python
def batches(iterable, size):
    batch = []
    for item in iterable:
        batch.append(item)
        if len(batch) == size:
            yield batch
            batch = []                       # start a NEW list (Topic 01: do not reuse the old one)
    if batch:                                # the leftover items
        yield batch

list(batches(range(7), 3))                   # [[0, 1, 2], [3, 4, 5], [6]]
```
### 15.3 Streaming an LLM answer
LLM APIs return the answer in small pieces so the user sees text appear immediately. Your code receives a **generator** and loops over it.
```python
def fake_llm_stream(answer):                 # stands in for the real API
    for word in answer.split():
        yield word + " "

for token in fake_llm_stream("Refunds take five days"):
    print(token, end="")                     # Refunds take five days
print()

full = "".join(fake_llm_stream("Refunds take five days"))
full                                         # 'Refunds take five days '
```
The real call looks like this (pseudo-code, needs an API key, not run here):
```python
stream = client.chat.completions.create(model="...", messages=[...], stream=True)
for chunk in stream:                         # a generator of small pieces
    piece = chunk.choices[0].delta.content or ""
    print(piece, end="", flush=True)
```
FastAPI streams the same way: a route returns `StreamingResponse(your_generator())` (you will do this in the FastAPI phase).

---

## 16. Pitfalls
```python
g = (x for x in range(3))
len(g)               # TypeError: object of type 'generator' has no len()
g[0]                 # TypeError: 'generator' object is not subscriptable
```
- A generator has **no length and no index**. Convert with `list(g)` only if the result is small.
- `list(gen)` **defeats the laziness** (everything is in memory). Loop directly instead.
- A generator is used **once** (sections 3 and 8.7). Keep the **function**, and call it again for a fresh one.
- Because generators are lazy, **errors appear late**, when the item is actually produced (section 8.8).
- Generators need the data to stay valid until you read them (if the source list changes meanwhile, you see the changes, section 9.3).
- A generator that holds an open file stays open until it finishes. Use `with` inside it (section 15.1).

---

## 17. Patterns to remember

| Pattern | Template | Used for |
|---|---|---|
| **Lazy filter** | `def f(items): for x in items: if ok(x): yield x` | Dropping empty or bad rows. |
| **Lazy transform** | `def f(items): for x in items: yield change(x)` | Cleaning each text. |
| **Pipeline** | `use(step3(step2(step1(source))))` | Read → clean → classify → batch. |
| **Batcher** | `batch=[]; for x: batch.append(x); if len(batch)==n: yield batch; batch=[]` then yield the leftover | Embedding calls, database inserts. |
| **Generator expression into a function** | `sum(x for x in items)` / `"".join(...)` / `any(...)` | One pass, no list. |
| **Take the first n** | `list(islice(gen, n))` | Previews, infinite generators. |
| **Stream** | `for piece in stream: handle(piece)` | LLM output, big files, API pages. |
| **Reuse** | keep the function (or the list), not the used-up generator | Avoiding the "empty second time" bug. |

**Structure to remember (a generator function):**
```
def step(source):
    for item in source:          # read from the previous step
        ...decide / change...
        yield result             # hand out one at a time
```

---

## 18. Use this when building (project guide)

- **Customer Support Copilot:** read the ticket file lazily (section 15.1), clean and classify with generators, and batch the texts before embedding (section 15.2). It keeps memory small even for a large dataset.
- **Embedding job:** `for batch in batched(texts, 100): vectors = embed(batch); save(vectors)`. Never `embed(all_texts)` in one call.
- **Chat UI:** stream the LLM answer to the user as it arrives (section 15.3) so the user sees text immediately.
- **Pandas later:** `pd.read_csv(path, chunksize=10_000)` returns an **iterator** of DataFrames. You loop over it exactly like a generator.
- **PyTorch later:** a `DataLoader` is an iterable that gives batches, and you write `for batch in loader:`.
- **FastAPI:** streaming responses take a generator.
- **Debug recipe:** if a loop or `list(...)` suddenly gives nothing, check whether you are reading a **used-up iterator or generator**. Print `type(x)` to see whether it is a list or a `generator`.

**Checklist:** do I need all items at once (list) or one at a time (generator)? Am I reading a generator twice? Did I call `list()` on something huge or infinite? Am I changing a collection while looping over it? Is the batch list **new** after each `yield`?

---

## 🧠 Quick Revision

- Iterable = can be looped (`__iter__`). Iterator = gives items one at a time, remembers its position, **one use** (`__next__`). `iter()` makes an iterator, `next()` reads one, `StopIteration` means the end (a `for` loop catches it quietly).
- `for x in items` = `iter(items)` plus repeated `next`. `next(it, default)` avoids the error.
- Lists, strings, dicts, files and `range` are iterables. `range` is not an iterator. `map`, `filter`, `zip`, `enumerate`, `reversed` and files are lazy iterators.
- `sum`, `list`, `in`, unpacking and `next` **consume** an iterator. A second pass gives nothing. Never change a dict or list's size while looping over it.
- Iterator class: `__iter__` returns `self` + `__next__`. Iterable class: `__iter__` returns a fresh iterator.
- Lazy keeps memory small; eager (a list) keeps everything. A generator object is tiny, however many items it will make.
- A function with `yield` is a **generator function**. Calling it runs nothing; each `next()` runs to the next `yield` and pauses with its variables kept. `return` ends it. It is used **once**.
- `(x for x in items)` is a generator expression (no `len`, no index, one use); `[x for x in items]` is a list. Pass generator expressions straight to `sum`, `any`, `max`, `join`.
- `yield from other` hands out all of `other`. `send()` and `close()` exist but are rare.
- Infinite generators are fine; limit them with `islice` or `break`.
- Pipelines of generators pass **one item at a time through every step**.
- `itertools`: `islice`, `count`, `cycle`, `repeat`, `chain`, `zip_longest`, `accumulate`, `product`, `combinations`, `permutations`, `takewhile`, `dropwhile`, `compress`, `starmap`, `pairwise`, `batched` (3.12+), `groupby` (sort first), `tee`.
- AI uses: read big files line by line, embed in batches, stream LLM output, Pandas `chunksize`, PyTorch `DataLoader`.
- Generator pitfalls: no `len` or index, used once, errors appear late, `list(gen)` defeats laziness.

---

## 19. Assignment (try it first, then check the answers below)

Create `07-assignment.ipynb`. Run each task, and compare with the answers.

**Task 1: iterator class.** Write class `CountUp(limit)` that gives `1, 2, ..., limit`, as an iterator (`__iter__` and `__next__`). Test it with a `for` loop, with `list(CountUp(3))`, with `list(CountUp(0))`, and by calling `list(...)` **twice** on the same object.

**Task 2: generator function.** Write a generator `evens(limit)` that gives the even numbers from 0 up to and including `limit`. Test `list(evens(10))`. Then take only the first 3 from `evens(10**12)` (without building the whole thing).

**Task 3: predict (no running first).**
```python
g = (n * n for n in range(4))
print(next(g))
print(sum(g))
print(list(g))
```
**Task 4: predict the order of the output.**
```python
def src():
    for i in range(3):
        print("make", i)
        yield i

def double(it):
    for x in it:
        print("double", x)
        yield x * 2

for v in double(src()):
    print("got", v)
```
**Task 5: ticket stream pipeline (the mini project).** Use `clean_text` and `classify` from Topic 05 (copy them in). Write three generators and connect them:
1. `clean_lines(lines)`: yields the cleaned text of each line, **skipping empty ones**.
2. `classify_lines(lines)`: yields `(category, text)`.
3. `batches(iterable, size)`: yields lists of up to `size` items (your own version, not `itertools`).

Run it on this data with batch size 3, print each batch with `f"batch {number}: {len(batch)} tickets"`, then each ticket as `f"  {category:<8} | {text}"`, and count categories with a `Counter`.
```python
raw_lines = [
    "  Order #4412: REFUND not received!!  ",
    "",
    "Cannot reset my PASSWORD   ",
    "   ",
    "Charged twice for order #5521 ???",
    "Late delivery... still waiting",
    "refund please",
    "Where is my parcel",
]
```
**Task 6: compare.** Check that `itertools.batched(range(7), 3)` gives the same groups as your `batches(range(7), 3)` (note: `batched` gives tuples).

---

## Answers

### Task 1
```python
class CountUp:
    def __init__(self, limit):
        self.limit = limit
        self.current = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.current >= self.limit:
            raise StopIteration
        self.current += 1
        return self.current

for n in CountUp(3):
    print(n, end=" ")                 # 1 2 3

list(CountUp(3)), list(CountUp(0))    # ([1, 2, 3], [])
c = CountUp(2)
list(c), list(c)                      # ([1, 2], [])      the object is an iterator: second time it is used up
```
Why: `current` keeps the position. After the last item `current` equals `limit`, so `__next__` raises `StopIteration` from then on. The same object cannot restart (use a new `CountUp(2)`).

### Task 2
```python
def evens(limit):
    for n in range(0, limit + 1, 2):    # step 2 from 0; limit + 1 so that limit itself can be included
        yield n

list(evens(10))                          # [0, 2, 4, 6, 8, 10]

from itertools import islice
list(islice(evens(10**12), 3))           # [0, 2, 4]     only 3 items are ever made
```
A `while` version works the same:
```python
def evens2(limit):
    n = 0
    while n <= limit:
        yield n
        n += 2

list(evens2(7))                          # [0, 2, 4, 6]
```
Why it works for `10**12`: `range` is lazy and the generator is lazy, and `islice` stops after 3 items, so almost nothing is computed.

### Task 3
```
0
14
[]
```
Why: `next(g)` takes the first item, `0` (0 squared). `sum(g)` then adds the **rest**: `1 + 4 + 9 = 14`. After that the generator is used up, so `list(g)` is `[]`. (This is section 5: reading moves the position forward.)

### Task 4
```
make 0
double 0
got 0
make 1
double 1
got 2
make 2
double 2
got 4
```
Why: items flow **one at a time** through the whole pipeline. The loop asks `double` for a value, `double` asks `src`, `src` prints "make 0" and yields 0, `double` prints "double 0" and yields 0, and the loop prints "got 0". Only then is the next item made. (Section 13.2.)

### Task 5
```python
from collections import Counter

def clean_text(raw):
    text = raw.strip().lower()
    text = text.replace("!", "").replace("?", "")
    return " ".join(text.split())

def classify(text):
    if "refund" in text or "charge" in text:
        return "billing"
    elif "password" in text or "login" in text:
        return "login"
    elif "delivery" in text or "late" in text:
        return "shipping"
    else:
        return "other"

def clean_lines(lines):
    for raw in lines:
        cleaned = clean_text(raw)
        if cleaned:                          # skip empty tickets (empty text is falsy)
            yield cleaned

def classify_lines(lines):
    for text in lines:
        yield classify(text), text

def batches(iterable, size):
    batch = []
    for item in iterable:
        batch.append(item)
        if len(batch) == size:
            yield batch
            batch = []                       # a NEW list for the next batch
    if batch:                                # the leftover
        yield batch

raw_lines = [
    "  Order #4412: REFUND not received!!  ",
    "",
    "Cannot reset my PASSWORD   ",
    "   ",
    "Charged twice for order #5521 ???",
    "Late delivery... still waiting",
    "refund please",
    "Where is my parcel",
]

pipeline = classify_lines(clean_lines(raw_lines))      # nothing runs yet: all lazy
counts = Counter()
for number, batch in enumerate(batches(pipeline, 3), start=1):
    print(f"batch {number}: {len(batch)} tickets")
    for category, text in batch:
        counts[category] += 1
        print(f"  {category:<8} | {text}")
print(dict(counts))
```
**Output:**
```
batch 1: 3 tickets
  billing  | order #4412: refund not received
  login    | cannot reset my password
  billing  | charged twice for order #5521
batch 2: 3 tickets
  shipping | late delivery... still waiting
  billing  | refund please
  other    | where is my parcel
{'billing': 3, 'login': 1, 'shipping': 1, 'other': 1}
```
Why it is built this way:
- The pipeline is a chain of three generators, so no step builds a full list. The same code would work for a file with millions of lines (replace `raw_lines` with `read_clean(path)` from section 15.1).
- Empty lines are removed in `clean_lines` (the "skip empty" pattern from Topic 05).
- The 8 raw lines give 6 tickets, which `batches` splits into 3 + 3. With 7 tickets the last batch would hold 1 (the leftover `if batch:` check).
- Mistake to avoid: reusing the **same list** for every batch (`batch.clear()` instead of `batch = []`). If you collect the batches (for example `list(batches(...))`), they would all be the same, emptied list (Topic 01: one list, many names).

### Task 6
```python
from itertools import batched

[list(b) for b in batched(range(7), 3)]                         # [[0, 1, 2], [3, 4, 5], [6]]
[list(b) for b in batched(range(7), 3)] == list(batches(range(7), 3))     # True
```
`batched` gives **tuples**, so convert with `list(b)` to compare. In real projects use `batched` (shorter and fast); the version you wrote shows what it does inside.