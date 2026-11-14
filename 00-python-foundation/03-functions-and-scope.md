# Functions and Scope

## 1. Definition
A **function** is a reusable block of code that takes inputs (parameters), does work, and gives back an output (return value). In Python a function is also an **object**: you can store it in a variable, pass it to another function, and return it from a function.

**Scope** is the region of code where a name (variable) can be seen and used.

## 2. Why it exists
- **Reuse:** write once, call many times.
- **Abstraction:** hide the details behind a name (`total(...)`).
- **Because functions are objects**, Python can support closures and decorators, which FastAPI, pytest and most libraries rely on.
- **Scope** keeps names from clashing: a variable inside one function does not accidentally overwrite one in another.

## 3. Mental model

### 3.1 A function is a recipe card with a label
- `def greet(name): ...` creates the recipe card and sticks the label `greet` on it.
- `greet("Ann")` (with parentheses) means **cook it now**.
- `greet` (without parentheses) means **the card itself**. You can hand the card to someone else, or stick a second label on it.

```python
def greet(name):
    return f"Hi {name}"

say = greet            # second label on the SAME card, nothing runs yet
print(say("Ann"))      # Hi Ann
```

### 3.2 Scope is a set of nested rooms (LEGB)
When Python meets a name, it searches rooms from the inside out and stops at the first match:

| Letter | Room | Meaning |
|---|---|---|
| **L** | Local | Names created inside the current function. |
| **E** | Enclosing | Names in any outer function that contains the current one. |
| **G** | Global | Names at the top level of the file or notebook. |
| **B** | Built-in | Names Python provides, like `len`, `print`, `sum`. |

### 3.3 A closure is a function with a backpack
When an inner function is created inside an outer function, it carries a **backpack** containing the outer variables it uses. Even after the outer function has finished, the inner function still has its backpack.

### 3.4 A decorator is a gift-wrapper
A decorator takes a function, wraps it in extra behavior (timing, logging, login check), and hands back the wrapped version. The caller uses the same name, but now gets the extra behavior.

## 4. How it works internally

### 4.1 `def` runs at definition time, the body runs at call time
```python
def f():
    print("running")   # NOT printed when `def` executes
f()                    # printed here
```
`def` creates a function object and binds the name. Nothing inside runs until the call.

### 4.2 Parameters and arguments
- **Parameter:** the name in the `def` line. **Argument:** the actual value you pass in the call.

| Kind | Example | Meaning |
|---|---|---|
| Positional | `def f(a, b)` | Matched by order: `f(1, 2)`. |
| Keyword | `f(b=2, a=1)` | Matched by name, so order does not matter. |
| Default | `def f(a, b=10)` | `b` is optional; the default is used if it's missing. |
| `*args` | `def f(*args)` | Collects extra **positional** arguments into a **tuple**. |
| `**kwargs` | `def f(**kwargs)` | Collects extra **keyword** arguments into a **dict**. |
| Keyword-only | `def f(*, key)` or `def f(*args, key)` | Everything after `*` or `*args` must be passed by name. |
| Positional-only | `def f(a, /, b)` | Everything before `/` must be passed by position. |

Order in a `def` line: positional → defaults → `*args` → keyword-only → `**kwargs`.

```python
def show(a, b=10, *args, key, **kwargs):
    print(a, b, args, key, kwargs)

show(1, 2, 3, 4, key="k", x=9)
# a=1, b=2, args=(3, 4), key="k", kwargs={"x": 9}
```

The stars also work in a **call**, where they **unpack**:
```python
nums = [1, 2]
opts = {"key": "k"}
show(*nums, **opts)     # same as show(1, 2, key="k")
```
- `*` spreads a list/tuple into separate positional arguments.
- `**` spreads a dict into separate keyword arguments.

### 4.3 Return values
- `return x` sends `x` back and **ends** the function immediately.
- No `return` (or a bare `return`) gives back `None`.
- `return a, b` actually returns one **tuple** `(a, b)`, which you can unpack: `x, y = f()`.

### 4.4 Arguments are passed by object reference
The function's parameter becomes a **second label on the same object** (the Topic 1 idea):
```python
def add_one(lst):
    lst.append(1)     # mutates the caller's list
def rebind(lst):
    lst = [99]        # only moves the local label; caller unaffected
```
Mutating a mutable argument changes the caller's object. Re-assigning the parameter does not.

### 4.5 Scope rules in detail
**Reading** an outer name works automatically. **Assigning** to a name inside a function creates a **new local** name, unless you declare otherwise.

```python
count = 0
def bad():
    count += 1          # UnboundLocalError
```
Python sees `count += 1` as an assignment, so it decides `count` is local to `bad`. But the local `count` has no value yet when `+=` tries to read it, hence `UnboundLocalError`.

Two fixes:
```python
def good_global():
    global count        # "use the module-level count"
    count += 1

def make_counter():
    count = 0
    def inc():
        nonlocal count  # "use the count from the enclosing function"
        count += 1
        return count
    return inc
```

| Keyword | Refers to | Use |
|---|---|---|
| `global x` | A name at the top level of the module. | Rarely; global state is usually a design smell. |
| `nonlocal x` | A name in the nearest enclosing **function**. | Closures that must update their own state. |

#### Extra clarity: how Python decides a name is local
Python decides **before running the function** (when it compiles it). The rule: **if a name is assigned anywhere inside the function, that name is local to the whole function.**
```python
x = 10

def read_only():
    print(x)        # 10. Only reading, so Python finds the global x.

def assign_only():
    x = 20          # creates a NEW local x; the global x is still 10
    print(x)        # 20

def read_then_assign():
    print(x)        # UnboundLocalError
    x = 20
```
Why `read_then_assign` fails: because of `x = 20` further down, Python marks `x` as local for the **entire** function. So `print(x)` looks for the **local** `x`, which has no value yet. It does **not** fall back to the global `x`. In Python 3.13 the message reads `UnboundLocalError: cannot access local variable 'x' where it is not associated with a value` (older versions say "local variable referenced before assignment").

What counts as an assignment (all of these make a name local): `x = ...`, `x += ...`, `for x in ...`, `import x`, `def x`, `with ... as x`.
`count += 1` secretly means `count = count + 1`, which is why it counts as an assignment.

