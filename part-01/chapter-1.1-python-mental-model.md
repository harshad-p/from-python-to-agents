# Chapter 1.1 — Python mental model

## Teaching

### Concept

When you run Python, an **interpreter** reads your source and executes it. The usual interpreter is **CPython**, the one started by `python` or `python3`.

While that program runs, the values you work with are **objects**. A number, a string, a list, a function, and `None` are all objects. Each object has a type. A **variable** is a name bound to an object. The name has no type of its own.

Some objects can be changed after they are created. Those are **mutable**. Others cannot. Those are **immutable**. Two names can refer to one object, so a change made through one name is visible through the other.

### Why

You already know how a C# local works: the compiler gives it a fixed type, and assignment of an `int` or a `struct` copies the value, while assignment of a `class` copies a reference. Python uses one rule for every assignment, and it checks types when a line runs.

That difference shows up as soon as you pass a list or a dictionary into a function, compare two values, or test for a missing result. This lesson is the model those later chapters rely on.

### Explanation

#### How a Python program runs

You write a `.py` file and hand it to the interpreter:

```bash
python script.py
```

You can also start `python` with no file. That opens the **REPL** (read-eval-print loop): you type a statement, the interpreter runs it, and it prints the result. Handy for trying a few lines.

CPython does compile your source, though you do not run a separate build. It translates the file into **bytecode**, a form its own virtual machine can execute, and then runs that bytecode. Failed operations, including operations that the object's type does not support, are reported when that line runs.

Unchanged files leave cached bytecode in a `__pycache__` directory, as `.pyc` files. That cache lets the next run skip recompiling. It is not an assembly you ship. You can ignore it for now.

#### Names and objects

Assignment binds a name to an object. It does not declare a type for the name, and it does not copy the object.

```python
status = "pending"
status = 200
```

`print` writes a readable form of a value to the output. After the first line, `print(status)` writes `pending`. After the second, the same name refers to the integer `200`, so `print(status)` writes `200`. The string `"pending"` was not converted. The name moved to a different object.

This is **dynamic typing**: the type belongs to the object, and Python checks an operation against that type at runtime. `len` is a built-in function that asks an object how many items it contains. A string has a length. An integer does not.

```python
status = "pending"
status = 200
length = len(status)  # TypeError: an int has no length
```

C# `var` is a different mechanism. `var status = "pending";` asks the compiler to infer `string` and then keep that type. Python does not lock `status` to `str` on the first assignment. Types are still real: `"pending"` is a `str`, and `200` is an `int`.

#### One assignment rule

A **list** is an ordered collection, written in square brackets. This creates one list object and binds `primary_regions` to it:

```python
primary_regions = ["eu-west-1", "us-east-1"]
active_regions = primary_regions
```

The second line binds `active_regions` to that same list. There is no second list.

A **method** is a function you call on an object with a dot, the same shape as an instance method in C#. `append` adds one item at the end of a list and changes that list.

```python
active_regions.append("ap-southeast-1")
print(primary_regions)
# ['eu-west-1', 'us-east-1', 'ap-southeast-1']
```

`primary_regions` was never assigned on that last step. It still refers to the original list, and `append` changed the list.

The closest C# picture is two `List<string>` variables referring to one instance. The picture that misleads is "assignment copies the value," which is true for a C# `int` or `struct` and false for every Python assignment. Copying in Python is a separate call, such as `some_list.copy()`.

#### Mutability

A mutable object can change after it is created. Lists are mutable, and so are **dictionaries** and **sets**, which later lessons cover properly. A dictionary maps keys to values and is written with braces: `{"Authorization": "Bearer token"}`.

An immutable object cannot change. Integers, strings, and tuples work this way. An expression that looks like an update builds a new object, and assignment rebinds the name.

```python
retry_count = 3
retry_count = retry_count + 1
```

The integer `3` is untouched. `retry_count` now refers to a different integer, `4`.

Strings follow the same rule. A method call such as `"ab".upper()` returns a new string, `"AB"`. The original `"ab"` stays as it was. If you do not bind the result, the new string is discarded.

#### Equality and identity

**Equality** asks whether two objects have the same value. You write that with `==`.

**Identity** asks whether two names refer to one object. You write that with `is`.

```python
configured = ["cache", "database"]
requested = ["cache", "database"]
alias = configured

configured == requested  # True: same contents
configured is requested  # False: two list objects
configured is alias      # True: one list object
```

Compare values with `==`. Use `is` when you mean the same object. In application code that almost always means a check for `None`.

