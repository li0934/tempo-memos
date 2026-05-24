# MPP Sessions：AI エージェント向け Web スケール決済（解説メモ）

出典: [MPP Sessions: Web-Scale Payments for AI Agents](https://tempo.xyz/blog/mpp-sessions)（Tempo Blog、2026年4月2日公開）

プロトコル詳細は **[Machine Payments Protocol (MPP)](https://mpp.dev/)** および Tempo 公式の **[machine payments guide](https://docs.tempo.xyz/guide/machine-payments)** を正とする。Tempo 上の session 実装は **[mpp.dev — Tempo session](https://mpp.dev/payment-methods/tempo/session)** も参照。

---

## 概要

**MPP Sessions** は [Machine Payments Protocol (MPP)](https://mpp.dev/) のプリミティブの一つ。Stripe と Tempo が共同で策定した MPP の上に載り、**エージェントやマシンが従量課金を大量に行う**ユースケース向けに、**オンチェーン取引を「開始」と「精算」の 2 回に集約**する。

中間の支払いは **オフチェーンの署名付き voucher（累積額）** で行い、最終的にサーバーが **最新の voucher だけ** をオンチェーンに提出して精算する。10 回でも 10,000 回でも、**レール上の確定取引は open / close の 2 回**というコスト構造が記事の中心メッセージ。

ビルドの入口: [mpp.dev — Tempo session](https://mpp.dev/payment-methods/tempo/session)、[machine payments guide](https://docs.tempo.xyz/guide/machine-payments)。

---

## なぜ必要か：エージェントの決済速度

エージェントが本当に有用になるには **支払い能力** が要る。カード系レールは秒間数千件規模でも、**人間起点の決済**を前提にした性能。エージェントが止まらず従量課金すると、その上限にすぐ当たる。

MPP Sessions は **基盤レールを 100 万 TPS 以上**までスケールさせるための仕組み、と記事は位置づけている（中間決済をオフチェーンに逃がす設計がその前提）。

---

## 速度のための二層アーキテクチャ

エージェントが「マシン速度」で支払うには、次の二つを分けて解く必要がある、という整理。

| レイヤ | 役割 |
|--------|------|
| **MPP** | マシンが **HTTP 上でリアルタイムに支払う**方法（[HTTP 402](https://mpp.dev/protocol/http-402) フローなど） |
| **MPP Sessions** | その下の **決済レールをスケール**させる方法（都度精算しない） |

MPP は **決済レール非依存**。Sessions はレール上の取引回数を抑え、高頻度・低単価の支払いを実用的にする。

### 2 つの payment intent（支払いモード）

MPP には現時点で 2 種類の intent がある（[payment methods](https://mpp.dev/payment-methods)）。

- **[charge](https://mpp.dev/payment-methods/tempo/charge)** — 一回きり（例: API 1 コール）
- **[session](https://mpp.dev/payment-methods/tempo/session)** — 継続利用（例: コンピュート従量、ストリーミング）

### ガソリンスタンドのオーソリ比喩

記事の比喩: **カードの事前オーソリ → 利用後に確定請求**。ポンプの流量（ガロン数）は都度計測するが、**レールに載るのは開始と確定の 2 取引**。MPP Sessions も同様に、**利用単位は計測するが、チェーン上は open / settle の 2 回**。

---

## Tempo 上での MPP Sessions の動き

Tempo は **ブロックチェーン上の MPP Sessions リファレンス実装**を提供する。2 取引だけでも **即時かつ確定的に決済**する必要があり、記事は **Tempo のサブ秒ファイナリティ**がそれを支える、と述べている。

### ライフサイクル（open → consume → settle）

1. **402 と支払いオプション**
   エージェントが有料リソースを要求すると、サーバーは **402** とサポートする支払い方法を返す。[Tempo payment method](https://mpp.dev/payment-methods/tempo) は **ステーブルコイン**で、`charge` と `session` の両 intent に対応（記事）。

2. **セッション開始（escrow へデポジット）**
   継続利用（例: LLM トークンのストリーム課金）では、エージェントが **オンチェーン escrow に資金を預けてセッションを開く**。サーバーはその後データをストリーム返却。

3. **消費ごとの voucher（オフチェーン）**
   サービス単位（トークン 1 個、動画チャンク、バイトなど）ごとに、エージェントは **累積額を増やした署名メッセージ（voucher）** を送る。ストリームの tick ごとに新しい voucher が交換される。

4. **サーバー側は署名検証のみ**
   voucher は **オフチェーン**でやり取り。サーバーは **暗号署名を検証**するだけでストリームを継続でき、マイクロ秒オーダー（記事）。各 voucher は前のものを置き換えるため、**精算時に必要なのは最後の 1 枚**。

5. **セッション終了と精算**
   エージェントがセッションを閉じ、サーバーが **最新 voucher をオンチェーン提出**。[settle / receipts](https://mpp.dev/protocol/receipts) により支払いが確定。**未使用分はエージェントのウォレットへ返却**。

結果: **中間 voucher が何千枚あっても、オンチェーン手数料は open + close の 2 回に摊分**され、大量マイクロペイメントが「理論」から「インフラ」に近づく、という説明。

### デモ（Tempo Moderato）

**公式デモの入口はブログ記事内の埋め込み UI**（独立したデモ専用ページ URL は公開されていない）。

- **記事（デモは「MPP session demo」見出し付近）:** [https://tempo.xyz/blog/mpp-sessions#mpp-session-demo](https://tempo.xyz/blog/mpp-sessions#mpp-session-demo)
- **デモの API バックエンド（UI ではない）:** `https://blog-tempo-mpp-sessions-demo.tempo-dev.workers.dev/api/mpp-chat` — 記事内の `MppSessionsDemo` コンポーネントがここへ接続する（JS バンドルより）

操作の流れ（記事本文）:

- ウォレット作成（Sign up）
- **PathUSD の $10 アクセスキー**を承認（Authorize）— voucher ごとの署名を人手で行わず session を回す
- クエリ送信（Send query）→ session open → トークンストリームと voucher 署名 → close で精算・未使用返却

**関連:** 同種の session 体験は [mpp.dev — Accept streamed payments](https://mpp.dev/guides/streamed-payments)（詩の SSE ストリーム課金）や [pay-as-you-go](https://mpp.dev/guides/pay-as-you-go)（写真ギャラリー）にも「Run demo」がある。ブログデモは **アクセスキー + LLM トークンストリーム** に特化したもの。

アクセスキーとの関係は **[docs/access-keys.md](access-keys.md)** および **[docs/subscriptions.md](subscriptions.md)**（periodic limits との組み合わせ）も参照。

---

## セキュリティ・保証（記事の FAQ 要約）

| 論点 | 記事の説明 |
|------|------------|
| 支払い保証 | 開始時に **escrow で資金をロック** |
| 過払い防止 | voucher は **クライアント署名**のみ有効。サーバーは署名額以上請求できない |
| 上限 | クライアントは **ロック額を超えられない** |
| 速度 | voucher 検証は **署名チェック**のみ。リクエストループ内でオンチェーン待ち不要 |
| コスト | **open（escrow 預入）+ close（精算）** の 2 オンチェーン取引。中間 voucher は **取引手数料ゼロ**（記事） |

---

## ユースケース（記事の例）

- **AI エージェントオーケストレーション** — コーディネータが検索・要約・ファクトチェックなどサブエージェントへタスク委譲。サブエージェントごとに 1 session、終了時に一度精算。定期ワークロードでは **[Subscriptions on Tempo](https://tempo.xyz/blog/subscriptions-on-tempo)** と periodic spend cap の組み合わせも言及（[docs/subscriptions.md](subscriptions.md)）。
- **LLM 推論マーケットプレイス** — 複数プロバイダへ並列クエリ、**トークン単位のストリーム課金**（[accept streamed payments](https://mpp.dev/guides/accept-streamed-payments)）。
- **動的コンピュート割当** — GPU マーケットプレイス間をリアルタイム価格で切替、**ミリ秒単位**の従量。
- **リアルタイムデータフィード** — トレーディングエージェントが tick ごとに課金（秒間数千更新）。
- **IoT マイクロトランザクション** — センサーがデータポイント単位で販売（気象、サプライチェーン位置情報など）。

いずれも **速度とコスト構造**がマイクロペイメントを実務インフラにする、という整理。エンタープライズ向け機能の文脈では **[New features for enterprise payments](https://tempo.xyz/blog/new-features-for-enterprise-payments)** も関連リンク。

---

## charge と session の使い分け（FAQ）

| 使う intent | 向く場面 |
|-------------|----------|
| **charge** | 1 リクエスト 1 回の支払い |
| **session** | LLM トークン、ストリーミングなど **継続的・従量ベース** |

---

## はじめるには・関連リンク（公式）

記事が案内しているもの:

| リソース | URL |
|----------|-----|
| MPP 仕様（HTTP 上の交渉） | [mpp.dev](https://mpp.dev/) |
| Tempo session payment method | [mpp.dev/payment-methods/tempo/session](https://mpp.dev/payment-methods/tempo/session) |
| Tempo charge payment method | [mpp.dev/payment-methods/tempo/charge](https://mpp.dev/payment-methods/tempo/charge) |
| Machine payments guide（Tempo docs） | [docs.tempo.xyz/guide/machine-payments](https://docs.tempo.xyz/guide/machine-payments) |
| Machine payments overview（Learn） | [docs.tempo.xyz/learn/tempo/machine-payments](https://docs.tempo.xyz/learn/tempo/machine-payments) |
| SDK（TypeScript / Python / Rust） | 記事は session 管理・voucher 署名・精算をライブラリが担うと案内 |

---

## このメモについて

本ファイルは上記ブログ記事の **要約・解説用メモ**であり、実装仕様の唯一の情報源ではない。voucher 形式、escrow コントラクト、402 ペイロードなど **実装の細部は MPP 仕様と Tempo 公式ドキュメントの現行版**で確認すること。