| Situation | What happens |
|---|---|
| Only **reading** an outer name | Works automatically; Python searches outward (LEGB). |
| **Assigning** with no declaration | Creates a new local name; the outer name is untouched. |
| Reading then assigning the same name | `UnboundLocalError`, because the name is local but has no value yet. |
| Assigning with `global` or `nonlocal` declared | Changes the outer name as you intended. |

### 4.6 Closures step by step
```python
def make_counter():
    count = 0
    def inc():
        nonlocal count
        count += 1
        return count
    return inc          # returns the function, NOT inc()

c = make_counter()      # make_counter finishes, but inc keeps `count` alive
print(c(), c(), c())    # 1 2 3
d = make_counter()      # a brand new call, so a brand new `count`
print(d())              # 1
```
Trace:
1. `make_counter()` runs: creates `count = 0` and the function `inc`.
2. It returns `inc` (the function object). The variable `count` is **not** destroyed, because `inc` still uses it.
3. `c` now points to `inc`. Each `c()` updates that same `count`: 1, 2, 3.
4. `d = make_counter()` runs the outer function again, creating a **separate** `count`. So `d()` starts at 1 and `c`'s count is unaffected.

Why `return inc` and not `return inc()`:
- `inc` is the function itself (what we want to hand back).
- `inc()` would call it right now and return the number `1`, and `c` would just be an int.

#### Extra clarity: why `nonlocal` is needed, and what the closure remembers
**Experiment 1: remove `nonlocal`.**
```python
def make_counter():
    count = 0
    def inc():
        count += 1          # no nonlocal
        return count
    return inc

c = make_counter()
c()    # UnboundLocalError: cannot access local variable 'count' ...
```
This is the same rule as 4.5: `count += 1` is an assignment, so `count` becomes a **local** variable of `inc`, and it has no value yet. `nonlocal count` tells Python "this name belongs to the enclosing function; do not create a new local".

**When you do NOT need `nonlocal`:** if the inner function only **reads** the outer variable.
```python
def make_multiplier(n):
    def multiply(x):
        return x * n        # only reading n, so no nonlocal needed
    return multiply

double = make_multiplier(2)
double(5)                   # 10
```
Rule: **read-only → nothing needed. Re-assigning → `nonlocal`.**

**Experiment 2: look inside the "backpack".** Every closure stores its remembered variables in "cells".
```python
c = make_counter()                 # the version WITH nonlocal
c.__code__.co_freevars             # ('count',)   names the function borrows from outside
c.__closure__[0].cell_contents     # 0   the current value of count
c(); c()
c.__closure__[0].cell_contents     # 2
```
This shows that `count` is not a copy: the inner function and the cell share the **same** variable, and every call updates it.

**Experiment 3: `return inc` vs `return inc()`.**

| Written as | What `c` becomes | What `c()` does |
|---|---|---|
| `return inc` | The inner **function** itself. | Runs it and returns `1`, then `2`, then `3`. |
| `return inc()` | The **number** `1` (the result of calling `inc` once, right now). | `TypeError: 'int' object is not callable`, because `c` is just an int. |

Rule of thumb: **parentheses mean "run it now"; no parentheses mean "hand the function over to be run later".**

### 4.7 Decorators step by step
```python
import time
from functools import wraps

def timed(fn):                              # fn = the original function
    @wraps(fn)
    def wrapper(*args, **kwargs):           # accepts anything
        start = time.perf_counter()
        result = fn(*args, **kwargs)        # run the original
        end = time.perf_counter()
        print(f"{fn.__name__} took {end - start:.4f}s")
        return result                       # give back the original's result
    return wrapper

@timed
def big_sum():
    return sum(range(1_000_000))

print(big_sum())
```
**What `@timed` really does.** This:
```python
@timed
def big_sum(): ...
```
is exactly the same as:
```python
def big_sum(): ...
big_sum = timed(big_sum)
```
Trace:
1. Python defines the original `big_sum`.
2. It calls `timed(big_sum)`. Inside, `fn` is the original `big_sum`.
3. `timed` returns `wrapper`, and the name `big_sum` is now bound to `wrapper`.
4. When you call `big_sum()`, you are really calling `wrapper()`, which starts the timer, calls the original via `fn(...)`, stops the timer, prints, and returns the original's result.
5. `wrapper` is a **closure**: it remembers `fn` in its backpack. This is why closures come before decorators.

Pieces explained:
- `*args, **kwargs` in `wrapper` lets it accept **any** arguments and pass them straight through to `fn`, so the decorator works on any function.
- `fn(*args, **kwargs)` calls the original (the stars unpack the collected arguments again).
- `time.perf_counter()` returns a high-precision clock reading in seconds. Subtract two readings to get elapsed time. (Use it instead of `time.time()` for timing code.)
- `functools.wraps(fn)` copies `fn`'s name and docstring onto `wrapper`. Without it, `big_sum.__name__` would say `"wrapper"`, which confuses debugging and tools such as FastAPI and pytest.
- `return result` is essential. If you forget it, the decorated function returns `None`.

**A decorator that takes arguments** needs one more layer:
```python
def repeat(n):                      # outer: receives the decorator's argument
    def decorator(fn):              # middle: receives the function
        @wraps(fn)
        def wrapper(*args, **kwargs):   # inner: runs on each call
            for _ in range(n):
                result = fn(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def hello(): print("hi")
```
`@repeat(3)` first calls `repeat(3)`, which returns `decorator`, and then `decorator` is applied to `hello`.

#### Extra clarity: when does each line of a decorator run?
Add print statements to see the order:
```python
def timed(fn):
    print("1. timed() is running, decorating", fn.__name__)
    def wrapper(*args, **kwargs):
        print("3. wrapper is running")
        result = fn(*args, **kwargs)
        print("4. original finished, returning its result")
        return result
    print("2. timed() finished, returning wrapper")
    return wrapper

@timed
def greet():
    return "hi"

print("--- decorated, now calling ---")
print(greet())
```
Output:
```
1. timed() is running, decorating greet
2. timed() finished, returning wrapper
--- decorated, now calling ---
3. wrapper is running
4. original finished, returning its result
hi
```
- Steps **1 and 2 run once**, at the moment the function is defined (that is when `@timed` is applied).
- Steps **3 and 4 run on every call** to `greet()`.

