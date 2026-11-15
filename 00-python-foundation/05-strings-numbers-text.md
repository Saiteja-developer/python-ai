# 05 — Strings, Numbers, Text and Control Flow

> Phase 0 · Python foundation. Almost every AI/RAG project starts with text: clean it, classify it, split it, put it into a prompt. This file is the toolbox for that, plus the `if` / loop logic that glues every program together.
> Running example: **support tickets** (the same data a Customer Support Copilot cleans and classifies).
> `🎯 Interview:` marks a point worth remembering for interviews. `⚠️ Common confusion:` marks a place where I got stuck, with the reason explained.
> How to read each section: Definition → Important points → Syntax → Examples (output in comments).

---

## 1. Core types, conversion and truthiness

### 1.1 Definition
Every value has a **type**. The five basic ones:

| Type | Example | What it is |
|---|---|---|
| `int` | `42` | A whole number of any size. |
| `float` | `3.14` | A decimal number (with small precision errors, see section 2). |
| `str` | `"hello"` | Text. |
| `bool` | `True`, `False` | A yes/no value. It is a kind of `int` (`True == 1`). |
| `NoneType` | `None` | "No value". There is only one `None`. |

### 1.2 Syntax and examples
**Syntax:** `type(value)`, `isinstance(value, type)`, `int(value)`, `float(value)`, `str(value)`, `bool(value)`
```python
type(42)      # <class 'int'>
type(3.14)    # <class 'float'>
type("hi")    # <class 'str'>
type(True)    # <class 'bool'>
type(None)    # <class 'NoneType'>
```
```python
1_000_000     # 1000000    underscores only make big numbers easier to read
10 ** 30      # 1000000000000000000000000000000    int has no size limit
True + True   # 2
```
```python
isinstance(3, int)             # True
isinstance("3", int)           # False    "3" is text, not a number
isinstance(3.0, (int, float))  # True     a tuple means "any of these types"
```
**Conversion:**
```python
int("5")            # 5
float("2.5")        # 2.5
str(42)             # '42'
int(3.9)            # 3         cuts the decimal part, does NOT round
int(float("3.9"))   # 3         the way to go from "3.9" to a whole number
int("3.9")          # ValueError: invalid literal for int() with base 10: '3.9'
float("3")          # 3.0
str(3) + "1"        # '31'      str(3) is text, so + joins text
```
Text from files and APIs is always `str`. Convert it before maths.

### 1.3 Truthiness: what counts as True or False
In an `if`, every value is treated as true or false. `bool(x)` shows which. It **always returns `True` or `False`**, never the value itself.

**Falsy:** `0`, `0.0`, `""`, `[]`, `{}`, `set()`, `None`, `False`. **Everything else is truthy.**
```python
bool(0)      # False
bool("")     # False
bool([])     # False
bool(None)   # False       None is falsy, so bool(None) is False (it is not "None")
bool("0")    # True        non-empty text, even though it says 0
bool([0])    # True        a list with one item is not empty
bool(" ")    # True        one space is still a character
```
Common use, "is it empty?":
```python
text = ""
if not text:
    print("empty")      # empty
```
⚠️ **Common confusion:** `bool(None)` is `False`. I answered `none` because I thought `bool` gives the value back. `bool(...)` only gives `True` or `False`.
⚠️ Many bugs come from `"0"` and `[0]` being truthy. Compare with `== 0` or `len(x) == 0` when you mean that.

---

## 2. Numbers

### 2.1 Operators
| Operator | Meaning | Example |
|---|---|---|
| `/` | Division. **Always gives a float.** | `10 / 2` gives `5.0` |
| `//` | Floor division: rounds **down**, gives an int for ints. | `7 // 2` gives `3` |
| `%` | Remainder. | `7 % 3` gives `1` |
| `**` | Power. | `2 ** 10` gives `1024` |

