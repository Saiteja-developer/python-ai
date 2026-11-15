# 04 — Modules, Packages, pip and Virtual Environments

> Phase 0 · Python foundation. Every real project (FastAPI app, RAG pipeline, Pandas job) is many files plus many installed libraries. This file is how they fit together without becoming a mess.
> Running example: a small package, `textkit`, that cleans and chunks support-ticket text (the same first steps of a RAG pipeline).
> `🎯 Interview:` marks a point worth remembering for interviews. `⚠️ Common confusion:` marks a place where I got stuck, with the reason explained.
> Commands are for **Windows PowerShell**.

---

## 1. Module

### 1.1 Definition
A **module** is **one `.py` file**. It holds code (functions, variables, classes) that other files can reuse. A **package** is a **folder of modules** (section 4). A **library** is a package (or a few) written for others to use.

### 1.2 Important points
- **File name:** `helpers.py`. **Module name** (in an import): `helpers`. **Never write `.py` in an import.**
- Three kinds of modules: **standard library** (comes with Python: `os`, `json`, `collections`), **third-party** (installed with pip: `requests`, `pandas`), **your own** (files in your project).
- Import forms:

| Written as | You use it as |
|---|---|
| `import helpers` | `helpers.hello("Ann")` |
| `import helpers as h` | `h.hello("Ann")` |
| `from helpers import hello` | `hello("Ann")` |
| `from helpers import *` (avoid) | Names get mixed up and it is unclear where they came from. |

- Convention (PEP 8): put imports at the top in three groups, separated by a blank line: standard library, third-party, your own.

**Syntax:**
```
import module_name
import module_name as alias
from module_name import name1, name2
from module_name import name as alias
from package.module import name           # a dot means "inside"
from module_name import *                 # avoid
```
**Each import form with a standard-library example:**
```python
import math                  # use as math.sqrt(16)
math.sqrt(16)                # 4.0

import math as m             # a short alias
m.sqrt(16)                   # 4.0

from math import sqrt, pi    # pull out chosen names
sqrt(16)                     # 4.0
round(pi, 2)                 # 3.14

from os import path as p     # a name inside a package, renamed
p.join("a", "b")             # 'a/b'  (on Windows: 'a\\b')
p.basename("/a/b/c.txt")     # 'c.txt'
p.splitext("c.txt")          # ('c', '.txt')

from math import *           # avoid: you cannot see where names came from
```

### 1.3 Example
```python
# helpers.py
print("helpers is loading")
PI = 3.14159

def hello(name):
    return f"Hello {name}"
```
```python
# notebook, same folder
import helpers
print(helpers.hello("Ann"), helpers.PI)
```
**Result:**
```
helpers is loading
Hello Ann 3.14159
```

### 1.4 What happens on `import`
1. Python **finds** the file (section 3).
2. It **runs the whole file once**, top to bottom (that is why "helpers is loading" printed).
3. It saves the module in a cache (`sys.modules`).
4. Later imports of the same module **reuse the cache**; the file does not run again.

Import `helpers` twice in two cells: the loading line prints **once**.

Prove the cache yourself:
```python
import sys
import helpers
"helpers" in sys.modules              # True     it is now cached
import helpers                        # prints nothing: the cached module is reused
sys.modules["helpers"].__name__       # 'helpers'
```

**Notebook consequence:** if you edit `helpers.py` and import it again in the same running kernel, you get the **old cached version**. Fix with a kernel restart, or:
```python
import importlib
importlib.reload(helpers)
# or auto-reload before every cell (a notebook "magic command", starts with %):
%load_ext autoreload
%autoreload 2
```
Also: importing **runs code**, so only import modules you trust. Heavy libraries (Pandas, PyTorch) take time to import, so import them once at the top, not inside loops.

### 1.5 ⚠️ Common confusion: `import helpers.py`
```python
import helpers.py as helper
# ModuleNotFoundError: No module named 'helpers.py'; 'helpers' is not a package
```
**A dot in an import means "inside a package".** Python read it as "the module `py` inside the package `helpers`". It **did find** `helpers` (a single file), saw it is not a package, and gave up. The message even says so: `'helpers' is not a package`.
**Fix:** `import helpers as helper`.