**What `fn(*args, **kwargs)` means.** The wrapper is defined as `wrapper(*args, **kwargs)`, which **packs** whatever arguments it receives into a tuple and a dict. Then `fn(*args, **kwargs)` **unpacks** them again and passes them to the original function.
```python
def greet2(name, punct="!"):
    return f"hi {name}{punct}"

# decorated greet2, called as: greet2("Ann", punct="?")
# wrapper receives  args = ("Ann",)   kwargs = {"punct": "?"}
# fn(*args, **kwargs) becomes  greet2("Ann", punct="?")
```

**Why we store `result` in a variable.** We have to call the original, then stop the clock, then return. A `return` ends the function immediately, so we first save the original's return value in `result`, do our extra work (print the time), and only then `return result`.

**What breaks if you leave out each part:**

| If you forget this | What happens |
|---|---|
| `return result` | The decorated function always returns `None`. |
| `*args, **kwargs` in the wrapper | `TypeError: wrapper() takes 0 positional arguments but 1 was given` as soon as the function has arguments. |
| The call `fn(...)` | The original function never runs. |
| `return wrapper` (written as `return wrapper()` instead) | The wrapper runs at decoration time and breaks. Parentheses mean "run now". |
| `@wraps(fn)` | Everything still works, but `greet.__name__` says `"wrapper"` and docstrings are lost. |

**The fill-in-the-blank version, solved:** in `result = ___(*args, **kwargs)` the blank is **`fn`** (the original function), and in `return ___` the blank is **`result`**.

### 4.8 Lambda
`lambda x: x * 2` is a small **anonymous function** limited to **one expression** (no statements, no assignments). It returns that expression's value automatically.
```python
double = lambda x: x * 2
sorted(["bb", "a", "ccc"], key=lambda s: len(s))   # ['a', 'bb', 'ccc']
```
Use lambdas for short throwaway functions, mainly as `key=` arguments. For anything longer, use `def`.

### 4.9 The late-binding closure trap
```python
fs = [lambda: i for i in range(3)]
print([f() for f in fs])     # [2, 2, 2]  (not [0, 1, 2])
```
- Each lambda does not store the **value** of `i`. It stores a link to the **variable** `i`, and looks it up **when it is called**.
- By the time you call them, the loop has finished and `i` is `2`, so all three return `2`.

**Fix:** use a default argument. Defaults are evaluated **when the lambda is created**, so each lambda keeps its own copy:
```python
fs = [lambda i=i: i for i in range(3)]
print([f() for f in fs])     # [0, 1, 2]
```
`i=i` means: the parameter named `i` gets its default from the current loop variable `i` at creation time.

#### Extra clarity: proving that all the lambdas share one variable
```python
fs = [lambda: i for i in range(3)]
fs[0].__closure__[0].cell_contents                 # 2   (the final value of i)
fs[0].__closure__[0] is fs[1].__closure__[0]       # True: all lambdas share the SAME cell
```
All three lambdas hold the same cell, and that cell holds the **latest** value of `i`. So whichever lambda you call, you get the latest value.

Three ways to fix it:

| Fix | Code | How it works |
|---|---|---|
| Default argument | `lambda i=i: i` | The default is evaluated when the lambda is created, so each lambda stores its own copy of the current `i`. |
| Factory function | `def make(i): return lambda: i` then `[make(i) for i in range(3)]` | Each call to `make` creates a new scope with its own `i`, so each lambda closes over a different variable. |
| `functools.partial` | `partial(show, i)` | `partial` stores the current value as a fixed argument at creation time. |

The same trap applies to a normal `def` inside a loop, not only to lambdas. The rule: **a closure remembers the variable, not the value it had at that moment.**

### 4.10 Docstrings
A **docstring** is a string written as the very first statement of a function. It documents what the function does.
```python
def area(r):
    """Return the area of a circle with radius r."""
    return 3.14159 * r * r

print(area.__doc__)     # Return the area of a circle with radius r.
help(area)              # prints the signature and the docstring
```
- Python stores it in the function's `__doc__` attribute.
- Convention: a one-line summary in triple quotes; for longer functions add a blank line, then details such as parameters and return value.
- Tools read it: `help()`, IDE hover text, and **FastAPI**, which shows it in the auto-generated API docs.

### 4.11 Recursion
A **recursive** function calls itself. Every recursive function needs:
1. A **base case**: the condition where it stops and returns directly.
2. A **recursive case**: a call to itself with a **smaller** problem that moves toward the base case.

```python
def fact(n):
    if n <= 1:              # base case
        return 1
    return n * fact(n - 1)  # recursive case

print(fact(5))              # 120
```
Trace of `fact(3)`:
```
fact(3) -> 3 * fact(2)
              fact(2) -> 2 * fact(1)
                           fact(1) -> 1          (base case)
              fact(2) = 2 * 1 = 2
fact(3) = 3 * 2 = 6
```
- Each call gets its **own local scope** and sits on the **call stack** until the call below it returns.
- Without a base case (or with a wrong one) the calls never stop, and Python raises `RecursionError` once the stack gets too deep. The default limit is about 1000 calls (`sys.getrecursionlimit()`).
- Python does **not** optimise tail calls, so very deep recursion should be rewritten as a loop.
- Use recursion for naturally nested data (trees, folders, nested JSON). Use loops for simple repetition.

### 4.12 `map`, `filter`, `reduce` and `key=` (in detail)
These three tools process a sequence **with a function instead of a loop**. They are **higher-order functions**: they receive another function as an argument.

| Tool | Its one job | Result size |
|---|---|---|
| `map` | **Transform** every item. | Same number of items as the input. |
| `filter` | **Keep** only the items that pass a test. | Same or fewer items. |
| `reduce` | **Combine** all items into one value. | A single value. |

#### `map(function, iterable, *more_iterables)`
Calls `function` on every item and gives back the results.
```python
nums = [1, 2, 3, 4]
result = map(lambda x: x * 2, nums)
print(result)          # <map object at 0x...>   NOT a list
print(list(result))    # [2, 4, 6, 8]
print(list(result))    # []   the map object is already used up
```
How it works:
- In Python 3, `map` returns a **lazy iterator**: it computes **one item at a time, only when you ask for the next one** (by looping or calling `list()`). Nothing is calculated when you write `map(...)`.
- An iterator can be consumed **only once**. After you loop over it, it is **exhausted** (empty). To use the results again, store them with `list(...)`.
- Laziness saves memory for big inputs, because the full result list never exists at once.

