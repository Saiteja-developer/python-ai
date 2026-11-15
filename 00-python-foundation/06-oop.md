# 06 — OOP: Classes and Objects (Python vs Java side by side)

> Phase 0 · Python foundation. FastAPI, Pydantic, scikit-learn, PyTorch and LangChain are all built from classes. You already know OOP from Java, so this file teaches **Python OOP by comparing it with Java**: **Python on the left, Java on the right**, with the same example and the same output.
> Running example: **tickets, accounts, transactions, notifiers and a RAG chunker**.
> `🎯 Interview:` marks a point worth remembering for interviews. `⚠️ Common confusion:` marks a place where I (or a Java programmer) can get stuck, with the reason explained.
> All code on both sides was run and gives the output shown in the comments (Java 21, Python 3). Each section follows: Definition → Important points → Syntax and examples.

---

## 0. The big picture: what changes from Java to Python

| Java | Python |
|---|---|
| Java checks types and visibility when you compile, so many mistakes are caught before the program runs. | Python checks almost everything while the program runs, so the same mistake appears as an error only when that line executes. |
| Java requires you to declare every field in the class body, and the constructor has the same name as the class. | Python has no field declarations, and the constructor is the method `__init__`, which creates the attributes by assigning `self.name = value`. |
| Java passes `this` to every method automatically and you never write it in the parameter list. | Python makes you write `self` as the first parameter of every instance method, and the name `self` is only a convention. |
| Java has `public`, `protected`, `private` and package-private keywords that the compiler enforces. | Python has only naming conventions (`_name`, `__name`) and trusts the programmer, so nothing is truly private. |
| Java creates objects with the `new` keyword and a class can have several constructors with different parameters (overloading). | Python creates objects by calling the class like a function (`Ticket(...)`), and one `__init__` with default values replaces many constructors. |
| Java needs `interface` and `implements` so that different classes can be used through one type. | Python uses duck typing ("if it has a `send` method, I can call `send`"), so no declaration is needed, though `ABC` and `Protocol` exist when you want one. |

Quick word map:

| Java word | Python word |
|---|---|
| `this` | `self` |
| `new Ticket(...)` | `Ticket(...)` |
| constructor `Ticket(...)` | `__init__` |
| `static` field | class attribute |
| `static` method | `@staticmethod` (or `@classmethod`) |
| `private` | `_name` convention (and `__name` mangling) |
| getter / setter | `@property` / `@name.setter` |
| `toString()` | `__str__` / `__repr__` |
| `equals()` / `hashCode()` | `__eq__` / `__hash__` |
| `compareTo()` | `__lt__` (and friends) |
| `extends` | `class Child(Parent):` |
| `super(...)` | `super().__init__(...)` |
| `abstract class` | `ABC` + `@abstractmethod` |
| `interface` | `Protocol` (or an `ABC`) |
| `instanceof` | `isinstance(obj, Class)` |
| `null` | `None` |
| `record` | `@dataclass` (Topic 09) |
| `public static void main` | `if __name__ == "__main__":` |

---

## 1. Class, object, constructor, `self`

