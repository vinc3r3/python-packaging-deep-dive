# 06 - pip Dependency Resolution

This chapter explains what `pip` is really doing when it chooses package versions.

Up to this point, we have covered environments, installed files, distribution artifacts, and wheels. Now we reach the next big source of packaging confusion:

> how does `pip` decide what actually gets installed?

This is where packaging stops being only about files and starts becoming a constraint-solving problem.

That matters because many frustrating installs are not caused by a broken environment or a missing wheel. They are caused by dependency relationships that are harder than they first appear.

---

# 1. The short version

When you run:

```bash
python -m pip install some-package
```

`pip` is usually not installing just one thing.

It is trying to build a compatible set of packages that satisfies:

- the package you asked for
- that package's declared dependencies
- version constraints across the dependency graph
- your current Python version
- artifact availability and compatibility

That is why a simple install command can sometimes lead to:

- many additional packages being installed
- version downgrades or upgrades
- long resolution time
- conflicts that are hard to read

---

# 2. Installing one package usually means installing a graph

Suppose you ask for:

```bash
python -m pip install fastapi
```

You are not really asking for one file or one directory.

You are asking for:

- `fastapi`
- its direct dependencies
- their dependencies
- and a set of versions that can coexist

So the real install target is not a single package.

It is a dependency graph.

That graph may be small for simple packages and surprisingly large for modern web, data, or ML stacks.

---

# 3. Direct dependencies vs transitive dependencies

It helps to separate two levels of dependency:

- direct dependency: a package you explicitly requested
- transitive dependency: a package required by some other dependency

Example:

- you install `fastapi`
- `fastapi` depends on `pydantic`
- `pydantic` may depend on additional packages

In that case:

- `fastapi` is direct
- the rest are transitive

Most environment complexity comes from transitive dependencies, because that is where version constraints begin to interact in non-obvious ways.

---

# 4. Version constraints are part of the graph

Packages usually do not declare dependencies as "anything at all."

They declare constraints such as:

- `requests>=2.31`
- `numpy<3`
- `pydantic>=2,<3`

These constraints matter because `pip` must find versions that satisfy all relevant requirements at once.

That can become difficult when different packages want overlapping but not identical version ranges.

Example:

- package A wants `numpy>=1.26,<2`
- package B wants `numpy>=2`

Those cannot both be true in one environment.

That is a real conflict, not just an inconvenience in tooling output.

---

# 5. Resolution is a compatibility search

At a high level, dependency resolution is a search problem.

`pip` needs to answer questions like:

- which version of each package should be chosen?
- do those versions satisfy all declared constraints?
- are compatible artifacts available for this environment?

So the resolver is not merely following a list.

It is exploring a space of possible version combinations and trying to find a consistent solution.

That is why installation can take longer when:

- the graph is large
- constraints are tight
- many versions are available
- conflicts force backtracking

---

# 6. What backtracking means

Modern `pip` may try one version choice, discover a conflict later, and then go back and try another.

That process is called backtracking.

A simplified example:

1. choose package X version 5
2. later discover that package Y requires X version `<5`
3. backtrack
4. try X version 4 instead

This is a normal part of resolution.

To users, it can look like:

- repeated downloading or metadata checks
- long install times
- confusing logs about multiple versions being considered

But conceptually it is just the resolver exploring alternatives.

---

# 7. Why installs can feel non-deterministic

Beginners often expect:

> same command, same result

In practice, plain `pip install ...` is often less reproducible than that.

Why?

Because the outcome may depend on:

- what package versions are currently available on the index
- whether dependencies released new versions yesterday
- which Python version you are using
- which wheels exist for your platform
- whether the resolver had to choose among several valid candidates

So if you run the same install command on different days or machines, you may not get exactly the same environment.

This is one of the key reasons lockfile-oriented workflows have become more important.

---

# 8. `pip` resolves against the current environment too

Resolution does not happen in a vacuum.

`pip` also has to consider the environment it is operating in, including:

- Python version
- platform and architecture
- already installed packages in some workflows
- availability of compatible wheels

That means dependency solving and artifact compatibility are linked.