**Syntax:** `a + b`, `a - b`, `a * b`, `a / b`, `a // b`, `a % b`, `a ** b`, `a += 1` (same as `a = a + 1`)
```python
10 / 2      # 5.0      float, even though the answer is whole
7 / 2       # 3.5
10 // 2     # 5        int
7 // 2      # 3
-7 // 2     # -4       "down" means toward minus infinity, not toward zero
7 % 3       # 1
10 % 2      # 0        remainder 0 means even
2 ** 10     # 1024
```
⚠️ **Common confusion:** `type(10 / 2)` is `float`, not `int`. `/` **always** returns a float. I answered `int`. Use `//` when you need a whole number (for example, how many chunks fit).

Useful built-ins:
```python
abs(-5)            # 5
max(3, 9, 4)       # 9
min(3, 9, 4)       # 3
sum([1, 2, 3])     # 6
divmod(17, 5)      # (3, 2)    quotient and remainder together
import math
math.floor(2.7)    # 2         round down
math.ceil(2.1)     # 3         round up
math.sqrt(16)      # 4.0
```

### 2.2 ⚠️ Float precision (a classic bug)
Computers store decimals in binary, so many decimals cannot be stored exactly.
```python
0.1 + 0.2            # 0.30000000000000004
0.1 + 0.2 == 0.3     # False
```
**Syntax:** `math.isclose(a, b)` and `Decimal("text")`
```python
import math
math.isclose(0.1 + 0.2, 0.3)    # True      compare floats like this, never with ==
round(0.1 + 0.2, 2)             # 0.3       or round when printing
```
For **money**, use `Decimal` and create it from a **string**:
```python
from decimal import Decimal
Decimal("0.1") + Decimal("0.2")   # Decimal('0.3')
Decimal(0.1)                      # Decimal('0.1000000000000000055511151231257827021181583404541015625')
                                  # made from a float it copies the float's error: use a string
```
🎯 **Interview:** never compare floats with `==`; use `math.isclose`. Use `Decimal` for money. In AI work this appears when you compare similarity scores or thresholds.

### 2.3 ⚠️ `round()` uses banker's rounding
**Syntax:** `round(number)`, `round(number, ndigits)`

A `.5` goes to the nearest **even** number. It does not always go up or down.
```python
round(2.5)          # 2     2 is even
round(3.5)          # 4     4 is even
round(0.5)          # 0
round(1.5)          # 2
round(3.14159, 2)   # 3.14  second argument = decimal places
round(1234, -2)     # 1200  negative = round to hundreds
```
I answered `round(3.5)` as `3`. The rule is "go to even", not "go down": for 3.5 the neighbours are 3 and 4, and 4 is even.

---

## 3. Strings: the basics

### 3.1 Definition
A **string** is text: an **immutable** sequence of characters (Topic 01). Index and slice it like a list (Topic 02), but you cannot change it in place.

### 3.2 Indexing, slicing, length
**Syntax:** `s[index]`, `s[start:stop:step]`, `len(s)`. Start is included, stop is not. `0` is the first character, `-1` the last.
```python
s = "Hello, World"
s[0]       # 'H'
s[-1]      # 'd'
s[0:5]     # 'Hello'
s[::-1]    # 'dlroW ,olleH'
len(s)     # 12
```

### 3.3 ⚠️ Strings are immutable: methods return a NEW string
This is the most important rule in strings.
```python
t = "hello"
t.upper()        # makes a NEW string 'HELLO'... and nothing keeps it
print(t)         # hello      t did not change
t = t.upper()    # the fix: point the name t at the new string
print(t)         # HELLO
t[0] = "J"       # TypeError: 'str' object does not support item assignment
```
In my practice I printed `hello` and said "it was not updated". The full reason: strings are immutable, so `upper()` builds a new string, and I threw it away because I did not store it. **Rule: a string method returns a new string. Catch it with `=`.**

### 3.4 Joining with `+` and `*`
```python
"ab" + "cd"    # 'abcd'
"ab" * 3       # 'ababab'
"-" * 10       # '----------'     handy for printing a line
```
`+=` in a loop builds a new string every time. For many pieces use `"".join(parts)` (section 4.3).

---

## 4. String methods

