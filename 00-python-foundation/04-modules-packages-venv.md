# Modules, Packages, pip and Virtual Environments

## 1. Definition
- **Module:** one `.py` file. It holds code (functions, variables, classes) that other files can reuse.
- **Package:** a **folder of modules**, usually with an `__init__.py` file.
- **Import:** bringing code from a module or package into the file you are working in.
- **pip:** the tool that downloads and installs libraries written by other people.
- **Virtual environment (venv):** a private copy of Python plus its own installed libraries, **one per project**.
- **`requirements.txt` / `pyproject.toml`:** files that record which libraries your project needs, so anyone can rebuild the same environment.

## 2. Why it exists
- **Organise code.** One huge file becomes unreadable. Splitting into modules and packages keeps each file small and focused.
- **Reuse code.** Write `clean_text` once, import it everywhere.
- **Use other people's code.** Libraries such as Pandas, FastAPI and requests are packages you install with pip.
- **Avoid clashes between projects.** Project A needs Pandas 1.5, Project B needs Pandas 2.2. A virtual environment per project keeps them apart.
- **Reproducibility.** A requirements file lets a teammate (or a server) rebuild the exact same setup.

Every Track 2 project (FastAPI, RAG, Pandas pipelines) is many files plus many installed libraries. This topic is how they fit together.

## 3. Mental model
| Idea | Picture |
|---|---|
| Module | One **toolbox**. |
| Package | A **cupboard of toolboxes** (a folder of modules). |
| `import` | "Bring that toolbox to my desk." |
| `sys.path` | The **list of places** Python looks for toolboxes, in order. |
| pip | A **shop** (PyPI) that delivers new toolboxes to your computer. |
| Virtual environment | A **separate kitchen for each project**: ingredients installed in one kitchen do not appear in another. |
| `requirements.txt` | The **shopping list** that lets someone stock a new kitchen the same way. |

## 4. How it works internally

### 4.1 Modules
A module is just a file. Make `helpers.py`:
```python
# helpers.py
print("helpers is loading")
PI = 3.14159

def hello(name):
    return f"Hello {name}"
```
Use it from another file or a notebook:
```python
import helpers
print(helpers.hello("Ann"), helpers.PI)     # Hello Ann 3.14159
```

**The file name has `.py`, but the module name does not.**
- File: `helpers.py`
- Module (in an import): `helpers`

**Common confusion: `import helpers.py`.** This fails with `ModuleNotFoundError: No module named 'helpers.py'; 'helpers' is not a package`. A **dot in an import means "inside a package"**. So Python reads `helpers.py` as "the module `py` inside the package `helpers`". It found `helpers` (a single file, not a folder), saw it is not a package, and gave up. Write `import helpers`.

**Common confusion: `print` vs `return` in a function.**
```python
def hello(name):
    print("My name:", name)        # shows text, returns None
print(helpers.hello("Ann"), helpers.PI)
# My name: Ann
# None 3.14159          <- the stray None is the clue
```
`print` only displays text. `return` hands a value back so the caller can store or use it. A function that only prints returns `None`. Use `return` for logic and `print` only for display. Fixed: `return f"My name: {name}"`.

### 4.2 Ways to import
| Style | Example | How you use it |
|---|---|---|
| Import the module | `import helpers` | `helpers.hello("Ann")` |
| Import with a short name | `import helpers as h` | `h.hello("Ann")` |
| Import only some names | `from helpers import hello, PI` | `hello("Ann")` |
| Import everything (avoid) | `from helpers import *` | Names get mixed up, and it is unclear where each came from. |

Use `import x` or `from x import y`. Avoid `import *`.

**Import order convention (PEP 8):** group imports in three blocks separated by a blank line: standard library, then third-party libraries, then your own modules.
```python
import os
import sys

import requests

from textkit import clean_text
```

### 4.3 What happens when you `import`
1. Python **finds** the module (see 4.5).
2. It **runs the whole file once**, from top to bottom (so the `print("helpers is loading")` runs).
3. It stores the module in a cache, the dictionary `sys.modules`.
4. Later imports of the same module **reuse the cache** and do not run the file again.

Test it: import `helpers` twice in different cells. "helpers is loading" prints only once.

