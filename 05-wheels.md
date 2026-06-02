# 05 - Wheels

This chapter explains what a wheel really is and why it matters so much in modern Python packaging.

If the previous chapter introduced the distribution model in general, this chapter focuses on the artifact users encounter most often in successful installs:

> the wheel

Wheels are one of the main reasons modern Python packaging is usable at scale, especially for scientific computing, AI, and ML workloads.

---

# 1. The short version

A wheel is a built Python distribution designed for installation.

In the common case, installing a wheel is much simpler than building from source:

- download the wheel
- unpack it
- place files into the environment
- record metadata
- generate scripts if needed

That is why wheels are usually:

- faster to install
- more reliable for end users
- especially important for packages with native code

If no compatible wheel exists, installation may fall back to a source build, which is where complexity often begins.

---

# 2. Why wheels exist

Before wheels, installation often meant running a build process at install time.

That caused recurring problems:

- slow installs
- inconsistent builds across machines
- user systems missing compilers or headers
- failures in packages with C, C++, Rust, or CUDA code

Wheels improve this by separating:

- build time
- install time

The package author or release pipeline can build once.

Then many users can install the result without repeating that build work locally.

That separation is a huge win for both ergonomics and reliability.

---

# 3. A wheel is a built artifact, not a source tree

This distinction is worth repeating because it unlocks a lot of intuition.

A wheel is not:

- your Git checkout
- an editable install
- a source release meant for arbitrary rebuilding

A wheel is:

- a packaged installable result
- already structured for environment installation
- accompanied by metadata describing compatibility and contents

That is why wheels tend to feel smooth when they are available and compatible.

Much of the difficult work has already happened before the user runs `pip install`.

---

# 4. What a wheel file looks like

A wheel usually has a filename like:

```text
numpy-2.1.1-cp311-cp311-manylinux_2_17_x86_64.whl
```

That name is not random decoration.

It encodes important compatibility information, including:

- distribution name
- version
- Python tag
- ABI tag
- platform tag

Those tags help installers decide whether a wheel can be used in the current environment.

This is one of the key reasons wheels are powerful:

> a wheel is not just packaged code; it is packaged code plus an explicit compatibility contract

---

# 5. The broad shape of wheel contents

A wheel is an archive format, structurally similar to a zip file, with a standardized internal layout.

It commonly contains:

- importable package files
- compiled extension modules if needed
- a `.dist-info` directory with metadata

Conceptually, a wheel contains almost everything needed for installation in a ready-to-place form.

That is why wheel installation is often mostly a file placement operation rather than a build operation.

---

# 6. Pure Python wheels vs platform wheels

Not all wheels have the same compatibility story.

Some wheels are pure Python:

- mostly `.py` files
- no platform-specific compiled binaries
- often usable across many systems for the same Python compatibility range

Other wheels are platform-specific:

- contain compiled extensions
- may depend on a particular ABI
- may target a specific operating system and architecture

Examples of packages that often need platform-specific wheels:

- `numpy`
- `pandas`
- `torch`
- `orjson`
- many tokenizer, audio, and GPU-related libraries

This is where wheel compatibility becomes central.

---

# 7. Python tag, ABI tag, and platform tag

The three most important wheel compatibility dimensions are:

- Python tag
- ABI tag
- platform tag

In a filename like:

```text
package-1.0.0-cp311-cp311-manylinux_2_17_x86_64.whl
```

you can read it roughly as:

- `cp311`: built for CPython 3.11
- second `cp311`: built for the CPython 3.11 ABI
- `manylinux_2_17_x86_64`: built for a compatible Linux platform on x86_64

The precise specification is more detailed, but this mental model is enough to explain most user-facing behavior.

When a wheel does not match your environment, the installer should reject it and look for another compatible option.

---

# 8. Why ABI matters

ABI, or Application Binary Interface, becomes important when compiled code is involved.

If a package includes native extension modules, those binaries are not universally portable across all Python versions and interpreter builds.

A wheel built for:

- CPython 3.11

is not automatically safe for:

- CPython 3.12
- PyPy
- every possible platform build

That is why binary compatibility is part of the wheel identity.

For pure Python code, ABI is often much less important.

For compiled packages, it is critical.

---

# 9. Why wheels matter so much for AI and ML

Once you enter the AI, ML, and scientific Python ecosystem, wheels become far more than a convenience.

They often determine whether installation is practical at all.

Packages in this space may involve:

- C and C++ extensions
- Rust components
- BLAS or LAPACK integration
- GPU runtimes
- CUDA-specific builds
- architecture-specific optimizations

Building these from source on an end user's machine can be slow, fragile, or outright unrealistic.

That is why the existence of a compatible wheel is often the difference between:

- "installs in 20 seconds"
- "fails after 20 minutes of native build errors"

This is especially visible with packages like:

- `torch`
- `flash-attn`
- `xformers`
- `tokenizers`

For these packages, wheel availability is a major part of user experience.

---

# 10. Why a missing wheel changes everything

If `pip` can find a compatible wheel, installation is usually straightforward.

If it cannot, several things may happen:

1. it may search for another compatible wheel
2. it may fall back to an `sdist`
3. it may attempt a local build

That fallback can suddenly introduce requirements such as:

- a compiler toolchain
- Python development headers
- Rust
- CMake
- NVIDIA CUDA tooling
- system libraries

This is why users often say:

- "it works on one machine but not another"
- "installing this package suddenly started compiling things"

The root cause is often not `pip` being arbitrary.

It is that a compatible wheel was not available for that exact environment.

---

# 11. Wheels improve stability too

Speed is the obvious benefit of wheels, but stability is just as important.

A prebuilt wheel can reduce variation by avoiding:

- machine-specific compiler differences
- missing local build dependencies
- accidental source-build drift
- inconsistent user environments during compilation

This does not make wheels perfect.

But it does move complexity away from each individual install and toward controlled build pipelines.

That is a much better place for complexity to live.

---

# 12. Wheel installation is still not magic

Even with a wheel, installation can still fail.

Common reasons include:

- the wheel targets a different Python version
- the wheel targets a different architecture
- the wheel expects platform behavior your system does not satisfy
- external shared libraries are missing at runtime

So a wheel reduces build-time problems, but it does not erase all compatibility concerns.

In particular, packages that depend on GPU stacks or system-level native libraries can still fail after a successful install.

---

# 13. A practical way to read wheel behavior

When a package install behaves differently across systems, ask these questions:

1. Was a wheel available?
2. Was it compatible with this interpreter version?
3. Was it compatible with this OS and architecture?
4. Did the installer fall back to a source build?
5. Does the package depend on external native libraries at runtime?

That line of thinking often explains installation failures much faster than reading packaging as a purely Python-level problem.

---

# 14. The installer's decision is compatibility-driven

An installer does not simply grab the newest wheel and hope for the best.

It has to evaluate whether the wheel matches the current environment.

That decision depends on factors such as:

- interpreter type
- interpreter version
- ABI compatibility
- operating system
- architecture

This is why two machines can request the "same package version" and still receive different artifacts.

The package version may be the same.
The wheel selected for each environment may not be.

---

# 15. A practical mental model

Use this model:

```text
wheel
  =
[ built package files ]
      +
[ install metadata ]
      +
[ compatibility tags ]
```

That last part, compatibility tags, is the reason wheels can be both fast and selective.

They are optimized for installation, but only when the environment matches the wheel's contract.

---

# 16. What this chapter should give you

After this chapter, the useful takeaway is:

- a wheel is a built installable artifact
- wheel filenames encode compatibility information
- wheels are crucial for packages with native code
- no compatible wheel often means source-build pain
- ML packaging problems are frequently wheel-availability problems in disguise

This is one of the most important concepts in practical Python packaging.

---

# 17. What comes next

Next: `06 - pip Dependency Resolution`

There we will shift from artifact format to dependency selection:

- how `pip` decides which versions to install
- why dependency conflicts can become hard to reason about
- why reproducibility is harder than it first appears