### 4.1 Cleaning and case
**Syntax:** `s.strip()`, `s.lstrip()`, `s.rstrip()`, `s.strip(chars)`, `s.lower()`, `s.upper()`, `s.title()`, `s.capitalize()`, `s.swapcase()`
```python
"  hi ".strip()          # 'hi'       both ends
"  hi ".lstrip()         # 'hi '      left only
"  hi ".rstrip()         # '  hi'     right only
"xxhixx".strip("x")      # 'hi'       strip chosen characters
"Hello".lower()          # 'hello'    lowercase before comparing text
"Hello".upper()          # 'HELLO'
"hello world".title()    # 'Hello World'
"hello world".capitalize()   # 'Hello world'    only the first letter
"Hello".swapcase()       # 'hELLO'
```

### 4.2 `replace`
**Syntax:** `s.replace(old, new)`, `s.replace(old, new, count)`
```python
"a!b!".replace("!", "")        # 'ab'       remove by replacing with nothing
"a-a-a".replace("a", "x", 2)   # 'x-x-a'    only the first 2
```

### 4.3 `split` and `join` (very important for text work)
`split` cuts a string into a **list**. `join` glues a list into a **string**.

**Syntax:** `s.split()`, `s.split(separator)`, `s.split(separator, maxsplit)`, `s.splitlines()`, `"glue".join(list_of_strings)`
```python
"a,b,c".split(",")        # ['a', 'b', 'c']
"a,b,c".split(",", 1)     # ['a', 'b,c']      cut at most once
"a  b   c".split()        # ['a', 'b', 'c']   NO argument: any whitespace, no empty pieces
"a  b   c".split(" ")     # ['a', '', 'b', '', '', 'c']   with " ": empty pieces appear
"a\nb\nc".splitlines()    # ['a', 'b', 'c']
"-".join(["a", "b", "c"])       # 'a-b-c'      the glue goes in FRONT of .join
" ".join(["hello", "world"])    # 'hello world'
" ".join("too    many   spaces".split())   # 'too many spaces'    the famous whitespace cleaner
```
🎯 **Interview:** `join` is the fast way to build one string from many pieces. Do not add strings in a loop with `+`.

### 4.4 Searching: `in`, `find`, `index`, `count`, `startswith`, `endswith`
**Syntax:** `sub in s`, `sub not in s`, `s.find(sub)`, `s.index(sub)`, `s.count(sub)`, `s.startswith(prefix)`, `s.endswith(suffix)`
```python
"ref" in "refund"       # True
"z" in "refund"         # False
"z" not in "refund"     # True
"hello".find("l")       # 2     position of the first match
"hello".find("h")       # 0     found at position 0
"hello".find("z")       # -1    find gives -1 when missing
"hello".index("z")      # ValueError: substring not found     index raises an error instead
"banana".count("a")     # 3
"refund.pdf".startswith("ref")   # True
"refund.pdf".endswith(".pdf")    # True
```
⚠️ **Common confusion 1: `find` returns a number, not True/False.** I answered `false` for `"hello".find("z")`; the answer is `-1`. This causes a real bug: `-1` is truthy and `0` is falsy, so `if s.find("h"):` is **False when found** and `if s.find("z"):` is **True when missing**. To ask "is it inside?" use `in`.
```python
if "refund" in text:              # correct
    ...
if text.find("refund") != -1:     # only if you also need the position
    ...
```
⚠️ **Common confusion 2: the direction of `in`.** `A in B` asks "is **A inside B**?". The thing you look for goes on the **left**, the text on the **right**.
```python
"refund" in "refund not received"      # True     is the word inside the ticket?
"refund not received" in "refund"      # False    is the ticket inside the word? (backwards)
"" in "refund"                         # True     the empty string is inside EVERY string
```
That last line is a trap. In my assignment I wrote `text in "refund"` (backwards). Every real ticket gave `False` and became `"other"`, and the **empty** tickets gave `True` and became `"billing"`.

