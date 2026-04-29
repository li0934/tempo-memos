# Agent instructions (Cursor / automation)

Human-facing narrative and conventions: **[README.md](README.md)** (Japanese).

Machine-readable index + English hints: **[llms.txt](llms.txt)**.

## When you change this repo

1. **Memos** live under **`docs/`** only (optional subfolders). Body text is **Japanese** Markdown.
2. **Filenames** must describe content (English-style slugs are fine), not bare dates.
3. When you **add or rename** a memo under `docs/`, update the **“Index of Japanese articles”** section in **`llms.txt`** with the repo-relative path (e.g. `docs/foo.md`).
4. **`llms.txt`** body stays **English**; do not translate it to Japanese unless the maintainer asks.
5. Do **not** invent Tempo protocol or product facts—cite official docs or leave gaps.
6. Whenever you learn something **non-obvious** about Tempo docs, tooling, or this repo’s workflow while editing, **append a short bullet to [Learnings](#learnings) below** (one line is enough). If a prior bullet is wrong, fix it and keep the section honest.

## Learnings

Operational and documentation insights discovered while working here. **Update this section whenever new 気づき arise** (see rule 6 above).

- **Access Keys — where the real detail lives:** fetching or summarizing only [`guide/tempo-transaction#access-keys`](https://docs.tempo.xyz/guide/tempo-transaction#access-keys) often yields a thin outline. For implementable detail, cross-read [`guide/use-accounts/authorize-access-keys`](https://docs.tempo.xyz/guide/use-accounts/authorize-access-keys) (Wagmi / `authorizeAccessKey` examples) and [`TIP-1011`](https://docs.tempo.xyz/protocol/tips/tip-1011) (periodic limits, call scoping). See also [`AccountKeychain`](https://docs.tempo.xyz/protocol/transactions/AccountKeychain) from the same doc set.

## Authority order (avoid drift)

- Editorial rules for people: **README.md**
- Discoverability for tools: **llms.txt**
- Imperative checklist + **accumulated repo learnings**: **this file**
