# 03 — Functions and Scope

> Phase 0 · Python foundation. FastAPI routes, pytest fixtures, Pandas `.apply()`, `lru_cache` for embeddings: all of them are built on the ideas in this file (closures and decorators especially).
> Running example: a **support-ticket cleaner**.
> `🎯 Interview:` marks a point worth remembering for interviews. `⚠️ Common confusion:` marks a place where I got stuck, with the reason explained.

---

## 1. Function basics

### 1.1 Definition
A **function** is a reusable block of code: it takes inputs (**parameters**), does work, and gives back an output (**return value**). In Python a function is also an **object**: you can store it in a variable, pass it to another function, and return it from a function.

### 1.2 Important points
- `def` only **creates** the function. The body runs when you **call** it.
- `f` is the function itself. `f()` **calls** it.
- No `return` (or a bare `return`) gives back `None`.
- `return a, b` returns one **tuple** `(a, b)`.
- A parameter is the name in `def f(x)`. An argument is the value in `f(5)`.

### 1.3 Example
**Syntax:**
```
def function_name(parameter1, parameter2):
    """Optional docstring."""
    ...body...
    return value              # optional; without it the function returns None

function_name(argument1, argument2)      # call it
other_name = function_name               # no brackets: a second name for the same function
```
```python
def clean(text):
    return text.strip().lower()

fix = clean                      # second label on the same function (no brackets)
print(fix("  Refund NOT received "))     # refund not received
```

### 1.4 ⚠️ Common confusion: `print` vs `return`
```python
def clean(text):
    print(text.strip().lower())          # shows text, returns None

result = clean("  Hello ")               # prints: hello
print(result)                            # None
```
`print` only **shows** text on the screen. `return` **hands a value back** so the caller can store it, test it or pass it on. A function that only prints returns `None`. Use `return` for logic and `print` only for display.

---

## 2. Parameters: the five kinds

| Kind | Written as | Meaning |
|---|---|---|
| Positional | `def f(a, b)` | Matched by order: `f(1, 2)`. |
| Default | `def f(a, b=10)` | `b` is optional. |
| `*args` | `def f(*args)` | Collects extra **positional** arguments into a **tuple**. |
| Keyword-only | `def f(*args, key)` | Comes after `*args` (or `*`), so it must be passed by name. |
| `**kwargs` | `def f(**kwargs)` | Collects extra **keyword** arguments into a **dict**. |

Order in the `def` line: positional → defaults → `*args` → keyword-only → `**kwargs`.

### 2.0 One example for each kind
**Syntax:**
```
def f(a, b)                       # positional
def f(a, b=10)                    # default value (defaults go after non-defaults)
def f(*args)                      # extra positional -> tuple
def f(*args, key)                 # key must be passed by name
def f(a, *, key)                  # bare * : everything after it is keyword-only
def f(a, b, /, c)                 # / : everything before it is positional-only
def f(**kwargs)                   # extra keyword -> dict
def f(a, b=1, *args, key, **kwargs)     # full order
f(*my_list, **my_dict)            # unpack when calling
return a, b                       # returns one tuple
```
```python
def greet(name, greeting="Hello"):        # positional + default
    return f"{greeting}, {name}"

greet("Ann")                  # 'Hello, Ann'      default used
greet("Ann", "Hi")            # 'Hi, Ann'         by position
greet(greeting="Yo", name="Bo")   # 'Yo, Bo'      by name: order no longer matters
```
```python
def show(*args):              # *args collects extra positional arguments into a tuple
    return args

show()           # ()
show(1)          # (1,)
show(1, 2, 3)    # (1, 2, 3)
```
```python
def info(**kw):               # **kwargs collects extra keyword arguments into a dict
    return kw

info()           # {}
info(a=1, b=2)   # {'a': 1, 'b': 2}
```
```python
def f(a, *, key):             # everything after the bare * must be passed by name
    return a, key

f(1, key=2)      # (1, 2)
f(1, 2)          # TypeError: f() takes 1 positional argument but 2 were given
```
```python
def g(a, b, /, c):            # everything before the / must be passed by position
    return a, b, c

g(1, 2, c=3)           # (1, 2, 3)
g(a=1, b=2, c=3)       # TypeError: g() got some positional-only arguments passed as keyword arguments: 'a, b'
```
Returning many values:
```python
def mm(a, b):
    return a + b, a * b       # one tuple

mm(2, 3)          # (5, 6)
s, p = mm(2, 3)   # unpack: s = 5, p = 6
```