⚠️ **Common confusion 3: `==` vs `in`.** `text == "refund"` means the **whole text is exactly** `"refund"`. A real ticket is a sentence, so it never matches. `"refund" in text` means "the word appears somewhere inside". Lowercase the text first (`"Refund"` is not `"refund"`).

### 4.5 Checking: `isdigit`, `isalpha`, `isalnum`, `isspace`
**Syntax:** `s.isdigit()`, `s.isalpha()`, `s.isalnum()`, `s.isspace()` (all return True/False)
```python
"123".isdigit()    # True
"abc".isdigit()    # False
"abc".isalpha()    # True
"ab1".isalnum()    # True     letters and digits only
"   ".isspace()    # True
```

### 4.6 Padding: `zfill`, `center`, `ljust`, `rjust`
**Syntax:** `s.zfill(width)`, `s.center(width, fillchar)`, `s.ljust(width, fillchar)`, `s.rjust(width, fillchar)`
```python
"5".zfill(3)           # '005'
"ab".center(6, "*")    # '**ab**'
"ab".ljust(5, ".")     # 'ab...'
"ab".rjust(5)          # '   ab'
```

### 4.7 Chain them: the cleaning pipeline
```python
raw = "  Order #4412: REFUND not received!!  "
step1 = raw.strip()                 # 'Order #4412: REFUND not received!!'
step2 = step1.lower()               # 'order #4412: refund not received!!'
step3 = step2.replace("!", "")      # 'order #4412: refund not received'
words = step3.split()               # ['order', '#4412:', 'refund', 'not', 'received']
"-".join(words)                     # 'order-#4412:-refund-not-received'
```
Save each step in its own variable while learning. It makes mistakes easy to find. Once confident, chain: `raw.strip().lower().replace("!", "")`.
**Order matters:** remove unwanted characters first, then collapse spaces with `" ".join(text.split())`. If you collapse first and remove `?` after, you can leave a stray space.

---

## 5. f-strings and formatting

### 5.1 Definition
An **f-string** puts values inside text. Start with `f` and write expressions in `{}`.

**Syntax:** `f"text {expression} text"`
```python
name = "Ann"
f"Hi {name}"             # 'Hi Ann'
f"{2 + 3}"               # '5'       any expression works
f"{name.upper()}"        # 'ANN'
```

### 5.2 Format specs
**Syntax:** `f"{value:spec}"` where `spec = [align][width][,][.precision][type]`
- align: `<` left, `>` right, `^` centre. width: minimum characters (a `0` in front pads with zeros).
- `,` thousands separator. `.2` two digits after the point. type: `f` decimals, `d` integer, `%` percent, `b` binary, `x` hex.
```python
price = 1234567.891
f"{price:,.2f}"     # '1,234,567.89'
f"{3.14159:.2f}"    # '3.14'
f"{1e6:,.0f}"       # '1,000,000'
f"{0.8567:.1%}"     # '85.7%'
f"{42:05d}"         # '00042'
f"{'Ann':>8}|"      # '     Ann|'
f"{'Ann':<8}|"      # 'Ann     |'
f"{'Ann':^8}|"      # '  Ann   |'
f"{255:b}"          # '11111111'
f"{255:x}"          # 'ff'
n = 42
f"{n=}"             # 'n=42'      shows name and value (debugging)
f"{name!r}"         # "'Ann'"     !r shows the quotes
```
Older styles you will see in old code (use f-strings in new code):
```python
"{} is {}".format("Ann", 20)    # 'Ann is 20'
"%s is %d" % ("Ann", 20)        # 'Ann is 20'
```

### 5.3 ⚠️ Common confusion: building a prompt (a text template with holes)
A prompt is a **template with holes**. Three steps: (1) make variables for the changing parts, (2) write the fixed text in a triple-quoted f-string with `{variable}` where the changing parts go, (3) print it. Triple quotes allow several lines. **Without the `f`, the braces print literally** (`{policy}`), nothing is replaced.
```python
policy = "Items can be returned within 30 days."
question = "Can I return my shoes after 2 weeks?"

prompt = f"""You are a support assistant. Use only the policy below.

Policy:
{policy}

Customer question: {question}
Answer:"""
print(prompt)
# You are a support assistant. Use only the policy below.
#
# Policy:
# Items can be returned within 30 days.
#
# Customer question: Can I return my shoes after 2 weeks?
# Answer:
```
In a RAG pipeline, `policy` is the **retrieved chunks** and `question` is what the user typed. The code is identical.

