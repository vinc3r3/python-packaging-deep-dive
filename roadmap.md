# Roadmap — Python Packaging Deep Dive

This document defines the intended learning progression of this repository.

The goal is to build a complete mental model of Python’s packaging and environment system, starting from first principles and gradually moving toward modern tooling and native extensions.

---

# Phase 1 — Execution & Environment Model

Goal: Understand how Python resolves and executes code.

1. 01 — Environment Model
   - What a Python environment actually is
   - sys.path and import resolution
   - Role of interpreter, stdlib, site-packages

2. 02 — venv Internals
   - How virtual environments are constructed
   - What changes when you activate a venv
   - PATH manipulation and isolation model

3. 03 — site-packages
   - Where packages live
   - How Python discovers installed libraries
   - Editable installs and `.pth` files

---

# Phase 2 — Packaging System

Goal: Understand how Python packages are distributed and installed.

4. 04 — Package Distribution Model
   - Source distributions (sdist)
   - Binary distributions (wheel)
   - Metadata and packaging standards

5. 05 — Wheels
   - What a wheel really is
   - Why wheels matter for performance and stability
   - ABI and platform compatibility

---

# Phase 3 — Dependency Resolution

Goal: Understand installation behavior and reproducibility.

6. 06 — pip Dependency Resolution
   - How pip resolves dependency graphs
   - Why installs are often non-deterministic
   - Role of version constraints and conflicts

7. 07 — uv Model
   - Modern deterministic resolution approach
   - Lockfiles and reproducibility
   - Why uv improves on pip workflows

---

# Phase 4 — Native & Compiled Extensions

Goal: Understand the boundary between Python and compiled code.

8. 08 — Native Extensions
   - C/C++ extensions in Python
   - CUDA-based packages
   - Why packages like flash-attn require prebuilt wheels
   - Build complexity and ABI coupling

---

# End Goal

By the end of this series, the reader should have a mental model of Python packaging that explains:

- what happens when you run `pip install`
- why environments break
- why wheels exist
- why modern tooling (uv, lockfiles) is emerging
- how Python interacts with compiled systems