### 1.6 ⚠️ Common confusion: `print` vs `return` in your module's function
If `hello` does `print("My name:", name)` instead of `return`, then `print(helper.hello("Ann"), helper.PI)` shows:
```
My name: Ann
None 3.14159
```
The function printed, then returned `None`, so the outer `print` shows `None`. (More in Topic 03, section 1.4.)

---

## 2. `if __name__ == "__main__":`

### 2.1 Definition
Every module has a built-in variable `__name__`.

| How the file is used | `__name__` is | Does the guarded block run? |
|---|---|---|
| Run directly: `python helpers.py` | `"__main__"` | Yes |
| Imported: `import helpers` | `"helpers"` (the module name) | No |

### 2.2 Example
**Syntax:**
```
if __name__ == "__main__":
    ...code that runs only when this file is run directly...
```
```python
# helpers.py
def hello(name):
    return f"Hello {name}"

print("__name__ is", __name__)

if __name__ == "__main__":
    print(hello("test"))
```
**Result:**
- `python helpers.py` → `__name__ is __main__` then `Hello test`
- `import helpers` → `__name__ is helpers` only (the demo line is skipped)

**Important points:**
- It lets one file work as **both** a script and a module, so demo or test code (or starting a server) does not run by accident on import.
- If you `import main` and **nothing prints**, that is the guard doing its job: `__name__` is `"main"`, so the block is skipped.
- Almost every runnable script and FastAPI entry file has this guard.

🎯 **Interview:** what does `if __name__ == "__main__":` do? It runs the block only when the file is run directly, not when imported.

---

## 3. How Python finds a module: `sys.path`