### 5.4 Raw strings, escapes, triple quotes
`\n` new line, `\t` tab, `\\` one backslash, `\"` a quote. A **raw string** `r"..."` keeps backslashes as typed.
```python
len("\n")                # 1      one new-line character
len(r"\n")              # 2      a backslash and an n
print("C:\\new")        # C:\new
print(r"C:\new\table")  # C:\new\table     use r"" for Windows paths and regex
```

---

## 6. Unicode: `str` vs `bytes`

### 6.1 Definition
`str` is **text** (characters). `bytes` is **raw data** (numbers 0 to 255) that files and networks use. **Encode** turns text into bytes. **Decode** turns bytes into text. Use **UTF-8** almost always.

### 6.2 Syntax and examples
**Syntax:** `text.encode("utf-8")`, `raw.decode("utf-8")`, `text.encode("ascii", errors="replace")`, `ord(char)`, `chr(number)`
```python
t = "café"
len(t)                       # 4       characters
b = t.encode("utf-8")        # b'caf\xc3\xa9'
len(b)                       # 5       bytes
b.decode("utf-8")            # 'café'
t.encode("ascii")            # UnicodeEncodeError     ascii cannot hold é
t.encode("ascii", errors="replace")    # b'caf?'
t.encode("ascii", errors="ignore")     # b'caf'       é is dropped
"café".encode("utf-8").decode("latin-1")   # 'cafÃ©'   wrong decoding = garbled text (mojibake)
ord("A")      # 65
chr(97)       # 'a'
ord("é")      # 233
```
⚠️ **Common confusion: why 4 characters but 5 bytes?** `len(text)` counts **characters** (what a human sees). `len(text.encode())` counts **bytes** (what the computer stores). UTF-8 uses a different number of bytes per character:

| Character | Bytes in UTF-8 | Why |
|---|---|---|
| `c`, `a`, `f` | 1 byte each | Plain English letters need 1 byte. |
| `é` | 2 bytes (`\xc3\xa9`) | Accented and non-English letters need 2 or more. |

So `1 + 1 + 1 + 2 = 5`. An emoji is 1 character and 4 bytes:
```python
len("😀")                   # 1
len("😀".encode("utf-8"))   # 4
```
File sizes and API limits count **bytes**. Token limits count **tokens**. Do not assume `len(text)` equals either.

### 6.3 Same-looking text can be different (normalization)
`é` can be stored as one character or as `e` plus a separate accent. They look the same but compare as different:
```python
import unicodedata
a = "caf\u00e9"                 # é as one character
b = "cafe\u0301"                # e + combining accent
a == b                          # False     looks identical, is not equal
len(a), len(b)                  # (4, 5)
unicodedata.normalize("NFC", a) == unicodedata.normalize("NFC", b)   # True
```
**Syntax:** `unicodedata.normalize("NFC", text)`. Normalize text before comparing or embedding it, so identical-looking words match.
Garbled text like `Ã©` means "decoded with the wrong encoding". Always open files with `encoding="utf-8"` (Topic 08). Text from PDFs and web pages often has odd characters; a wrong encoding silently damages the text before it becomes embeddings.

---

## 7. Control flow

### 7.1 `if` / `elif` / `else`
**Syntax:**
```
if condition1:
    ...
elif condition2:
    ...
else:
    ...

value = a if condition else b       # one-line version (ternary)
```
Python checks top to bottom and runs only the **first** true branch.
```python
for score in (0.95, 0.8, 0.3):
    if score > 0.9:
        label = "strong"
    elif score > 0.7:
        label = "ok"
    else:
        label = "weak"
    print(score, label)
# 0.95 strong
# 0.8 ok
# 0.3 weak

label = "strong" if 0.95 > 0.9 else "weak"     # 'strong'
```