**Consequence in notebooks:** if you edit `helpers.py` and import it again in the same running kernel, Python gives you the **old cached version**. Fixes:
```python
# Option 1: restart the kernel (clears everything)
# Option 2: reload just that module
import importlib
importlib.reload(helpers)
# Option 3: auto-reload on every cell run (an IPython "magic command", starts with %)
%load_ext autoreload
%autoreload 2
```
Importing **runs code**, so only import modules you trust.

**Speed note.** Python also saves a compiled version of each module in a `__pycache__` folder (`.pyc` files) so the next start is faster. You never edit it, and it goes in `.gitignore`. Heavy libraries (Pandas, PyTorch) take a noticeable time to import, so import them once at the top of your file, not inside loops.

### 4.4 `if __name__ == "__main__":`
Every module has a built-in variable `__name__`.

| How the file is used | Value of `__name__` | Does the guarded code run? |
|---|---|---|
| Run directly: `python helpers.py` | `"__main__"` | Yes. |
| Imported: `import helpers` | `"helpers"` (the module name) | No. |

```python
# helpers.py
def hello(name):
    return f"Hello {name}"

if __name__ == "__main__":
    print(hello("test"))        # runs only when you run the file directly
```
Verify it yourself: add `print("__name__ is", __name__)` to the file. Running it directly prints `__name__ is __main__`. Importing it prints `__name__ is helpers`.

**Why it matters:** you can keep demo or test code (or start a server) in a file without it running by accident every time the file is imported. If you import a script and **nothing prints**, that is the guard doing its job: `import main` runs `main.py`, but `__name__` is `"main"`, so the guarded block is skipped.

