# Tempo Transactions — アクセスキー（Access Keys）解説メモ

出典: [Use Tempo Transactions — Access Keys](https://docs.tempo.xyz/guide/tempo-transaction#access-keys)（Tempo 公式ドキュメント）

### 関連ページ（実装・詳細はこちらが厚い）

（※ `tempo-transaction` の単ページ抽出だけだと概要に留まることがある。**コード付きガイド**と **TIP** をあわせて読む。）

| ページ | 内容 |
|--------|------|
| [Authorize access keys](https://docs.tempo.xyz/guide/use-accounts/authorize-access-keys) | Wagmi / Tempo Wallet / Passkey で `authorizeAccessKey` を渡す **実装例**（有効期限・上限・スコープ） |
| [TIP-1011: Enhanced Access Key Permissions](https://docs.tempo.xyz/protocol/tips/tip-1011) | 周期的上限・呼び出しスコープ・転送先制限などの **仕様（構造体レベル）** |
| [Account Keychain（仕様）](https://docs.tempo.xyz/protocol/transactions/AccountKeychain) | アカウントキーチェーン・プリコンパイルまわり |
| [Tempo Transaction 仕様](https://docs.tempo.xyz/protocol/transactions/spec-tempo-transaction) | EIP-2718 トランザクション形 |

---

## Tempo Transaction の位置づけ（このページ全体の文脈）

**Tempo Transactions** は Tempo 専用の **EIP-2718 の新しいトランザクション種別**。公式は通常の Ethereum トランザクションではなく **Tempo Transactions の利用を強く推奨**している。

同じガイドページで、次のような機能が並列で説明されている（アクセスキーはその一つ）。

| 機能 | ざっくりした意味 |
|------|------------------|
| Configurable Fee Tokens | TIP-20 を `fee_token` にすると Fee AMM が手数料換算 |
| Fee Sponsorship | 送信者とは別の fee payer が手数料を肩代わり |
| Batch Calls | `calls` で複数操作を 1 トランザクションに束ねる |
| **Access Keys** | メインアカウントから別鍵へ署名権限を委譲 |
| Concurrent Transactions | nonce key などで並列送信 |
| Scheduled Transactions | `validAfter` / `validBefore` で実行可能時間帯を指定 |

公式が案内している SDK 例: TypeScript（tempo-ts）、Rust（tempo-alloy）、Go（tempo-go）、Python（pytempo）、Foundry など。統合の目安として「< 1 時間」などの表がある。

---

## アクセスキーでできること

**Access keys** は、**プライマリ（メイン）アカウントの署名権限を、セカンダリな鍵に委譲**するための仕組み。

例として公式は **デバイスにバインドされた、取り出せない WebCrypto 鍵**（non-extractable）などをセカンダリ鍵のイメージとして挙げている。

### 流れ（ドキュメントの説明どおり）

1. **プライマリが「鍵の承認（key authorization）」に署名する**
   → その内容により、アクセスキーが **プライマリの代理でトランザクションに署名できる**権限が付与される。

2. その **承認は「次の 1 トランザクション」に添付**される。
   そのトランザクション自体は **プライマリでもアクセスキーのどちらでも署名できる**。

3. **それ以降のトランザクションは、アクセスキーだけで署名できる**（ドキュメントの「thereafter」）。

サブスクリプションや定期上限の話は別記事・別セクションとセットになるが、本ページの **T3 に関する注記**では `key_authorization` が **`TokenLimit.period`（周期的な上限）** や **`allowed_calls`（呼び出しスコープ）** を扱える、とされている（下記「T2 → T3」）。

---

## T2 → T3 でのアクセスキー周りの変更（公式の箇条書き）

ドキュメントは **T3** のトランザクション形を前提に例を示している。アクセスキーまわりでは概ね次が書かれている。

### 1. `key_authorization` の表現力

- **周期的なトークン上限**: `TokenLimit.period` など **periodic** な値がサポートされる、との記載がある。
- **呼び出しスコープ**: **`allowed_calls`** により **どの呼び出しに鍵を使えるか** を絞れる（サブスクリプション記事でいう「コントラクト／関数へのスコープ」と対応するイメージ）。

### 2. SDK・ヘルパーの利用が推奨

レガシーの **`authorizeKey(...)` ABI を直に叩くのではなく**、T3 対応のヘルパーを使うよう案内されている（名称はドキュメント原文どおり）。

| 例として挙げられている API / コマンド（公式） |
|----------------------------------------------|
| TypeScript 寄り: `client.accessKey.authorizeSync` |
| `AccountKeychain.authorize_key` |
| Go など: `keychain.AuthorizeKey` |
| Foundry / CLI: `cast keychain authorize` |

※ 実際の import パスや引数は **使用中の SDK バージョンとガイドの該当節** を見ること（このメモは名前の索引）。

### 3. 低レベルエンコードを自分で書く場合

- エンベロープを手で組むときは **`authorization_list` を `aa_authorization_list` にリネーム**する、という移行注意がある（フィーペイメント周りと同様の命名変更）。

### 4. アクセスキー署名トランザクションの制約

- **アクセスキーで署名したトランザクションからはコントラクトを新規作成できない**（**`CREATE` が実行されるフローは不可**）。
- **デプロイやそれに相当するフローはルート鍵（プライマリ側）で行う**必要がある、と明記されている。

---

## コード例が載っているページ — Wagmi（Authorize access keys）

[tempo-transaction#access-keys](https://docs.tempo.xyz/guide/tempo-transaction#access-keys) は概念・T2→T3 が中心。**実コードは** [Authorize access keys](https://docs.tempo.xyz/guide/use-accounts/authorize-access-keys) が本文付き。

公式の流れの要約:

1. Wagmi + Tempo アカウント（Tempo Wallet **または** domain-bound Passkeys）をセットアップする。
2. コネクタに **`authorizeAccessKey`** を渡す。接続時にアクセスキーが承認され、**以降はパスキー再プロンプトなし**で送金できる。
3. `Hooks.token.useTransferSync()` などで送金（サンプルでは AlphaUSD）。

コネクタ設定の骨格（公式サンプルを日本語コメントだけ付けて短縮。**動かすときは必ず原文のフルコードを参照**）:

```typescript
// tempoWallet または webAuthn コネクタの例（公式: Authorize access keys）
import { Expiry } from "accounts";
import { tempoWallet } from "accounts/wagmi"; // または webAuthn from accounts/wagmi

tempoWallet({
  authorizeAccessKey: () => ({
    expiry: Expiry.days(7), // 満了後は鍵が無効
    limits: [
      {
        token: "0x20c0000000000000000000000000000000000001",
        limit: parseUnits("100", 6),
      },
    ], // トークンごとの上限
    scopes: [
      {
        target: "0x20c0000000000000000000000000000000000001",
        selector: "transfer(address,uint256)",
      },
    ], // 呼び出せるコントラクトと関数
  }),
});
```

公式がこの例で説明しているポイント:

- **7 日で失効** — その後は鍵が無効になる。
- **スコープ** — 例では AlphaUSD 上の **`transfer` だけ**。
- **上限** — 例では **100 AlphaUSD** まで。

---

### tempo-transaction 側で名前だけ出ている API（低レイヤー / CLI）

同じ [Use Tempo Transactions](https://docs.tempo.xyz/guide/tempo-transaction#access-keys) には、**SDK ヘルパー名**が列挙されている（T3 ではレガシー `authorizeKey(...)` ABI よりこちらを推奨という趣旨）。

| 例（公式に列挙） |
|------------------|
| `client.accessKey.authorizeSync` |
| `AccountKeychain.authorize_key` |
| `keychain.AuthorizeKey` |
| `cast keychain authorize` |

Foundry / CLI:

```bash
cast keychain authorize --help
```

低レベルでエンベロープを自分で組む場合:

- **`authorization_list` → `aa_authorization_list`** にリネーム（T3）。
- **`key_authorization`** に **`TokenLimit.period`**（周期的上限）や **`allowed_calls`**（呼び出しスコープ）などを載せる、という説明がある。

---

## TIP-1011（詳細仕様の参照先）

[Enhanced Access Key Permissions](https://docs.tempo.xyz/protocol/tips/tip-1011) は、アクセスキーを次の 3 つで拡張する TIP として書かれている（要約）。

1. **周期的な支出上限** — `period` を秒で持つ `TokenLimit`（`period == 0` は一回きりの上限、`period > 0` は期間ごとのキャップとリセット）。
2. **呼び出しスコープ** — `CallScope`（`target` と `selector_rules`）。ベクタが空ならそのターゲット上の任意セレクタ、などルールが細かく定義されている。
3. **転送先などの絞り込み** — `transfer` / `approve` / `transferWithMemo` など **限定されたセレクタ**について、ABI の **第 1 引数（address）** が許可リストに入っているときだけ、など。

データ構造の全文・Solidity ABI は **TIP 本文** を正とする。このメモでは名前だけ索引に留める。

---

## 実装・運用で押さえるチェックリスト

- [ ] Tempo Transaction を使う（通常 ETH tx ではなく）。
- [ ] T3 前提か確認し、レガシー `authorizeKey` ABI ではなく **推奨ヘルパー** を検討。
- [ ] アクセスキーで **コントラクト作成をさせない**（デプロイはルート鍵）。
- [ ] periodic limits・allowed_calls を使う場合は **公式の Access Keys 詳細**と SDK の型で一致確認。
- [ ] 手動エンコード時は **`authorization_list` → `aa_authorization_list`** のリネームを忘れない。

---

## このメモについて

本ファイルは [tempo-transaction の Access Keys 節](https://docs.tempo.xyz/guide/tempo-transaction#access-keys) と [Authorize access keys（コード付き）](https://docs.tempo.xyz/guide/use-accounts/authorize-access-keys)、および [TIP-1011](https://docs.tempo.xyz/protocol/tips/tip-1011) を **読み合わせたメモ**である。実行可能なコードや確定 ABI は **常に公式の現行版** を正とする。