### 7.2 Loops: `for`, `range`, `while`, `break`, `continue`
**Syntax:**
```
for item in iterable:           range(stop)    range(start, stop)    range(start, stop, step)
    ...                         # stop is NOT included
while condition:
    ...
break       # leave the loop now
continue    # skip the rest of this round, go to the next item
```
```python
for i in range(3):
    print(i, end=" ")             # 0 1 2
for i in range(2, 10, 3):
    print(i, end=" ")             # 2 5 8

a = 0
while a < 3:
    a += 1                        # change something inside, or it never stops
print(a)                          # 3

for x in [1, 2, 3, 4]:
    if x == 3:
        break                     # stop the whole loop
    print("x", x)                 # x 1 / x 2

for x in range(1, 11):
    if x % 3 == 0:
        continue                  # skip numbers divisible by 3
    print(x, end=" ")             # 1 2 4 5 7 8 10
```

### 7.3 ⚠️ `for ... else`: the "search" pattern
The `else` runs **only if the loop finished without `break`**.

**Syntax:**
```
for item in iterable:
    if found_it:
        ...
        break
else:
    ...          # runs only when nothing was found
```
```python
for i in [1, 3, 5]:
    if i % 2 == 0:
        print("found", i)
        break
else:
    print("none found")        # none found

for i in [1, 3, 6]:
    if i % 2 == 0:
        print("found", i)      # found 6
        break
else:
    print("none found")        # (does not run)
```
⚠️ **Common confusion: indentation decides who owns the `else`.** I lined the `else` up with the `if`, so it belonged to the `if` and ran on **every** odd item:
```python
for x in [1, 3, 5]:
    if x % 2 == 0:
        print("found")
        break
    else:                       # belongs to the IF: "this item is not even"
        print("not found")
# not found
# not found
# not found                     printed 3 times, and no "none found" ever
```
Lined up with the **`for`**, it runs once, only if the whole loop found nothing. Under `if` = "this item failed". Under `for` = "the whole search failed".

### 7.4 Short-circuit `or` / `and`
`a or b` gives `a` if `a` is truthy, else `b`. `a and b` gives `a` if `a` is falsy, else `b`. They return one of the **values**, not only True/False.
```python
"" or "default"        # 'default'
"hi" or "default"      # 'hi'
0 or "default"         # 'default'
None or "default"      # 'default'
"" and "x"             # ''
"a" and "b"            # 'b'
```
⚠️ `name or "guest"` also replaces `0` and `""`. If `0` is a valid value, write `x if x is not None else default`.

### 7.5 Chained comparison and walrus
```python
0 < 0.5 < 1               # True     same as 0 < 0.5 and 0.5 < 1
0 < 5 < 1                 # False

if (n := len("hello world")) > 10:     # := assigns and tests in one line
    print("long", n)                    # long 11
```
**Syntax:** `low < x < high`, `if (name := expression) > value:`

### 7.6 `match` (Python 3.10+)
**Syntax:**
```
match value:
    case pattern1:
        ...
    case pattern2 | pattern3:     # | means "or"
        ...
    case _:                       # default
        ...
```
```python
for cmd in ["refund", "hello"]:
    match cmd:
        case "refund":
            print("go billing")      # go billing
        case "login" | "password":
            print("go login")
        case _:
            print("other")           # other
```
`match` compares the **whole value** (like `==`), not "contains". For keyword search in a sentence, use `in`.

---

## 8. Worked example: the ticket cleaner (assignment)

