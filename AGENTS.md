# Agent instructions (Cursor / automation)

Human-facing narrative and conventions: **[README.md](README.md)** (Japanese).

Machine-readable index + English hints: **[llms.txt](llms.txt)**.

## What these memos are for

Memos under `docs/` are **not** mere summaries of [tempo.xyz blog](https://tempo.xyz/blog) posts (or other official announcements). Their job is to **answer questions readers are likely to have** — confusion points, “how does this actually work?”, comparisons, and “what do I do today?” — by **researching ahead of time** (official docs, mpp.dev, SDK source when needed) and explaining clearly in Japanese.

**Do not** stop at paraphrasing the blog. **Do** anticipate reader friction and resolve it with cited, honest answers.

**Scope limit:** Readers do **not** want exhaustive treatment of things **not yet surfaced** in product or standard demos — unmerged PRs, integrator-only paths, protocol edge cases the blog never mentions. Mention briefly or link out when relevant; **do not** make that the memo’s center of gravity. Prefer **what works now** and what the source material actually presents.

When trimming, ask: *Would a listener trying to use or explain Tempo today care?* If not, cut or defer to official docs.

## When you change this repo

1. **Memos** live under **`docs/`** only (optional subfolders). Body text is **Japanese** Markdown.
2. **Filenames** must describe content (English-style slugs are fine), not bare dates.
3. When you **add or rename** a memo under `docs/`, update **both** indexes: **「記事一覧」** in **`README.md`** (Japanese) and **“Index of Japanese articles”** in **`llms.txt`** (English one-liner per path, e.g. `docs/foo.md`).
4. **`llms.txt`** body stays **English**; do not translate it to Japanese unless the maintainer asks.
5. Do **not** invent Tempo protocol or product facts—cite official docs or leave gaps.
6. Whenever you learn something **non-obvious** about Tempo docs, tooling, or this repo’s workflow while editing, **append a short bullet to [Learnings](#learnings) below** (one line is enough). If a prior bullet is wrong, fix it and keep the section honest.

## Learnings

Operational and documentation insights discovered while working here. **Update this section whenever new 気づき arise** (see rule 6 above).

- **Access Keys — where the real detail lives:** fetching or summarizing only [`guide/tempo-transaction#access-keys`](https://docs.tempo.xyz/guide/tempo-transaction#access-keys) often yields a thin outline. For implementable detail, cross-read [`guide/use-accounts/authorize-access-keys`](https://docs.tempo.xyz/guide/use-accounts/authorize-access-keys) (Wagmi / `authorizeAccessKey` examples) and [`TIP-1011`](https://docs.tempo.xyz/protocol/tips/tip-1011) (periodic limits, call scoping). See also [`AccountKeychain`](https://docs.tempo.xyz/protocol/transactions/AccountKeychain) from the same doc set.
- **MPP Sessions — client today:** Shipped pattern = keep `tempo.session()` **SessionManager in one process** until `close()`. No `save()`/`restore()`; cross-process reuse needs extra server+client work (see mpp.dev SessionManager docs — not standard demo path).
- **MPP Credits — fund vs spend:** blog + CLI center on **card → Credits in Tempo Wallet** (`tempo wallet fund`, `whoami --credits`, [wallet.tempo.xyz/agent](https://wallet.tempo.xyz/agent)). Credits ≠ on-chain USDC.e (non-refundable, MPP-only). Coinflow handles card + merchant stablecoin settlement. **Japan vs MoonPay:** on-ramp sells withdrawable crypto (CAESP-heavy); Credits = Coinflow closed-loop prepaid ([terms](https://coinflow.cash/terms-of-service/)) — different product/regulatory posture; MoonPay lists JP unsupported, Coinflow lists JP in pay-in countries; neither explains the delta officially. Pair with [mpp-sessions.md](docs/mpp-sessions.md) for high-frequency spend, [access-keys.md](docs/access-keys.md) for signer caps.
- **cbBTC on Tempo — addresses + DEX:** Token = [Explorer `0x20C000…c412Ec89D0c08be5`](https://explore.tempo.xyz/address/0x20C000000000000000000000c412Ec89D0c08be5?tab=token) (`currency` BTC, 6 decimals). CCIP Directory Tempo token string may not match 20-byte RPC form — use Directory for **BurnMint pool** only. **Stablecoin DEX** ≠ cbBTC; non-stable swaps → **Uniswap v4** (pathUSD↔cbBTC pool on PoolManager `0x33620f62…`, swap tx confirmed) + **0x Swap API** (4217). Explorer transfer tab may be empty while RPC shows activity. Circulating supply tiny (~0.076 cbBTC, 2026-06); quote before swap.

## Authority order (avoid drift)

- Editorial rules for people: **README.md**
- Discoverability for tools: **llms.txt**
- Imperative checklist + **accumulated repo learnings**: **this file**