### 2.1 Example
```python
def total(*nums, **opts):
    round_to = opts.get("round_to")        # None if not given
    if round_to is not None:
        return round(sum(nums), round_to)
    return sum(nums)

print(total(1.234, 2.345, round_to=2))     # 3.58
print(total(1, 2, 3))                      # 6
print(total())                             # 0
```
Here `nums = (1.234, 2.345)` and `opts = {"round_to": 2}`.

### 2.2 Stars in a **call** do the reverse (unpack)
```python
nums = [1, 2]
opts = {"round_to": 1}
total(*nums, **opts)             # same as total(1, 2, round_to=1)
```
`*` spreads a list or tuple into separate arguments. `**` spreads a dict into keyword arguments.

**Real project use:** `Model(**data)` builds a Pydantic model from a dict, and wrappers pass `*args, **kwargs` through to another function.

---

## 3. Scope (LEGB)

### 3.1 Definition
**Scope** = where a name can be seen. Python searches from the inside out and stops at the first match:

| Letter | Scope | Meaning |
|---|---|---|
| **L** | Local | Names created inside the current function. |
| **E** | Enclosing | Names in an outer function that contains this one. |
| **G** | Global | Names at the top level of the file or notebook. |
| **B** | Built-in | Names Python provides (`len`, `print`, `sum`). |

🎯 **Interview:** "What is the LEGB rule?" Local, Enclosing, Global, Built-in: the order Python looks up a name.

### 3.2 ⚠️ Common confusion: why `count += 1` gives `UnboundLocalError`
```python
x = 10

def read_only():
    print(x)              # 10  (only reading: Python finds the global x)

def read_then_assign():
    print(x)              # UnboundLocalError
    x = 20
```
Python decides **before running** the function: **if a name is assigned anywhere inside it, that name is local to the whole function.** So `print(x)` looks for the **local** `x`, which has no value yet. It does not fall back to the global.

`count += 1` secretly means `count = count + 1`, which is an assignment. So this fails too:
```python
count = 0
def bad():
    count += 1            # UnboundLocalError
```
**Fixes:** `global count` (module-level variable) or `nonlocal count` (variable of an enclosing function).

| Situation | What happens |
|---|---|
| Only **reading** an outer name | Works (Python searches outward). |
| **Assigning** it with no declaration | Creates a new local name; the outer one is untouched. |
| Reading, then assigning the same name | `UnboundLocalError`. |
| Assigning with `global` / `nonlocal` declared | Changes the outer name. |

**Syntax:**
```
global name        # inside a function: "name is the module-level variable"
nonlocal name      # inside an inner function: "name is the enclosing function's variable"
```
**`global` in action:**
```python
x = 10
def change():
    global x          # "x means the module-level x"
    x = 20

change()
print(x)              # 20
```
**Reading an enclosing name (no keyword needed):**
```python
def outer():
    y = 1
    def inner():
        return y + 1  # only reads y
    return inner()

outer()               # 2
```

---

## 4. Closures

### 4.1 Definition
A **closure** is an inner function that **remembers the variables** of the outer function it was created in, even after the outer function has finished. Think of it as a function carrying a **backpack**.

