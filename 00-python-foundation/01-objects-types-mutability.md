# Objects, Types and Mutability

## 1. Definition
In Python everything is an object. A variable is a name bound to an object, not a container of a value.

## 2. Why it exists
Uniform object model gives flexibility. Assignment binds names, so it is cheap and never copies data.

## 3. Mental model
Objects = items in a warehouse. Variables = sticky labels. `b = a` adds a second label to the same item.

## 4. How it works internally
- Every object has identity (id), type, value.
- `is` compares identity; `==` compares value.
- Reference counting frees an object when its count reaches 0.
- Small ints and some strings are cached, so `is` can mislead.

## 5. Important terminology
- **Mutable:** can change in place (list, dict, set).
- **Immutable:** cannot change in place (int, str, tuple).
- **Reference:** a name pointing to an object.
- **Shallow copy:** new outer object, shared inner objects.
- **Deep copy:** fully independent copy.

## 6. Example
```python
a = [1, 2]; b = a; b.append(3)   # a is [1, 2, 3]
```

## 7. Code example
```python
import copy
a = [[1], [2]]
b = a.copy()              # shallow
c = copy.deepcopy(a)      # deep

# mutable default fix
def add_item(item, bucket=None):
    if bucket is None:
        bucket = []
    bucket.append(item)
    return bucket

# 2D grid
grid = [[0] * 3 for _ in range(3)]   # NOT [[0]*3]*3
```

## 8. Real-world usage
Safe defaults in functions; avoiding shared state in APIs; understanding views vs copies in Pandas/NumPy.

## 9. Common mistakes
- Mutable default arguments (`def f(x, l=[])`)
- `is` instead of `==`
- `[[0]*3]*3` aliasing (all rows are the same list)
- Mutating a list while iterating
- `t[1] += [5]` on a tuple holding a list: raises TypeError but the list still changes

## 10. Performance considerations
- Assignment is O(1) (no copy). Copies cost time and memory.
- Strings are immutable, so repeated `+=` in a loop is slow. Use `"".join(parts)`.

## 11. Security considerations
Shared mutable state across requests can leak one user's data to another.

## 12. Interview questions
- `is` vs `==`?
- Mutable default argument problem and fix?
- Shallow vs deep copy?
- Tuple containing a list: can it change?
- Why does `t[1] += [5]` raise an error but still change the list?

## 13. Things to remember
- Use `is` only for `None`.
- Default to `None`, then create the mutable object inside the function.
- `b = a` never copies.
- A tuple's immutability covers which objects it points to, not the contents of those objects.

## 14. One-page revision summary
Names point to objects. Mutable objects change in place, so aliases see the change. Immutable objects are replaced, so aliases are safe. Use `copy.deepcopy` for independence. Never use mutable defaults.