### 4.5 How Python finds a module: `sys.path`
`sys.path` is a **list of folders** Python searches, in this order:
1. The folder of the script you ran (for a notebook, the notebook's own folder).
2. Folders in the `PYTHONPATH` setting (rarely used).
3. The standard library (built-in modules such as `os`, `json`).
4. `site-packages`: the folder where `pip install` puts libraries (inside your `.venv`).

```python
import sys, os
print(sys.executable)       # the Python that is running right now
print(sys.path)             # the folders searched
print(os.getcwd())          # the folder the notebook or script runs in
print(os.listdir())         # files in that folder
```
If a module is in none of those folders, you get **`ModuleNotFoundError`**. It means "I searched every folder and found nothing with that name".

**Common confusion: `ModuleNotFoundError` for your own file.** The usual reason is that the file is **not in the folder the notebook is running in** (for example `helpers.py` is in `practice\` but the notebook is in `practice\00-python-foundation\`). Python does not look in the parent folder. Fixes, best first:
1. Put the module in the same folder as the notebook or script.
2. Make it part of a package inside that folder.
3. As a quick workaround only: `sys.path.append(r"C:\path\to\folder")`. This lasts only for that session.

How to read the two similar messages:
- `No module named 'helpers'` → Python **did not find** any `helpers`.
- `No module named 'helpers.py'; 'helpers' is not a package` → Python **found** `helpers`; the mistake is the `.py` in your import.

**Shadowing: never name your file after a library.** Because Python searches **your own folder first**, a file named `random.py`, `math.py` or `pandas.py` is imported **instead of the real library**. Then the real library's functions seem to be missing: `AttributeError: module 'random' has no attribute 'randint'`. This is not about "reserved keywords". It is about search order. Pick names that don't match any library (`my_random.py`, `textkit`).

**Three kinds of modules**
| Kind | Where it lives | Example |
|---|---|---|
| Standard library | Comes with Python. | `os`, `json`, `pathlib`, `collections`. |
| Third-party | Installed with pip into `site-packages`. | `requests`, `pandas`, `fastapi`. |
| Your own (local) | Your project folder. | `helpers`, `textkit`. |

### 4.6 Packages
A **package is a folder of modules**. Example you built:
```
shop\
├── __init__.py       <- marks the folder as a package (can be empty)
├── prices.py         <- module
└── tax.py            <- module
```
```python
# shop/prices.py
def total(items):
    return sum(items)

# shop/tax.py
def add_tax(amount, rate=0.18):
    return amount * (1 + rate)
```
```python
from shop.prices import total
from shop import tax
print(total([100, 200]), tax.add_tax(100))      # 300 118.0
```
A dot means "inside": `shop.prices` is the module `prices` inside the package `shop`.

**Module vs package (a common mix-up).** A module is **one file**. A package is **a folder of modules**. So `helpers.py` is a module, and `shop/` and `textkit/` are packages. (It is not "a module is a collection of packages".)

**What `__init__.py` does.**
- It marks the folder as a package.
- It **runs once when the package is imported**.
- It decides which names are available at the top level of the package.

**Common confusion: `AttributeError: module 'textkit' has no attribute 'clean_text'`.** `import textkit` runs **only `__init__.py`**. It does not automatically import the other files in the folder. If `__init__.py` is empty, the functions in `clean.py` and `split.py` are not visible as `textkit.clean_text`. Fix it by re-exporting them in `__init__.py`:
```python
# textkit/__init__.py
from .clean import clean_text
from .split import split_words, chunk_text
```
Now both of these work:
```python
import textkit
textkit.clean_text("  Hi   there ")                 # 'hi there'

from textkit import chunk_text
chunk_text("abcdefghij", 4)                        # ['abcd', 'efgh', 'ij']
```
You can also import the submodule directly: `from textkit.clean import clean_text`.

**Optional: `__all__`.** A list in a module or `__init__.py` that names what `from package import *` should export: `__all__ = ["clean_text", "chunk_text"]`.

### 4.7 Absolute vs relative imports (inside a package)
```python
# inside textkit/__init__.py or another file of the package
from textkit.clean import clean_text      # absolute: full path from the top (recommended)
from .clean import clean_text             # relative: "from this same package" (one dot)
from ..other import something             # two dots: "from the parent package"
```

**Common confusion: running a package file directly.** Running `__init__.py` (or any file that uses relative imports) directly gives:
```
ImportError: attempted relative import with no known parent package
```
Why: a relative import means "from the package this file belongs to". When you **run** a file directly, Python treats it as a loose script (`__name__` is `"__main__"`), with **no parent package**, so the dot has nothing to refer to. When the same file is **imported** (`import textkit`), Python knows it belongs to the package and the dot works.

**Rule:** run only **entry-point scripts** (such as `main.py`). Package files (`__init__.py`, `clean.py`, `split.py`) are only **imported**. If you must run a file inside a package, run it as a module from the parent folder: `python -m textkit.clean`. The `-m` flag tells Python "treat this as part of its package", so relative imports work. (If `__init__.py` already imports that same module you may see a harmless `RuntimeWarning`; it still runs.)

**Circular imports.** If module A imports B and B imports A, Python may fail or hand back a half-loaded module. Fix by moving the shared code into a third module, or by importing inside a function.

### 4.8 pip: the package installer
**PyPI** (the Python Package Index) is the online shop. **pip** downloads from it and installs into the **current environment**.

| Command | What it does |
|---|---|
| `python -m pip install requests` | Install a library. |
| `pip install requests==2.34.2` | Install an exact version. |
| `pip install -U requests` | Upgrade to the newest version. |
| `pip uninstall requests` | Remove it. |
| `pip list` | Show everything installed in this environment. |
| `pip show requests` | Show details (version, location). |
| `pip freeze` | List installed packages with exact versions. |
| `pip install -r requirements.txt` | Install everything listed in a file. |
| `pip install --only-binary :all: numpy` | Install ready-made binaries only; never compile. |

Prefer **`python -m pip ...`** over plain `pip ...`. It guarantees the pip belongs to the Python you are running.

**Dependencies.** A library often needs other libraries. For example, `requests` needs `certifi`, `charset-normalizer`, `idna` and `urllib3`. pip installs those automatically. They are called **dependencies** (the extra ones are *transitive dependencies*).

**Version specifiers** (used in requirements files):
| Written as | Meaning |
|---|---|
| `requests==2.34.2` | Exactly this version (a **pin**). |
| `requests>=2.30` | This version or newer. |
| `requests~=2.30` | A compatible release: 2.30 or newer, but below 3.0. |
| `requests!=2.31.0` | Anything except this version. |

### 4.9 Virtual environments
**The problem:** all projects share one global Python, so their library versions clash and the global install gets cluttered.
**The solution:** each project gets its own environment.

```powershell
python -m venv .venv                  # create it (folder named .venv)
.venv\Scripts\Activate.ps1            # activate it (Windows PowerShell); the prompt shows (.venv)
deactivate                            # leave it
```
While active, `python` and `pip` are the ones **inside `.venv`**, so installs go into `.venv\Lib\site-packages` only.

**How a venv relates to the base Python.** The venv shares the base installation's standard library (for example `...\AppData\Local\Programs\Python\Python315\Lib`, which is why you see `AppData` paths in `sys.path`; this is normal). Only the installed libraries are separate. Check that you are really inside a venv:
```python
import sys
print(sys.prefix != sys.base_prefix)         # True means you are inside a venv
print(sys.prefix)                            # should contain ...\.venv
```

**Notebook kernel.** In VS Code, the notebook must use the `.venv` kernel (top right: `.venv (3.15.2)`), or `pip install` in the terminal will not be visible to the notebook. For notebooks you only need **`ipykernel`**, not the full `jupyter` package: `pip install ipykernel`.

**Windows problems you may hit, and fixes**
| Problem | Cause and fix |
|---|---|
| `Activate.ps1` is blocked. | PowerShell's execution policy. Run `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned` once. |
| A library tries to **compile from source** and fails (for example PyYAML). | There is no ready-made binary (a *wheel*) for your Python version, which happens with very new Python versions such as 3.15. Use `pip install --only-binary :all: <package>`, or create a second venv with Python 3.12 or 3.13 where wheels exist. |
| `An Application Control policy has blocked this file`. | Windows (Smart App Control) blocked an unsigned file produced during the compile step. Avoid compiling by using wheels (above), and install only what you need (`ipykernel` instead of `jupyter`). |

### 4.10 Recording what the project needs

**`requirements.txt`** is a plain list of packages:
```
requests==2.34.2
ipykernel==7.4.0
```
```powershell
pip freeze > requirements.txt            # save everything installed right now
pip install -r requirements.txt          # rebuild on another machine
```
`pip freeze` lists **everything installed, including dependencies of dependencies** (that is why your file had `certifi`, `idna`, `tornado`, `pyzmq` and many others). That is fine for exact reproduction. In real projects you usually write **only your direct dependencies** (`requests`, `pandas`, `fastapi`) and let pip find the rest. Notebook-only tools such as `ipykernel` are often kept in a separate file (`requirements-dev.txt`) so production does not carry them.

**`pyproject.toml`** is the modern standard project file. It describes the project's name, Python version and dependencies in one place (TOML is a simple settings-file format).
```toml
[project]
name = "textkit"
version = "0.1.0"
requires-python = ">=3.10"
dependencies = [
    "requests>=2.30",
]

[project.optional-dependencies]
dev = ["pytest", "ipykernel"]
```
Newer tools (**uv**, **poetry**) manage environments and dependencies using this file. For now, know that `requirements.txt` is the simple list and `pyproject.toml` is the full project description.

**`.gitignore`** tells Git which files never to commit:
```
.venv/
__pycache__/
.ipynb_checkpoints/
.env
```
- **Never commit `.venv`.** It is not confidential. It is a huge folder of installed libraries, specific to your PC and operating system. It can be **rebuilt** from `requirements.txt` or `pyproject.toml`, so **commit those files instead**.
- **Never commit `.env`.** This is the file for secrets (API keys, passwords). It is a different thing from `.venv` and a very common mix-up.

### 4.11 Typical project layout
```
my-project\
├── .venv\                  (never committed)
├── .gitignore
├── .env                    (secrets; never committed)
├── requirements.txt        (or pyproject.toml)
├── README.md
├── src\
│   └── app\
│       ├── __init__.py
│       ├── main.py
│       └── utils.py
└── tests\
    └── test_utils.py
```
With a `src\` layout, you normally run `pip install -e .` once. The `-e` means "editable": it links your package into the environment so `import app` works from anywhere, and your edits take effect immediately.

## 5. Important terminology
| Term | Simple meaning |
|---|---|
| Module | One `.py` file. |
| Package | A folder of modules (with `__init__.py`). |
| Library | A package (or set of packages) written for others to use. |
| Import | Loading code from another module into this file. |
| `__init__.py` | Marks a folder as a package; runs on import; decides the package's top-level names. |
| `__name__` | A built-in variable: `"__main__"` if the file is run directly, else the module's name. |
| `sys.path` | The list of folders Python searches for modules. |
| `sys.modules` | The cache of modules already imported. |
| Shadowing | Your file hides a library with the same name because your folder is searched first. |
| Absolute import | Full path from the top (`from textkit.clean import x`). |
| Relative import | Path relative to the current package (`from .clean import x`). |
| PyPI | The online index where pip downloads packages from. |
| pip | The package installer. |
| Wheel | A ready-made binary package file (`.whl`) that installs without compiling. |
| Dependency | A library that another library needs. |
| Pin | An exact version requirement (`==`). |
| Virtual environment | A private Python plus its own libraries, one per project. |
| `site-packages` | The folder where installed libraries live. |
| Kernel | The Python process that runs a notebook's cells. |
| `requirements.txt` | A plain list of the libraries a project needs. |
| `pyproject.toml` | The modern project description file. |
| `.gitignore` | A file listing what Git must ignore. |
| `.env` | A file holding secrets and settings; never committed. |
| Editable install | `pip install -e .`: installs your project by linking, so edits apply at once. |
| Magic command | A notebook command starting with `%` (for example `%autoreload 2`). |

## 6. Example (small)
```python
# helpers.py
def hello(name):
    return f"Hello {name}"

if __name__ == "__main__":
    print(hello("test"))
```
```python
# notebook (same folder)
import helpers
print(helpers.hello("Ann"))       # Hello Ann
```

## 7. Code example (full): the `textkit` package
```
00-python-foundation\
├── main.py
└── textkit\
    ├── __init__.py
    ├── clean.py
    └── split.py
```
```python
# textkit/clean.py
def clean_text(text):
    """Lowercase the text and collapse runs of whitespace into one space."""
    return " ".join(text.lower().split())
```
```python
# textkit/split.py
def split_words(text):
    """Return the words of the text as a list."""
    return text.split()

def chunk_text(text, size):
    """Cut text into pieces of at most `size` characters."""
    return [text[i:i + size] for i in range(0, len(text), size)]
```
```python
# textkit/__init__.py
from .clean import clean_text
from .split import split_words, chunk_text
```
```python
# main.py
from textkit import clean_text, split_words, chunk_text

def main():
    messy = "  Here   is   a   messy     line  "
    cleaned = clean_text(messy)
    print(split_words(cleaned))
    for chunk in chunk_text(cleaned, 10):
        print(repr(chunk))

if __name__ == "__main__":
    main()
```
Run it with `python main.py` (from that folder). Do **not** run `__init__.py` directly.

**What the output teaches.** `chunk_text` cuts at exactly `size` characters, so it can split words in the middle (`li` / `ne`) and start chunks with a space. This is the weakness of **fixed-size chunking**. Real RAG chunkers split on sentences or words and add overlap between chunks to avoid this. You will build that in Phases 6 and 7.

**Rebuilding the environment from a file (practice for a new machine):**
```powershell
cd C:\Users\saite\resources\track-2\practice
python -m venv .venv_test                  # create a fresh, empty environment
.venv_test\Scripts\Activate.ps1            # switch into it; prompt shows (.venv_test)
pip install -r requirements.txt            # install everything from the file
python 00-python-foundation\main.py        # prove it runs there
deactivate                                 # leave it
Remove-Item -Recurse -Force .venv_test     # delete the test environment
```

## 8. Real-world usage
- **FastAPI projects** are packages: `app/main.py`, `app/routers/`, `app/models/`, each with an `__init__.py`. `uvicorn app.main:app` imports your code by its package path.
- **pytest** finds your code through the same import rules. A wrong layout leads to `ModuleNotFoundError` in tests.
- **Docker and servers** rebuild the environment with `pip install -r requirements.txt` (the step you practised).
- **RAG projects** keep ingestion, chunking, embedding and retrieval in separate modules (`chunking.py`, `embeddings.py`, `retriever.py`) so each can be tested and replaced.
- **Teams** commit `requirements.txt` or `pyproject.toml` and `.gitignore`, never `.venv` or `.env`.
- **The `__name__` guard** is on almost every runnable script, so the same file can be run or imported.

## 9. Common mistakes
- Writing `import helpers.py` instead of `import helpers`.
- Using `print` where you meant `return`, so a function returns `None`.
- Putting the module in a different folder from the notebook, then getting `ModuleNotFoundError`.
- Naming your file after a library (`random.py`, `pandas.py`), which shadows it.
- An empty `__init__.py` and then expecting `textkit.clean_text` to exist.
- Running `__init__.py` or another package file directly, which gives "attempted relative import with no known parent package".
- Editing a module and re-importing in the same notebook, which uses the cached old version.
- Using `pip install` in the terminal while the notebook uses a different kernel.
- Installing with plain `pip` that belongs to a different Python. Use `python -m pip`.
- Committing `.venv` (huge, PC-specific) or `.env` (secrets).
- Mixing up module and package (module = file, package = folder).
- Using `from x import *`, which hides where names come from.
- Running very new Python versions that have no wheels, causing compile errors.
- Forgetting to activate the environment before installing, so packages land in the wrong place.

## 10. Performance considerations
- A module is run **once** and then cached in `sys.modules`; repeated imports are almost free.
- Importing big libraries (Pandas, PyTorch, transformers) takes noticeable time. Import them once at the top of the file, not inside a loop. For rarely used heavy imports, import inside the function that needs them.
- `__pycache__` stores compiled bytecode so later starts are faster.
- Wheels install in seconds; compiling from source can take minutes and may fail.
- Keep dependencies small. Each extra library adds install time, image size and risk.

## 11. Security considerations
- **Importing runs code.** Only import modules you trust.
- **Installing packages runs code too** (and can bring in dependencies you never chose). Install only trusted packages, and check the spelling: attackers publish look-alike names (typosquatting), such as `reqeusts`.
- **Pin versions** in `requirements.txt` so you know exactly what runs, and review upgrades.
- **Never commit `.env`** or any file with API keys and passwords. Add it to `.gitignore`. If a secret is ever committed, treat it as leaked and rotate it.
- Windows Application Control or Smart App Control may block unsigned files. Do not bypass it casually; prefer prebuilt wheels.
- Do not `pip install` into the global Python as an administrator; use a virtual environment.

## 12. Interview questions and answers

**1. What is the difference between a module and a package?**
A module is a single `.py` file. A package is a directory of modules, normally containing an `__init__.py`, which lets you organise code into a hierarchy (`shop.prices`).

**2. What does `if __name__ == "__main__":` do?**
`__name__` is `"__main__"` when the file is run directly and the module's name when it is imported. The guard runs its block only on direct execution, so importing the file does not trigger demo code or start a server.

**3. How does Python find the modules you import?**
It searches the folders in `sys.path` in order: the script's folder (or the notebook's), `PYTHONPATH`, the standard library, and `site-packages` (installed libraries). If nothing matches, it raises `ModuleNotFoundError`.

**4. What does `__init__.py` do?**
It marks a folder as a package, runs when the package is imported, and can re-export names so users can write `from textkit import clean_text`. It can be empty.

**5. Absolute vs relative imports?**
An absolute import gives the full path from the top-level package (`from textkit.clean import clean_text`). A relative import starts from the current package using dots (`from .clean import clean_text`). Absolute imports are clearer and recommended; relative imports only work when the file is imported as part of a package.

**6. Why do you get "attempted relative import with no known parent package"?**
You ran a file that uses relative imports directly as a script. It then has no parent package. Run the entry script instead, or use `python -m package.module`.

**7. Why shouldn't you name your file `random.py`?**
Python searches your own folder first, so `import random` would load your file instead of the standard library. The real module's functions then appear to be missing. This is called shadowing.

**8. What is a virtual environment and why use one?**
It is an isolated copy of Python with its own installed libraries, one per project. It prevents version clashes between projects, keeps the global Python clean, and makes the setup reproducible.

**9. `requirements.txt` vs `pyproject.toml`?**
`requirements.txt` is a simple list of packages (often exact pins). `pyproject.toml` is the modern project file that describes the project's name, Python version, dependencies and optional dependency groups, and is read by tools such as pip, uv and poetry.

**10. Why is `.venv` not committed to Git?**
It is large, specific to one machine and operating system, and fully rebuildable from `requirements.txt` or `pyproject.toml`. Commit those files instead. (Secrets live in `.env`, which is also never committed.)

**11. What does `pip freeze` show?**
Every installed package, including dependencies of dependencies, with exact versions. Redirect it into a file with `pip freeze > requirements.txt`.

**12. What is a wheel, and why do wheel errors happen on very new Python versions?**
A wheel (`.whl`) is a prebuilt package that installs without compiling. A new Python version may not have wheels yet, so pip downloads the source and tries to compile it, which can fail (especially on Windows). Use an older stable Python or `--only-binary :all:`.

**13. What happens when you import the same module twice?**
The first import runs the file and stores the module in `sys.modules`. The second import reuses the cache and does not run the file again. To pick up edits, reload it (`importlib.reload`) or restart the interpreter.

**14. What is a circular import and how do you fix it?**
Two modules import each other, so one is only partly loaded when the other needs it. Fix it by moving shared code to a third module, or by importing inside a function.

**15. Why use `python -m pip` instead of `pip`?**
It guarantees you are using the pip that belongs to the Python interpreter you are running, not a different one earlier on your PATH.

## 13. Things to remember
- Module = file; package = folder of modules.
- Import the module name **without** `.py`.
- Importing runs the file once and caches it; edits need a reload or kernel restart.
- `__name__ == "__main__"` only when the file is run directly.
- Python searches your own folder first, so never name a file like a library.
- `import pkg` runs only `__init__.py`; re-export names there.
- Run entry scripts (`main.py`); only import package files.
- One virtual environment per project; activate it before `pip install`.
- Use `python -m pip`.
- Check a venv with `sys.prefix != sys.base_prefix`.
- Commit `requirements.txt` / `pyproject.toml` and `.gitignore`; never `.venv` or `.env`.
- `print` shows; `return` hands back.

## 14. One-page revision summary
- **Module** = one `.py` file. **Package** = folder of modules with `__init__.py`. **Library** = installed package(s).
- **Import** forms: `import x`, `import x as y`, `from x import a`. No `.py` in the name. Avoid `import *`.
- **First import** runs the file once and caches it in `sys.modules`; edits need `importlib.reload` or a kernel restart.
- **`__name__`**: `"__main__"` when run directly, module name when imported. The guard keeps demo code from running on import.
- **`sys.path`** order: script/notebook folder, PYTHONPATH, standard library, site-packages. Not found means `ModuleNotFoundError`. Your own folder comes first, so a file named like a library shadows it.
- **Packages:** `import pkg` runs only `__init__.py`; re-export with `from .module import name`. Relative imports (`.`) only work when the file is imported as part of a package; run `main.py`, not package files (or `python -m pkg.module`).
- **pip:** `python -m pip install / list / show / freeze`; `-r requirements.txt` rebuilds. Dependencies install automatically. Pins use `==`.
- **venv:** `python -m venv .venv`, activate with `.venv\Scripts\Activate.ps1`, `deactivate` to leave. One per project. Notebook kernel must be the venv. Check with `sys.prefix != sys.base_prefix`.
- **Windows tips:** execution policy for activation; very new Python may lack wheels, so use `--only-binary :all:` or Python 3.12/3.13; install `ipykernel`, not `jupyter`.
- **Files:** `requirements.txt` (simple list), `pyproject.toml` (full project description), `.gitignore` (`.venv/`, `__pycache__/`, `.env`).
- **Never commit** `.venv` (huge, rebuildable) or `.env` (secrets).
- **Layout:** `src/app/` with `__init__.py`, `tests/`, `README.md`, requirements, `.gitignore`.
- **Security:** importing and installing run code; trust sources, watch for typosquatting, pin versions, protect secrets.

## 15. Practice to redo, with solutions

**1. Module and `__name__`**
```python
# helpers.py
print("helpers is loading")
PI = 3.14159
print("__name__ is", __name__)

def hello(name):
    return f"Hello {name}"

if __name__ == "__main__":
    print(hello("test"))
```
- `python helpers.py` prints `helpers is loading`, `__name__ is __main__`, `Hello test`.
- `import helpers` in a notebook prints `helpers is loading`, `__name__ is helpers`, and **not** `Hello test`.
- Importing it twice in one kernel prints the loading line only once.

**2. `sys.path` and venv check**
```python
import sys
print(sys.executable, sys.prefix != sys.base_prefix)   # ...\.venv\Scripts\python.exe True
print(sys.path[:3])
```

**3. The `shop` package**
```python
from shop.prices import total
from shop import tax
print(total([100, 200]), tax.add_tax(100))   # 300 118.0
```

**4. pip**
```powershell
python -m pip install requests
python -m pip show requests
pip freeze > requirements.txt
```
```python
import requests
print(requests.__version__)
```

**5. Deliberate error**
```python
import nothing_here      # ModuleNotFoundError: No module named 'nothing_here'
```

**6. The `textkit` package** (section 7): write the three modules and `main.py`, run `python main.py`, then `import main` in a notebook and confirm that **nothing prints**.

**7. Rebuild from `requirements.txt`:** use the command block at the end of section 7 and confirm that `main.py` runs inside `.venv_test`.

**8. Short answers**
1. Module = one `.py` file; package = a folder of modules with `__init__.py`.
2. A file named `random.py` shadows the standard library because Python searches your own folder first.
3. `.venv` is huge, specific to your machine and rebuildable, so commit `requirements.txt` or `pyproject.toml` instead. Secrets belong in `.env`, which is also never committed.