# 02 - venv Internals

This chapter explains what Python virtual environments actually do.

Many beginners hear "virtual environment" and imagine something closer to a container or sandbox. That is not what `venv` provides.

A virtual environment mainly changes which Python executable, package directory, and script entry points your shell and interpreter resolve to.

That sounds simple, but it explains most of the behavior people care about:

- why `python` and `pip` suddenly point somewhere else
- why packages installed in one project are not visible in another
- why activation helps, but is not strictly required
- why virtual environments do not solve system-library problems

---

# 1. The short version

A virtual environment is a lightweight Python environment built from an existing base interpreter.

Its job is to give you:

- a Python executable for that environment
- a private `site-packages` directory
- local script entry points such as `pip`
- startup configuration that points Python at the environment

It does not give you:

- a container
- a security sandbox
- kernel or process isolation
- isolation from all system dependencies

The real effect of a virtual environment is:

> path redirection for Python and Python-installed tools

---

# 2. Why `venv` exists

Without virtual environments, Python packages are often installed into one shared location.

That creates familiar problems:

- project A needs one version of a library
- project B needs another
- global installs accidentally affect unrelated work
- `pip install` changes behavior outside the current project

`venv` solves this by giving each project its own package installation area while still reusing a base Python installation.

---

# 3. Creating a virtual environment

A typical command looks like this:

```bash
python -m venv .venv
```

This does not create a whole new Python implementation from scratch.

Instead, it creates a directory structure that points back to a base interpreter while giving the new environment its own local package area and helper scripts.

---

# 4. Typical directory structure

After creation, a virtual environment usually looks roughly like this:

```text
.venv/
├── bin/                  # Linux/macOS command entry points
│   ├── python
│   ├── pip
│   └── activate
├── Scripts/              # Windows equivalents
│   ├── python.exe
│   ├── pip.exe
│   └── activate.bat
├── lib/
│   └── python3.x/
│       └── site-packages/
└── pyvenv.cfg
```

The exact details vary by platform, but these are the important pieces conceptually:

- a Python launcher for the environment
- environment-local install locations
- activation scripts
- a small config file that records how the environment was created

---

# 5. `pyvenv.cfg` is the key file

One of the most important files is:

```text
.venv/pyvenv.cfg
```

It often looks something like:

```text
home = /usr/bin
include-system-site-packages = false
version = 3.11.5
```

This file tells Python important facts about the environment, including:

- which base installation the venv came from
- whether system `site-packages` should be visible
- which interpreter version created it

The key setting for isolation is usually:

```text
include-system-site-packages = false
```

That means the environment should use its own package directory rather than automatically exposing globally installed packages.

---

# 6. What actually changes in a venv

The easiest way to understand a venv is to compare before and after.

Without a virtual environment, `python` may resolve to a system or globally installed interpreter.

Inside a virtual environment, you want commands like:

```bash
python
pip
```

to resolve to the environment-local versions instead.

At runtime, the important changes are usually:

- `sys.prefix` points to the virtual environment
- `sys.path` includes the environment's `site-packages`
- shell command lookup finds the environment's `python` and `pip`

Those three shifts account for most of the "it works in this venv but not that one" behavior.

---

# 7. `sys.prefix` vs `sys.base_prefix`

These values are one of the clearest ways to see a virtual environment in action:

```python
import sys
print("prefix:", sys.prefix)
print("base_prefix:", sys.base_prefix)
```

In a typical venv:

- `sys.prefix` points at the venv directory
- `sys.base_prefix` points at the original base Python installation

That gives you a quick venv check:

```python
import sys
print(sys.prefix != sys.base_prefix)
```

If that prints `True`, you are probably inside a virtual environment.

This is one of the most useful inspection tricks when debugging environment confusion.

---

# 8. What happens to `sys.path`

When Python starts inside a virtual environment, it builds `sys.path` differently from the base interpreter.

Most importantly, it adds the venv's own package directory, typically something like:

```text
.venv/lib/pythonX.Y/site-packages/
```

That means imports are resolved against packages installed in this environment.

If `include-system-site-packages` is disabled, globally installed packages should not be part of the normal third-party import set.

So the practical effect is:

- packages installed into this venv are visible here
- packages installed into some other venv are not
- global packages are hidden unless explicitly allowed

---

# 9. What gets copied, and what does not

