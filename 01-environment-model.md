# 01 - Python Environment Model

This chapter explains what a Python environment is at runtime.

If you are new to packaging, it is easy to picture an environment as "a folder that contains packages." That is useful as a first approximation, but it is not the real model Python uses.

At runtime, a Python environment is defined by the interpreter you launched, the paths Python searches, and the packages visible on those paths.

That model matters because many packaging problems are really environment-selection problems in disguise:

- `pip install` succeeds, but `import` fails
- a package exists, but Python loads the wrong one
- one terminal works, another does not
- native extensions work in one interpreter and break in another

---

# 1. The short version

A Python environment is the combination of:

1. the Python interpreter binary
2. the standard library (`stdlib`) for that interpreter
3. the third-party package directories, usually `site-packages`
4. the runtime configuration that builds `sys.path`

Together, these determine:

- what `import` resolves to
- which package versions are visible
- which native binaries can be loaded
- which `python` and `pip` you are actually using

If one of those pieces changes, you may effectively be in a different environment.

---

# 2. Start with the interpreter

The interpreter is the anchor for everything else.

When you run:

```bash
python3
```

you are not just launching "Python in general." You are launching one specific executable on your machine.

That executable determines:

- the Python version, such as 3.10, 3.11, or 3.12
- the standard library location
- the default import search paths
- the ABI compatibility for compiled extensions

ABI means Application Binary Interface. In practice, it is part of what decides whether a compiled package built for one Python build can run in another.

You can inspect the active interpreter:

```python
import sys
print(sys.executable)
print(sys.version)
```

If you remember only one debugging rule, remember this:

> The real identity of your environment starts with `sys.executable`.

---

# 3. A useful mental model

When Python starts, it builds an import world around the interpreter.

You can think of the environment like this:

```text
[ Python executable ]
         +
[ standard library ]
         +
[ sys.path ]
         +
[ installed packages on those paths ]
```

Tools like `venv`, `pip`, `uv`, and `poetry` do not create magic. They mostly influence one or more parts of this structure.

---

# 4. `sys.path` is the import search list

Every `import` statement uses `sys.path`.

`sys.path` is an ordered list of locations Python will search when it tries to load a module.

You can inspect it:

```python
import sys
for p in sys.path:
    print(p)
```

For a typical interpreter session, the list usually includes:

1. the current project or working directory
2. paths from `PYTHONPATH`, if that variable is set
3. standard library directories
4. `site-packages` directories for third-party packages

The important part is not just what is in `sys.path`, but the order.

---

# 5. How import resolution works

Suppose you write:

```python
import requests
```

In the normal case, Python roughly does this:

1. walk through `sys.path` from top to bottom
2. look for something that can satisfy `requests`
3. stop at the first matching result
4. load it and cache it in `sys.modules`

For regular packages, that often means checking for things like:

- `requests.py`
- `requests/__init__.py`

The key consequence is:

> Python imports are order-dependent, not version-aware.

Python does not ask, "Which `requests` is newest?" It asks, "Which `requests` do I find first on `sys.path`?"

That is why path leaks and interpreter mix-ups cause so much confusion.

---

# 6. What `site-packages` is

`site-packages` is the main directory where third-party packages are installed.

A common path looks like:

```text
.../lib/pythonX.Y/site-packages/
```

This directory usually contains:

- installed libraries
- compiled extensions such as `.so` or `.pyd`
- metadata directories such as `.dist-info`

That metadata is how packaging tools keep track of what was installed.

This is usually where `pip install ...` puts packages.

---

# 7. How Python builds `sys.path` at startup

At startup, Python does not begin with one fixed universal path list. It builds one.

A simplified model is:

### Step 1: find the interpreter and its standard library

Some paths come from how that interpreter was built and installed.

### Step 2: process environment configuration

Environment variables can affect startup, especially:

- `PYTHONPATH`
- `PYTHONHOME`

### Step 3: run the `site` startup logic

This is where Python typically adds `site-packages`, user-site locations, and path customizations.

That is why the same machine can produce different import behavior depending on which interpreter and startup configuration you used.

---

# 8. `sys.prefix` and `sys.base_prefix`

These two values are especially useful when virtual environments enter the picture:

```python
import sys
print(sys.prefix)
print(sys.base_prefix)
```

You can think of them like this:

### `sys.base_prefix`

- the original base Python installation
- the "real" interpreter root before virtual environment redirection

### `sys.prefix`

- the currently active environment root
- often changes when you are inside a virtual environment

In a normal virtual environment, these differ.

That makes this a handy quick check:

```python
import sys
print(sys.prefix != sys.base_prefix)
```

If it prints `True`, you are probably running inside a virtual environment.

---

# 9. The role of `site`

The `site` module is part of Python startup behavior in normal runs.

Its job includes:

- adding `site-packages` directories to `sys.path`
- processing `.pth` files
- enabling per-user package directories

You can inspect some of what it sees:

```python
import site
print(site.getsitepackages())
```

Depending on platform and interpreter, you may see one or several package locations.

---

# 10. `.pth` files are a quiet but important mechanism

Inside `site-packages`, Python may process files ending in `.pth`.

A simple example:

```text
myproject.pth
```

containing:

```text
/custom/path/to/libs
```

That path can be added to `sys.path` during startup.

This mechanism is important because it helps explain behavior that can otherwise feel mysterious, including:

- editable installs
- custom path injection
- some environment-specific tooling

For newcomers, the key idea is simple:

> Not every importable package has to be physically copied into `site-packages`.

Sometimes `site-packages` only contains a pointer to somewhere else.

---

# 11. Editable installs

When you run:

```bash
pip install -e .
```

Python usually does not copy your project code into `site-packages` the way a normal install would.

Instead, packaging tooling typically makes your source directory importable from the environment, often through a `.pth` file or a link-like mechanism.

The result is:

> imports resolve directly to your working source tree

That is why editing your project files immediately affects what Python imports.

This is convenient for development, but it also means your environment may depend on your current checkout state.

---

# 12. Why environments feel isolated

Python environments often feel isolated, but the isolation is limited.

What usually changes between environments is:

- `sys.prefix`
- the interpreter path
- `sys.path`
- the visible `site-packages` directories

What does not automatically change:

- operating system permissions
- filesystem access
- network access
- access to arbitrary paths if you manually add them

So a virtual environment is not a sandbox in the operating-system sense.

It is better understood as:

> a different interpreter context with different import paths

That distinction is important. Many people hear "isolated environment" and imagine stronger separation than Python actually provides.

---

# 13. Virtual environment preview

A virtual environment is essentially a base interpreter plus redirected environment paths.

In practice, it usually changes:

- `sys.prefix` so the active environment has its own root
- `sys.path` so imports resolve to environment-specific packages
- shell `PATH` during activation so commands like `python` and `pip` point to the environment-local executables

What it does not create:

- process isolation
- filesystem isolation
- kernel-level sandboxing

So the core behavior of a virtual environment is path redirection, not security isolation.

---

# 14. The practical debugging model

When something packaging-related breaks, these are the first things to inspect:

```python
import sys
import site

print("executable:", sys.executable)
print("version:", sys.version)
print("prefix:", sys.prefix)
print("base_prefix:", sys.base_prefix)
print("site-packages:", site.getsitepackages())
print("sys.path:")
for p in sys.path:
    print(" ", p)
```

This usually tells you which interpreter you launched, which environment root is active, and where Python is looking for imports.

That is enough to explain a large share of packaging issues.

---

# 15. Key mental model

A Python environment is best understood as:

```text
[ interpreter ]
      +
[ ordered import search path ]
      +
[ packages visible on that path ]
```

Everything else, including `venv`, `pip`, and `uv`, is mainly manipulating this structure.

---

# 16. Why this matters

Many packaging problems reduce to one of these:

- wrong interpreter
- wrong `sys.path` order
- wrong `site-packages` directory
- incompatible native extension for the active interpreter
- unexpected path injection from `PYTHONPATH`, `.pth`, or user-site packages

Once you see the environment as a runtime model instead of a folder, those failures become much easier to reason about.

---

# 17. What comes next

Next: `02 - venv Internals`

There we will look at:

- how virtual environments rewrite interpreter context
- how activation changes command resolution
- why `python` and `pip` can drift apart
- what virtual environment isolation really does and does not do