A version may be logically allowed by dependency constraints but still be unusable because:

- no compatible wheel exists
- source build requirements are missing
- the package does not support your Python version

So successful resolution is not only about package names and versions.

It is also about installability in the real environment.

---

# 9. Why conflicts are often hard to read

When `pip` reports a dependency conflict, the message can feel dense.

Usually, the underlying issue is one of these:

- two packages require incompatible versions of the same dependency
- your Python version is outside a package's supported range
- the resolver cannot find a compatible artifact for a required version

The logs may look noisy because the resolver is describing a search failure.

But the core question is usually simple:

> is there at least one version combination that satisfies all constraints?

If the answer is no, the install fails.

---

# 10. Why ML and data stacks get complicated fast

Dependency resolution becomes especially painful in scientific and ML ecosystems because those stacks often combine:

- large dependency trees
- compiled packages
- platform-specific wheels
- strict Python-version support windows
- CUDA-specific package combinations

A package might be logically compatible at the dependency level but still become unusable in practice because the required binary artifacts do not exist for your interpreter or platform.

That is why an ML environment can fail for reasons that mix several layers:

- dependency constraints
- wheel availability
- native compatibility
- external runtimes

From the user's perspective, this often looks like one problem.

Under the hood, it is several packaging layers interacting at once.

---

# 11. Requirements files help, but only partly

A `requirements.txt` file is useful because it records packages you want to install.

For example:

```text
fastapi
uvicorn
requests
```

Or with pinned versions:

```text
fastapi==0.115.0
uvicorn==0.30.6
requests==2.32.3
```

This helps with repeatability, but there is an important distinction:

- a loose requirements file describes intent
- a fully pinned set gets closer to reproducibility

Even then, full reproducibility can still depend on:

- Python version
- platform
- wheel availability
- index state

So requirements files are useful, but they are not the whole story.

---

# 12. Pinning reduces uncertainty

Version pinning means choosing exact versions instead of open-ended ranges.

Example:

- loose: `numpy>=1.26`
- pinned: `numpy==1.26.4`

Pinning helps because it reduces the number of choices the resolver must make.

That usually means:

- more predictable installs
- fewer surprises across machines
- easier debugging when something breaks

The tradeoff is that pinning also reduces flexibility.

If a pinned version becomes incompatible with a new Python release or platform need, you must update intentionally.

That is usually a good trade in production workflows.

---

# 13. `pip install` is not a lockfile workflow

This is one of the most important takeaways in the whole packaging story.

Plain `pip install package-name` is a resolution workflow, not a reproducibility guarantee.

It says:

> solve this request now, against the current ecosystem state

That is different from a lockfile workflow, which says:

> recreate this previously solved environment exactly or as closely as possible

This difference is why teams eventually move beyond ad hoc installs and start caring about deterministic environment management.

---

# 14. A practical debugging checklist for resolution issues

When an install goes wrong, ask:

1. Which package did I request directly?
2. What transitive dependencies did that pull in?
3. Are there incompatible version constraints?
4. Am I on a Python version the ecosystem fully supports?
5. Is a compatible wheel available for the chosen versions?
6. Am I trying to reproduce an environment without pinned versions?

Those questions usually surface the real problem faster than treating the error as random installer behavior.

---

# 15. A mental model for `pip`

Use this model:

```text
requested packages
       +
dependency metadata
       +
version constraints
       +
environment compatibility
       ->
resolver searches for a valid install set
```

That is what makes `pip` feel more complicated than "copy package into environment."

Because it is doing much more than that.

---

# 16. What this chapter should give you

After this chapter, the important takeaway is:

- `pip install` usually means solving a graph, not installing one thing
- transitive dependencies create most of the hidden complexity
- version constraints can conflict in real ways
- environment compatibility and wheel availability affect resolution outcomes
- plain `pip` workflows are useful, but not inherently deterministic

This is the transition point where packaging starts to become dependency management.

---

# 17. What comes next

Next: `07 - uv Model`

There we will look at:

- why newer tools emphasize determinism
- how lockfiles change the packaging story
- how `uv` improves environment reproduction and install speed
- why modern workflows are shifting beyond plain `pip install`