### 4.2 Example: a counter that remembers
**Syntax (closure pattern):**
```
def outer(parameter):
    remembered = ...
    def inner(argument):
        nonlocal remembered       # only if you re-assign it
        ...use parameter / remembered...
        return result
    return inner                  # no brackets: return the function itself
```
```python
def make_counter():
    count = 0
    def inc():
        nonlocal count        # we will re-assign the enclosing variable
        count += 1
        return count
    return inc                # return the function itself, NOT inc()

c = make_counter()
print(c(), c(), c())          # 1 2 3
d = make_counter()            # a brand new call: a brand new count
print(d())                    # 1
```
**How it works:**
1. `make_counter()` creates `count = 0` and the function `inc`, then returns `inc`.
2. `count` is **not** destroyed, because `inc` still uses it (it is in `inc`'s backpack).
3. Each `c()` updates that same `count`: 1, 2, 3.
4. `make_counter()` again creates a **separate** `count`, so `d()` starts at 1.

### 4.3 ⚠️ Common confusion: `nonlocal`, and `return inc` vs `return inc()`
**Why `nonlocal`?** Without it, `count += 1` is an assignment, so `count` becomes local to `inc` and fails with `UnboundLocalError` (same rule as 3.2). `nonlocal count` says "this name belongs to the enclosing function".

**When you do not need it:** if the inner function only **reads** the variable.
```python
def make_multiplier(n):
    def multiply(x):
        return x * n          # only reading n: no nonlocal needed
    return multiply

double = make_multiplier(2)
print(double(5))              # 10
```
**Rule:** read-only → nothing needed. Re-assigning → `nonlocal`.

**`return inc` vs `return inc()`:**

| Written as | What `c` becomes |
|---|---|
| `return inc` | The inner **function**, so `c()` works and counts 1, 2, 3. |
| `return inc()` | The **number** `1` (the function was run right now), so `c()` gives `TypeError: 'int' object is not callable`. |

**Rule of thumb:** brackets mean "run it now". No brackets mean "hand the function over to run later".

**See the backpack:**
```python
c = make_counter()
c.__code__.co_freevars            # ('count',)   what it borrows from outside
c.__closure__[0].cell_contents    # 0   current value (updates as you call c())
```

**Real project use:** build a configured function once and reuse it.
```python
def make_score_filter(min_score):
    def keep(result):
        return result["score"] >= min_score
    return keep

good = list(filter(make_score_filter(0.8), results))    # keep only strong search matches
```

---

## 5. Decorators

### 5.1 Definition
A **decorator** is a function that **takes a function and returns a new function** that wraps it with extra behaviour (timing, logging, a login check, caching). Writing `@name` above a `def` is shorthand:
```python
@timed
def load_tickets(): ...
# is exactly the same as:
def load_tickets(): ...
load_tickets = timed(load_tickets)
```

### 5.2 Example: time any function
**Syntax (decorator pattern):**
```
from functools import wraps

def decorator_name(fn):
    @wraps(fn)
    def wrapper(*args, **kwargs):
        ...do something before...
        result = fn(*args, **kwargs)
        ...do something after...
        return result
    return wrapper

@decorator_name              # same as: my_func = decorator_name(my_func)
def my_func(...): ...
```
```python
import time
from functools import wraps

def timed(fn):                                  # fn = the original function
    @wraps(fn)                                  # keep the original name and docstring
    def wrapper(*args, **kwargs):               # accept ANY arguments
        start = time.perf_counter()             # precise clock, in seconds
        result = fn(*args, **kwargs)            # run the original function
        end = time.perf_counter()
        print(f"{fn.__name__} took {end - start:.4f}s")
        return result                           # give back the original's result
    return wrapper

@timed
def big_sum():
    return sum(range(1_000_000))

print(big_sum())
```
**Result:** a line like `big_sum took 0.0085s`, then `499999500000`.

### 5.3 ⚠️ Common confusion: when does each line run?
```python
def timed(fn):
    print("1. timed() is running, decorating", fn.__name__)
    def wrapper(*args, **kwargs):
        print("3. wrapper is running")
        result = fn(*args, **kwargs)
        print("4. original finished")
        return result
    print("2. timed() finished, returning wrapper")
    return wrapper

@timed
def greet():
    return "hi"

print("--- decorated, now calling ---")
print(greet())
```
**Result:**
```
1. timed() is running, decorating greet
2. timed() finished, returning wrapper
--- decorated, now calling ---
3. wrapper is running
4. original finished
hi
```
- Steps **1 and 2 run once**, when the function is defined.
- Steps **3 and 4 run on every call**. After decoration, the name `greet` points to `wrapper`.
- `wrapper` is a **closure**: it remembers `fn` in its backpack. That is why closures come before decorators.

**What `fn(*args, **kwargs)` means:** the wrapper's `*args, **kwargs` **pack** whatever it receives, and `fn(*args, **kwargs)` **unpacks** it again for the original. Called as `f("Ann", punct="?")`, the wrapper gets `args=("Ann",)`, `kwargs={"punct": "?"}` and calls `fn("Ann", punct="?")`.

**Why save `result` in a variable?** We must run the original, **then** stop the clock, **then** return. `return` ends the function at once, so we store the value first.

| If you forget this | What happens |
|---|---|
| `return result` | The decorated function always returns `None`. |
| `*args, **kwargs` in the wrapper | `TypeError` as soon as the function has arguments. |
| The call `fn(...)` | The original never runs. |
| `@wraps(fn)` | It still works, but `greet.__name__` becomes `"wrapper"` and the docstring is lost. |

🎯 **Interview:** `@decorator` means `f = decorator(f)`. The decorator runs once at definition; the wrapper runs on every call.

### 5.4 A decorator that takes an argument needs one more layer
**Syntax:**
```
def decorator_with_arg(arg):          # 1st layer: gets the argument
    def decorator(fn):                # 2nd layer: gets the function
        @wraps(fn)
        def wrapper(*args, **kwargs): # 3rd layer: runs on each call
            ...
        return wrapper
    return decorator

@decorator_with_arg(3)
def my_func(...): ...
```
```python
def repeat(n):                    # outer: receives the argument
    def decorator(fn):            # middle: receives the function
        @wraps(fn)
        def wrapper(*args, **kwargs):     # inner: runs on each call
            for _ in range(n):
                result = fn(*args, **kwargs)
            return result
        return wrapper
    return decorator
```
`@repeat(3)` first calls `repeat(3)`, which returns `decorator`, which is then applied to the function.

**Real project use:** `@app.get("/tickets")` in FastAPI registers your function as a route (it is a decorator that stores your function in a table of routes). Logging, auth checks, retry and caching are usually decorators.

---

## 6. Lambda and the late-binding trap

### 6.1 Definition
A **lambda** is a small anonymous function with **one expression**: `lambda x: x * 2`. It returns that expression's value. Use it for short throwaway functions, mostly as a `key=`. For anything longer, use `def`.

### 6.2 ⚠️ Common confusion: why `[2, 2, 2]`?
```python
fs = [lambda: i for i in range(3)]
print([f() for f in fs])         # [2, 2, 2]   (not [0, 1, 2])
```
Each lambda does **not** store the **value** of `i`. It stores a link to the **variable** `i` and looks it up **when it is called**. By then the loop has finished and `i == 2`. All three lambdas share the same variable:
```python
fs[0].__closure__[0] is fs[1].__closure__[0]     # True
```

| Fix | Code | Why it works |
|---|---|---|
| Default argument | `lambda i=i: i` | The default is evaluated when the lambda is **created**, so each lambda keeps its own copy. |
| Factory function | `def make(i): return lambda: i` | Each call to `make` creates a new scope with its own `i`. |
| `functools.partial` | `partial(show, i)` | `partial` stores the current value at creation time. |

```python
fs = [lambda i=i: i for i in range(3)]
print([f() for f in fs])         # [0, 1, 2]
```
**Rule:** a closure remembers the **variable**, not the value it had at that moment. The same trap applies to a normal `def` inside a loop.

**Syntax:**
```
lambda parameters: expression       # no return keyword; one expression only
lambda x: x * 2                     lambda a, b: a + b       lambda: 5
```
**Lambda basics (3 examples):**
```python
sq = lambda x: x * x
sq(4)                                    # 16
(lambda a, b: a + b)(2, 3)               # 5     a lambda used on the spot
sorted(["bb", "a", "ccc"], key=lambda s: -len(s))   # ['ccc', 'bb', 'a']   longest first
list(map(lambda x: x * 2, [1, 2, 3]))    # [2, 4, 6]
list(filter(lambda x: x > 1, [1, 2, 3])) # [2, 3]
```
A function can be passed like any value:
```python
def apply(fn, value):
    return fn(value)

apply(str.upper, "hi")     # 'HI'
apply(len, "abc")          # 3
callable(apply)            # True     is it something you can call?
callable(5)                # False
```

---

## 7. map, filter, reduce and `key=`

### 7.1 Definition
These process a sequence **with a function instead of a loop**. They are **higher-order functions** (they take a function as an argument).

| Tool | Its one job | Result size |
|---|---|---|
| `map(f, items)` | **Transform** every item. | Same number of items. |
| `filter(f, items)` | **Keep** items where `f(item)` is true. | Same or fewer. |
| `reduce(f, items, start)` | **Combine** all items into one value. | One value. |

### 7.2 map and filter
**Syntax:**
```
map(function, iterable)                  # new iterator of function(item) for every item
map(function, iterable1, iterable2)      # function gets one item from each
filter(function, iterable)               # keeps items where function(item) is truthy
filter(None, iterable)                   # keeps truthy items
list(map(...))   list(filter(...))       # needed to see the results
```
```python
texts = ["  Refund ", "", "LOGIN issue", "   "]

stripped = map(str.strip, texts)             # a map object, not a list!
print(list(stripped))                        # ['Refund', '', 'LOGIN issue', '']
print(list(stripped))                        # []   already used up

non_empty = list(filter(None, map(str.strip, texts)))
print(non_empty)                             # ['Refund', 'LOGIN issue']
```
**Important points:**
- In Python 3, `map` and `filter` return **lazy iterators**. They compute one item at a time, only when you loop over them or call `list()`.
- An iterator can be used **once**. After that it is **exhausted** (empty).
- `filter(None, items)` keeps only **truthy** items (drops `""`, `0`, `None`, `[]`).
- `map(print, items)` prints nothing, because nothing consumes it. For side effects use a `for` loop.
- `map` with two iterables passes one item from each and stops at the shortest.
- The same results with comprehensions: `[t.strip() for t in texts if t.strip()]`. Prefer comprehensions when the logic is new; use `map`/`filter` when a named function already exists.

### 7.3 reduce
**Syntax:**
```
from functools import reduce
reduce(function, iterable, initial_value)      # function(accumulator, item) -> new accumulator
```
`reduce` is in `functools` (not built-in), so `from functools import reduce`. It keeps an **accumulator** (the running result) and combines it with the next item. The function takes exactly **two arguments**.
```python
from functools import reduce

texts = ["refund", "login issue", "late delivery"]
total_chars = reduce(lambda acc, t: acc + len(t), texts, 0)
print(total_chars)                           # 6 + 11 + 13 = 30
```
Step by step (start value `0`): `0 + 6 = 6` → `6 + 11 = 17` → `17 + 13 = 30`.
- The third argument is the **initializer** (the starting value).
- Empty input **without** an initializer raises `TypeError`. **With** one, it returns the initializer. So always give one if the input might be empty.
- Prefer built-ins when they exist: `sum`, `math.prod`, `max`, `min`, `all`, `any`, `"".join(...)`. Here `sum(map(len, texts))` does the same job. Use `reduce` only for a custom combining rule.

### 7.4 `key=` functions (`sorted`, `min`, `max`)
**Syntax:**
```
sorted(iterable, key=function, reverse=False)
min(iterable, key=function)      max(iterable, key=function)
lst.sort(key=function, reverse=False)
```
`key` is called **once per item** to get a sort value. Items are compared by those values, but the **original items** are returned.
```python
words = ["bb", "a", "ccc"]
sorted(words, key=len)                       # ['a', 'bb', 'ccc']
max(words, key=len)                          # 'ccc'
sorted(results, key=lambda r: r["score"], reverse=True)[:3]    # top-3 search results
```
Sorting is stable (equal keys keep their order). A tuple key sorts by several fields: `key=lambda p: (p.age, p.name)`.

| `map` / `filter` | List comprehension |
|---|---|
| They return lazy iterators that can be used only once, so nothing is computed until you loop or call `list()`. | A list comprehension builds the whole list in memory at once, and a list can be looped over many times. |
| They need a function, and a long lambda makes them hard to read. | The expression and the condition are written inline, which is usually easier to read. |

🎯 **Interview:** `map` returns a lazy iterator, not a list. `reduce` needs an import and an initializer for safety. Prefer comprehensions and built-ins.

**Real project use:** Pandas `Series.map()` and `.apply()` pass a function to every value. In RAG: `map(embed, chunks)` to embed every chunk, `filter` to drop results below a score threshold.

---

## 8. functools: partial and lru_cache

### 8.1 partial: pre-fill some arguments
**Syntax:**
```
from functools import partial
new_function = partial(original_function, argument_or_keyword=value)
```
```python
from functools import partial

def chunk_text(text, size):
    return [text[i:i + size] for i in range(0, len(text), size)]

chunk_100 = partial(chunk_text, size=100)      # a new function with size fixed
print(chunk_100("a" * 250)[1][:3])             # 'aaa'
```
Use it to adapt a function to something that expects fewer arguments (callbacks, `map`).

### 8.2 lru_cache: remember results (memoization)
**Syntax:**
```
from functools import lru_cache

@lru_cache(maxsize=128)        # maxsize=None means unlimited
def my_func(hashable_args): ...

my_func.cache_info()           my_func.cache_clear()
```
**Memoization** = store a function's result so the same call returns instantly next time. (LRU = least recently used: when full, the oldest unused result is dropped.)
```python
from functools import lru_cache

@lru_cache(maxsize=1000)
def embed(text):                    # imagine this calls a paid embedding API
    print("calling the model for:", text)
    return [float(len(text))]

embed("refund")                     # prints, computes
embed("refund")                     # instant, no print: served from the cache
print(embed.cache_info())           # hits=1, misses=1, ...
```
**Important points:**
- The cache is a **dict keyed by the arguments**, so arguments must be **hashable** (a `str` is fine; a `list` raises `TypeError`).
- Only cache **pure** functions (same input → same output). Caching something that reads the time or a database returns stale answers.
- Famous example: `fib(n)` without a cache is exponential (about 2ⁿ calls); with `@lru_cache` it is linear (about n).
- Another common use: `@lru_cache def get_settings(): ...` in FastAPI, so configuration loads once.

---

## 9. Recursion, docstrings, pure functions

### 9.1 Recursion
**Syntax (template):**
```
def solve(problem):
    if problem_is_small_enough:        # base case: stop here
        return answer
    return combine(solve(smaller_problem))      # the function calls itself
```
A **recursive** function calls itself on a **smaller** problem. It needs a **base case** (when to stop), or you get `RecursionError` (the default limit is about 1000 nested calls).
```python
def fact(n):
    if n <= 1:                 # base case
        return 1
    return n * fact(n - 1)     # smaller problem each time

print(fact(5))                 # 120
```
Each call gets its own local scope on the **call stack** and waits until the call below it returns.

**Real project use:** the **recursive text splitter** in RAG chunking. Split by paragraphs; if a piece is still too long, split it by lines, then by words, until every piece fits. Simplified version:
```python
def split_recursive(text, max_len, seps=("\n\n", "\n", " ")):
    if len(text) <= max_len:                       # base case: it already fits
        return [text]
    if not seps:                                   # nothing left to split on: hard cut
        return [text[i:i + max_len] for i in range(0, len(text), max_len)]
    pieces = text.split(seps[0])
    out = []
    for p in pieces:
        out.extend(split_recursive(p, max_len, seps[1:]))
    return out
```
(Real splitters also merge small pieces back together; this shows the recursive idea.)

Two more recursion examples:
```python
def countdown(n):
    if n == 0:                 # base case
        print("done")
        return
    print(n)
    countdown(n - 1)

countdown(3)                   # 3 / 2 / 1 / done

def total(items):              # sum of a nested list
    s = 0
    for x in items:
        s += total(x) if isinstance(x, list) else x
    return s

total([1, [2, 3], [4, [5]]])   # 15
```

### 9.2 Docstring
**Syntax:**
```
def name(params):
    """One-line summary."""          # first statement in the function

    # or several lines:
    """Summary line.

    Longer explanation, arguments, return value.
    """
```
A string written as the **first statement** of a function. It documents it. Python keeps it in `f.__doc__`, `help(f)` shows it, and FastAPI uses it in the generated API docs.
```python
def area(r):
    """Return the area of a circle with radius r."""
    return 3.14159 * r * r
```
```python
area.__doc__          # 'Return the area of a circle with radius r.'
area.__name__         # 'area'
help(area)            # prints the docstring in a readable form
```

### 9.3 Pure vs impure
- A **pure** function depends only on its inputs and has **no side effects**.
- A **side effect** is anything besides returning a value: printing, writing a file, changing a global or a mutable argument, calling an API.

Pure functions are easy to test, cache and reuse. Keep the core logic pure and push side effects (I/O, API calls) to the edges of your program.

---

## 10. Patterns to remember

| Pattern | Template | Used for |
|---|---|---|
| **Pure helper** | `def f(x): return ...` (no print, no globals) | Cleaning, scoring, chunking: easy to test. |
| **Config factory (closure)** | `def make_x(setting): def x(value): ...; return x` | Build a function once with settings, reuse it. |
| **Decorator** | `def deco(fn): @wraps(fn) def wrapper(*a, **k): ...; return wrapper` | Timing, logging, retry, auth. |
| **Decorator with argument** | three layers: `outer(arg)` → `decorator(fn)` → `wrapper(...)` | `@retry(3)`. |
| **Registry decorator** | store `fn` in a dict and `return fn` | Routes, tool lists, plugins. |
| **Cache** | `@lru_cache(maxsize=...)` on a pure function with hashable arguments | Embeddings, settings, slow lookups. |
| **Sort or filter by rule** | `sorted(items, key=...)`, `filter(fn, items)` | Top-K, thresholds. |
| **Recursion** | base case, then a smaller call | Splitters, nested data. |

**Config factory** (a closure, verified):
```python
def make_cleaner(lower=True, min_len=0):
    def clean(text):
        text = " ".join(text.split())
        if lower:
            text = text.lower()
        return text if len(text) >= min_len else None
    return clean

clean = make_cleaner(min_len=3)
clean("  Hello   World ")      # 'hello world'
clean("a")                     # None   too short
```
**Retry decorator** (a decorator with an argument, verified):
```python
from functools import wraps

def retry(times):
    def decorator(fn):
        @wraps(fn)
        def wrapper(*args, **kwargs):
            for attempt in range(1, times + 1):
                try:
                    return fn(*args, **kwargs)
                except Exception as e:
                    print("attempt", attempt, "failed:", e)
                    if attempt == times:
                        raise              # out of attempts: let the error out
        return wrapper
    return decorator

calls = {"n": 0}

@retry(3)
def flaky():
    calls["n"] += 1
    if calls["n"] < 3:
        raise ValueError("boom")
    return "ok"

print(flaky())
# attempt 1 failed: boom
# attempt 2 failed: boom
# ok
```
(`try` / `except` is explained in Topic 08. For now: `try` runs the code, `except` runs if it raised an error.)

**Registry decorator** (this is how `@app.get("/tickets")` works inside FastAPI, simplified, verified):
```python
routes = {}

def route(path):
    def decorator(fn):
        routes[path] = fn      # store the function under its path
        return fn              # give the same function back
    return decorator

@route("/tickets")
def list_tickets():
    return ["t1", "t2"]

@route("/health")
def health():
    return "ok"

print(sorted(routes))          # ['/health', '/tickets']
print(routes["/tickets"]())    # ['t1', 't2']
```

---

## 11. Use this when building (project guide)

- **FastAPI:** `@app.get(...)` is a registry decorator. Route functions are plain functions with parameters, so the parameter kinds (defaults, keyword arguments) decide the URL options.
- **RAG pipeline:** write each step as a small pure function (`clean`, `split`, `embed`, `score`). Use `lru_cache` on `embed`, `partial` or a config factory to fix chunk size, and `sorted(..., key=...)` for top-K.
- **Pandas:** `df["col"].apply(clean)` and `.map(fn)` take a function. Your `clean` from Topic 03 plugs straight in.
- **Pytest later:** pure functions are the easiest to test. Fixtures and parametrization are built on decorators.
- **Production code:** timing, logging, retry and auth are decorators, so you write them once.

**Checklist:** does the function **return** (not print) its result? Are mutable defaults avoided? Can I make the core logic pure? Am I looping and building lambdas (late-binding trap)? Do I need `@wraps`?

---

## 🧠 Quick Revision

- A function is an **object**. `f` is the function, `f()` calls it. No `return` → `None`. `print` shows, `return` hands back.
- Parameters: positional, default, `*args` (tuple), keyword-only, `**kwargs` (dict). Stars in a **call** unpack.
- **LEGB** lookup. Assigning inside a function makes the name local, so `count += 1` needs `nonlocal` (enclosing function) or `global` (module). Reading needs nothing.
- **Closure** = inner function + remembered outer variables. `return inner` (the function), not `inner()`.
- **Decorator**: `@d` above `def f` means `f = d(f)`. The decorator runs once; the wrapper runs on every call. Wrapper pattern: `*args, **kwargs`, call `fn`, `return result`, add `@wraps(fn)`.
- **Lambda** in a loop captures the **variable**, not the value: use `lambda i=i: i`.
- `map` transforms, `filter` keeps, `reduce` combines (give an initializer). `map`/`filter` are lazy and one-use: wrap in `list()`. Prefer comprehensions and built-ins (`sum`, `max`, `"".join`).
- `key=` is called once per item; `sorted(x, key=..., reverse=True)[:k]` gives top-K.
- `partial(f, size=100)` pre-fills arguments. `@lru_cache` remembers results: hashable arguments, pure functions only.
- Recursion needs a base case and a smaller problem each call.
- Pure function = same input, same output, no side effects.
- Keyword-only: put `*` before the name. Positional-only: put `/` after the name. `callable(x)` tests if you can call it.