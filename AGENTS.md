# Agent instructions (Cursor / automation)

Human-facing narrative and conventions: **[README.md](README.md)** (Japanese).

Machine-readable index + English hints: **[llms.txt](llms.txt)**.

## When you change this repo

1. **Memos** live under **`docs/`** only (optional subfolders). Body text is **Japanese** Markdown.
2. **Filenames** must describe content (English-style slugs are fine), not bare dates.
3. When you **add or rename** a memo under `docs/`, update the **“Index of Japanese articles”** section in **`llms.txt`** with the repo-relative path (e.g. `docs/foo.md`).
4. **`llms.txt`** body stays **English**; do not translate it to Japanese unless the maintainer asks.
5. Do **not** invent Tempo protocol or product facts—cite official docs or leave gaps.

## Authority order (avoid drift)

- Editorial rules for people: **README.md**
- Discoverability for tools: **llms.txt**
- Imperative checklist for agents editing files: **this file**