Newcomers sometimes assume a venv contains a full duplicate of Python. Usually it does not.

A virtual environment typically includes:

- a Python executable or launcher
- `pip` and other installed script entry points
- activation scripts
- its own `site-packages` directory
- `pyvenv.cfg`

It usually does not duplicate:

- the full standard library in a completely independent form
- the entire interpreter toolchain
- operating system libraries
- external system dependencies

The exact implementation can vary by platform. Some setups use copies, some use symlinks, and some use launcher behavior.

The big picture stays the same:

> the environment is lightweight because it reuses a base interpreter and changes path resolution

---

# 10. Activation is a shell convenience layer

On Unix-like systems, activation often looks like:

```bash
source .venv/bin/activate
```

On Windows, it uses the matching script under `Scripts/`.

Activation mostly changes shell state, especially:

- `PATH`, so `python` and `pip` resolve to the venv first
- the prompt, so you can see which environment is active

It does not rewrite Python itself.

That is why this works even without activation:

```bash
./.venv/bin/python script.py
```

If you call the environment's interpreter directly, you get the same Python behavior whether or not your shell was activated first.

This is an important distinction:

> activation changes command resolution in your shell, not the meaning of the interpreter binary itself

---

# 11. Why activation is still useful

If activation is optional, why do people use it?

Because it reduces mistakes.

It helps by:

- avoiding long executable paths
- keeping `python` and `pip` paired to the same environment
- making the active environment visible in the prompt
- lowering the chance that you accidentally install into the wrong interpreter

So activation is not technically required, but it is very useful ergonomically.

---

# 12. How `pip` behaves inside a venv

Inside a virtual environment, `pip install ...` usually installs into that environment's `site-packages`.

For example:

```bash
pip install requests
```

typically targets a path like:

```text
.venv/lib/pythonX.Y/site-packages/
```

This is why packages installed in one project do not automatically appear in another project's environment.

In practice, the safest habit is:

```bash
python -m pip install requests
```

That makes the interpreter-package-manager pairing explicit.

It removes one common source of confusion:

- `pip` from one interpreter
- `python` from another

---

# 13. What a venv isolates

A virtual environment does isolate some things well.

Usually isolated:

- third-party Python packages
- Python-installed console scripts
- import resolution for those installed packages

Usually not isolated:

- operating system files
- system shared libraries such as `libc`
- GPU drivers and CUDA installations
- network access
- arbitrary environment variables unless you change them

So if a wheel depends on a missing system library, a venv will not fix that.

That is why native packages can still fail even when your virtual environment setup looks correct.

---

# 14. Common misconception

A virtual environment is not:

- Docker
- a VM
- a process sandbox
- a separate operating system environment

It is better described as:

> a Python interpreter context with redirected package and script paths

That definition is less flashy, but much closer to what really happens.

---

# 15. Why venvs still fail sometimes

Even when using venv correctly, problems still happen.

Common reasons include:

- using the wrong interpreter to create the environment
- using one `python` and a different `pip`
- broken or unexpected `PATH` ordering
- editable installs adding surprising import paths
- incompatible wheels for the active interpreter
- missing system libraries or external runtimes

A venv solves package-location problems very well.

It does not solve every problem that happens to involve Python.

---

# 16. A practical inspection checklist

When a virtual environment behaves strangely, inspect these first:

```python
import sys
import site

print("executable:", sys.executable)
print("prefix:", sys.prefix)
print("base_prefix:", sys.base_prefix)
print("in venv:", sys.prefix != sys.base_prefix)
print("site-packages:", site.getsitepackages())
print("sys.path:")
for p in sys.path:
    print(" ", p)
```

And from the shell:

```bash
which python
which pip
python -m pip --version
```

Those checks usually reveal whether:

- you launched the interpreter you thought you launched
- the venv is actually active
- `pip` is installing into the same environment your `python` is using

---

# 17. Key mental model

A virtual environment is best understood as:

```text
[ base Python interpreter ]
           +
[ redirected prefixes and paths ]
           +
[ private site-packages and scripts ]
```

That is the core of `venv`.

Everything else is convenience, platform detail, or tooling around that mechanism.

---

# 18. What comes next

Next: `03 - site-packages`

There we will look at:

- how installed packages are laid out on disk
- what `.dist-info` directories are for
- how imports relate to installed files
- why namespace packages can feel surprising