Task: clean each ticket, skip empty ones, classify, print a numbered line, count per category, print percentages.
```python
def clean_text(raw):
    text = raw.strip().lower()
    text = text.replace("!", "").replace("?", "")
    return " ".join(text.split())

def classify(text):
    if "refund" in text or "charge" in text:        # word on the LEFT of in
        return "billing"
    elif "password" in text or "login" in text:
        return "login"
    elif "delivery" in text or "late" in text:
        return "shipping"
    else:
        return "other"

tickets = [
    "  Order #4412: REFUND not received!!  ",
    "Cannot reset my PASSWORD   ",
    "",
    "   ",
    "Charged twice for order #5521 ???",
    "Late delivery... still waiting",
]

skipped = 0
kept = 0
counts = {}
cleaned_tickets = []

for raw in tickets:
    cleaned = clean_text(raw)
    if not cleaned:                       # empty text is falsy: skip it
        skipped += 1
        continue
    kept += 1                             # number only the tickets we keep
    category = classify(cleaned)
    counts[category] = counts.get(category, 0) + 1      # dict counter
    cleaned_tickets.append((category, cleaned))
    print(f"{kept:02d} | {category:<8} | {cleaned}")

print("skipped", skipped)
for category, count in counts.items():
    print(f"{category}: {count} ({count / kept:.0%})")

for category, cleaned in cleaned_tickets:         # for-else search
    if category == "login":
        print("first login ticket:", cleaned)
        break
else:
    print("no login ticket")
```
**Output:**
```
01 | billing  | order #4412: refund not received
02 | login    | cannot reset my password
03 | billing  | charged twice for order #5521
04 | shipping | late delivery... still waiting
skipped 2
billing: 2 (50%)
login: 1 (25%)
shipping: 1 (25%)
first login ticket: cannot reset my password
```

### Mistakes in my first attempt, and why
1. **`text in "refund"` (backwards).** See 4.4. All real tickets became `"other"`, and empty tickets became `"billing"` (the empty string is inside every string). Write `"refund" in text`.
2. **Counting `"other"` as skipped.** Skipping is for **empty** tickets, and the test for empty is truthiness: `if not cleaned:`. `"other"` is a real category and must be counted. My `skipped` showed 4 instead of 2.
3. **Variable named `str`.** It hides Python's built-in `str` (the name-shadowing problem from Topic 04). After `str = "billing"`, calling `str(5)` fails:
   ```python
   str = "billing"
   str(5)       # TypeError: 'str' object is not callable
   ```
   Never name a variable `str`, `list`, `dict`, `type`, `max`, `sum`, `id`. Use `category`.
4. **Numbering every ticket.** My `total` went up for every ticket, including skipped ones, and its name was misleading. Number only the **kept** ones (`kept`). Name variables by what they hold.
5. **Four separate counters** (`billing`, `login`, `shipped`...). A new category needs a new variable and a new `elif`. One **dict** (`counts[category] = counts.get(category, 0) + 1`) handles any category.
6. **Alignment and percentage were missing.** `{category:<8}` aligns the columns and `{count / kept:.0%}` makes a percentage (section 5.2).

What was already right: `clean_text` was correct, written step by step, and the `{total:02d}` format and the `if / elif` structure were good.

---

## 9. Patterns to remember

| Pattern | Template | Used for |
|---|---|---|
| **Cleaning pipeline** | `strip` → `lower` → remove characters → `" ".join(text.split())` | Preparing text for chunking and embedding. |
| **Skip empty** | `if not cleaned: continue` | Dropping blank rows. |
| **Keyword rule** | `if "word" in text or "word2" in text: return "category"` | Simple classifiers and routing. |
| **Rules table** | `RULES = [("billing", ["refund", "charge"]), ...]` with a loop | Many categories without many `elif`s. |
| **Count with a dict** | `counts[key] = counts.get(key, 0) + 1` | Counting (or use `Counter`, Topic 02). |
| **Search with for-else** | `for ...: if found: ...; break` / `else: not found` | "Find the first match, or say none". |
| **Default with `or`** | `value = given or "default"` | Fallback text (careful with `0`). |
| **Prompt template** | triple-quoted f-string with `{context}` and `{question}` | Building LLM prompts. |
| **Normalize first** | `unicodedata.normalize("NFC", text)` then lowercase | Matching text that looks the same. |