Passing a named function is the most common use:
```python
list(map(int, ["1", "2", "3"]))        # [1, 2, 3]  convert strings to ints
list(map(str.upper, ["a", "b"]))       # ['A', 'B']
list(map(len, ["hi", "hello"]))        # [2, 5]
```
`map` with two (or more) iterables passes one item from each, and stops at the shortest (like `zip`):
```python
list(map(lambda a, b: a + b, [1, 2, 3], [10, 20]))   # [11, 22]
```
Same result with a comprehension: `[x * 2 for x in nums]`.

#### `filter(function, iterable)`
Keeps an item only if `function(item)` is **truthy**.
```python
nums = [1, 2, 3, 4]
list(filter(lambda x: x % 2 == 0, nums))      # [2, 4]
list(filter(None, [0, 1, "", None, [], "a"])) # [1, 'a']   None as the function keeps only truthy items
```
- `filter` is also a **lazy iterator** and is exhausted after one pass.
- The function is a **test**: it should answer yes or no. It does not change the items. (To change items, use `map`.)
- Truthy and falsy: `0`, `""`, `None`, `[]`, `{}` and `False` are falsy; most other values are truthy.
- Same result with a comprehension: `[x for x in nums if x % 2 == 0]`.

#### `reduce(function, iterable, initializer)`
`reduce` lives in `functools` (it is **not** a built-in in Python 3), so you must import it: `from functools import reduce`.

It repeatedly combines two things: the **accumulator** (the running result so far) and the **next item**. The function takes exactly **two arguments** and returns the new accumulator.
```python
from functools import reduce
reduce(lambda acc, x: acc + x, [1, 2, 3, 4])     # 10
```
Step by step (no initializer, so the first item becomes the starting accumulator):
```
acc = 1                        (first item)
acc = 1 + 2 = 3                (next item 2)
acc = 3 + 3 = 6                (next item 3)
acc = 6 + 4 = 10               (next item 4)   -> result 10
```
With an **initializer** (a starting value), the accumulator starts from it:
```python
reduce(lambda acc, x: acc + x, [1, 2, 3, 4], 100)   # 110
```
```
acc = 100
acc = 100 + 1 = 101 ... acc = 110
```
Empty input:
- **Without** an initializer, `reduce(f, [])` raises `TypeError: reduce() of empty iterable with no initial value`.
- **With** an initializer, it simply returns the initializer. So use an initializer whenever the input could be empty.

More examples:
```python
reduce(lambda a, b: a * b, [1, 2, 3, 4])               # 24   product
reduce(lambda a, b: a if a > b else b, [3, 9, 2])      # 9    maximum
reduce(lambda a, b: a + b, [[1], [2, 3]], [])          # [1, 2, 3]  flatten (works but slow for big data)
```
**Prefer built-ins when they exist.** They are clearer and faster:

| Instead of this `reduce` | Use this |
|---|---|
| `reduce(lambda a, b: a + b, nums)` | `sum(nums)` |
| `reduce(lambda a, b: a * b, nums)` | `math.prod(nums)` |
| `reduce(lambda a, b: a if a > b else b, nums)` | `max(nums)` (or `min(nums)` for the smallest) |
| `reduce(lambda a, b: a and b, flags)` | `all(flags)` |
| `reduce(lambda a, b: a or b, flags)` | `any(flags)` |
| `reduce(lambda a, b: a + b, list_of_strings)` | `"".join(list_of_strings)` |

Use `reduce` only when the combining rule is custom and no built-in matches.

#### Chaining the three
Sum of the squares of the even numbers from 1 to 10:
```python
from functools import reduce
nums = range(1, 11)
evens   = filter(lambda x: x % 2 == 0, nums)       # 2, 4, 6, 8, 10
squares = map(lambda x: x * x, evens)              # 4, 16, 36, 64, 100
total   = reduce(lambda a, b: a + b, squares)      # 220
```
The same job in the usual Python style: `sum(x * x for x in range(1, 11) if x % 2 == 0)`, which gives `220`.

#### `map` / `filter` vs list comprehension
| `map` / `filter` | List comprehension |
|---|---|
| `map` and `filter` return lazy iterators, so nothing is computed until you loop over them or call `list()`. | A list comprehension builds the whole list in memory immediately, while a generator expression with round brackets `(...)` is lazy. |
| They need a function, and with a lambda the logic becomes hard to read once it is more than trivial. | The expression and the condition are written inline, which is usually easier to read. |
| They are neat when a named function already exists, such as `map(int, strings)`. | They are neat when the logic is new, because you do not need to create a function first. |
| A `map` or `filter` object can be consumed only once and is then empty. | A list can be looped over any number of times. |

#### The `operator` module: functions for operators
Writing `lambda a, b: a + b` is wasteful. The standard-library module `operator` provides ready-made functions for operators.
```python
import operator
from functools import reduce

reduce(operator.add, [1, 2, 3])        # 6    same as a + b
reduce(operator.mul, [1, 2, 3, 4])     # 24   same as a * b

from operator import itemgetter, attrgetter
people = [("bob", 30), ("ann", 25)]
sorted(people, key=itemgetter(1))      # sort by index 1 (age): [('ann', 25), ('bob', 30)]
# itemgetter("score") works on dicts too; attrgetter("age") works on object attributes
```

#### `key=` functions (`sorted`, `min`, `max`)
`key` is a function that is called **once per item** to produce a **sort value**. The items are compared by their key values, but the **original items** are returned.
```python
words = ["bb", "a", "ccc"]
sorted(words, key=len)                      # ['a', 'bb', 'ccc']
max(words, key=len)                         # 'ccc'
sorted(words, key=len, reverse=True)        # ['ccc', 'bb', 'a']   biggest first
results = [{"id": 1, "score": 0.4}, {"id": 2, "score": 0.9}]
sorted(results, key=lambda r: r["score"], reverse=True)   # best score first
sorted(people, key=lambda p: (p[1], p[0]))  # sort by age, ties broken by name (a tuple as the key)
```
Sorting is **stable**: items with equal keys keep their original order.

