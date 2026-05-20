# 02 — venv Internals

This note explains how Python virtual environments are actually constructed and what changes when you “activate” one.

A virtual environment is not isolation in the OS sense. It is a controlled rewrite of interpreter paths and environment variables.

---

# 1. What a venv really is

A virtual environment is:

```

A lightweight, self-contained Python installation that redirects:

* interpreter resolution
* package installation path
* script execution paths

````

It does NOT:
- sandbox system access
- isolate kernel resources
- create containers or processes

It only modifies *how Python finds things*.

---

# 2. venv directory structure

When you run:

```bash
python -m venv .venv
````

Python creates:

```text id="p3n8yy"
.venv/
├── bin/ (Linux/macOS)
│   ├── python
│   ├── pip
│   └── activate
│
├── Scripts/ (Windows)
│   ├── python.exe
│   ├── pip.exe
│   └── activate.bat
│
├── lib/
│   └── python3.x/
│       └── site-packages/
│
├── pyvenv.cfg
```

---

# 3. The key file: pyvenv.cfg

This file defines how the venv behaves.

Example:

```text id="c0tq9g"
home = /usr/bin
include-system-site-packages = false
version = 3.11.5
```

### Meaning:

* `home`: original Python installation
* `include-system-site-packages`: whether global packages leak in
* `version`: interpreter version

This file is read by the interpreter at startup.

---

# 4. What changes when you enter a venv

When you activate a venv:

```bash
source .venv/bin/activate
```

you are NOT changing Python itself.

You are modifying environment variables:

### PATH modification

Before:

```text id="7b1qkz"
/usr/bin/python
```

After:

```text id="1g3xzn"
.venv/bin/python
```

So `python` resolves to the venv interpreter.

---

# 5. sys.prefix behavior

Inside a venv:

```python id="l9xq2m"
import sys
print(sys.prefix)
```

Output:

```text id="v8kq9a"
/path/to/project/.venv
```

But:

```python id="z1m0rt"
print(sys.base_prefix)
```

Output:

```text id="p2n8sd"
/usr (or system Python root)
```

### Interpretation:

* base_prefix = original Python install
* prefix = active environment

This is the core venv detection mechanism.

---

# 6. How Python modifies sys.path in a venv

When a venv is active, Python injects:

```text id="u7m1ld"
.venv/lib/pythonX.Y/site-packages
```

into `sys.path` at runtime.

So imports now resolve to:

* venv packages first
* system packages only if allowed

---

# 7. site-packages duplication model

Each venv has its own:

```text id="b4n0xq"
.venv/lib/pythonX.Y/site-packages/
```

This means:

* pip installs are local
* packages are not shared
* dependency graphs are isolated per project

This is the core isolation mechanism.

---

# 8. What actually gets “copied” into a venv

A venv does NOT copy the full Python installation.

It typically includes:

### Lightweight copies / symlinks:

* python executable (or link to system python)
* pip launcher scripts
* activation scripts

### Not copied:

* standard library
* compiled interpreter core
* system dependencies

---

# 9. Why venv is lightweight

Instead of duplicating Python, venv relies on:

```
shared interpreter + redirected paths
```

So multiple environments can share:

* same Python binary
* same stdlib
* different site-packages

This is why venv creation is fast.

---

# 10. Activation is just shell manipulation

Activation scripts modify:

## PATH

so `python` and `pip` resolve to venv

## shell prompt (optional)

visual indicator only

Example:

```bash id="k2v7we"
(.venv) user@machine$
```

Important:

> Activation does NOT modify Python itself.

If you call Python directly:

```bash id="r4q9lm"
./.venv/bin/python script.py
```

it behaves exactly the same without activation.

---

# 11. Why activation exists at all

Activation is purely convenience:

* avoids typing full path
* ensures correct pip/python pairing
* reduces human error

But technically unnecessary.

---

# 12. pip inside a venv

Once activated:

```bash id="t7n3qp"
pip install requests
```

installs into:

```text id="a9m2wx"
.venv/lib/pythonX.Y/site-packages
```

pip detects venv via:

* sys.prefix
* environment markers
* installation path resolution

---

# 13. venv isolation boundary

A venv isolates ONLY:

### Isolated:

* Python packages
* installed binaries (console scripts)
* sys.path resolution

### NOT isolated:

* system libraries (libc, CUDA, etc.)
* OS files
* environment variables (except PATH)
* process execution

This is NOT a container.

---

# 14. Common misconception

A venv is NOT:

* a Docker container
* a sandbox
* a separate OS environment

It is:

> a redirected Python interpreter context

---

# 15. Why venv still fails sometimes

Even with venv, problems occur due to:

* system-level shared libraries (e.g. libstdc++)
* CUDA mismatches
* pip installing incompatible wheels
* mixing interpreters (python vs python3 vs system python)
* editable installs leaking paths

---

# 16. Key mental model

A venv is best understood as:

```
System Python
   +
Path redirection layer
   +
Private site-packages directory
```

Nothing more.

---

# 17. What comes next

Next layer:

→ 03 — site-packages

We will go deeper into:

* how packages are physically stored
* how `.dist-info` works
* how imports map to filesystem structures
* why namespace packages behave strangely
