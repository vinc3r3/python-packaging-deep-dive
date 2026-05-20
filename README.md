# Python Packaging Deep Dive

This is a collection of notes from my late-night conversations with GPT about how Python actually works under the hood.

Not tutorials in the usual sense. More like trying to reverse-engineer the mental model behind Python’s packaging system—step by step, from venvs to wheels to modern tools like uv.

---

## What this is

A structured attempt to understand:

- how Python environments actually work
- what pip really does
- why wheels exist
- why some packages (like flash-attn) are painful to install
- how modern tools like uv change the picture

---

## Why this exists

Because at some point, “just pip install it” stops being an explanation.

And I wanted to know what is actually happening.

---

## Start here

Follow the learning path in `roadmap.md`.