### 1.1 Definition
- A **class** is a blueprint. It says what data an object has and what it can do.
- An **object** (instance) is one thing made from the blueprint. `Ticket` is the class, ticket #101 is an object.
- The **constructor** sets up a new object. In Python it is `__init__`. `self` is the object being built (Java's `this`).

### 1.2 Important points
- Python has **no field declarations**. `self.id = id` inside `__init__` both creates and sets the attribute.
- `self` must be the **first parameter** of every instance method. Python passes the object there for you, so you call `t.short()` and never write `self` at the call.
- `t.short()` is the same as `Ticket.short(t)`. This shows what `self` really is: the object, passed as the first argument.
- There is **no `new`**. `Ticket(...)` builds the object and calls `__init__`.
- Python allows **one** `__init__`. Instead of overloading, give parameters **default values** (`category="other"`).
- Technically `__new__` creates the object and `__init__` fills it, but you almost never write `__new__`.

### 1.3 Syntax
```
class ClassName:
    def __init__(self, param1, param2="default"):
        self.param1 = param1
        self.param2 = param2

    def method_name(self, other_param):
        return ...self.param1...

obj = ClassName(value1)          # create an object
obj.param1                       # read an attribute
obj.method_name(arg)             # call a method
```

### 1.4 Example: a ticket

```text
PYTHON                                              | JAVA
----------------------------------------------------+----------------------------------------------------------------------
class Ticket:                                       | class Ticket {
    def __init__(self, id, text, category="other"): |     int id;
        self.id = id                                |     String text;
        self.text = text                            |     String category;
        self.category = category                    |
                                                    |     Ticket(int id, String text, String category) {
    def short(self):                                |         this.id = id;
        return f"#{self.id}: {self.text[:10]}"      |         this.text = text;
                                                    |         this.category = category;
t = Ticket(101, "Refund not received", "billing")   |     }
t2 = Ticket(102, "Cannot login now")                |
print(t.short())        # #101: Refund not          |     // no default values in Java: overload instead
print(t2.category)      # other                     |     Ticket(int id, String text) {
                                                    |         this(id, text, "other");
                                                    |     }
                                                    |
                                                    |     String shortText() {
                                                    |         return "#" + id + ": " + text.substring(0, 10);
                                                    |     }
                                                    | }
                                                    |
                                                    | public class Main {
                                                    |     public static void main(String[] args) {
                                                    |         Ticket t = new Ticket(101, "Refund not received", "billing");
                                                    |         Ticket t2 = new Ticket(102, "Cannot login now");
                                                    |         System.out.println(t.shortText());      // #101: Refund not
                                                    |         System.out.println(t2.category);        // other
                                                    |     }
                                                    | }
```


Other things you can do with an object:
```python
t.id                              # 101
type(t)                           # <class '__main__.Ticket'>
isinstance(t, Ticket)             # True            (Java: t instanceof Ticket)
t.category = "refund"             # change an attribute
t.priority = "high"               # Python even allows NEW attributes later (Java does not); avoid it
t.__dict__                        # {'id': 101, 'text': 'Refund not received', 'category': 'refund', 'priority': 'high'}
Ticket.short(t)                   # same as t.short()
```
`obj.__dict__` shows all attributes as a dict. It is useful for debugging.

### 1.5 Default values replace overloaded constructors
Java has no default values, so it needs several methods with the same name (**overloading**). Python needs one function.

```text
PYTHON                          | JAVA
--------------------------------+----------------------------------------------------
# one function, a default value | public class Main {
def add(a, b, c=0):             |     // two methods, same name, different parameters
    return a + b + c            |     static int add(int a, int b) {
                                |         return a + b;
print(add(1, 2))        # 3     |     }
print(add(1, 2, 3))     # 6     |
                                |     // this is "overloading"
                                |     static int add(int a, int b, int c) {
                                |         return a + b + c;
                                |     }
                                |
                                |     public static void main(String[] args) {
                                |         System.out.println(add(1, 2));      // 3
                                |         System.out.println(add(1, 2, 3));   // 6
                                |     }
                                | }
```


Python has **no method overloading**. If you define two methods with the same name, the second one silently replaces the first. Use default values, `*args`, or `if` checks inside one function.
🎯 **Interview:** Python has no overloading; use default arguments or `*args`. `__init__` is not the real constructor (`__new__` is), but `__init__` is where you set up attributes.

### 1.6 ⚠️ Common confusion: forgetting `self`
Java forgives forgetting `this` because the compiler resolves the field. In Python, a name without `self.` is just a **local variable** that disappears when the method ends.

**Mistake I made in practice:** `owner = owner` inside `__init__`.

```text
PYTHON                                              | JAVA
----------------------------------------------------+-------------------------------------------------------------------
class Account:                                      | class Account {
    def __init__(self, owner):                      |     String owner;
        # BUG: only changes the local variable      |     Account(String owner) {
        owner = owner                               |         // BUG too: the parameter is assigned to itself
                                                    |         owner = owner;
a = Account("Ann")                                  |     }
print(a.owner)                                      | }
# AttributeError: 'Account' object has no attribute | // new Account("Ann").owner  ->  null     (no error, just a silent
# 'owner'                                           | // null)
# Fix:  self.owner = owner                          | // Fix:  this.owner = owner;
```


The same idea in both languages: you must say **which** `owner` you mean (`self.owner` / `this.owner`). Python fails loudly with `AttributeError`; Java silently leaves `null`.

**The opposite mistake: forgetting `self` in the method's parameter list.**

```text
PYTHON                                            | JAVA
--------------------------------------------------+---------------------------------------------------------------------
class NoSelf:                                     | class NoSelf {
    def hello():                   # forgot self  |     // no "self" parameter exists in Java:
        return "hi"                               |     String hello() {
                                                  |         return "hi";               // `this` is passed automatically
NoSelf().hello()                                  |     }
# TypeError: NoSelf.hello() takes 0 positional    | }
# arguments but 1 was given                       |
# Python passed the object as the first argument, |
# but there was no parameter to receive it.       |
```


**Rule:** in Python, every instance method starts with `self`, and every attribute is read and written as `self.name`.

### 1.7 ⚠️ Common confusion: `print` an object
Without `__repr__` / `__str__` the output is useless (like Java's default `Ticket@1b6d3586`):
```python
print(t)     # <__main__.Ticket object at 0x7ff4fa336f90>
```
Fixing it is in section 5.

---

## 2. Instance attributes vs class attributes

### 2.1 Definition
- **Instance attribute:** set with `self.x = ...`. Every object has its **own** copy. (Java: a normal field.)
- **Class attribute:** written directly in the class body, outside any method. **All objects share one copy.** (Java: a `static` field.)

### 2.2 Syntax and example
```
class ClassName:
    shared = 0                    # class attribute (Java: static int shared = 0;)
    def __init__(self):
        self.own = 1              # instance attribute
ClassName.shared                  # read through the class name
```

```text
PYTHON                                       | JAVA
---------------------------------------------+-------------------------------------------------------------------
class Counter:                               | class Counter {
    # class attribute: shared                |     static int total = 0;        // static field: shared
    total = 0                                |     int n;                       // instance field: one per object
                                             |
    def __init__(self):                      |     Counter() {
        Counter.total += 1                   |         total++;
        # instance attribute: one per object |         n = total;
        self.n = Counter.total               |     }
                                             | }
a = Counter()                                |
b = Counter()                                | public class Main {
print(a.n, b.n, Counter.total)   # 1 2 2     |     public static void main(String[] args) {
                                             |         Counter a = new Counter();
                                             |         Counter b = new Counter();
                                             |         // 1 2 2
                                             |         System.out.println(a.n + " " + b.n + " " + Counter.total);
                                             |     }
                                             | }
```


Important points:
- Change a class attribute through the **class name** (`Counter.total += 1`). If you write `self.total += 1`, Python makes a **new instance attribute** that hides the class one, and the shared counter does not change.
- An object can **read** a class attribute (`a.total`), so reading through `self` or the class both work.

### 2.3 ⚠️ Common confusion: the Java habit that creates a bug
In Java, `List<String> tags = new ArrayList<>();` written in the class body is a **field with its own list per object**. In Python, anything assigned in the class body is a **class attribute, shared by all objects**. A Java programmer writes the Python version below by habit and gets a bug. (This is the same "one list, many names" idea as Topic 01.)

```text
PYTHON                                          | JAVA
------------------------------------------------+-----------------------------------------------
class Bad:                                      | import java.util.*;
    # class-body assignment = CLASS attribute = |
    # shared!                                   | class Good {
    tags = []                                   |     // field initializer = one list PER object
                                                |     List<String> tags = new ArrayList<>();
    def add(self, x):                           |
        self.tags.append(x)                     |     void add(String x) {
                                                |         tags.add(x);
p = Bad()                                       |     }
q = Bad()                                       | }
p.add("a")                                      |
# ['a']   q got p's tag                         | public class Main {
print(q.tags)                                   |     public static void main(String[] args) {
                                                |         Good p = new Good();
                                                |         Good q = new Good();
                                                |         p.add("a");
                                                |         // []   q is not affected
                                                |         System.out.println(q.tags);
                                                |     }
                                                | }
```


**Fix in Python:** create the list inside `__init__` with `self.`, so each object gets its own:
```python
class Good:
    def __init__(self):
        self.tags = []          # a NEW list for each object

x = Good(); y = Good()
x.tags.append("a")
y.tags                          # []
x.tags                          # ['a']
```
**Rule:** put **changing data** (lists, dicts, counters per object) in `__init__` with `self.`. Use class attributes only for **constants** and **shared counters**.
This was my Practice 1 task: `items = []` in the class body made `p.add(1)` appear in `q.items` (`[1]`). The fix was `self.items = []` in `__init__`, because `__init__` runs again for every new object, while a class-body list is created only once.
🎯 **Interview:** instance vs class attributes, and why a mutable class attribute is shared.

---

## 3. Methods: instance, class and static

### 3.1 Definition
| Kind | Python | Java equivalent | First parameter |
|---|---|---|---|
| Instance method | `def m(self, ...)` | normal method | `self` (the object) |
| Class method | `@classmethod def m(cls, ...)` | a `static` factory method (not exactly equal) | `cls` (the class) |
| Static method | `@staticmethod def m(...)` | `static` method | nothing |

### 3.2 Important points
- A **class method** receives the **class** as `cls`. Its main use is an **alternative constructor**: Python allows only one `__init__`, so `from_csv`, `from_dict` or `from_json` class methods give other ways to build an object. They return `cls(...)`.
- A **static method** is a plain function placed inside the class because it belongs with it. It uses neither `self` nor `cls`.
- Choose: needs the object's data → instance method. Builds objects or changes class data → class method. Just a related helper → static method.

### 3.3 Syntax
```
class ClassName:
    def method(self, ...): ...                  # instance method

    @classmethod
    def method(cls, ...):                       # class method
        return cls(...)

    @staticmethod
    def method(...): ...                        # static method
```

### 3.4 Example: an account

```text
PYTHON                                               | JAVA
-----------------------------------------------------+---------------------------------------------------------------------
class Account:                                       | class Account {
    def __init__(self, owner, balance=0.0):          |     String owner;
        self.owner = owner                           |     double balance;
        self.balance = balance                       |
                                                     |     Account(String owner, double balance) {
    # instance method                                |         this.owner = owner;
    def deposit(self, amount):                       |         this.balance = balance;
        self.balance += amount                       |     }
        return self.balance                          |
                                                     |     // instance method
    # class method: another way to build an object   |     double deposit(double amount) {
    @classmethod                                     |         balance += amount;
    def from_csv(cls, line):                         |         return balance;
        owner, bal = line.split(",")                 |     }
        return cls(owner, float(bal))                |
                                                     |     // static factory method
    # static method: needs no object, no class       |     static Account fromCsv(String line) {
    @staticmethod                                    |         String[] p = line.split(",");
    def is_valid_amount(amount):                     |         return new Account(p[0], Double.parseDouble(p[1]));
        return amount > 0                            |     }
                                                     |
a = Account("Ann", 100.0)                            |     // static method
print(a.deposit(50.0))                       # 150.0 |     static boolean isValidAmount(double amount) {
b = Account.from_csv("Bob,20")                       |         return amount > 0;
# Bob 20.0                                           |     }
print(b.owner, b.balance)                            | }
print(Account.is_valid_amount(-5))           # False |
                                                     | public class Main {
                                                     |     public static void main(String[] args) {
                                                     |         Account a = new Account("Ann", 100.0);
                                                     |         System.out.println(a.deposit(50.0));                // 150.0
                                                     |         Account b = Account.fromCsv("Bob,20");
                                                     |         // Bob 20.0
                                                     |         System.out.println(b.owner + " " + b.balance);
                                                     |         System.out.println(Account.isValidAmount(-5));      // false
                                                     |     }
                                                     | }
```


### 3.5 Class methods and subclasses (a real difference from Java)
In Python, `cls` is **whichever class you called it on**. A Java `static` method always builds the same class.

```text
PYTHON                                      | JAVA
--------------------------------------------+---------------------------------------------------------------------
class Account:                              | class Account {
    def __init__(self, owner, balance=0.0): |     static Account fromCsv(String line) {
        self.owner = owner                  |         String[] p = line.split(",");
        self.balance = balance              |         // always an Account
                                            |         return new Account(p[0], Double.parseDouble(p[1]));
    @classmethod                            |     }
    def from_csv(cls, line):                | }
        owner, bal = line.split(",")        | class Savings extends Account { }
        # cls = the class it was called on  |
        return cls(owner, float(bal))       | // Savings.fromCsv("Cy,10") still returns an Account, not a Savings.
                                            | // Static methods are not "polymorphic" in Java.
class Savings(Account):                     |
    pass                                    |
                                            |
s = Savings.from_csv("Cy,10")               |
# Savings   (not Account)                   |
print(type(s).__name__)                     |
```


🎯 **Interview:** instance vs class vs static method, and why alternative constructors are class methods. Real examples: Pandas `DataFrame.from_dict(...)`, Pydantic `Model.model_validate(data)`.

---

## 4. Encapsulation: access control and `@property`

### 4.1 Definition
**Encapsulation** = keep an object's data together with the code that changes it, and control how outsiders touch it.

### 4.2 Access control: Java keywords vs Python conventions
| Java | Python |
|---|---|
| `public` means anyone can use the member, and it is enforced by the compiler. | A plain name (`balance`) means anyone can use it, and that is the normal default for everything. |
| `protected` means the class, its subclasses and the same package can use it. | A single underscore (`_balance`) means "internal, please do not touch from outside", but Python does not stop you. |
| `private` means only the same class can use it, and outside access is a compile error. | A double underscore (`__pin`) triggers name mangling: Python renames it to `_ClassName__pin`, which avoids clashes in subclasses but is not real security. |

```text
PYTHON                                            | JAVA
--------------------------------------------------+-----------------------------------------------------------------
class Acc2:                                       | class Acc2 {
    def __init__(self):                           |     protected int balance = 10;
        # internal by convention                  |     private int pin = 1234;
        self._balance = 10                        | }
        self.__pin = 1234          # name-mangled |
                                                  | // Acc2 x = new Acc2();
x = Acc2()                                        | // x.pin;       // COMPILE ERROR: pin has private access in Acc2
# 10       works (only a convention)              | // Java stops you before the program even runs.
x._balance                                        |
x.__pin                                           |
# AttributeError: 'Acc2' object has no attribute  |
# '__pin'                                         |
# 1234     still reachable                        |
x._Acc2__pin                                      |
# ['_balance', '_Acc2__pin']                      |
list(x.__dict__)                                  |
```


⚠️ **Common confusion:** `_name` and `__name` do **not** make data private. Python trusts programmers ("we are all consenting adults"). Use `_name` to say "internal", and use `@property` (next) when you need real control.

### 4.3 `@property`: getter and setter without `get`/`set` methods
A **property** lets outside code write `a.balance` (no brackets) while Python runs a method behind it. Use it to **validate** or **compute** values.

**Syntax:**
```
@property
def name(self):                 # getter: runs on  obj.name
    return self._name

@name.setter
def name(self, value):          # setter: runs on  obj.name = value
    ...validate...
    self._name = value
```

```text
PYTHON                                              | JAVA
----------------------------------------------------+-------------------------------------------------------
class Account:                                      | class Account {
    def __init__(self, owner, balance):             |     private String owner;
        self.owner = owner                          |     // private: only this class can touch it
        # goes through the setter below             |     private double balance;
        self.balance = balance                      |
                                                    |     Account(String owner, double balance) {
    # getter                                        |         this.owner = owner;
    @property                                       |         // validate at creation too
    def balance(self):                              |         setBalance(balance);
        return self._balance                        |     }
                                                    |
    # setter with validation                        |     double getBalance() {                    // getter
    @balance.setter                                 |         return balance;
    def balance(self, value):                       |     }
        if value < 0:                               |
            raise ValueError(                       |     // setter with validation
                "balance cannot be negative")       |     void setBalance(double value) {
        self._balance = value                       |         if (value < 0) {
                                                    |             throw new IllegalArgumentException(
a = Account("Ann", 100.0)                           |                 "balance cannot be negative");
# looks like a normal attribute                     |         }
a.balance = 50.0                                    |         this.balance = value;
print(a.balance)                             # 50.0 |     }
try:                                                | }
    a.balance = -5                                  |
except ValueError as e:                             | public class Main {
    # balance cannot be negative                    |     public static void main(String[] args) {
    print(e)                                        |         Account a = new Account("Ann", 100.0);
                                                    |         // you must call the method
                                                    |         a.setBalance(50.0);
                                                    |         System.out.println(a.getBalance());  // 50.0
                                                    |         try {
                                                    |             a.setBalance(-5);
                                                    |         } catch (IllegalArgumentException e) {
                                                    |             // balance cannot be negative
                                                    |             System.out.println(e.getMessage());
                                                    |         }
                                                    |     }
                                                    | }
```


Important points:
- Outside code is just `a.balance = 50.0`. In Java you must write `a.setBalance(50.0)`.
- The property is named `balance`, the stored value is `_balance`. **Two different names.** If both were `balance`, the setter would call itself forever.
- In `__init__` write `self.balance = balance` (no underscore) so the **setter runs at creation too**. Then `Account("Bob", -1)` fails immediately.
- A property keeps the **same outside syntax** even if you add validation later, so old code does not break. (In Java, changing a public field to getter/setter changes every caller.)
- Many Java programmers write `get_balance()` / `set_balance()` in Python. Do not; use properties.
- `raise ValueError(...)` stops with an error, like Java's `throw new IllegalArgumentException(...)`. Full error handling is Topic 08.

### 4.4 Read-only and computed properties
A property with **no setter** is read-only. It is also the way to expose a **computed** value without brackets.

```text
PYTHON                                         | JAVA
-----------------------------------------------+-----------------------------------------------------------------
class Acc:                                     | class Acc {
    def __init__(self, balance):               |     private double balance;
        self._balance = balance                |     Acc(double balance) { this.balance = balance; }
                                               |
    # computed; no setter                      |     boolean isEmpty() {              // a getter only, no setter
    @property                                  |         return balance == 0;
    def is_empty(self):                        |     }
        return self._balance == 0              | }
                                               |
a = Acc(0)                                     | // new Acc(0).isEmpty()   -> true     (brackets needed)
# True     (no brackets)                       |
a.is_empty                                     |
a.is_empty = False                             |
# AttributeError: property 'is_empty' of 'Acc' |
# object has no setter                         |
```


🎯 **Interview:** why `@property` instead of `get_x()` / `set_x()`? It keeps attribute syntax, allows validation later and can compute values. `_x` is convention, `__x` is name mangling, and neither is real privacy. Real example: Pydantic and FastAPI validate fields so bad data is rejected early.

---

## 5. Printing, equality, hashing, ordering: the dunder methods

### 5.1 Definition
Methods with **double underscores on both sides** ("dunder" methods) are called by Python itself at special moments. They are Python's version of overriding `toString`, `equals`, `hashCode` and `compareTo`.

| Java | Python | When Python calls it |
|---|---|---|
| `toString()` | `__repr__` (for developers) and `__str__` (for users) | `print(obj)` uses `__str__`; a list or the notebook uses `__repr__`. |
| `equals(other)` | `__eq__(self, other)` | When you write `a == b`. |
| `hashCode()` | `__hash__(self)` | When the object goes into a `set` or is used as a dict key. |
| `compareTo(other)` | `__lt__(self, other)` (and `__gt__`, `__le__`, `__ge__`) | When you use `<` or `sorted(...)`. |
| `size()` / `length()` | `__len__(self)` | `len(obj)` |
| `get(i)` | `__getitem__(self, i)` | `obj[i]` |
| (a functional-interface method like `apply`) | `__call__(self, ...)` | `obj(...)`, the object behaves like a function. |

### 5.2 Important points
- **`==` in Java compares references; `.equals()` compares values. In Python `==` compares values** (it calls `__eq__`), and `is` compares identity (Java's `==`).
- If you write `__eq__`, also write `__hash__` (equal objects must have equal hashes), otherwise the object cannot be used in sets or as a dict key.
- If you only write one printing method, write **`__repr__`**. `print` falls back to it when there is no `__str__`.
- Make `__repr__` look like the code that builds the object, and show the **real class name** and every attribute.
- `sorted(...)` only needs `__lt__`, or you can give a `key=` function (Topic 03) and write no method at all.

### 5.3 Syntax
```
def __repr__(self):             return f"ClassName(attr={self.attr!r})"
def __str__(self):              return "friendly text"
def __eq__(self, other):        return isinstance(other, ClassName) and self.attr == other.attr
def __hash__(self):             return hash(self.attr)
def __lt__(self, other):        return self.attr < other.attr
```
`!r` inside an f-string shows the **repr** (text keeps its quotes), so `status='pending'` looks like code.

### 5.4 Example: transactions

```text
PYTHON                                                            | JAVA
------------------------------------------------------------------+--------------------------------------------------------------------
class Transaction:                                                | import java.util.*;
    def __init__(self, id, amount):                               |
        self.id = id                                              | class Transaction implements Comparable<Transaction> {
        self.amount = amount                                      |     int id;
                                                                  |     int amount;
    # like toString()                                             |
    def __repr__(self):                                           |     Transaction(int id, int amount) {
        return f"Transaction(id={self.id}, amount={self.amount})" |         this.id = id;
                                                                  |         this.amount = amount;
    # like equals()                                               |     }
    def __eq__(self, other):                                      |
        return (isinstance(other, Transaction)                    |     @Override
                and self.id == other.id)                          |     public String toString() {
                                                                  |         return "Transaction(id=" + id + ", amount=" + amount + ")";
    # like hashCode()                                             |     }
    def __hash__(self):                                           |
        return hash(self.id)                                      |     @Override
                                                                  |     public boolean equals(Object other) {
    # like compareTo() (used by sorted)                           |         return other instanceof Transaction t && this.id == t.id;
    def __lt__(self, other):                                      |     }
        return self.amount < other.amount                         |
                                                                  |     @Override
a = Transaction(1, 500)                                           |     public int hashCode() {
b = Transaction(1, 900)                                           |         return Objects.hash(id);
c = Transaction(2, 100)                                           |     }
# Transaction(id=1, amount=500)                                   |
print(a)                                                          |     @Override
print(a == b)               # True   (same id)                    |     public int compareTo(Transaction other) {
# False  (different objects)                                      |         return Integer.compare(this.amount, other.amount);
print(a is b)                                                     |     }
# 2      (a and b count as one)                                   | }
print(len({a, b, c}))                                             |
# [Transaction(id=2, amount=100), Transaction(id=1,               | public class Main {
# amount=500)]                                                    |     public static void main(String[] args) {
print(sorted([a, c]))                                             |         Transaction a = new Transaction(1, 500);
                                                                  |         Transaction b = new Transaction(1, 900);
                                                                  |         Transaction c = new Transaction(2, 100);
                                                                  |         // Transaction(id=1, amount=500)
                                                                  |         System.out.println(a);
                                                                  |         // true
                                                                  |         System.out.println(a.equals(b));
                                                                  |         // false
                                                                  |         System.out.println(a == b);
                                                                  |         // 2
                                                                  |         System.out.println(new HashSet<>(List.of(a, b, c)).size());
                                                                  |         List<Transaction> list = new ArrayList<>(List.of(a, c));
                                                                  |         Collections.sort(list);
                                                                  |         // [Transaction(id=2, amount=100), Transaction(id=1,
                                                                  |         // amount=500)]
                                                                  |         System.out.println(list);
                                                                  |     }
                                                                  | }
```


⚠️ **Common confusion (my Practice 1 mistake):** I copied the `T2(...)` example into `__repr__` for `Transaction`, so it printed the wrong class name and left out `status`. A `__repr__` must show the **real class name and every attribute**:
```python
class Transaction:
    def __init__(self, id, amount, status="pending"):
        self.id = id
        self.amount = amount
        self.status = status

    def __repr__(self):
        return f"Transaction(id={self.id!r}, amount={self.amount!r}, status={self.status!r})"

print(Transaction(1, 500))
# Transaction(id=1, amount=500, status='pending')
```
Also: an f-string only **makes** text. `f"{t}"` on its own line shows nothing; you must `print(...)` it (in a notebook only the **last** line of a cell is shown automatically).

### 5.5 `__len__`, `__getitem__`, `__call__`
These make your objects behave like built-in things. In Java you must call named methods; in Python the **syntax** works directly.

```text
PYTHON                                             | JAVA
---------------------------------------------------+------------------------------------------------------------------
class Chunks:                                      | import java.util.*;
    def __init__(self, items):                     |
        self.items = items                         | class Chunks {
                                                   |     List<String> items;
    def __len__(self):                    # len(c) |
        return len(self.items)                     |     Chunks(List<String> items) {
                                                   |         this.items = items;
    def __getitem__(self, i):             # c[i]   |     }
        return self.items[i]                       |
                                                   |     int size() {                           // c.size()
class Doubler:                                     |         return items.size();
    # d(5): the object behaves like a function     |     }
    def __call__(self, x):                         |
        return x * 2                               |     String get(int i) {                    // c.get(i)
                                                   |         return items.get(i);
c = Chunks(["a", "b", "c"])                        |     }
print(len(c), c[1])           # 3 b                | }
d = Doubler()                                      |
print(d(5))                   # 10                 | class Doubler {
                                                   |     // d.apply(5): you must name the method
                                                   |     int apply(int x) {
                                                   |         return x * 2;
                                                   |     }
                                                   | }
                                                   |
                                                   | public class Main {
                                                   |     public static void main(String[] args) {
                                                   |         Chunks c = new Chunks(List.of("a", "b", "c"));
                                                   |         System.out.println(c.size() + " " + c.get(1));     // 3 b
                                                   |         System.out.println(new Doubler().apply(5));        // 10
                                                   |     }
                                                   | }
```


Why `__call__` matters for AI: in PyTorch, `model(x)` works because `nn.Module` defines `__call__`, which runs your `forward` method. scikit-learn and LangChain objects use the same idea.

---

## 6. Inheritance, `super()` and overriding

### 6.1 Definition
**Inheritance** lets a class reuse another class's attributes and methods. The child ("is a" parent) can **override** methods to change them. Java: `extends`.

### 6.2 Important points
- **Syntax:** `class Child(Parent):`. The parent goes in brackets.
- **`super()`** gives access to the parent. `super().__init__(...)` calls the parent's constructor, `super().method(...)` calls the parent's version of a method.
- **Python does not call the parent constructor for you.** Java inserts an implicit `super()` call if you forget; in Python, if you write your own `__init__` and skip `super().__init__(...)`, the parent's attributes are never created.
- Overriding needs **no keyword**: define a method with the same name. (Java's `@Override` is optional help; Python has no equivalent, so a typo in the name creates a *new* method silently.)
- Every class inherits from the built-in class `object` (Java's `Object`).
- Check with `isinstance(obj, Parent)` (true for children too) and `issubclass(Child, Parent)`.

### 6.3 Syntax
```
class Child(Parent):
    def __init__(self, parent_param, child_param):
        super().__init__(parent_param)      # call the parent constructor
        self.child_attr = child_param

    def method(self, ...):                  # override
        ...super().method(...)...           # optionally reuse the parent's version
```
### 6.4 Example: notifiers

```text
PYTHON                                              | JAVA
----------------------------------------------------+-------------------------------------------------------------------
class Notifier:                                     | class Notifier {
    def __init__(self, name):                       |     String name;
        self.name = name                            |
                                                    |     Notifier(String name) {
    def send(self, msg):                            |         this.name = name;
        return f"{self.name}: {msg}"                |     }
                                                    |
# EmailNotifier "is a" Notifier                     |     String send(String msg) {
class EmailNotifier(Notifier):                      |         return name + ": " + msg;
    def __init__(self, name, address):              |     }
        # call the parent constructor               | }
        super().__init__(name)                      |
        self.address = address                      | // EmailNotifier "is a" Notifier
                                                    | class EmailNotifier extends Notifier {
    # override the parent method                    |     String address;
    def send(self, msg):                            |
        return (f"EMAIL to {self.address} | "       |     EmailNotifier(String name, String address) {
                + super().send(msg))                |         // call the parent constructor
                                                    |         super(name);
n = EmailNotifier("alerts", "a@x.com")              |         this.address = address;
# EMAIL to a@x.com | alerts: Refund done            |     }
print(n.send("Refund done"))                        |
print(isinstance(n, Notifier))        # True        |     // override the parent method
print(issubclass(EmailNotifier, Notifier))   # True |     @Override
                                                    |     String send(String msg) {
                                                    |         return "EMAIL to " + address + " | " + super.send(msg);
                                                    |     }
                                                    | }
                                                    |
                                                    | public class Main {
                                                    |     public static void main(String[] args) {
                                                    |         EmailNotifier n = new EmailNotifier("alerts", "a@x.com");
                                                    |         // EMAIL to a@x.com | alerts: Refund done
                                                    |         System.out.println(n.send("Refund done"));
                                                    |         System.out.println(n instanceof Notifier);        // true
                                                    |         System.out.println(
                                                    |             // true
                                                    |             Notifier.class.isAssignableFrom(EmailNotifier.class));
                                                    |     }
                                                    | }
```


⚠️ **Common confusion: forgetting `super().__init__(...)`.**
```python
class Bad(Notifier):
    def __init__(self, name, address):
        self.address = address            # forgot super().__init__(name)

n = Bad("alerts", "a@x.com")
n.send("hi")      # AttributeError: 'Bad' object has no attribute 'name'
```
In Java the same code would work because `super()` is called implicitly (as long as the parent has a no-argument constructor). In Python, always call it.

---

## 7. Polymorphism and duck typing

### 7.1 Definition
**Polymorphism** = one call, different behaviour depending on the object. `n.send("hi")` does something different for an `Email`, an `Sms` or a base `Notifier`.

### 7.2 Polymorphism through inheritance (same idea as Java)

```text
PYTHON                                 | JAVA
---------------------------------------+---------------------------------------------------------------------
class Notifier:                        | class Notifier {
    def send(self, msg):               |     String send(String msg) { return "base: " + msg; }
        return "base: " + msg          | }
                                       |
class Email(Notifier):                 | class Email extends Notifier {
    def send(self, msg):               |     @Override String send(String msg) { return "EMAIL: " + msg; }
        return "EMAIL: " + msg         | }
                                       |
class Sms(Notifier):                   | class Sms extends Notifier {
    def send(self, msg):               |     @Override String send(String msg) { return "SMS: " + msg; }
        return "SMS: " + msg           | }
                                       |
# one loop, three different behaviours | public class Main {
for n in [Email(), Sms(), Notifier()]: |     public static void main(String[] args) {
    print(n.send("hi"))                |         // parent type holds children
# EMAIL: hi                            |         Notifier[] all = { new Email(), new Sms(), new Notifier() };
# SMS: hi                              |         for (Notifier n : all) {
# base: hi                             |             System.out.println(n.send("hi"));
                                       |         }
                                       |         // EMAIL: hi
                                       |         // SMS: hi
                                       |         // base: hi
                                       |     }
                                       | }
```


### 7.3 Duck typing (the Python difference)
Python does not need a shared parent or `implements`. If the object **has the method**, you can call it. ("If it walks like a duck and quacks like a duck, it is a duck.")

In Java, `Email` and `Slack` must both say `implements Sender` before one method can accept them. In Python they do nothing special. A `Protocol` (Python's interface) only **describes** the shape for readers and type checkers; classes do not have to declare it.

```text
PYTHON                                     | JAVA
-------------------------------------------+----------------------------------------------------------------------
from typing import Protocol                | import java.util.*;
                                           |
# like a Java interface                    | interface Sender {                                    // interface
class Sender(Protocol):                    |     String send(String msg);
    def send(self, msg: str) -> str: ...   | }
                                           |
# does NOT say "implements Sender"         | // must say "implements Sender"
class Email:                               | class Email implements Sender {
    def send(self, msg):                   |     public String send(String msg) {
        return f"EMAIL: {msg}"             |         return "EMAIL: " + msg;
                                           |     }
# also no declaration                      | }
class Slack:                               |
    def send(self, msg):                   | // must say it here too
        return f"SLACK: {msg}"             | class Slack implements Sender {
                                           |     public String send(String msg) {
# works with anything that has send()      |         return "SLACK: " + msg;
def broadcast(senders: list[Sender], msg): |     }
    return [s.send(msg) for s in senders]  | }
                                           |
# ['EMAIL: hi', 'SLACK: hi']               | public class Main {
print(broadcast([Email(), Slack()], "hi")) |     static List<String> broadcast(List<Sender> senders, String msg) {
                                           |         List<String> out = new ArrayList<>();
                                           |         for (Sender s : senders) {
                                           |             out.add(s.send(msg));
                                           |         }
                                           |         return out;
                                           |     }
                                           |
                                           |     public static void main(String[] args) {
                                           |         List<Sender> all = List.of(new Email(), new Slack());
                                           |         // [EMAIL: hi, SLACK: hi]
                                           |         System.out.println(broadcast(all, "hi"));
                                           |     }
                                           | }
```


Important points:
- `Protocol` is checked by tools like mypy, not at run time. The program works even without it.
- Duck typing is why Python libraries are flexible: anything with `.fit()` and `.predict()` works as a scikit-learn model; anything with `.read()` works as a file.
- The risk: a typo in a method name shows up only when that line runs. Tests (Topic 10) and type hints (Topic 09) protect you.
🎯 **Interview:** polymorphism via inheritance and via duck typing; `Protocol` vs `ABC`.

---

## 8. Abstract classes and interfaces

### 8.1 Definition
An **abstract class** is a class you cannot create objects from. It defines methods that child classes **must** implement. Java: `abstract class`. Python: inherit from `ABC` and mark methods with `@abstractmethod`.

| Java | Python |
|---|---|
| An `abstract class` can have both abstract methods (no body) and normal methods, and `new` on it is a compile error. | A class that inherits `ABC` can have both `@abstractmethod` methods and normal ones, and creating it raises `TypeError` when the program runs. |
| An `interface` lists methods that a class promises to provide, and the class must write `implements`. | A `Protocol` lists the methods an object should have, and classes do not need to declare anything; an `ABC` with only abstract methods is the "must inherit" version. |

### 8.2 Syntax
```
from abc import ABC, abstractmethod

class Base(ABC):
    @abstractmethod
    def required_method(self, ...): ...        # no body

class Child(Base):
    def required_method(self, ...):            # must implement it
        ...
```
### 8.3 Example: a chunker for a RAG pipeline

```text
PYTHON                                                   | JAVA
---------------------------------------------------------+---------------------------------------------------------------------
from abc import ABC, abstractmethod                      | import java.util.*;
                                                         |
# abstract class                                         | // abstract class
class Chunker(ABC):                                      | abstract class Chunker {
    # abstract method: no body                           |     // abstract method: no body
    @abstractmethod                                      |     abstract List<String> chunk(String text);
    def chunk(self, text):                               | }
        ...                                              |
                                                         | class FixedChunker extends Chunker {
class FixedChunker(Chunker):                             |     int size;
    def __init__(self, size):                            |
        self.size = size                                 |     FixedChunker(int size) {
                                                         |         this.size = size;
    # must implement it                                  |     }
    def chunk(self, text):                               |
        return [text[i:i + self.size]                    |     // must implement it
                for i in range(0, len(text), self.size)] |     @Override
                                                         |     List<String> chunk(String text) {
try:                                                     |         List<String> out = new ArrayList<>();
    # cannot create an abstract class                    |         for (int i = 0; i < text.length(); i += size) {
    Chunker()                                            |             int end = Math.min(i + size, text.length());
except TypeError as e:                                   |             out.add(text.substring(i, end));
    # Can't instantiate abstract class Chunker           |         }
    # without an implementation for abstract method      |         return out;
    # 'chunk'                                            |     }
    print(e)                                             | }
                                                         |
# ['abcd', 'efgh', 'ij']                                 | public class Main {
print(FixedChunker(4).chunk("abcdefghij"))               |     public static void main(String[] args) {
                                                         |         // compile error: Chunker is abstract; cannot be
                                                         |         // instantiated
                                                         |         // new Chunker();
                                                         |         // [abcd, efgh, ij]
                                                         |         System.out.println(new FixedChunker(4).chunk("abcdefghij"));
                                                         |     }
                                                         | }
```


Important points:
- If a child forgets to implement an abstract method, **it** cannot be created either (`TypeError`).
- Use an `ABC` when you want to **force** a shared base (and share code in it). Use a `Protocol` or plain duck typing when you only care that the method exists.
- Real example: you can define `Embedder` and `Retriever` ABCs, then swap OpenAI, a local model or a fake one in tests without changing the pipeline.

---

## 9. Composition vs inheritance

### 9.1 Definition
- **Inheritance** = "is a" (`EmailNotifier` **is a** `Notifier`).
- **Composition** = "has a" (a `Pipeline` **has a** chunker). One object holds another object as an attribute.

### 9.2 Important points
- Prefer **composition** unless "is a" is really true. It is easier to change, test and swap.
- Inheritance ties the child tightly to the parent; a change in the parent can break every child.
- The "has a" part is often passed in through `__init__` (called **dependency injection**). FastAPI's `Depends` uses the same idea.

### 9.3 Example

```text
PYTHON                                                   | JAVA
---------------------------------------------------------+-----------------------------------------------------------
class FixedChunker:                                      | import java.util.*;
    def __init__(self, size):                            |
        self.size = size                                 | class FixedChunker {
                                                         |     int size;
    def chunk(self, text):                               |
        return [text[i:i + self.size]                    |     FixedChunker(int size) {
                for i in range(0, len(text), self.size)] |         this.size = size;
                                                         |     }
# a Pipeline HAS a chunker                               |
class Pipeline:                                          |     List<String> chunk(String text) {
    def __init__(self, chunker):                         |         List<String> out = new ArrayList<>();
        self.chunker = chunker                           |         for (int i = 0; i < text.length(); i += size) {
                                                         |             int end = Math.min(i + size, text.length());
    def run(self, text):                                 |             out.add(text.substring(i, end));
        cleaned = " ".join(text.lower().split())         |         }
        return self.chunker.chunk(cleaned)               |         return out;
                                                         |     }
p = Pipeline(FixedChunker(5))                            | }
# ['hello', ' worl', 'd']                                |
print(p.run("  Hello   WORLD  "))                        | // a Pipeline HAS a chunker
                                                         | class Pipeline {
                                                         |     FixedChunker chunker;
                                                         |
                                                         |     Pipeline(FixedChunker chunker) {
                                                         |         this.chunker = chunker;
                                                         |     }
                                                         |
                                                         |     List<String> run(String text) {
                                                         |         String cleaned = String.join(
                                                         |             " ", text.toLowerCase().trim().split("\\s+"));
                                                         |         return chunker.chunk(cleaned);
                                                         |     }
                                                         | }
                                                         |
                                                         | public class Main {
                                                         |     public static void main(String[] args) {
                                                         |         Pipeline p = new Pipeline(new FixedChunker(5));
                                                         |         // [hello,  worl, d]
                                                         |         System.out.println(p.run("  Hello   WORLD  "));
                                                         |     }
                                                         | }
```


A bigger one that mixes everything so far (Python only; it runs): an ABC, a concrete class, composition and a fake embedder you can swap for a real model.
```python
from abc import ABC, abstractmethod

class Embedder(ABC):
    @abstractmethod
    def embed(self, text): ...

class FakeEmbedder(Embedder):
    def embed(self, text):
        return [float(len(text)), float(text.count(" "))]

class Chunker:
    def __init__(self, size):
        self.size = size
    def chunk(self, text):
        return [text[i:i + self.size] for i in range(0, len(text), self.size)]

class Pipeline:
    def __init__(self, chunker, embedder):          # composition: Pipeline HAS these parts
        self.chunker = chunker
        self.embedder = embedder

    def run(self, text):
        return [(c, self.embedder.embed(c)) for c in self.chunker.chunk(text)]

p = Pipeline(Chunker(8), FakeEmbedder())
for chunk, vec in p.run("refund not received"):
    print(repr(chunk), vec)
# 'refund n' [8.0, 1.0]
# 'ot recei' [8.0, 1.0]
# 'ved' [3.0, 0.0]
```
To use a real embedding model later, write `OpenAIEmbedder(Embedder)` and pass it in. `Pipeline` does not change.

---

## 10. Multiple inheritance, mixins and MRO

### 10.1 Definition
Python lets a class inherit from **several** classes: `class Service(Loggable, Cacheable):`. Java classes can have only **one** parent (`extends`), but can implement **many interfaces**.

### 10.2 Mixins
A **mixin** is a small class that adds one ability (logging, caching) and is meant to be combined with others. It is Python's version of Java interfaces with `default` methods.

```text
PYTHON                               | JAVA
-------------------------------------+--------------------------------------------------------------
class Loggable:                      | interface Loggable {
    def log(self, m):                |     default String log(String m) { return "LOG: " + m; }
        return "LOG: " + m           | }
                                     |
class Cacheable:                     | interface Cacheable {
    def cache(self, k):              |     default String cache(String k) { return "CACHED: " + k; }
        return "CACHED: " + k        | }
                                     |
# inherits from TWO classes          | // many interfaces: allowed
class Service(Loggable, Cacheable):  | class Service implements Loggable, Cacheable {
    pass                             | // (class Service extends A, B is NOT allowed)
                                     | }
s = Service()                        |
print(s.log("x"))        # LOG: x    | public class Main {
print(s.cache("y"))      # CACHED: y |     public static void main(String[] args) {
                                     |         Service s = new Service();
                                     |         System.out.println(s.log("x"));      // LOG: x
                                     |         System.out.println(s.cache("y"));    // CACHED: y
                                     |     }
                                     | }
```


### 10.3 MRO: which parent wins?
**MRO** (Method Resolution Order) is the order Python searches for a method: the class itself, then its parents left to right, with each class appearing once and always before its own parents. `super()` follows this order, and `Class.__mro__` shows it.

```text
PYTHON                                 | JAVA
---------------------------------------+--------------------------------------------------------------------
class A:                               | // COMPILE ERROR: a class can have only ONE parent
    def hello(self):                   | // class D extends B, C { }
        return "A"                     | //
                                       | // Java avoids the "diamond problem" by allowing only one parent
class B(A):                            | // class.
    def hello(self):                   | // For "many abilities", use interfaces:  class D implements I1, I2
        return "B->" + super().hello() | // { }
                                       |
class C(A):                            |
    def hello(self):                   |
        return "C->" + super().hello() |
                                       |
class D(B, C):                         |
    def hello(self):                   |
        return "D->" + super().hello() |
                                       |
# D->B->C->A                           |
print(D().hello())                     |
# ['D', 'B', 'C', 'A', 'object']       |
print([k.__name__ for k in D.__mro__]) |
```


Reading the output: `D().hello()` runs `D.hello`, whose `super()` goes to the next class in the MRO (`B`), whose `super()` goes to `C` (not straight to `A`), and so on. Rule: `super()` means "the **next class in the MRO**", not "the parent".
🎯 **Interview:** what is MRO? What is the diamond problem? How does Python solve it (C3 linearization)? Keep multiple inheritance to **mixins**.

---

## 11. Dataclass preview (Java `record`)
Writing `__init__`, `__repr__` and `__eq__` by hand for a simple data holder is boring. A **dataclass** writes them for you. (Full coverage with Pydantic is Topic 09.)

```text
PYTHON                                         | JAVA
-----------------------------------------------+-----------------------------------------------------------------
from dataclasses import dataclass              | record Point(int x, int y) { }
                                               |
@dataclass                                     | public class Main {
class Point:                                   |     public static void main(String[] args) {
    x: int                                     |         Point p = new Point(1, 2);
    y: int                                     |         // Point[x=1, y=2]
                                               |         System.out.println(p);
p = Point(1, 2)                                |         System.out.println(p.equals(new Point(1, 2)));   // true
print(p)                     # Point(x=1, y=2) |     }
print(p == Point(1, 2))      # True            | }
```


Note: `x: int` is a **type hint** (a label for readers and tools); Python does not enforce it.

---

## 12. Things Java has that Python does not (or does differently)

| Java | Python |
|---|---|
| Methods can be overloaded, meaning several methods with the same name and different parameter lists. | There is no overloading; the last `def` with a name wins, so use default values or `*args`. |
| `final` stops a variable from being reassigned or a class from being extended. | There is no `final`; by convention constants are written `UPPER_CASE`, and `typing.Final` helps type checkers only. |
| Generics such as `List<String>` are part of the language and checked at compile time. | Type hints such as `list[str]` are optional labels and are ignored when the program runs. |
| Checked exceptions force you to declare or catch them. | There are no checked exceptions; you decide where to use `try` / `except` (Topic 08). |
| `null` is the "no object" value and calling a method on it throws `NullPointerException`. | `None` is the "no value" object, and calling a missing method on it raises `AttributeError`; check with `x is None`. |
| `public static void main(String[] args)` is the entry point. | The entry point is any file you run, usually guarded by `if __name__ == "__main__":` (Topic 04). |
| Each public class lives in its own file named after the class. | Many classes can live in one file, and the file is a module (Topic 04). |
| Fields, methods and types must be declared before use. | Attributes can be added at any time, which is flexible but easy to misuse. |

---

## 13. Patterns to remember

**Pattern 1: a standard class template**
```
class Name:
    CONSTANT = ...                         # class attribute: constants only
    def __init__(self, a, b=default):
        self.a = a                         # all changing data here, with self.
        self.items = []                    # lists/dicts created per object
    def __repr__(self):
        return f"Name(a={self.a!r}, ...)"
    def action(self, ...):
        ...
```
**Pattern 2: alternative constructor**
```
@classmethod
def from_xxx(cls, data):
    return cls(data["a"], data.get("b", default))
```
**Pattern 3: validated attribute**
```
@property
def x(self): return self._x
@x.setter
def x(self, v):
    if bad(v): raise ValueError("message")
    self._x = v
# in __init__:  self.x = x      (no underscore, so the setter runs)
```
**Pattern 4: required interface (swap parts)**
```
class Base(ABC):
    @abstractmethod
    def do(self, ...): ...
class Impl(Base):
    def do(self, ...): ...
class Pipeline:
    def __init__(self, part: Base):        # composition: pass in the part
        self.part = part
```
**Pattern 5: child class**
```
class Child(Parent):
    def __init__(self, parent_arg, own_arg):
        super().__init__(parent_arg)       # never forget this
        self.own = own_arg
```
**Debug recipe:** `obj.__dict__` (what attributes does it have?), `type(obj)`, `isinstance(obj, Class)`, `Class.__mro__`, and `print(repr(obj))`.

---

## 14. Use this when building (project guide)

- **Pydantic / FastAPI:** request and response models are classes (`class Ticket(BaseModel): id: int; text: str`). Pydantic is a dataclass plus validation, so Topic 09 builds directly on this file.
- **scikit-learn:** every model is a class with `.fit()` and `.predict()` (duck typing). Your own transformer is a class with those methods.
- **PyTorch:** `class Net(nn.Module):` with `__init__` (create layers, call `super().__init__()`) and `forward` (use them). `model(x)` calls `__call__`, which calls `forward`.
- **LangChain and similar frameworks:** retrievers, loaders and splitters are classes with a common method (`invoke`, `split_text`). You swap them because of polymorphism.
- **RAG project:** `Chunker`, `Embedder`, `Retriever` and `Pipeline` classes, built with composition and an ABC or Protocol (section 9), so every part can be tested and replaced.
- **State and shared data:** a class attribute that is a list or dict is shared by every object and every request (a real FastAPI bug). Put per-object data in `__init__`.

**Checklist for any class:** Every method starts with `self`? Every attribute is `self.name`? Lists and dicts created in `__init__`, not in the class body? A useful `__repr__`? Called `super().__init__(...)` in a child? Used composition instead of inheritance unless "is a" is true? Used `@property` instead of `get_x`/`set_x`?

---

## 🧠 Quick Revision

- Class = blueprint, object = instance. `__init__` is the constructor; `self` is Java's `this` and must be the first parameter of every instance method. No `new`: call `Class(...)`.
- No field declarations: `self.x = ...` creates the attribute. No overloading: use default values.
- Forgetting `self.` makes a local variable (`owner = owner` stores nothing); forgetting `self` in the parameters gives `TypeError`.
- Instance attributes (`self.x`, one per object) vs class attributes (class body, shared; Java `static`). A **list or dict in the class body is shared by all objects**: create it in `__init__`.
- Instance method (`self`), class method (`@classmethod`, `cls`, alternative constructors like `from_csv`), static method (`@staticmethod`, no self/cls).
- Privacy is a convention: `_x` internal, `__x` name-mangled; nothing is truly private. Use `@property` + `@x.setter` for validation, and set `self.x = ...` (no underscore) in `__init__` so the setter runs at creation.
- Dunders: `__repr__`/`__str__` (toString), `__eq__` + `__hash__` (equals/hashCode), `__lt__` (compareTo), `__len__`, `__getitem__`, `__call__`. `==` compares values in Python; `is` compares identity.
- `__repr__` shows the real class name and every attribute. An f-string must be printed to be seen.
- Inheritance: `class Child(Parent)`, call `super().__init__(...)`, override by redefining the method, `isinstance` / `issubclass`. `super()` means "next class in the MRO".
- Polymorphism works through inheritance and through **duck typing** (no `implements` needed). `Protocol` describes a shape; `ABC` + `@abstractmethod` forces children to implement methods (`TypeError` if you create the abstract class).
- Prefer composition ("has a") over inheritance ("is a"); pass parts in through `__init__`.
- Multiple inheritance is allowed (mixins); `Class.__mro__` shows the search order. Java allows one parent class but many interfaces.
- `@dataclass` writes `__init__`, `__repr__`, `__eq__` for you (Java `record`), Topic 09.
- Python checks types and visibility at run time and has no overloading, `final`, generics enforcement or checked exceptions.