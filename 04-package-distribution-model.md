# 04 - Package Distribution Model

This chapter explains what a Python package distribution actually is.

Up to this point, we have focused on runtime environments and installed files. Now we shift one step earlier:

> before a package appears in `site-packages`, what form does it exist in?

That question matters because Python packaging is not just about imports. It is also about how projects are built, described, published, and turned into installable artifacts.

If you understand the distribution model, you understand what `pip` is downloading and why installation sometimes triggers a build.

---

# 1. The short version

There are two main distribution forms in modern Python packaging:

- source distribution (`sdist`)
- built distribution, usually a wheel (`.whl`)

They serve different purposes:

- an `sdist` is source code plus packaging metadata needed to build the project
- a wheel is a prebuilt installable artifact

When you run `pip install some-package`, `pip` is typically trying to obtain one of those forms and install it into your environment.

---

# 2. The three different "shapes" of a package

For learning purposes, it helps to separate three forms of the same project:

1. source tree
2. distribution artifact
3. installed environment contents

Example mental flow:

```text
project source tree
      ->
sdist or wheel
      ->
files installed into site-packages
```

These are not the same thing.

That distinction is one of the biggest steps toward understanding packaging rigorously.

---

# 3. The source tree is for development

The source tree is your working project directory.

It may contain things like:

```text
myproject/
├── pyproject.toml
├── README.md
├── src/
│   └── myproject/
├── tests/
└── ...
```

This form is designed for development, not for direct distribution.

It may include:

- tests
- docs
- CI configuration
- local scripts
- development-only files

Some of those belong in released artifacts. Some do not.

Packaging is partly about converting this source tree into a standardized form that installers can understand.

---

# 4. A distribution is an installable or buildable artifact

A distribution is the packaged form of a project that tools can publish and install.

The two most important kinds are:

- `sdist`
- wheel

These are standardized artifact formats, not just arbitrary zip files with code inside.

They exist so tools can agree on:

- how a package identifies itself
- what version it is
- what dependencies it declares
- how installation should proceed

Without common artifact formats, packaging tooling would be much more fragile and ad hoc.

---

# 5. What an `sdist` is

An `sdist`, or source distribution, is the source form of a release artifact.

It usually contains:

- the project source code
- packaging metadata
- build configuration
- files needed to build a wheel

It is often distributed as a `.tar.gz` archive.

Conceptually, an `sdist` says:

> here is the source release of this project in a standard form that build tools can work from

This is different from "whatever happened to be in the Git repository at the moment."

An `sdist` is a packaging artifact, not just a source checkout.

---

# 6. What a wheel is

A wheel is a built distribution.

It is usually a `.whl` file, which is a standardized archive format designed for fast installation.

Conceptually, a wheel says:

> this project has already been built into an installable form

That is why wheels are usually much faster to install than source distributions.

With a wheel, the installer often only needs to:

- unpack files
- place them in the right locations
- record metadata
- generate scripts if needed

No full build step is required in the common case.

---

# 7. Why both forms exist

If wheels are so convenient, why keep source distributions at all?

Because they solve different problems.

`sdist` is useful because:

- it preserves the source form of a release
- it allows building on platforms where no wheel is available
- it supports transparency and reproducibility at the source level

Wheels are useful because:

- they install much faster
- they avoid local build complexity
- they improve the user experience for packages with compiled code

So the ecosystem keeps both:

- source for buildability and release completeness
- wheels for fast, reliable installation

---

# 8. The role of `pyproject.toml`

In modern packaging, `pyproject.toml` is a central file.

It typically tells tools things like:

- which build backend to use
- what build requirements are needed
- project metadata such as name and version

A simplified example:

```toml
[build-system]
requires = ["setuptools>=61", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "myproject"
version = "0.1.0"
dependencies = ["requests>=2.31"]
```

This file is part of how Python packaging became more standardized.

Instead of every tool inventing its own private installation story, modern packaging leans on shared metadata and build interfaces.

---

# 9. Build backend vs installer

Another important distinction:

- the build backend knows how to build the project
- the installer knows how to install the resulting artifact

Examples of build backends include:

- `setuptools`
- `hatchling`
- `flit`
- `poetry-core`

Examples of installers or environment-facing tools include:

- `pip`
- `uv`

This division of responsibility is important.

`pip` is not the author of your package's build logic.

Instead, it often delegates building to the backend declared by the project.

That is why installation can sometimes involve a build environment and backend-specific behavior.

---

# 10. What `pip install` may actually do

When you run:

```bash
python -m pip install some-package
```

the installer may have to choose between different paths.

In a simplified model, it may:

1. find a compatible wheel and install it
2. fall back to an `sdist`
3. build a wheel locally from that source distribution
4. then install the built result

This is one reason installation can feel unpredictable to beginners.

The same command may:

- finish in seconds on one machine
- trigger a full local build on another

That difference is often about artifact availability and compatibility, not randomness.

---

# 11. Metadata is part of the contract

A package distribution is not just code.

It also carries metadata that tools rely on, such as:

- distribution name
- version
- dependency requirements
- Python version compatibility
- entry points

This metadata helps answer questions like:

- can this package be installed on Python 3.12?
- what other packages must be installed with it?
- should this package expose a command-line tool?

That metadata eventually shows up again in installed `.dist-info` directories.

So there is a through-line from:

- project metadata in the source
- metadata embedded in distribution artifacts
- metadata recorded after installation

---

# 12. Standards are what make interoperability possible

Modern Python packaging can feel fragmented because many tools exist.

But the ecosystem works as well as it does because those tools increasingly meet at shared standards.

Standards define things like:

- how build requirements are declared
- how metadata is represented
- what a wheel looks like
- how installers talk to build backends

This means a project can often switch build tooling without inventing an entirely new installation model.

That interoperability is one of the quiet strengths of modern packaging.

---

# 13. Why source builds can be painful

Once native code enters the picture, source distributions become much more demanding.

Installing from source may require:

- a compiler
- Python headers
- platform-specific build tools
- system libraries
- CUDA or other external toolchains

This is where packaging stops being only about Python and starts touching the operating system and native build ecosystem.

That is why many ML and scientific packages strongly prefer wheels for end users.

For those packages, "no compatible wheel available" can mean "you are about to attempt a complex local build."

---

# 14. A practical mental model for release artifacts

If you want one clean picture to keep in your head, use this:

```text
source tree
   ->
build backend produces artifacts
   ->
sdist and/or wheel
   ->
installer selects a compatible artifact
   ->
files are installed into site-packages
```

That flow connects development, publishing, and installation.

Once you see that flow clearly, `pip install` becomes much less mysterious.

---

# 15. What this chapter should give you

After this chapter, the useful takeaway is:

- a source tree is not the same as a distribution artifact
- an `sdist` is source prepared for packaging workflows
- a wheel is a built artifact prepared for installation
- installers and build backends have different responsibilities
- metadata is part of the package, not an optional extra

These distinctions are foundational for everything that comes next.

---

# 16. What comes next

Next: `05 - Wheels`

There we will go deeper into:

- what a wheel really contains
- how wheel compatibility tags work
- why wheels matter so much for speed and stability
- why ML packages become painful when a compatible wheel does not exist