#### Mistakes to avoid with `map` / `filter` / `reduce`
- Printing a `map` or `filter` object and expecting items. Wrap it in `list(...)`.
- Using the same `map` object twice. The second use is empty.
- Calling `map(print, items)` for its side effect: nothing prints until something consumes it. Use a normal `for` loop for side effects.
- Forgetting `from functools import reduce`.
- Calling `reduce` on a possibly empty list without an initializer.
- A `reduce` function that does not take exactly two arguments.
- Giving `filter` a function that returns a transformed value instead of a yes/no. `filter(lambda x: x * 2, [0, 1, 2])` keeps `[1, 2]` because it tests whether `x * 2` is truthy; it does not double anything.

### 4.13 `functools.partial`
`partial` **fixes some arguments** of an existing function and gives you a new function that needs only the rest.
```python
from functools import partial

def power(base, exp):
    return base ** exp

square = partial(power, exp=2)     # exp is now fixed at 2
cube   = partial(power, exp=3)
print(square(5), cube(5))          # 25 125
```
Use it to adapt a function to an interface that wants fewer arguments (callbacks, `map`, `sorted(key=...)`), instead of writing a small wrapper or lambda.

### 4.14 `functools.lru_cache` (memoization)
**Memoization** means remembering the result of a call so that repeating the same call returns instantly. `lru_cache` does this with a decorator. (LRU = "least recently used": when the cache is full it drops the entry used longest ago.)
```python
from functools import lru_cache

@lru_cache(maxsize=None)           # maxsize=None means unlimited
def fib(n):
    return n if n < 2 else fib(n - 1) + fib(n - 2)

print(fib(35))                     # instant
print(fib.cache_info())            # hits, misses, size
fib.cache_clear()                  # empty the cache
```
- Without the cache, `fib(n)` recomputes the same values again and again, which takes **exponential** time (about 2^n calls). With the cache each value is computed once, so it takes **linear** time (about n calls).
- Internally the cache is a dict keyed by the call's arguments, so **arguments must be hashable** (Topic 2). Lists and dicts as arguments raise `TypeError`.
- Python 3.9+ also has `functools.cache`, a shortcut for `lru_cache(maxsize=None)`.
- Only cache **pure** functions (see 4.15). Caching a function that reads changing data (the time, a database) returns stale results. Unlimited caches can also use a lot of memory.

### 4.15 Pure and impure functions
- A **pure function** depends only on its arguments and has **no side effects**: same input always gives the same output, and it changes nothing outside itself.
- A **side effect** is anything a function does besides returning a value: printing, writing a file, modifying a global, changing a mutable argument, calling an API.
```python
def add(a, b):            # pure
    return a + b

total = 0
def add_to_total(x):      # impure: changes a global
    global total
    total += x

def append_one(lst):      # impure: mutates its argument
    lst.append(1)
```
Pure functions are easy to **test** (no setup), easy to **cache**, and safe to reuse. Aim to keep the core logic pure and push side effects (I/O, printing, database calls) to the edges of your program.

### 4.16 Inspecting functions and placeholders
```python
callable(len)               # True: can it be called?
add.__name__                # 'add'
add.__doc__                 # its docstring
import inspect
inspect.signature(add)      # <Signature (a, b)>

def todo():
    pass                    # does nothing; a placeholder so the body is not empty
def todo2():
    ...                     # Ellipsis; also commonly used as a placeholder
```
`inspect.signature` is how frameworks such as FastAPI read your function's parameters to build routes and validation.

### 4.17 Function topics covered in other notes
| Feature | Where it is covered |
|---|---|
| Type hints on parameters and returns (`def f(x: int) -> str`) | Topic 8: type hints, dataclasses, Pydantic. |
| Generator functions (`yield`) | Topic 6: iterators and generators. |
| `__call__` (making an object callable like a function) | Topic 5 and 6: OOP and magic methods. |
| Raising and handling errors inside functions | Topic 7: error handling. |
| `async def` functions | Phase 3: async Python and FastAPI. |

## 5. Important terminology
| Term | Simple meaning |
|---|---|
| Parameter | The name in the `def` line (`def f(a)`: `a`). |
| Argument | The actual value passed when calling (`f(5)`: `5`). |
| First-class function | A function that can be stored, passed and returned like any value. |
| Higher-order function | A function that takes or returns another function (`sorted(key=...)`, decorators). |
| `*args` | Extra positional arguments collected into a tuple. |
| `**kwargs` | Extra keyword arguments collected into a dict. |
| Keyword-only argument | Must be passed by name; comes after `*` or `*args`. |
| Scope | The region where a name is visible. |
| LEGB | Lookup order: Local, Enclosing, Global, Built-in. |
| Closure | An inner function that remembers variables of its enclosing function. |
| `nonlocal` | Lets an inner function re-assign an enclosing function's variable. |
| `global` | Lets a function re-assign a module-level variable. |
| Decorator | A function that takes a function and returns a wrapped replacement. |
| Lambda | A one-expression anonymous function. |
| Late binding | A closure looks up a variable when called, not when created. |
| `functools.wraps` | Copies the original function's metadata onto a wrapper. |
| `perf_counter` | A high-precision clock for timing code. |
| Docstring | A string as the first statement of a function that documents it; stored in `__doc__`. |
| Recursion | A function calling itself on a smaller problem. |
| Base case | The condition where a recursive function stops and returns directly. |
| Call stack | The pile of active function calls; each call sits on it until it returns. |
| Higher-order function | A function that takes or returns functions (`map`, `filter`, `sorted(key=)`, decorators). |
| `map` / `filter` / `reduce` | Apply a function to every item / keep matching items / combine all items into one. |
| `partial` | Makes a new function with some arguments of another function pre-filled. |
| Memoization | Remembering a function's results so repeated calls return instantly. |
| `lru_cache` | A decorator that memoizes; drops the least recently used entries when full. |
| Pure function | Output depends only on inputs, with no side effects. |
| Side effect | Anything a function does besides returning a value (printing, I/O, changing outside state). |
| Iterator | An object that hands out items one at a time with `next()`. `map` and `filter` return iterators. |
| Lazy evaluation | Computing a value only when it is asked for, not in advance. |
| Exhausted | An iterator that has already given all its items and is now empty. |
| Accumulator | The running result that `reduce` carries from step to step. |
| Initializer | The optional starting value for `reduce`'s accumulator. |
| Truthy / falsy | A value that counts as true / false in a condition (`0`, `""`, `None`, `[]` are falsy). |
| Cell | The storage slot where a closure keeps a remembered variable (`f.__closure__`). |
| `operator` module | Standard-library functions for operators (`operator.add`, `itemgetter`) used instead of lambdas. |

