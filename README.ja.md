<div align="center">

```
██████╗ ██████╗ ██████╗  █████╗ ██╗
██╔══██╗██╔══██╗██╔══██╗██╔══██╗██║
██║  ██║██║  ██║██║  ██║███████║██║
██║  ██║██║  ██║██║  ██║██╔══██║██║
██████╔╝██████╔╝██████╔╝██║  ██║██║
╚═════╝ ╚═════╝ ╚═════╝ ╚═╝  ╚═╝╚═╝
```

# dddai-bulk-apikey

**APIキー付きのDDD AIアカウントを一括作成 — OpenAI互換の推論 + 9Routerコネクタ。**

[![Node](https://img.shields.io/badge/Node-%E2%89%A518-3DA639?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-blue?style=for-the-badge)](#)
[![License](https://img.shields.io/badge/license-MIT-6ea8fe?style=for-the-badge)](LICENSE)
[![Models](https://img.shields.io/badge/models-2%20served-412991?style=for-the-badge)](https://api.dddai.dev/v1/models)
[![9Router](https://img.shields.io/badge/9Router-connector-8B5CF6?style=for-the-badge)](#-connect-to-9router)
[![OmniRoute](https://img.shields.io/badge/OmniRoute-connector-8B5CF6?style=for-the-badge)](#-connect-to-omniroute)
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 クイックスタート](#-quick-start) · [🎁 無料枠](#-free-tier--how-much-you-get) · [🤖 モデル](#-models--all-2-of-them) · [🧪 APIの使い方](#-how-to-use-the-api) · [✅ 検証済み](#-verified-working--every-key-is-tested) · [🧭 仕組み](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [🔌 OmniRoute](#-connect-to-omniroute) · [⚙️ オプション](#️-cli-options) · [🛰️ プロキシ](#️-behind-a-proxy) · [🐞 問題報告](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ これが存在する理由

DDD AIは **`turnstile_check: false` のnew-apiフォーク** を運用しています — サインアップのゲートはメールアドレス、パスワード、そしてパネルが送ってくる6文字のコードだけです。だからこそバッチ処理が可能なのです: captchaソルバーもブラウザも不要です。各アカウントは独自のクォータで始まり、APIはOpenAI互換で、**モデルはツールを呼び出せます**(実測: 実際の引数を伴う `tool_calls`)。そのため、キーは単なるチャットだけでなく、エージェント型クライアントでも利用できます。

## 🎁 無料枠 — どれだけ得られるか

| 項目 | 値 |
| --- | --- |
| プラン | 無料アカウント、発行されたキーごとに1つ |
| サインアップ時のクォータ | アカウントあたり約 **7,300** パネル単位(既存の2アカウントで実測) |
| 1回の呼び出しコスト | ライブゲートは `deepseek-v4.1-flash` で動作(`ratio 37.5`、`completion 1`、`quota_type 0`)、短いチャットで数単位を消費 |
| アカウントあたりのキー数 | 任意の数(`POST /api/token/`)、ただし **アカウント上のすべてのキーは同じアカウントクォータを消費します** — だからこそこのツールは **キーごとに1アカウント** を発行します |
| グループ | トークンは `deepseek` グループに属する必要があります。パネルの `default` グループは `503 No available channel for model deepseek-v4.1-flash under group default (distributor)` を返します |

> 💡 無料枠はDDD AIが設定しており、変更される可能性があります。最新の数値は [dddai.dev](https://dddai.dev) で確認してください — そしてツール自身のゲートに判断させてください:

```bash
node src/cli.js --verify-only     # every key in keys.txt is re-tested with a real chat call
```

## 🤖 モデル — 全部で2つ

`GET https://api.dddai.dev/v1/models` は **2つのID** を返し、どちらもチャット呼び出しでライブ検証済みです(終了コード固定、実際のコンテンツ):

| モデルID | チャット | ツール |
| --- | --- | --- |
| `deepseek-v4.1-flash` | **200** — このツールが書き込むすべてのキーのゲートモデル | **200**、引数を伴う `finish_reason: tool_calls` |
| `deepseek-v4-pro` | 200 — 同じエンドポイント、同じキー | 同じツールサポートを継承 |

> ゲートは1つのIDではなく、短い候補リストをたどります: アカウントが正常でも、特定のモデルでチャネルがダウンしていることがあります。

## 🧪 APIの使い方

ベースURL **`https://api.dddai.dev/v1`** — OpenAI互換。

```python
from openai import OpenAI
c = OpenAI(base_url="https://api.dddai.dev/v1", api_key="<your key>")
print(c.chat.completions.create(model="deepseek-v4.1-flash",
      messages=[{"role": "user", "content": "hi"}]).choices[0].message.content)
```

ツール呼び出しも同じように機能します — 通常の `tools` 配列を渡し、`choices[0].message.tool_calls` を読み取るだけです。

## 🛠 クイックスタート

```bash
cd dddai-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # optional — without it the tool runs direct
node src/launcher.js        # interactive: mint / verify / screen / settings

# or directly:
node src/cli.js --count 3 --pool proxies.txt
node src/cli.js --verify-only
```

メールボックスのレールは **Emailnator** で、アカウントごとにプレーンな `@gmail.com` を発行します — パネルはサーバー側でドメイン/構文ポリシーを強制しているため(実測: `user@example.com`、ドット付きgmail、outlook/hotmail/mailtrap は `EMAIL_ADDRESS_REJECTED`)、プレーンなgmailだけが受け入れられるアドレス種別です。パネルには **captchaがありません** ので、トークンやソルバーは一切不要で、レール呼び出しはパネル呼び出しと同じ出口プールを利用します。

### 🖱️ ランチャー

ランチャーはプロキシリスト(任意)、カウント、ゲートモデルを尋ね、それらを `launcher.config.json` に記憶し、CLIを実行します — mint、verify、screen exits、settings、help、Connect to 9Router。

## ✅ 検証済み — すべてのキーがテストされています

| チェック | 結果 |
| --- | --- |
| `GET /api/status` | `turnstile_check: false` · `email_verification: true` · `password_register: true` |
| `GET /api/verification?email=…` | **200** `{"success":true}` — パネルが6文字のコードをメール送信 |
| `POST /api/user/register` | **200** — コードを要求したのと同じcookie jarで |
| `POST /api/user/login` | **200** → `access_token`; 以降のすべての呼び出しは `New-Api-User: <id>` も伴います |
| `POST /api/token/` の後に `POST /api/token/<id>/key` | リストはキーをマスクします; revealエンドポイントが平文を返します |
| `POST /v1/chat/completions` | **200** 実際のコンテンツ付き — キーはこれを通った後にのみ `keys.txt` に到達します |
| `tools` プローブ(`get_time` スキーマ) | **200** `finish_reason: tool_calls`、`{"timezone":"Europe/Istanbul"}` を伴う1回の呼び出し |

> **品質ゲート。** ここにあるすべてのキーは、`keys.txt` に書き込まれる前にライブの `chat-200` を通過しています
> (死んだキーは決して出荷されません)。ルーターのコンボもモデルレベルで同じルールに従います: エントリは
> **モデルが `chat-200` を返し、ツールを呼び出せる場合にのみ** 追加されます — ツールサポートのないモデル、
> 死んだキー、死んだプロバイダは追加されずに破棄されます。検証ステップはいつでも再実行できます;
> 失敗は報告され、決して隠されません。

## 🧭 仕組み

```
inbox (one address per account)                                  a fresh address per account
   │
   ├─▶ sign-up: DDD AI page → e-mail address → code / link from the inbox
   ├─▶ session: the account's cookies or access token in hand
   ├─▶ key mint: the account's key endpoint → plaintext key returned once (written to disk immediately)
   └─▶ live proof: POST https://api.dddai.dev/v1/chat/completions → real content = the key is live
```

| ステップ | 呼び出し | 備考 |
| --- | --- | --- |
| メールボックス | Emailnator `POST /api/generate-email` | アカウントごとに新しいプレーンな `@gmail.com` — パネルのポリシーは `+`/ドット付きのエイリアスや他のすべてのレールのドメインを拒否します |
| コード | `GET /api/verification?email=…` | コードメールはパネル自身のリレードメインから届きます |
| 登録 | `POST /api/user/register` | `{username,password,email,verification_code}` — コード要求と同じcookie jar |
| セッション | `POST /api/user/login` | `access_token` + アカウントID; `/api/*` は `New-Api-User: <id>` も要求します |
| トークン | `POST /api/token/` `{group:'deepseek'}` → `POST /api/token/<id>/key` | グループが重要です: `default` にはこのモデルのチャネルがありません |
| ゲート | `POST /v1/chat/completions` | 実際のコンテンツ付きの200 = 保持; 閉じたグループは `/api/pricing` の別のグループで再試行されます |

## ⚙️ CLIオプション

```bash
node src/cli.js --count N        # mint N keys (each stops at the live chat gate)
node src/cli.js --verify-only    # re-prove every key in keys.txt
node src/cli.js --screen         # exit pool self-test against api.dddai.dev
node src/cli.js --connect-9router
node src/cli.js --help
```

| 環境変数 | デフォルト | 意味 |
| --- | --- | --- |
| `DDDAI_PROXIES` | `proxies.txt` | 出口プール、1行につき1プロキシ — 任意: プールがなければCLIは直接接続します |
| `DDDAI_MODEL` | `deepseek-v4.1-flash` | ライブ検証用のモデル |

| フラグ | 意味 |
| --- | --- |
| `--pool <file>` | プロキシリスト(環境変数より優先) |
| `--proxy <url>` | すべてのパネル呼び出しに1つの出口を固定 |
| `--group <name>` | トークングループ(デフォルト `deepseek`) |
| `--exits K` | アカウントごとに試す出口候補(デフォルト6) |

## 🛰️ プロキシの背後で

**任意 — 推奨(サインアップ制限は出口IPごとです)。** このツールはプロキシなしでも動作します。

- **プールあり:** `--pool <file>` を渡す(または `DDDAI_PROXIES` を設定する)と、すべてのリクエストがそのリストの出口を経由します; 死んだ回線はその試行を失敗させ、次の出口が試されます。
- **プールなし:** CLIは `DIRECT (no proxy)` と表示し、直接接続します。サインアップ、キー発行、チャットゲートは変わりません — パケットが出ていく場所だけが異なります。
- まず `--screen` を実行してください: リスト内のすべての出口を `api.dddai.dev/api/status` に対してテストし、行ごとにOK/DEADを表示します。

## 📄 ファイル

| パス | 内容 |
| --- | --- |
| `keys.txt` | ライブ検証済みのキーが1行に1つ — git-ignored |
| `accounts.json` | email / username / password / group / gate status — git-ignored |
| `proxies.txt` | あなたの出口 — git-ignored |

## 🔌 9Routerへの接続

[9Router](https://9router.com) はセルフホスト型のOpenAI互換ルーターです:

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

デフォルト: `--prefix dddai`、`--base-url https://api.dddai.dev/v1`、`--model deepseek-v4.1-flash`。モデルは `dddai/<model id>` として提供されます。

## 🔌 OmniRouteへの接続

[OmniRoute](https://omniroute.online) はセルフホスト型のOpenAI互換ルーターです:

```bash
node src/connect-omniroute.js --base http://localhost:20128 --password 123456
```

デフォルト: `--prefix dddai`、`--base-url https://api.dddai.dev/v1`、`--model deepseek-v4.1-flash`。モデルは `dddai/<model id>` として提供されます。

## 🔑 1つのキーで得られるもの

| | |
| --- | --- |
| 🏢 プロバイダ | **DDD AI** — [dddai.dev](https://dddai.dev) |
| 🎁 プロバイダが提供するもの | 独自のクォータを持つ無料アカウント(captchaゲートなし) |
| 🔑 1つのキーで得られるもの | 独自のアカウント上の1つのキー: `https://api.dddai.dev/v1` の両方のID、チャットおよびツール検証済み |
| 🌐 APIの形式 | OpenAI互換: `POST https://api.dddai.dev/v1/chat/completions` |
| 🔒 あなたのデータ | 接触するのは `dddai.dev` / `api.dddai.dev`、Emailnator、および設定した受信トレイバックエンドのみ |

## 🐞 問題報告

DDD AIがサインアップエンドポイントを変更したり、Turnstileを追加したり、メールボックスポリシーを厳格化したり、動作するグループを移動させたりした場合、ツールはそのステップで停止し、その旨を報告します。コマンドと正確な出力を添えて **[issueを開いてください](../../issues)**。

## 📄 ライセンス

MIT © 2026 — [LICENSE](LICENSE) を参照。

<div align="center">

**[MIT](LICENSE)** © 2026 dddai-bulk-apikey contributors

</div>