CPython reuses one object for some small integers and some strings, so this can be true:

```python
a = 256
b = 256
a is b  # often True, because both names were bound to a cached object
```

That reuse is a property of this interpreter, and it is not a rule you should code against. `256 == 256` is the comparison that asks about the value.

#### `None`

`None` is a single object meaning "no value" or "not present." It plays the role `null` plays in C#. Because there is only one `None`, the accurate test is identity:

```python
cached_result = None

if cached_result is None:
    print("Cache miss")
```

`if` runs its indented block when the condition holds. You will look at branches in detail in a later lesson. For now, the condition to remember is `is None`, and the negative form `is not None`.

`== None` often gives the same answer, and it can also give a different one, because a class is allowed to define its own equality. `is` ignores that definition and only checks whether both sides are the `None` object. That is the check to write.

#### Truthiness

A condition also accepts values that are not booleans. Python asks the object whether it should count as true or false. `bool(value)` returns that answer as `True` or `False`. An `if` asks the same question on its own, so you can write `if errors:` without calling `bool`.

These values are **falsy**:

- `False`
- `None`
- zero, including `0` and `0.0`
- empty text and empty collections: `""`, `[]`, `{}`, `()`

Other objects are **truthy**, including non-empty collections and numbers other than zero.

```python
errors = []

if not errors:
    print("No validation errors")
```

`not` inverts the truthiness. An empty list is falsy, so `not errors` is true, and the message prints. That is a good fit when an empty collection and a missing one mean the same thing in the domain.

It is a poor fit when those states mean different things. A retry count of `0` is real data, and it is falsy. An omitted field and an empty list are also different facts. Say so directly:

```python
if payload is None:
    print("field omitted")
elif not payload:
    print("field present, but empty")
```

`elif` is the branch that runs when the earlier condition was false and this one is true. Explicit checks like these match a Python habit you will see everywhere: say the thing you mean. When absence matters, write `is None`. When emptiness matters, test the empty value. Use bare truthiness when those really are the same case.

#### Conventions

Python's style guide is **PEP 8**. Variables and functions use `snake_case`. Classes use `PascalCase`. Constants use `UPPER_SNAKE_CASE`. Indent with four spaces.

Indentation is syntax. The indented lines under `if` or `def` are the block. There are no braces around it.

A block starts with a colon. This defines a function named `add_default_headers`. Calling it runs the indented body, with the argument bound to the parameter `headers`:

```python
def add_default_headers(headers):
    headers["X-Service"] = "billing"
```

`headers["X-Service"] = "billing"` stores a value under that key. Dictionaries are mutable, so this changes the existing dictionary.

### Examples

The function above and the caller's variable can be two names for one dictionary:

```python
def add_default_headers(headers):
    headers["X-Service"] = "billing"

request_headers = {"Authorization": "Bearer token"}
add_default_headers(request_headers)

print(request_headers)
# {'Authorization': 'Bearer token', 'X-Service': 'billing'}
```

`add_default_headers` receives a name for the same dictionary the caller holds. Setting a key changes that object, so `request_headers` shows `X-Service` after the call. That is what you want when the function's job is to update the caller's mapping. It is a surprise when the caller expected the original mapping to stay as it was.

To leave the input alone, make a new dictionary and return it. `copy()` returns a new dictionary with the same keys and values. For a flat dictionary like this one, later changes to the copy stay on the copy.

```python
def with_default_headers(headers):
    result = headers.copy()
    result["X-Service"] = "billing"
    return result
```

`return` hands that dictionary back to the caller. The caller's original dictionary has no `X-Service` key unless the caller binds the returned object and uses that.

### Exercise

Predict every printed line without running the code. Say whether `regions` changes, and for each line say which idea explains it: name binding, mutability, equality, identity, `None`, or truthiness.

```python
regions = ["eu", "us"]
replica = regions
snapshot = ["eu", "us"]

replica.append("ap")

print(regions)
print(regions == snapshot)
print(regions is replica)
print(snapshot is regions)

value = None
print(value is None)
print(bool(value))
print(bool(regions))
```

### Checkpoint

You are ready to continue when you can explain:

1. Why `replica.append("ap")` changes what `print(regions)` shows.
2. Why `regions == snapshot` and `snapshot is regions` answer different questions.
3. Why a missing value is tested with `is None`.
4. When `if not errors` matches the domain, and when `is None` is the clearer test.

Next lesson: **Chapter 1.2 — Basic syntax**.