## 6. Example (small)
```python
def total(*nums, **opts):
    round_to = opts.get("round_to")       # None if the key is missing
    if round_to is not None:
        return round(sum(nums), round_to)
    return sum(nums)

total(1.234, 2.345, round_to=2)   # 3.58
total(1, 2, 3)                    # 6
total()                           # 0 (sum of an empty tuple)
```
`nums` is a tuple `(1.234, 2.345)` and `opts` is a dict `{"round_to": 2}`. `.get()` avoids a `KeyError` when the option is absent.

## 7. Code example (full)
```python
import time
from functools import wraps

# 1. Closure: a counter that remembers its own state
def make_counter():
    count = 0
    def inc():
        nonlocal count
        count += 1
        return count
    return inc

c = make_counter()
print(c(), c(), c())      # 1 2 3
print(make_counter()())   # 1  (a fresh, separate counter)

# 2. Lambda trap and fix
bad  = [lambda: i for i in range(3)]
good = [lambda i=i: i for i in range(3)]
print([f() for f in bad])    # [2, 2, 2]
print([f() for f in good])   # [0, 1, 2]

# 3. Decorator
def timed(fn):
    @wraps(fn)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = fn(*args, **kwargs)
        print(f"{fn.__name__} took {time.perf_counter() - start:.4f}s")
        return result
    return wrapper

@timed
def big_sum():
    return sum(range(1_000_000))

print(big_sum())
```

## 8. Real-world usage (and how it connects to Track 2)

**FastAPI routes are decorators.** `@app.get("/hello")` stores your function in a table of routes. The core idea in a few lines:
```python
routes = {}

def get(path):                   # a decorator that takes an argument
    def decorator(fn):
        routes[path] = fn        # register the function under a URL
        return fn
    return decorator

@get("/hello")
def hello():
    """Say hello."""
    return {"msg": "hi"}

routes["/hello"]()               # {'msg': 'hi'}
```
FastAPI does the same, and also uses `inspect.signature` to read your parameters (to validate input) and your docstring (to write the API docs).

**Dependency injection passes functions around.** In `Depends(get_db)`, you give FastAPI the function `get_db`, and FastAPI calls it for you on every request.

**Caching with `@lru_cache`.** Two very common uses:
```python
from functools import lru_cache

@lru_cache
def get_settings():              # load configuration once, reuse everywhere
    return load_config()

@lru_cache(maxsize=1000)
def embed(text):                 # str arguments are hashable, so they can be cached
    return call_embedding_model(text)
```
Caching embeddings of repeated text saves time and API cost. Only do this for a function whose result depends only on its input.

**Closures build configured functions.** A relevance filter for RAG search results:
```python
def make_score_filter(min_score):
    def keep(result):
        return result["score"] >= min_score
    return keep

good = list(filter(make_score_filter(0.8), results))
```
**Sorting with `key=` gives top-K results:** `sorted(results, key=lambda r: r["score"], reverse=True)[:5]`.

**Recursion.** Walking nested JSON or folders, and the **recursive text splitter** used in RAG chunking: try to split by paragraphs; if a piece is still too big, split it by sentences; then by words, until every piece fits the chunk size. The base case is "this piece already fits".

**`map`, `filter`, `reduce` and their cousins:**
- Pandas has `Series.map(func)` and `.apply(func)` (transform each value), boolean masks like `df[df["amount"] > 100]` (a vectorised `filter`), and `groupby(...).agg(...)` (a `reduce` per group).
- In RAG code: embed each chunk (`map`), drop results below a similarity threshold (`filter`), and merge many dictionaries of metadata into one (`reduce`).

**`*args` and `**kwargs`.** Decorators forward them. Pydantic models are built from dictionaries with `Model(**data)`. `requests.get(url, **options)` forwards options.

**Decorators for timing and logging.** Measure how long an embedding call or an API request takes; log calls to find slow paths. `@pytest.fixture` and `@pytest.mark.parametrize` are decorators too.

**Pure functions.** Data-cleaning steps in a pipeline written as pure functions are trivial to test with pytest. Keep printing, file writing and database calls at the edges.

## 9. Common mistakes
- Forgetting `nonlocal`/`global` when re-assigning an outer variable, which causes `UnboundLocalError`.
- Writing `return inc()` instead of `return inc` when you want to hand back the function.
- Forgetting `return result` in a decorator's wrapper, so the decorated function returns `None`.
- Forgetting `*args, **kwargs` in a wrapper, so it breaks on functions with arguments.
- Forgetting `@wraps`, so the function's name becomes `wrapper`.
- Late binding in loops (`lambda: i`). Fix it with `lambda i=i: i`.
- A mutable default argument (`def f(x, l=[])`).
- Calling a function when you meant to pass it (`apply(fn(), x)` instead of `apply(fn, x)`).
- Using `global` for state that should be passed in or returned.
- Putting long logic in a lambda instead of a normal `def`.
- Recursion with no base case, or a base case that is never reached, which gives `RecursionError`.
- Forgetting that `map` and `filter` return lazy iterators; printing them shows an object, not the items. Wrap in `list(...)`.
- Using `lru_cache` on an impure function (stale results) or passing unhashable arguments (lists, dicts).
- Hiding side effects (printing, changing globals or arguments) inside functions that look like they only compute a value.
- `map`/`filter`/`reduce` pitfalls: printing the iterator object, reusing an exhausted iterator, forgetting `from functools import reduce`, `reduce` on empty input without an initializer, and `map(print, ...)` doing nothing because it is lazy.

