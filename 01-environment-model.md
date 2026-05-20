# 01 — Python Environment Model

This note defines what a “Python environment” actually is at runtime, not conceptually.

Most confusion in packaging comes from treating Python environments as a single isolated folder. In reality, it is a combination of interpreter state, filesystem layout, and import resolution rules.

---

# 1. What a Python environment actually is

A Python environment is composed of four tightly coupled components:

```

1. Python interpreter binary
2. Standard library (stdlib)
3. Third-party package directory (site-packages)
4. Runtime configuration (sys.path + environment variables)

````

These components together define:

- what `import` resolves to
- which binaries are loaded
- which packages are visible
- how modules are prioritized

---

# 2. The interpreter is the anchor

Everything starts from the Python executable:

```bash
python3
````

This binary defines:

* Python version (e.g. 3.10, 3.11, 3.12)
* ABI compatibility
* location of stdlib
* default search paths

You can inspect it:

```python
import sys
print(sys.executable)
```

This is the *true identity* of the environment.

---

# 3. sys.prefix and sys.base_prefix

Python distinguishes between:

```python
sys.prefix
sys.base_prefix
```

### sys.base_prefix

* original Python installation path
* system interpreter root

### sys.prefix

* active environment root
* changes when using virtual environments

If they differ, you are inside a virtual environment.

---

# 4. sys.path — the import resolution core

Every `import` statement is resolved using `sys.path`.

You can inspect it:

```python
import sys
for p in sys.path:
    print(p)
```

Typical structure:

```
1. current working directory
2. PYTHONPATH (if set)
3. stdlib paths
4. site-packages paths
```

---

# 5. Import resolution algorithm (simplified but accurate)

When you write:

```python
import requests
```

Python performs:

1. Iterate over `sys.path`
2. For each entry:

   * check for `requests.py`
   * or `requests/__init__.py`
3. First match wins
4. Load module into `sys.modules`

Important consequence:

> Python import system is order-dependent, not version-aware.

---

# 6. site-packages — the third-party layer

Third-party packages live in:

```
.../lib/pythonX.Y/site-packages/
```

This directory is automatically appended to `sys.path`.

It contains:

* installed libraries
* compiled extensions (.so, .pyd)
* metadata (.dist-info)

This is where pip installs packages.

---

# 7. How Python builds sys.path at startup

At interpreter startup, Python:

### Step 1: initializes stdlib paths

Compiled into interpreter build.

### Step 2: processes environment variables

* PYTHONPATH
* PYTHONHOME

### Step 3: adds site-packages

This happens via:

```
site.py module
```

which dynamically injects paths.

---

# 8. The role of site.py

`site.py` is executed automatically on startup.

It is responsible for:

* adding site-packages to sys.path
* loading `.pth` files
* enabling user site directories

You can observe it indirectly:

```python
import site
print(site.getsitepackages())
```

---

# 9. .pth files (hidden mechanism)

Inside site-packages, `.pth` files can modify sys.path.

Example:

```
myproject.pth
```

contains:

```
/custom/path/to/libs
```

This mechanism allows:

* editable installs
* legacy path injection
* environment customization

---

# 10. Editable installs (preview)

When you run:

```bash
pip install -e .
```

pip does NOT copy files.

Instead it:

* creates a `.pth` entry OR
* links project directory into site-packages

Result:

> Python imports directly from source directory

---

# 11. Why environments feel “isolated”

Isolation is not a kernel-level sandbox.

It is purely:

* different sys.prefix
* different sys.path
* different site-packages location

Nothing prevents:

* system library access
* file system access
* cross-environment imports (if paths leak)

---

# 12. Virtual environment preview (full model)

A virtual environment is essentially:

```
copy of interpreter + redirected paths
```

It modifies:

* sys.prefix → new directory
* sys.path → environment-specific site-packages
* PATH → points to local python binary

No process isolation exists.

---

# 13. Key mental model

A Python environment is best understood as:

```
[ Interpreter ]
      +
[ sys.path (ordered search list) ]
      +
[ site-packages (installed distributions) ]
```

Everything else (venv, pip, uv) manipulates this structure.

---

# 14. Why this matters

Almost all packaging issues come from:

* incorrect sys.path ordering
* multiple site-packages locations
* interpreter mismatch
* ABI mismatch (native extensions)
* accidental global package leakage

---

# 15. What comes next

Next layer:

→ 02 — venv Internals

We will see how:

* sys.prefix is changed
* executables are copied or symlinked
* activation scripts modify PATH
* isolation is actually implemented