### 3.1 Definition
`sys.path` is the **list of folders** Python searches, in this order:
1. The folder of the script you ran (for a notebook: the notebook's own folder).
2. `PYTHONPATH` folders (rarely used).
3. The standard library.
4. `site-packages`: where `pip install` puts libraries (inside your `.venv`).

If nothing matches: **`ModuleNotFoundError`** ("I searched every folder and found nothing with that name").
```python
import sys, os
print(sys.executable)      # the Python that is running
print(sys.path)            # folders searched
print(os.getcwd())         # the folder the notebook runs in
print(os.listdir())        # files in that folder
```
Example output (yours will differ):
```
C:\Users\you\track-2\practice\.venv\Scripts\python.exe      <- sys.executable
['C:\\...\\00-python-foundation', '...\\python315.zip', '...\\Lib', '...\\.venv\\Lib\\site-packages']   <- sys.path
C:\Users\you\track-2\practice\00-python-foundation          <- os.getcwd()
['helpers.py', '04-modules.ipynb']                               <- os.listdir()
```
Add a folder for the current session:
```python
import sys
sys.path.append(r"C:\path\to\folder")
sys.path[-1]                         # 'C:\\path\\to\\folder'   it is now searched last
```

**Syntax for looking at the search path:**
```
import sys
sys.path                       # list of folders searched
sys.path.append(r"C:\folder")   # add one for this session
sys.modules                    # dict of modules already imported
import importlib
importlib.reload(module)       # re-run an edited module
```
### 3.2 ⚠️ Common confusion: `ModuleNotFoundError` for your own file
Usually the file is **not in the folder the notebook runs in** (for example `helpers.py` is in `practice\` but the notebook is in `practice\00-python-foundation\`). Python does **not** look in the parent folder. Check `os.getcwd()` and `os.listdir()`. Fixes, best first:
1. Put the module in the **same folder** as the notebook or script.
2. Make it part of a **package** inside that folder.
3. Quick workaround only: `sys.path.append(r"C:\path\to\folder")` (lasts for that session).

Two similar messages, two different meanings:
- `No module named 'helpers'` → Python **did not find** any `helpers`.
- `No module named 'helpers.py'; 'helpers' is not a package` → it **found** `helpers`; your import has a `.py` in it.

### 3.3 Shadowing: never name a file after a library
Python searches **your own folder first**. A file named `random.py`, `math.py` or `pandas.py` is imported **instead of the real library**, and then the real library's functions seem to be missing (`AttributeError: module 'random' has no attribute 'randint'`). This is not about "reserved keywords". It is about search order.

🎯 **Interview:** why not name your file `random.py`? It shadows the standard library because the script's folder is searched first.

---

## 4. Package

### 4.1 Definition
A **package** is a **folder of modules**, normally containing an **`__init__.py`**.
```
textkit\
├── __init__.py       <- marks the folder as a package; runs when the package is imported
├── clean.py          <- module
└── split.py          <- module
```
A dot means "inside": `textkit.clean` is the module `clean` inside the package `textkit`.
```python
import textkit.clean                       # the module, as a path
textkit.clean.clean_text("  A  B ")        # 'a b'

from textkit import clean                  # the module, by name
clean.clean_text("  A  B ")                # 'a b'

from textkit.split import chunk_text       # one function from one module
chunk_text("abcdef", 4)                    # ['abcd', 'ef']
```

**Module vs package:** a module is **one file**. A package is **a folder of modules**. (`helpers.py` is a module; `textkit/` is a package.)

### 4.2 Example: the `textkit` package
**Syntax (folder layout and imports):**
```
package_name\
├── __init__.py            # can be empty, or re-export names
├── module_a.py
└── module_b.py

# inside __init__.py (re-export):
from .module_a import function_a

# in another file:
from package_name import function_a
from package_name.module_b import function_b
```
```python
# textkit/clean.py
def clean_text(text):
    """Lowercase the text and collapse extra spaces."""
    return " ".join(text.lower().split())
```
```python
# textkit/split.py
def split_words(text):
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
# main.py  (next to the textkit folder)
from textkit import clean_text, split_words, chunk_text

def main():
    ticket = "  Refund   NOT   received  for order 4412  "
    cleaned = clean_text(ticket)
    print(split_words(cleaned))
    for chunk in chunk_text(cleaned, 12):
        print(repr(chunk))

if __name__ == "__main__":
    main()
```
Run: `python main.py`. **Result:**
```
['refund', 'not', 'received', 'for', 'order', '4412']
'refund not r'
'eceived for '
'order 4412'
```
The chunker cuts at exactly 12 characters, so it splits words in the middle. That is the weakness of **fixed-size chunking**. Real RAG chunkers split on sentences or words and add **overlap**. This is what Phases 6 and 7 build.

### 4.3 ⚠️ Common confusion: `AttributeError: module 'textkit' has no attribute 'clean_text'`
`import textkit` runs **only `__init__.py`**. It does **not** automatically import the other files in the folder. If `__init__.py` is empty, `textkit.clean_text` does not exist. **Fix:** re-export the names in `__init__.py` (as above). Or import the submodule directly: `from textkit.clean import clean_text`.

### 4.4 Absolute vs relative imports
**Syntax:**
```
from package.module import name        # absolute
from .module import name               # relative: same package (one dot)
from ..module import name              # relative: parent package (two dots)
python -m package.module               # run a module inside a package (terminal)
```
```python
from textkit.clean import clean_text     # absolute: full path from the top (recommended)
from .clean import clean_text            # relative: "from this same package" (one dot)
from ..other import something            # two dots: the parent package
```
Layout to picture it:
```
app\
├── __init__.py
├── main.py
├── utils\
│   ├── __init__.py
│   └── text.py
└── other.py

# inside app\utils\text.py:
from . import something_in_utils_init     # "." = app.utils
from .. import other                      # ".." = app (the parent)
from ..other import helper                # a name from app\other.py
```

### 4.5 ⚠️ Common confusion: `ImportError: attempted relative import with no known parent package`
This appears when you **run** a package file (for example `__init__.py`) directly with the Run button or `python textkit/__init__.py`. A relative import means "from the package this file belongs to". A file that is **run directly** is a loose script (`__name__ == "__main__"`), so it has **no parent package** and the dot has nothing to point to. When the same file is **imported**, Python knows its package, so it works.

**Rule:** run **entry-point scripts** (like `main.py`). Package files are only **imported**. If you must run one, do it as a module from the parent folder: `python -m textkit.clean` (the `-m` flag keeps the package context).

**Circular imports:** if module A imports B and B imports A, Python may fail or hand back a half-loaded module. Fix: move the shared code into a third module, or import inside a function.

🎯 **Interview:** module vs package; absolute vs relative imports; what `__init__.py` does (marks the package, runs on import, can re-export names).

---

## 5. pip: the package installer

### 5.1 Definition
**PyPI** is the online index of Python libraries. **pip** downloads from it and installs into the **current environment**.

| Command | What it does |
|---|---|
| `python -m pip install requests` | Install a library. |
| `pip install requests==2.34.2` | Install an exact version. |
| `pip install -U requests` | Upgrade it. |
| `pip uninstall requests` | Remove it. |
| `pip list` / `pip show requests` | List everything installed / show details. |
| `pip freeze` | List installed packages with exact versions. |
| `pip install -r requirements.txt` | Install everything in a file. |
| `pip install --only-binary :all: numpy` | Use ready-made binaries only; never compile. |

**Syntax:**
```
python -m pip install <package>
python -m pip install <package>==<version>        # exact
python -m pip install "<package>>=<version>"      # at least
python -m pip install -U <package>                # upgrade
python -m pip install -r <file>                   # from a requirements file
python -m pip uninstall <package>
python -m pip list    |    python -m pip show <package>    |    python -m pip freeze
python -m pip freeze > requirements.txt           # save to a file
```
**What each command looks like when you run it** (text after `#` is what you will see):
```powershell
python -m pip install requests
# Collecting requests ... Successfully installed certifi-... charset-normalizer-... idna-... requests-2.34.2 urllib3-...

pip show requests
# Name: requests
# Version: 2.34.2
# Requires: certifi, charset-normalizer, idna, urllib3     <- its dependencies

pip list
# Package            Version
# ------------------ -------
# requests           2.34.2
# ipykernel          7.4.0

pip freeze
# requests==2.34.2
# ipykernel==7.4.0

pip install "requests>=2.30"       # at least this version
pip install -U requests            # upgrade to the newest
pip uninstall requests             # asks y/n, then removes it
```
Use it in code to prove it works:
```python
import requests
requests.__version__               # '2.34.2'
```

### 5.2 Important points
- **Use `python -m pip ...`**, so pip belongs to the Python you are running.
- A library often needs other libraries, called **dependencies**. `requests` needs `certifi`, `charset-normalizer`, `idna` and `urllib3`; pip installs them for you.
- Version rules: `==2.34.2` exact (a **pin**), `>=2.30` at least, `~=2.30` compatible (2.30 or newer, below 3.0).
- **Security:** installing a package runs code. Install only trusted packages, check the spelling (look-alike names such as `reqeusts` exist), and pin versions.

---

## 6. Virtual environment (venv)

### 6.1 Definition
A **virtual environment** is a **private copy of Python with its own installed libraries, one per project**. Project A can use Pandas 1.5 while Project B uses Pandas 2.2, with no clash. (Think: a separate kitchen for each project.)

```powershell
python -m venv .venv                  # create it
.venv\Scripts\Activate.ps1            # activate; the prompt shows (.venv)
deactivate                            # leave it
```
While active, `python` and `pip` are the ones **inside `.venv`**, so installs go only into `.venv\Lib\site-packages`.

Check which Python you are using:
```powershell
where.exe python
# C:\...\practice\.venv\Scripts\python.exe     <- first line is the venv (good)
# C:\Users\you\AppData\Local\Programs\Python\...\python.exe

python -m pip --version
# pip 25.x from C:\...\.venv\Lib\site-packages\pip (python 3.15)
```
Prove two venvs are separate:
```powershell
python -m venv .venv_a
.venv_a\Scripts\Activate.ps1
pip install requests
pip list           # requests is here...
deactivate
python -m venv .venv_b
.venv_b\Scripts\Activate.ps1
pip list           # ...but NOT here: it only shows pip itself
```

**Syntax:**
```
python -m venv <folder_name>             # create (usually .venv)
<folder_name>\Scripts\Activate.ps1        # activate (Windows PowerShell)
deactivate                               # leave
```
### 6.2 Notebook kernel
In VS Code the notebook must use the `.venv` kernel (top right shows `.venv (3.x)`). Otherwise `pip install` in the terminal is invisible to the notebook. For notebooks you only need **`ipykernel`**, not the full `jupyter` package.

### 6.3 ⚠️ Common confusion: "is my notebook really inside the venv?"
`sys.path` may show `...\AppData\Local\Programs\Python\...`. That is **normal**: a venv shares the base Python's **standard library**. Only the installed libraries are separate. The reliable check:
```python
import sys
print(sys.prefix != sys.base_prefix)    # True means you ARE inside a venv
print(sys.prefix)                       # contains ...\.venv
```

### 6.4 Windows problems seen, and fixes

| Problem | Cause and fix |
|---|---|
| `Activate.ps1` is blocked. | PowerShell's execution policy. Run once: `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned`. |
| A library tries to **compile from source** and fails (for example PyYAML). | There is no ready-made binary (**wheel**) for your Python version. This happens with very new versions such as 3.15. Use `pip install --only-binary :all: <pkg>`, or a second venv with Python 3.12 or 3.13. |
| `An Application Control policy has blocked this file`. | Windows (Smart App Control) blocked an unsigned file made during the compile step. Avoid compiling by using wheels, and install only what you need (`ipykernel` instead of `jupyter`). |

---

## 7. Recording what the project needs

### 7.1 requirements.txt
A plain list of packages so anyone can rebuild your environment:
```
requests==2.34.2
ipykernel==7.4.0
```
```powershell
pip freeze > requirements.txt          # save what is installed now
pip install -r requirements.txt        # rebuild it elsewhere
```
**⚠️ Common confusion:** `pip freeze` lists **everything installed, including dependencies of dependencies** (that is why your file had `certifi`, `idna`, `tornado`, `pyzmq`...). That is fine for exact reproduction. In real projects you usually list **only your direct dependencies** (`requests`, `pandas`, `fastapi`); pip finds the rest. Notebook-only tools like `ipykernel` often go in a separate `requirements-dev.txt`.

**Syntax of requirements.txt (one requirement per line):**
```
package                 # any version
package==1.2.3          # exactly this
package>=1.2            # at least this
package~=1.2            # compatible (1.2 or newer, below 2.0)
```
### 7.2 pyproject.toml
The modern standard project file (TOML is a simple settings format). It describes the project's name, Python version and dependencies.
```toml
[project]
name = "textkit"
version = "0.1.0"
requires-python = ">=3.10"
dependencies = ["requests>=2.30"]

[project.optional-dependencies]
dev = ["pytest", "ipykernel"]
```
Newer tools (**uv**, **poetry**) manage environments from this file. For now: `requirements.txt` = simple list, `pyproject.toml` = full project description.

### 7.3 .gitignore, `.venv` vs `.env`
```
.venv/
__pycache__/
.ipynb_checkpoints/
.env
```
- **Never commit `.venv`.** It is **not confidential**. It is a huge folder of installed libraries, specific to your PC, and **rebuildable** from `requirements.txt` / `pyproject.toml`. Commit those files instead.
- **Never commit `.env`.** That is the file for **secrets** (API keys, passwords). It is a different thing from `.venv`, and a very common mix-up. If a secret is ever committed, treat it as leaked and replace it.
- `__pycache__` holds compiled copies Python makes to start faster. You never edit it.

🎯 **Interview:** why not commit `.venv`? Large, machine-specific, rebuildable. `requirements.txt` vs `pyproject.toml`? Simple list vs full project description. What does `pip freeze` show? Everything installed with exact versions, including indirect dependencies.

### 7.4 Rebuild an environment from a file (practice for servers and Docker)
```powershell
python -m venv .venv_test                # a fresh, empty environment
.venv_test\Scripts\Activate.ps1          # prompt shows (.venv_test)
pip install -r requirements.txt          # install everything in the file
python main.py                           # prove it runs
deactivate
Remove-Item -Recurse -Force .venv_test   # delete the test environment
```

---

## 8. A typical project layout (what you will build later)
```
my-project\
├── .venv\                 (never committed)
├── .env                   (secrets; never committed)
├── .gitignore
├── requirements.txt       (or pyproject.toml)
├── README.md
├── src\
│   └── app\
│       ├── __init__.py
│       ├── main.py
│       └── utils.py
└── tests\
    └── test_utils.py
```
- FastAPI projects are packages like this (`app/main.py`, `app/routers/`, `app/models/`).
- RAG projects split work into modules (`chunking.py`, `embeddings.py`, `retriever.py`) so each part can be tested and replaced.
- With a `src\` layout you normally run `pip install -e .` once ("editable": it links your package into the environment so `import app` works anywhere and edits apply at once).

---

## 9. Patterns to remember

**Pattern 1: start a new project (do these in order).**
```powershell
mkdir my-project; cd my-project
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install <what you need>
pip freeze > requirements.txt
```
Then add `.gitignore` (with `.venv/`, `.env`, `__pycache__/`), a `README.md`, and your first package folder with `__init__.py`.

**Pattern 2: package skeleton.**
```
package\
├── __init__.py      # re-export the public names: from .module import name
├── module_a.py      # one job per file
└── module_b.py
main.py              # entry point with: if __name__ == "__main__": main()
```
**Pattern 3: debug an import error (follow the order).**

| Error message | Check this first |
|---|---|
| `ModuleNotFoundError: No module named 'x'` (a library) | Is the right venv active? Run `python -m pip show x`. Install it. |
| `ModuleNotFoundError: No module named 'x'` (your file) | Is `x.py` in the folder the code runs from? Check `os.getcwd()` and `os.listdir()`. |
| `No module named 'x.py'; 'x' is not a package` | Remove `.py` from the import. |
| `AttributeError: module 'x' has no attribute 'f'` | Empty `__init__.py`? Or does your file shadow a library (`random.py`)? Or old cached import: restart or reload. |
| `ImportError: attempted relative import with no known parent package` | You ran a package file directly. Run `main.py`, or `python -m package.module`. |
| Nothing prints after `import main` | The `if __name__ == "__main__"` guard did its job. |
| Works in the terminal, fails in the notebook | The notebook kernel is a different Python. Select the `.venv` kernel. |

**Pattern 4: share or rebuild a project.** `pip freeze > requirements.txt` on one machine, then new venv plus `pip install -r requirements.txt` on the other.

---

## 10. Use this when building (project guide)

Your Customer Support Copilot (RAG) project can start with this layout:
```
support-copilot\
├── .venv\                (not committed)
├── .env                   (API keys; not committed)
├── .gitignore
├── requirements.txt
├── README.md
├── src\
│   └── copilot\
│       ├── __init__.py
│       ├── cleaning.py    # clean_text (Topic 03/04 textkit)
│       ├── chunking.py    # split and chunk
│       ├── embeddings.py  # embed with lru_cache
│       └── retriever.py   # top-K search
├── tests\
└── main.py                # runs the pipeline, guarded by __name__
```
- One file = one job, so you can test and replace each part.
- Run `pip install -e .` (needs a `pyproject.toml`) so `import copilot` works from anywhere.
- Install only what you use, pin versions, and commit `requirements.txt`.
- The same layout works for a FastAPI app: add `routers\` and `models\` inside the package.

**Checklist for any new project:** venv created and active, kernel selected, `.gitignore` written (with `.env`), dependencies recorded, one module per job, an entry file with the `__name__` guard.

---

## 🧠 Quick Revision

- **Module** = one `.py` file. **Package** = folder of modules with `__init__.py`. Never write `.py` in an import.
- Import runs the file **once** and caches it. After editing, reload or restart the kernel.
- `__name__` is `"__main__"` only when the file is **run directly**; the guard keeps demo code from running on import.
- `sys.path` order: script/notebook folder → PYTHONPATH → standard library → site-packages. Not found = `ModuleNotFoundError`. Your own folder comes first, so a file named like a library **shadows** it.
- `import pkg` runs only `__init__.py`: re-export with `from .module import name`.
- Relative imports (`.`) only work when the file is **imported** as part of a package. Run `main.py`, never package files (or `python -m pkg.module`).
- pip: `python -m pip install / list / show / freeze`, `-r requirements.txt` rebuilds. Dependencies install automatically.
- venv: `python -m venv .venv`, `.venv\Scripts\Activate.ps1`, `deactivate`. One per project. Check with `sys.prefix != sys.base_prefix`.
- Very new Python may lack wheels: use `--only-binary :all:` or Python 3.12/3.13. Install `ipykernel`, not `jupyter`.
- Commit `requirements.txt` / `pyproject.toml` and `.gitignore`. **Never** commit `.venv` (huge, rebuildable) or `.env` (secrets).
- Importing and installing both run code: trust sources, pin versions.
- Practice each import form on a standard-library module (`math`, `os.path`) so the syntax becomes automatic.