## 10. Performance considerations
- Function calls have a small cost; do not avoid functions for speed in normal code.
- A decorator adds one extra call layer per invocation; usually negligible, but it matters in extremely hot loops.
- `functools.lru_cache` can make a repeated pure function dramatically faster by remembering results.
- Use `time.perf_counter()` for timing; `time.time()` is less precise.
- Closures keep their captured variables alive in memory, so avoid capturing huge objects you don't need.

## 11. Security considerations
- Decorators are the standard place for authentication and permission checks (`@login_required`). A missing or mis-ordered decorator can leave an endpoint unprotected.
- Do not log secrets in timing or logging decorators: `*args, **kwargs` may contain passwords or tokens.
- Never use `eval()` or `exec()` on user input to build functions.
- Global mutable state shared across requests can leak one user's data to another.

## 12. Interview questions and answers

**1. What is the LEGB rule?**
Python looks up a name in four scopes in order: Local (current function), Enclosing (outer functions), Global (module level), Built-in (Python's own names). It uses the first match it finds.

**2. `*args` vs `**kwargs`?**
`*args` collects extra positional arguments into a tuple. `**kwargs` collects extra keyword arguments into a dict. In a call, `*` and `**` do the reverse and unpack a sequence or a dict into arguments.

**3. What is a closure? What does `nonlocal` do?**
A closure is an inner function that remembers the variables of the enclosing function it was created in, even after that function has returned. `nonlocal` lets the inner function re-assign such a variable. Without it, assignment would create a new local variable.

**4. Why does `[lambda: i for i in range(3)]` give `[2, 2, 2]`?**
The lambdas capture the variable `i`, not its value at that moment, and look it up when called. After the loop `i` is 2. Fix with a default argument: `lambda i=i: i`.

**5. What are keyword-only arguments?**
Parameters after `*` or `*args` in a `def` line. They must be passed by name, for example `def f(*, key)` requires `f(key=1)`. This makes calls clearer and prevents mistakes with argument order.

**6. What is a decorator, and what does `@timed` do?**
A decorator is a function that takes a function and returns a new function wrapping it with extra behavior. `@timed` above `def f` is equivalent to `f = timed(f)`.

**7. Why use `functools.wraps`?**
It copies the original function's `__name__`, docstring and other metadata onto the wrapper, so debugging, documentation and tools such as FastAPI and pytest still see the real function.

**8. Why use `*args, **kwargs` inside a decorator's wrapper?**
So the same decorator works on functions with any signature. The wrapper accepts everything and forwards it unchanged to the original.

**9. `global` vs `nonlocal`?**
`global x` refers to a variable at module level. `nonlocal x` refers to a variable in the nearest enclosing function (not the module). Both are needed only to re-assign, not to read.

**10. What does a function return if there is no `return` statement?**
`None`.

**11. Is Python pass-by-value or pass-by-reference?**
Neither exactly: it is pass-by-object-reference. The parameter becomes another label on the same object, so mutating a mutable argument affects the caller, but re-assigning the parameter does not.

**12. What is a lambda, and when should you avoid it?**
A one-expression anonymous function, handy for `key=` arguments. Avoid it for anything complex or reused, because a named `def` is clearer and easier to debug.

**13. What is recursion? What is a base case?**
Recursion is a function calling itself on a smaller version of the problem. The base case is the condition that stops the calls and returns directly. Without it Python raises `RecursionError` once the call stack passes its limit (about 1000 calls by default).

**14. `map`/`filter` vs a list comprehension?**
They do similar jobs. `map` and `filter` return lazy iterators and take a function; a comprehension builds a list directly and is usually more readable. In Python the comprehension is the preferred style, and `map`/`filter` shine when you already have a named function to pass.

**15. What does `functools.partial` do?**
It creates a new function from an existing one with some arguments fixed. `square = partial(power, exp=2)` gives a function that only needs `base`.

**16. How does `lru_cache` work, and when should you not use it?**
It stores results in a dict keyed by the call's arguments and returns the stored result on repeat calls, dropping the least recently used entries when `maxsize` is reached. Arguments must be hashable. Do not use it on impure functions (results depend on time, files, a database) or when the cache could grow without bound.

**17. What is a pure function and why does it matter?**
A pure function returns the same output for the same input and has no side effects. It is easy to test, cache and reuse, and it makes bugs easier to locate.

**18. Why is a naive recursive Fibonacci slow, and how do you fix it?**
Each call spawns two more, recomputing the same values many times, so the work grows exponentially (about 2^n). Memoization (`@lru_cache`) computes each value once, making it linear (about n calls). A loop also works.

**19. What does `map` return in Python 3, and what is the catch?**
A lazy iterator (a `map` object), not a list. It computes items only when you loop over it or call `list()`, and it can be consumed only once; a second pass gives nothing. Wrap it in `list(...)` if you need the results more than once.

**20. How does `reduce` work? What is the initializer for?**
`reduce(f, items, init)` keeps an accumulator. It starts with `init` (or the first item if there is no initializer), then repeatedly sets `acc = f(acc, next_item)`, and finally returns `acc`. The initializer gives a starting value and makes empty input safe; without it, an empty iterable raises `TypeError`.

**21. How does `filter(None, items)` behave?**
It keeps only the truthy items and drops `0`, `""`, `None`, `[]`, `{}` and `False`.

**22. Why does `map(print, [1, 2])` print nothing?**
Because `map` is lazy. Nothing runs until the iterator is consumed (for example by `list(...)` or a `for` loop). Use a plain `for` loop when you want side effects.

**23. When should you use `reduce`, and when not?**
Use it only for a custom combining rule that no built-in covers. For sums, products, minimum, maximum, all/any and joining strings, use `sum`, `math.prod`, `min`, `max`, `all`, `any` and `"".join`, which are clearer and faster.

**24. How does `sorted(items, key=f)` work?**
It calls `f` once on each item to get a sort value, orders the items by those values, and returns the original items in that order (a new list). The sort is stable, so equal keys keep their original order. Use a tuple key such as `(age, name)` to sort by several fields.

**25. Why does a closure need `nonlocal` to update a variable, but not to read one?**
Any assignment inside a function makes that name local to the function. Reading an outer variable just finds it by the LEGB search. Re-assigning without `nonlocal` would create a new local variable, so `count += 1` fails with `UnboundLocalError`.

**26. In `@timed`, when does the decorator body run versus the wrapper body?**
The decorator function (`timed`) runs once, when the decorated function is defined, and returns the wrapper. The wrapper runs on every call of the decorated function.

## 13. Things to remember
- `f` is the function, `f()` calls it.
- Extra positional arguments go into `*args` (tuple); extra keyword arguments go into `**kwargs` (dict).
- Re-assigning an outer variable needs `nonlocal` (enclosing function) or `global` (module).
- To hand back a function, `return inner`, not `return inner()`.
- `@decorator` means `f = decorator(f)`.
- A decorator's wrapper needs `*args, **kwargs`, a call to the original, and `return result`.
- Always add `@wraps(fn)` in a decorator.
- Lambdas in loops: use `lambda i=i: i`.
- Defaults are evaluated once at definition time; never use a mutable default.
- Recursion needs a base case and a smaller problem each call.
- `map`/`filter` are lazy: use `list(...)`; prefer comprehensions for readability.
- `partial(f, x=1)` pre-fills arguments; `@lru_cache` remembers results (hashable arguments, pure functions only).
- Pure function = same input gives same output, no side effects.
- The first string in a function is its docstring; `help(f)` shows it.

## 14. One-page revision summary
- **Function = object.** `def` creates it; `f()` calls it; `f` alone is the object. No `return` means `None`.
- **Parameters:** positional, default, `*args` (tuple), keyword-only, `**kwargs` (dict). Stars in a call unpack.
- **Scope:** LEGB lookup. Assigning inside a function makes a new local unless `global` or `nonlocal` is declared. `count += 1` without the declaration raises `UnboundLocalError`.
- **Closure:** inner function + remembered outer variables. `return inner` (the function), not `inner()`.
- **Decorator:** `@d` above `def f` equals `f = d(f)`. Pattern: `def d(fn): @wraps(fn) def wrapper(*a, **k): ...; r = fn(*a, **k); ...; return r; return wrapper`.
- **Lambda:** one-expression anonymous function. Late-binding trap: `lambda: i` sees the final `i`; fix with `lambda i=i: i`.
- **Timing:** `time.perf_counter()` before and after; subtract.
- **Arguments are passed by object reference** (Topic 1): mutation is visible to the caller, re-assignment is not.
- **Docstring:** first string in the function; read with `help(f)` or `f.__doc__`.
- **Recursion:** base case + smaller recursive call; default depth limit about 1000; use loops for simple repetition.
- **map / filter / reduce:** `map` transforms every item, `filter` keeps items passing a test, `reduce` combines all items into one value (`from functools import reduce`; accumulator + next item; add an initializer for empty input). `map` and `filter` are lazy one-use iterators, so wrap in `list()`. Prefer comprehensions, and prefer built-ins (`sum`, `max`, `all`, `"".join`) over `reduce`. `key=` takes a function called once per item (`sorted(x, key=len)`).
- **Why `nonlocal`:** any assignment inside a function makes the name local, so re-assigning an enclosing variable needs `nonlocal`; reading needs nothing.
- **Decorator timing:** the decorator body runs once at definition; the wrapper runs on every call. A wrapper needs `*args, **kwargs`, a call to `fn`, and `return result`.
- **Closure rule:** a closure remembers the variable, not its value at that moment (late binding).
- **partial:** `partial(f, exp=2)` returns a new function with `exp` fixed.
- **lru_cache:** memoizes results in a dict keyed by arguments (must be hashable); turns exponential Fibonacci into linear; use on pure functions only.
- **Pure vs impure:** pure = same input, same output, no side effects; keep core logic pure, side effects at the edges.

## 15. Practice to redo, with solutions

**1. `@log_calls` decorator**
```python
from functools import wraps

def log_calls(fn):
    @wraps(fn)
    def wrapper(*args, **kwargs):
        print(f"Calling {fn.__name__} with args={args} kwargs={kwargs}")
        return fn(*args, **kwargs)
    return wrapper

@log_calls
def add(a, b=0):
    return a + b

print(add(2, b=3))
# Calling add with args=(2,) kwargs={'b': 3}
# 5
```

**2. `make_multiplier(n)` closure**
```python
def make_multiplier(n):
    def multiply(x):
        return x * n          # n lives in the closure's "backpack"
    return multiply

double = make_multiplier(2)
triple = make_multiplier(3)
print(double(5), triple(5))   # 10 15
```
No `nonlocal` is needed because `n` is only read, never re-assigned.

**3. Why does this fail?**
```python
x = 10
def f():
    print(x)      # UnboundLocalError
    x = 20
f()
```
Because `x = 20` appears inside `f`, Python treats `x` as a **local** variable for the **whole** function. So `print(x)` tries to read the local `x` before it has a value. Fix: remove the assignment, or add `global x` at the top of `f` if you really mean the module-level `x`.

**4. Recursion and caching**
```python
from functools import lru_cache

def fact(n):
    return 1 if n <= 1 else n * fact(n - 1)

def fib_slow(n):
    return n if n < 2 else fib_slow(n - 1) + fib_slow(n - 2)

@lru_cache(maxsize=None)
def fib_fast(n):
    return n if n < 2 else fib_fast(n - 1) + fib_fast(n - 2)

# time fib_slow(32) vs fib_fast(32) using your @timed decorator
```

**5. `partial`**
```python
from functools import partial
def power(base, exp): return base ** exp
square = partial(power, exp=2)
cube = partial(power, exp=3)
print(square(5), cube(5))     # 25 125
```

**6. `map`, `filter`, `reduce`**
```python
from functools import reduce
nums = [1, 2, 3, 4, 5, 6]

print(list(map(lambda x: x * 3, nums)))               # [3, 6, 9, 12, 15, 18]
print(list(filter(lambda x: x % 2 == 1, nums)))       # [1, 3, 5]
print(reduce(lambda acc, x: acc + x, nums))           # 21
print(reduce(lambda acc, x: acc + x, [], 0))          # 0  (initializer makes empty input safe)

# Sum of squares of even numbers, chained:
print(reduce(lambda a, b: a + b,
             map(lambda x: x * x, filter(lambda x: x % 2 == 0, nums))))   # 4 + 16 + 36 = 56
```
Predict first, then run: what does `m = map(lambda x: x + 1, nums); list(m); list(m)` return the second time? (Answer: `[]`, because the iterator is exhausted.)