**Rules table** (verified; the same classifier with data instead of repeated `elif`):
```python
RULES = [
    ("billing",  ["refund", "charge"]),
    ("login",    ["password", "login"]),
    ("shipping", ["delivery", "late"]),
]

def classify(text):
    for category, words in RULES:
        if any(w in text for w in words):      # any(...) = is at least one word inside?
            return category
    return "other"

classify("cannot reset my password")    # 'login'
classify("hello")                       # 'other'
```
To add a category, add one line to `RULES`.

**Structure to remember (a text-processing function):**
```
def process(raw):
    1. clean it        (strip, lower, remove, collapse spaces)
    2. check it        (if not cleaned: skip)
    3. decide          (classify with `in`)
    4. report          (f-string line, counts in a dict)
```

---

## 10. Use this when building (project guide)

- **RAG ingestion:** every document goes through the cleaning pipeline before chunking. Normalize Unicode, strip, collapse whitespace, drop empty chunks (`if not chunk: continue`).
- **Prompt building:** the final prompt is a triple-quoted f-string with the retrieved chunks and the question. Check it with `print(prompt)` before sending it.
- **Sizes:** characters, bytes and tokens are three different counts. Chunk size in characters is fine to start with; API limits use tokens.
- **Scores and thresholds:** compare floats with `math.isclose` or with `>=`/`<=` rather than `==`. Format scores for logs with `f"{score:.3f}"`.
- **Money or exact decimals:** use `Decimal("...")`, never `float`.
- **Routing and rules:** keyword rules (`in`) are a good first classifier and a baseline to beat later with embeddings or an LLM.
- **Files:** always `open(path, encoding="utf-8")` (Topic 08).

**Checklist for any text function:** Did I store the result of each string method (`s = s.strip()`)? Is the search word on the **left** of `in`? Did I lowercase before comparing? Did I handle empty text? Is the `else` lined up with the `for` or the `if` I meant? Did I avoid names like `str`, `list`, `max`?

---

## 🧠 Quick Revision

- Types: `int`, `float`, `str`, `bool`, `None`. `type()`, `isinstance()`. `int("3.9")` fails; `int(3.9)` cuts to 3.
- Falsy: `0`, `0.0`, `""`, `[]`, `{}`, `set()`, `None`, `False`. `"0"` and `[0]` are truthy. `bool(...)` gives only `True`/`False`.
- `/` always gives a float. `//` rounds down (`-7 // 2` is `-4`). `%` remainder. `**` power.
- Floats are inexact: use `math.isclose`; use `Decimal("...")` for money.
- `round(2.5)` is 2 and `round(3.5)` is 4 (ties go to even).
- Strings are **immutable**: methods return a new string, so write `s = s.strip()`.
- `split()` (no argument) splits on any whitespace; `"-".join(list)` glues; `" ".join(s.split())` collapses spaces.
- `find` returns a position or `-1` (not True/False). For "is it inside?" use `"word" in text`, with the word on the **left**. `""` is inside every string.
- `==` means exactly equal, `in` means contained.
- f-string: `f"{value:,.2f}"`, `f"{x:.1%}"`, `f"{n:05d}"`, `f"{s:<8}"`, `f"{n=}"`. Triple-quoted f-strings build prompts; no `f` means no replacement.
- `r"..."` raw strings for Windows paths and regex.
- `str` = text, `bytes` = raw. `encode` / `decode` with `"utf-8"`. Characters, bytes and tokens differ. Wrong decoding gives mojibake. Normalize with `unicodedata.normalize("NFC", text)`.
- `if / elif / else` runs the first true branch. `for` + `range(start, stop, step)` (stop excluded). `break`, `continue`.
- `for ... else`: `else` lined up with `for` runs only if there was no `break`.
- `a or b` returns the first truthy value; `x or "default"`. Careful with `0`.
- `match` compares whole values. Use `in` to search inside text.
- Never name a variable `str`, `list`, `dict`, `max`, `sum`, `type`, `id`.
- Count with `counts[key] = counts.get(key, 0) + 1`. Rules table + `any(...)` replaces long `elif` chains.