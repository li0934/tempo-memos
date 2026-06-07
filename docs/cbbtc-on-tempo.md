# cbBTC on Tempo（Chainlink CCIP 経由）解説メモ

出典: [Coinbase Wrapped BTC is now live on Tempo, powered by Chainlink CCIP](https://tempo.xyz/blog/cbbtc-on-tempo)（Tempo Blog、2026年5月15日公開）

クロスチェーン実装の参照先:

- **cbBTC レジストリ:** [Chainlink CCIP Directory — cbBTC](https://docs.chain.link/ccip/directory/mainnet/token/cbBTC)
- **Tempo Explorer（トークン）:** [cbBTC on Tempo Mainnet](https://explore.tempo.xyz/address/0x20C000000000000000000000c412Ec89D0c08be5?tab=token)
- **Tempo × CCIP:** [Tempo Integration Guide](https://docs.chain.link/ccip/tools-resources/network-specific/tempo-integration-guide)
- **Tempo 上の BTC 系トークン命名:** [TIP-20 — currency フィールド](https://docs.tempo.xyz/protocol/tip20/overview)

---

## 概要

**Coinbase Wrapped BTC（cbBTC）** が **Tempo メインネット** で利用可能になった、という **プロダクト告知**。Tempo エコシステムに **Bitcoin（ラップ資産として）が初めて載る** タイミングで、ブリッジ基盤は **Chainlink CCIP**（Cross-Chain Interoperability Protocol）。

記事の主メッセージは次のとおり。

| 論点 | 内容 |
|------|------|
| **何が増えたか** | Tempo 上で **cbBTC** を保有・利用できる（流通量 **$5B+** の cbBTC エコシステムへの接続） |
| **どう届くか** | **Chainlink CCIP** が Coinbase のラップ資産向け **専用ブリッジ** として機能 |
| **誰向けか** | Tempo 上でビルドする **エンタープライズ・機関投資家** |
| **何に使うか（記事が強調）** | **DeFi** — 利回り（earn）、レンディング、BTC 担保クレジット、トレードなど |
| **Tempo との組み合わせ** | CCIP のセキュリティ設計 × Tempo の **高スループット決済インフラ** で、担保・DEX 流動性・BTC 残高の運用に **本番向けの道** を用意、という位置づけ |

冒頭では **「ステーブルコイン決済と同じ環境で Bitcoin も使える」** とも書かれているが、**具体的な決済フロー（MPP・手数料トークン・payment lane など）の説明は本記事にはない**。後述の「読み分け」を参照。

---

## 背景：payments-first チェーンに BTC 担保が載る意味

Tempo は **payments-first** の L1 として設計され、メインネットでも **ステーブルコイン決済・エンタープライズ向け入金/定期課金** が前面に出てきている（例: [enterprise payments](https://tempo.xyz/blog/new-features-for-enterprise-payments)、[MPP Sessions](https://tempo.xyz/blog/mpp-sessions)）。

本記事はその流れの中で、**エンタープライズが決済プロダクトに DeFi 機能を組み込みたい** という需要への応答として書かれている（Tempo GTM の Dan Romero コメント）。つまり **「支払いレールそのものを BTC にする」** というより **「同じチェーン上でステーブルコイン決済と BTC 担保・運用を両立する」** イメージに近い。

---

## 仕組み：CCIP 経由で cbBTC が Tempo に届く

### ブログが述べる CCIP 選定理由

Tempo は **機関向けユースケース** に必要な **機関グレードのセキュリティ** のために CCIP を選んだ、と記載。

- **各 CCIP ブリッジレーン** は、独立した **セキュリティレビュー済みノードオペレーターが最低 16 台** 以上で冗長検証する（他ブリッジとの差別化として記載）
- **ネイティブのレートリミット** が **サーキットブレーカー** として機能するリスク管理

Coinbase 側は、**Coinbase が発行するラップ資産（cbBTC など）を他チェーンへ運ぶ公式ブリッジは CCIP のみ** である点を強調している（記事原文: *exclusive bridging infrastructure for Coinbase's wrapped assets*）。**cbBTC だけのための CCIP** や **Tempo 専用の CCIP** という意味ではない — CCIP は汎用のクロスチェーンプロトコルで、Coinbase が自社ラップ資産の **唯一の認可ブリッジ** として CCIP を指定している、という関係。

### Tempo 上の cbBTC（Explorer / TIP-20）

[Tempo Explorer — Token タブ](https://explore.tempo.xyz/address/0x20C000000000000000000000c412Ec89D0c08be5?tab=token) で確認できるオンチェーン情報（2026年6月時点）:

| 項目 | 値 |
|------|-----|
| 名称 | Coinbase Wrapped BTC |
| シンボル | cbBTC |
| Token address | `0x20C000000000000000000000c412Ec89D0c08be5` |
| 種別 | **TIP-20 Native Token**（プリコンパイル） |
| **Decimals** | **6**（TIP-20 固定 — Base 等の 8 decimals cbBTC とは異なる） |
| **`currency`** | **`BTC`**（TIP-20 の参照資産識別子） |
| `paused` | `false`（RPC `eth_call` で確認） |

[TIP-20 仕様](https://docs.tempo.xyz/protocol/tip20/overview) どおり、BTC を 1:1 で追跡するブリッジトークンは `currency = "BTC"` — cbBTC もそれに従う。

**アドレスの権威:** 実装・残高照会・Explorer リンクは **上記 Explorer アドレス** を正とする。[CCIP Directory](https://docs.chain.link/ccip/directory/mainnet/token/cbBTC) に載る Tempo 向け token address は **桁数が Tempo の 20-byte 形式と合わない表記** になっており、RPC では `invalid string length` となる。**ブリッジ設定（プール種別・アドレス）の参照は CCIP Directory、実トークン本体は Explorer** と役割を分ける。

### CCIP ブリッジ設定（Directory）

Chainlink [cbBTC ディレクトリ](https://docs.chain.link/ccip/directory/mainnet/token/cbBTC) の Tempo Mainnet 行（ブリッジ用）:

| 項目 | Tempo Mainnet |
|------|----------------|
| Token pool type | **Burn/Mint** |
| Token pool address | `0x3b54B8000000000000000000000000000046c71E` |

Tempo への CCIP 統合は **Burn/Mint** モデルが標準（[Tempo Integration Guide](https://docs.chain.link/ccip/tools-resources/network-specific/tempo-integration-guide)）。ソースチェーン側で burn / ロックし、Tempo 側の **BurnMintTokenPool** が TIP-20 トークンを **mint** する、という一般的な CCIP パターン。

### 一般ユーザーが Tempo に資産を移す方法との違い

Tempo 公式の [Getting Funds on Tempo](https://docs.tempo.xyz/guide/getting-funds) が列挙する **LayerZero / Relay / Squid / Across / Bungee** などは、主に **ステーブルコイン等の汎用ブリッジ** 向けの案内。**cbBTC を Tempo に持ってくる公式パスは本告知どおり CCIP** であり、Coinbase ラップ資産は CCIP 専用、という整理。

実際の送金 UI は **Chainlink CCIP Transporter** 等（CCIP エコシステムのツール）が入口になりやすい。接続可能な **ソースチェーン** は CCIP Directory の **Outbound/Inbound lanes** で確認する（Tempo Mainnet の [チェーンエントリ](https://docs.chain.link/ccip/directory/mainnet/chain/tempo-mainnet) を参照）。

---

## 記事が想定するユースケース

ブログ本文と各社コメントが挙げる用途（**具体プロトコル名は未記載** — 記事スコープ外として扱う）。

| カテゴリ | 記事の言い方 |
|----------|--------------|
| **担保・レンディング** | cbBTC を **レンディングの担保** に |
| **流動性** | **DEX** で深い流動性にアクセス |
| **運用・利回り** | BTC 残高で **リターン** を得る（earn products） |
| **クレジット** | **BTC 担保クレジット** |
| **トレード** | 一般的な DeFi トレード |

Tempo 側の訴求は **「決済プロダクトに DeFi を載せ、ユーザーへの付加価値を増やす」** 方向。パートナーは **Coinbase**（cbBTC 発行・カストディ）と **Chainlink**（CCIP）。

---

## どの DEX / プロトコルで cbBTC を扱えるか

ブログは **「DEX で深い流動性」** と書くが **具体的な取引所名・プール名は載せていない**。Tempo 上の **公式インフラの役割分担** と照らすと、読者が探すべき場所は次のとおり。

### レイヤの整理

| レイヤ | 名称 | cbBTC | 根拠 |
|--------|------|-------|------|
| **L1 組込み** | **Stablecoin DEX**（`0xdec0000000000000000000000000000000000000`） | **対象外** | [Exchanging Stablecoins](https://docs.tempo.xyz/protocol/exchange) — **同一原資産のステーブル同士**（USDC.e ↔ USDT0 等）向け。[TIP-20 overview](https://docs.tempo.xyz/protocol/tip20/overview) も **StablecoinDEX で取引できるのは USD 建てステーブル** と明記。cbBTC は `currency = BTC`。 |
| **プロトコル DEX** | **Uniswap v2 / v3 / v4**（Tempo 上） | **ここが cbBTC スワップの想定レーン** | [Uniswap is Live on Tempo](https://blog.uniswap.org/uniswap-is-live-on-tempo): *「stable-to-stable はネイティブ DEX、**それ以外の資産は Uniswap が liquidity layer**」*。cbBTC ↔ pathUSD / USDC.e など **非ステーブル同士・BTC 建て資産** はこの側。 |
| **集約 API** | **0x Swap API**（chainId `4217`） | **ルーティング入口** | [0x on Tempo](https://0x.org/post/0x-powers-token-swaps-for-tempo-the-payments-first-blockchain): ネイティブ Stablecoin DEX と **第三者 venue（Uniswap 等）** をまとめてルート。アプリが「どの pool か」を自前で列挙しなくても **Swap API に sell/buy token を渡す** 形。 |
| **Uniswap API** | Uniswap Web App / Wallet / API | 同上（Uniswap 製品経由） | 上記 Uniswap 記事 — Stablecoin DEX と v4 pool を **aggregator hook** で一つの swap に統合。 |

**読み分け:** Tempo の **組込み Stablecoin DEX ≠ cbBTC を売買する場所**。cbBTC 記事の「DEX」は、**Uniswap（＋ 0x 等の集約）側** を指すと [Uniswap × Tempo の公式説明](https://blog.uniswap.org/uniswap-is-live-on-tempo) と整合する。

### cbBTC の TIP-20 設定（DEX 文脈）

Explorer / RPC から分かる関連フィールド:

| フィールド | 値 | 意味 |
|------------|-----|------|
| `currency` | `BTC` | 参照資産は Bitcoin（1 unit ≈ 1 BTC 設計） |
| `quoteToken` | pathUSD（`0x20c0000000000000000000000000000000000000`） | TIP-20 の **クォート通貨** として pathUSD を指定（`quoteToken()` RPC で確認） |

`quoteToken` が pathUSD でも、**Stablecoin DEX の「同一原資産ステーブル取引」には載らない**（`currency` が BTC のため）。実スワップは **Uniswap 側の pool** 経由（下節）。

### Uniswap で cbBTC を扱っているか（2026年6月 RPC 調査）

**結論: 扱っている — ただし流動性は極めて薄い。**

| 観点 | 調査結果 |
|------|----------|
| **プロトコル** | [Uniswap is Live on Tempo](https://blog.uniswap.org/uniswap-is-live-on-tempo) どおり v2/v3/v4 が Tempo（chainId `4217`）上に存在 |
| **トークン認識** | [Uniswap default token list — tempo.json](https://raw.githubusercontent.com/Uniswap/default-token-list/main/src/tokens/tempo.json) に cbBTC 登録あり（**リスト載り ≠ 深い流動性**） |
| **cbBTC プール** | **pathUSD ↔ cbBTC** の v4 プールが **PoolManager** 上に存在（下記 tx のログで `currency0` = pathUSD、`currency1` = cbBTC を確認） |
| **スワップ実績** | cbBTC の `Transfer` ログで **PoolManager への入出金が多数**（例: block `0x11f0000` 以降で **to PM 23 件 / from PM 29 件**、全チェーン transfer **219 件**）。Uniswap 経由の swap が動いている |
| **参考 tx** | [pathUSD ↔ cbBTC swap 例](https://explore.tempo.xyz/tx/0xd015985f573fea48e57c536ceabcfd40453501f3ad926487f83621ba4e9f9ade)（PoolManager `0x33620f62c5b9b2086dd6b62f4a297a9f30347029`） |
| **circulating supply** | RPC `totalSupply()` ≒ **0.076 cbBTC**（6 decimals、2026年6月）— Tempo 上の BTC 建て残高はごく小さい |
| **Explorer UI** | [cbBTC Transfers タブ](https://explore.tempo.xyz/address/0x20C000000000000000000000c412Ec89D0c08be5?tab=transfers) が **空に見える** ことがある。**インデクサ遅延/欠落の可能性** — プール有無の判断は RPC / tx ログを優先 |

**主要コントラクト（swap ログで確認したもの）:**

| 役割 | アドレス |
|------|----------|
| **Uniswap v4 PoolManager**（cbBTC swap で使用） | `0x33620f62c5b9b2086dd6b62f4a297a9f30347029` |
| **Permit2** | `0x000000000022D473030F116dDEE9F6B43aC78BA3` |

[Uniswap v4 Deployments](https://developers.uniswap.org/docs/protocols/v4/deployments) には Tempo 向けに **別の PoolManager アドレス**（`0x00000000fc1E66C9f582566EAd00108e55F1c0C6`）も記載される。**cbBTC の実スワップログが載るのは上記 `0x33620…` 側**（2026年6月時点）。integrator は quote / ルート結果の `PoolManager` を見て判断する。

**0x / Uniswap API:** 公開 quote API は **API key 必須** のため本調査では未検証。オンチェーン証拠と token list から、**Uniswap レーンで cbBTC ルーティングは想定どおり** — 実行前に quote でスリッページ・深さを確認するのが安全。

### 現状のオンチェーン（流動性）

- **プールはある / swap は走っている** が、**circulating cbBTC ≒ 0.076** と **極小** — 「深い流動性」とは言えない
- ブログの *deep liquidity* は **cbBTC 全体の流通（$5B+）** や **将来のエコシステム** の文脈であり、**Tempo 上に既に厚い cbBTC プールがある** とは読めない

**実務:** CCIP で Tempo に cbBTC を持ってきたあと、**Uniswap / 0x の quote** で pathUSD（または USDC.e 等）へのルートと **許容スリッページ** を必ず確認する。薄いプールでは **名目上ルート可能でも実効価格が悪い** ことがある。

### レンディング・担保・earn（DEX 以外）

| 用途 | Tempo 上の状況（本メモ執筆時点） |
|------|----------------------------------|
| **BTC 担保クレジット / レンディング** | cbBTC ブログ **プロトコル名なし**。[Token List](https://tokenlist.tempo.xyz/list/4217) に cbBTC は載るが、**Aave 等のレンディング統合を Tempo が告知した公式ソースは未確認** — パートナー構築待ち、または別チェーン流動性を参照する段階と読むのが安全 |
| **earn** | 同上 — 具体 vault / プロトコル名はブログ外 |

**Stablecoin Advisory**（[tempo.xyz/advisory](https://tempo.xyz/advisory)）がエンタープライズ向けパイロットを案内している文脈と合わせ、**「何が本番で使えるか」は告知より後から載る** タイプの情報になりやすい。

---

## 読者が混乱しやすい点

### 1. 「決済チェーン」なのに cbBTC 記事が DeFi 中心なのは矛盾か？

**矛盾ではないが、レイヤが違う。**

- Tempo の **コア決済**（手数料、MPP、payment lane の主役）は引き続き **USD 系ステーブルコイン（TIP-20）** が中心（[Machine Payments](https://docs.tempo.xyz/learn/tempo/machine-payments) も Tempo 上は **stablecoins** を明記）。
- cbBTC 告知は **同じ L1 上の資産多様性** — 特に **担保・DeFi** — を広げる話。記事も **「stablecoin payments alongside Bitcoin」** と **併記** するにとどまり、**cbBTC でガス代を払う / MPP で BTC を直接払う** とは書いていない。

### 2. 小数桁（6 vs 8）に注意

Tempo 上の cbBTC は **6 decimals**（TIP-20 慣行）。Base 等の **8 decimals** cbBTC から CCIP する際は、**プール側で単位変換される** 前提だが、**アプリは Tempo 上では 6 桁で表示・計算** すべき。他チェーンの cbBTC コードをそのまま流用すると桁ずれのバグになりやすい。

### 3. 手数料トークン・Stablecoin DEX との関係

[TIP-20 overview](https://docs.tempo.xyz/protocol/tip20/overview): **トランザクション手数料** および **Stablecoin DEX での取引** に使えるのは **`USD` 建てステーブル** のみ。cbBTC（`currency = BTC`）は **組込み Stablecoin DEX では売買できない** — 非ステーブル資産のスワップは **[Uniswap on Tempo](https://blog.uniswap.org/uniswap-is-live-on-tempo)** 側（上節参照）。

### 4. 本記事だけでは分からないこと

ブログは **告知・パートナーシップ・CCIP セキュリティ叙事** が中心。次は **Explorer・Uniswap/0x の quote・CCIP Directory** で補う。

- 接続済み **ソース/宛先チェーン一覧** と **CCIP レートリミット**（Directory 上の送金上限 — cbBTC ブリッジのサーキットブレーカー。Uniswap とは別レイヤ）
- Tempo 上で cbBTC を受け取れる **TIP-403 転送ポリシー**
- **cbBTC 以外のペア**（cbBTC/USDC.e 等）の Uniswap pool — 本調査で確認できたのは **pathUSD ↔ cbBTC** のみ
- **レンディング / earn の具体プロトコル名**（告知時点では未記載）

---

## タイムライン（記事が示す文脈）

本告知（2026年5月15日）は、記事周辺の Tempo ロードマップ叙述と合わせて読むと、**メインネット稼働 → エンタープライズ向け決済機能 → Stablecoin Advisory → BTC（cbBTC）DeFi 接続** という **機関向けスタックの拡張** の一コマに見える。正確な日付の権威は Tempo 公式発表に従う。

---

## まとめ：今日の Tempo ビルダー向けチェックリスト

1. **cbBTC は Tempo メインネットで CCIP 経由が公式ルート** — Coinbase ラップ資産は CCIP 専用ブリッジ。
2. **主用途は記事どおり DeFi 寄り**（担保・レンド・earn・トレード）。**ステーブルコイン決済インフラの代替** ではない。
3. **実装時は** Tempo 上 **6 decimals**、TIP-20 の **転送ポリシー**、**USD ステーブルでの gas** を前提に設計。
4. **トークンアドレス** は [Tempo Explorer](https://explore.tempo.xyz/address/0x20C000000000000000000000c412Ec89D0c08be5?tab=token) を正とする。**CCIP プール** は [CCIP Directory — cbBTC](https://docs.chain.link/ccip/directory/mainnet/token/cbBTC) で最新値を確認。
5. **cbBTC のスワップ** — 組込み **Stablecoin DEX では不可**。**Uniswap v4（pathUSD ↔ cbBTC プール確認済み、＋ 0x / Uniswap API 集約）** が公式に想定されるレーン。circulating ≒ **0.076 cbBTC** と極薄 — 実行前に quote で **実効流動性** を確認。
6. **決済・MPP・サブスク** との関係を整理するなら、同リポジトリの [enterprise-payments.md](enterprise-payments.md) / [mpp-sessions.md](mpp-sessions.md) と併読。

---

## 参考リンク

| リンク | 内容 |
|--------|------|
| [Tempo Blog — cbBTC on Tempo](https://tempo.xyz/blog/cbbtc-on-tempo) | 本メモの一次ソース |
| [Tempo Explorer — cbBTC](https://explore.tempo.xyz/address/0x20C000000000000000000000c412Ec89D0c08be5?tab=token) | オンチェーン TIP-20 メタデータ（`currency` = BTC など） |
| [Chainlink CCIP — cbBTC token](https://docs.chain.link/ccip/directory/mainnet/token/cbBTC) | CCIP ブリッジ用プール種別・アドレス |
| [Chainlink — Tempo Integration Guide](https://docs.chain.link/ccip/tools-resources/network-specific/tempo-integration-guide) | TIP-20 × BurnMintTokenPool × CCIP の技術フロー |
| [Tempo — TIP-20 overview](https://docs.tempo.xyz/protocol/tip20/overview) | BTC 建てブリッジトークンの `currency` 規約 |
| [Tempo — Exchanging Stablecoins](https://docs.tempo.xyz/protocol/exchange) | 組込み Stablecoin DEX（cbBTC 対象外） |
| [Uniswap — Live on Tempo](https://blog.uniswap.org/uniswap-is-live-on-tempo) | 非ステーブル資産の liquidity layer |
| [0x — Swap API on Tempo](https://0x.org/post/0x-powers-token-swaps-for-tempo-the-payments-first-blockchain) | DEX 集約ルーティング（chainId 4217） |
| [Tempo Token List](https://tokenlist.tempo.xyz/list/4217) | cbBTC 登録（bridgeInfo: Base cbBTC） |
| [Tempo — Getting Funds](https://docs.tempo.xyz/guide/getting-funds) | 汎用ブリッジ（cbBTC とは別入口） |
