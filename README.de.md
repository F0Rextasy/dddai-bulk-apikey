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

**DDD-AI-Konten im Bulk erstellen mit API-Schlüsseln — OpenAI-kompatible Inferenz + 9Router-Connector.**

[![Node](https://img.shields.io/badge/Node-%E2%89%A518-3DA639?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-blue?style=for-the-badge)](#)
[![License](https://img.shields.io/badge/license-MIT-6ea8fe?style=for-the-badge)](LICENSE)
[![Models](https://img.shields.io/badge/models-2%20served-412991?style=for-the-badge)](https://api.dddai.dev/v1/models)
[![9Router](https://img.shields.io/badge/9Router-connector-8B5CF6?style=for-the-badge)](#-connect-to-9router)
[![OmniRoute](https://img.shields.io/badge/OmniRoute-connector-8B5CF6?style=for-the-badge)](#-connect-to-omniroute)
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 Schnellstart](#-quick-start) · [🎁 Free-Tier](#-free-tier--how-much-you-get) · [🤖 Modelle](#-models--all-2-of-them) · [🧪 API verwenden](#-how-to-use-the-api) · [✅ Verifiziert](#-verified-working--every-key-is-tested) · [🧭 Funktionsweise](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [🔌 OmniRoute](#-connect-to-omniroute) · [⚙️ Optionen](#️-cli-options) · [🛰️ Proxys](#️-behind-a-proxy) · [🐞 Issues](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ Warum es das gibt

DDD AI betreibt einen **new-api-Fork mit `turnstile_check: false`** — das gesamte Registrierungs-Gate besteht aus einer E-Mail-Adresse, einem Passwort und dem 6-stelligen Code, den das Panel dir per Mail schickt. Genau das macht es batchfähig: kein Captcha-Solver, kein Browser. Jedes Konto startet mit eigenem Kontingent, die API ist OpenAI-kompatibel und **das Modell ruft Tools auf** (gemessen: `tool_calls` mit echten Argumenten), sodass die Schlüssel von agentenartigen Clients genutzt werden können und nicht nur für einfachen Chat.

## 🎁 Free-Tier — wie viel du bekommst

| Was | Wert |
| --- | --- |
| Plan | kostenloses Konto, eines pro erzeugtem Schlüssel |
| Kontingent bei der Registrierung | ~**7.300** Panel-Einheiten auf dem Konto (gemessen an den zwei bereits vorhandenen Konten) |
| Was ein Aufruf kostet | das Live-Gate läuft auf `deepseek-v4.1-flash` (`ratio 37.5`, `completion 1`, `quota_type 0`), ein kurzer Chat verbraucht also ein paar Einheiten |
| Schlüssel pro Konto | beliebig viele (`POST /api/token/`), aber **jeder Schlüssel eines Kontos zieht dasselbe Kontokontingent** — genau deshalb erzeugt dieses Tool **ein Konto pro Schlüssel** |
| Gruppe | das Token muss in der Gruppe `deepseek` liegen; die Gruppe `default` des Panels antwortet mit `503 No available channel for model deepseek-v4.1-flash under group default (distributor)` |

> 💡 Der Free-Tier wird von DDD AI festgelegt und kann sich ändern. Prüfe die Live-Zahlen auf [dddai.dev](https://dddai.dev) — und lass das Gate des Tools selbst entscheiden:

```bash
node src/cli.js --verify-only     # every key in keys.txt is re-tested with a real chat call
```

## 🤖 Modelle — alle 2 davon

`GET https://api.dddai.dev/v1/models` lieferte **2 IDs**, beide live mit einem Chat-Aufruf verifiziert (Exit gepinnt, echter Inhalt):

| Modell-ID | Chat | Tools |
| --- | --- | --- |
| `deepseek-v4.1-flash` | **200** — das Gate-Modell für jeden Schlüssel, den dieses Tool schreibt | **200**, `finish_reason: tool_calls` mit Argumenten |
| `deepseek-v4-pro` | 200 — derselbe Endpunkt, derselbe Schlüssel | erbt dieselbe Tool-Unterstützung |

> Das Gate geht eine kurze Kandidatenliste durch, nicht nur eine ID: Ein Kanal kann für ein einzelnes Modell ausgefallen sein, während das Konto in Ordnung ist.

## 🧪 So verwendest du die API

Base-URL **`https://api.dddai.dev/v1`** — OpenAI-kompatibel.

```python
from openai import OpenAI
c = OpenAI(base_url="https://api.dddai.dev/v1", api_key="<your key>")
print(c.chat.completions.create(model="deepseek-v4.1-flash",
      messages=[{"role": "user", "content": "hi"}]).choices[0].message.content)
```

Tool-Aufrufe funktionieren genauso — übergib ein normales `tools`-Array und lies `choices[0].message.tool_calls`.

## 🛠 Schnellstart

```bash
cd dddai-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # optional — without it the tool runs direct
node src/launcher.js        # interactive: mint / verify / screen / settings

# or directly:
node src/cli.js --count 3 --pool proxies.txt
node src/cli.js --verify-only
```

Die Mailbox-Schiene ist **Emailnator**, die pro Konto eine einfache `@gmail.com`-Adresse ausgibt — das Panel erzwingt serverseitig eine Domain-/Syntax-Richtlinie (gemessen: `EMAIL_ADDRESS_REJECTED` für `user@example.com`, gmails mit Punkten und outlook/hotmail/mailtrap), daher ist eine einfache Gmail die einzige Adressfamilie, die es akzeptiert. Das Panel hat **kein Captcha**, also wird nie ein Token oder Solver benötigt, und der Mailbox-Aufruf nutzt denselben Exit-Pool wie die Panel-Aufrufe.

### 🖱️ Der Launcher

Der Launcher fragt nach der Proxy-Liste (optional), der Anzahl und dem Gate-Modell, merkt sie sich in `launcher.config.json` und führt die CLI aus — mint, verify, screen exits, settings, help, Connect to 9Router.

## ✅ Verifiziert funktionierend — jeder Schlüssel wird getestet

| Prüfung | Ergebnis |
| --- | --- |
| `GET /api/status` | `turnstile_check: false` · `email_verification: true` · `password_register: true` |
| `GET /api/verification?email=…` | **200** `{"success":true}` — das Panel schickt einen 6-stelligen Code per Mail |
| `POST /api/user/register` | **200** mit demselben Cookie-Jar, der den Code angefordert hat |
| `POST /api/user/login` | **200** → `access_token`; jeder spätere Aufruf trägt außerdem `New-Api-User: <id>` |
| `POST /api/token/` dann `POST /api/token/<id>/key` | die Liste maskiert den Schlüssel; der Reveal-Endpunkt gibt den Klartext zurück |
| `POST /v1/chat/completions` | **200** mit echtem Inhalt — ein Schlüssel landet erst danach in `keys.txt` |
| `tools`-Probe (`get_time`-Schema) | **200** `finish_reason: tool_calls`, ein Aufruf mit `{"timezone":"Europe/Istanbul"}` |

> **Qualitäts-Gates.** Jeder Schlüssel hier hat ein Live-`chat-200` bestanden, bevor er in `keys.txt`
> geschrieben wurde (tote Schlüssel werden nie ausgeliefert). Router-Kombinationen folgen derselben Regel
> auf Modellebene: Ein Eintrag wird **nur hinzugefügt, wenn das Modell `chat-200` antwortet und Tools
> aufrufen kann** — ein Modell ohne Tool-Unterstützung, ein toter Schlüssel oder ein toter Anbieter wird
> verworfen statt hinzugefügt. Führe den Verifizierungsschritt jederzeit erneut aus;
> Fehler werden gemeldet, nie versteckt.

## 🧭 Funktionsweise

```
inbox (one address per account)                                  a fresh address per account
   │
   ├─▶ sign-up: DDD AI page → e-mail address → code / link from the inbox
   ├─▶ session: the account's cookies or access token in hand
   ├─▶ key mint: the account's key endpoint → plaintext key returned once (written to disk immediately)
   └─▶ live proof: POST https://api.dddai.dev/v1/chat/completions → real content = the key is live
```

| Schritt | Aufruf | Hinweise |
| --- | --- | --- |
| Mailbox | Emailnator `POST /api/generate-email` | eine frische einfache `@gmail.com` pro Konto — die Richtlinie des Panels lehnt Aliase mit `+`/Punkten und die Domain jeder anderen Schiene ab |
| Code | `GET /api/verification?email=…` | die Code-Mail kommt von der eigenen Relay-Domain des Panels |
| Registrierung | `POST /api/user/register` | `{username,password,email,verification_code}` — derselbe Cookie-Jar wie bei der Code-Anfrage |
| Session | `POST /api/user/login` | `access_token` + die Konto-ID; `/api/*` will außerdem `New-Api-User: <id>` |
| Token | `POST /api/token/` `{group:'deepseek'}` → `POST /api/token/<id>/key` | die Gruppe ist wichtig: `default` hat keinen Kanal für dieses Modell |
| Gate | `POST /v1/chat/completions` | 200 mit echtem Inhalt = behalten; eine geschlossene Gruppe wird mit einer anderen Gruppe aus `/api/pricing` erneut versucht |

## ⚙️ CLI-Optionen

```bash
node src/cli.js --count N        # mint N keys (each stops at the live chat gate)
node src/cli.js --verify-only    # re-prove every key in keys.txt
node src/cli.js --screen         # exit pool self-test against api.dddai.dev
node src/cli.js --connect-9router
node src/cli.js --help
```

| Env | Standard | Bedeutung |
| --- | --- | --- |
| `DDDAI_PROXIES` | `proxies.txt` | Exit-Pool, eine Zeile pro Proxy — optional: ohne Pool verbindet sich die CLI direkt |
| `DDDAI_MODEL` | `deepseek-v4.1-flash` | Modell für die Live-Verifizierung |

| Flag | Bedeutung |
| --- | --- |
| `--pool <file>` | Proxy-Liste (hat Vorrang vor der Env) |
| `--proxy <url>` | einen Exit für jeden Panel-Aufruf festlegen |
| `--group <name>` | Token-Gruppe (Standard `deepseek`) |
| `--exits K` | pro Konto versuchte Exit-Kandidaten (Standard 6) |

## 🛰️ Hinter einem Proxy

**Optional — empfohlen (das Registrierungslimit gilt pro Exit-IP).** Das Tool funktioniert auch ohne Proxy.

- **Mit einem Pool:** übergib `--pool <file>` (oder setze `DDDAI_PROXIES`), und jede Anfrage geht über einen Exit aus dieser Liste; eine tote Leitung lässt diesen Versuch scheitern und der nächste Exit wird versucht.
- **Ohne einen Pool:** die CLI gibt `DIRECT (no proxy)` aus und verbindet sich direkt. Registrierung, Schlüsselerzeugung und Chat-Gate bleiben unverändert — nur der Ursprung der Pakete ändert sich.
- Führe zuerst `--screen` aus: Es testet jeden Exit in deiner Liste gegen `api.dddai.dev/api/status` und gibt pro Zeile OK/DEAD aus.

## 📄 Dateien

| Pfad | Inhalt |
| --- | --- |
| `keys.txt` | ein live-verifizierter Schlüssel pro Zeile — von Git ignoriert |
| `accounts.json` | E-Mail / Benutzername / Passwort / Gruppe / Gate-Status — von Git ignoriert |
| `proxies.txt` | deine Exits — von Git ignoriert |

## 🔌 Mit 9Router verbinden

[9Router](https://9router.com) ist ein selbst gehosteter, OpenAI-kompatibler Router:

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

Standardwerte: `--prefix dddai`, `--base-url https://api.dddai.dev/v1`, `--model deepseek-v4.1-flash`. Modelle werden als `dddai/<model id>` bereitgestellt.

## 🔌 Mit OmniRoute verbinden

[OmniRoute](https://omniroute.online) ist ein selbst gehosteter, OpenAI-kompatibler Router:

```bash
node src/connect-omniroute.js --base http://localhost:20128 --password 123456
```

Standardwerte: `--prefix dddai`, `--base-url https://api.dddai.dev/v1`, `--model deepseek-v4.1-flash`. Modelle werden als `dddai/<model id>` bereitgestellt.

## 🔑 Was ein Schlüssel dir bringt

| | |
| --- | --- |
| 🏢 Anbieter | **DDD AI** — [dddai.dev](https://dddai.dev) |
| 🎁 Was der Anbieter gibt | ein kostenloses Konto mit eigenem Kontingent (kein Captcha-Gate) |
| 🔑 Was ein Schlüssel dir bringt | ein Schlüssel auf eigenem Konto: beide IDs von `https://api.dddai.dev/v1`, chat- und tool-verifiziert |
| 🌐 API-Form | OpenAI-kompatibel: `POST https://api.dddai.dev/v1/chat/completions` |
| 🔒 Deine Daten | nur `dddai.dev` / `api.dddai.dev`, Emailnator und das von dir konfigurierte Inbox-Backend werden kontaktiert |

## 🐞 Issues

Wenn DDD AI seine Registrierungs-Endpunkte ändert, Turnstile hinzufügt, die Mailbox-Richtlinie verschärft oder die funktionierende Gruppe verschiebt, stoppt das Tool an diesem Schritt und sagt es dir. **[Öffne ein Issue](../../issues)** mit dem Befehl und der exakten Ausgabe.

## 📄 Lizenz

MIT © 2026 — siehe [LICENSE](LICENSE).

<div align="center">

**[MIT](LICENSE)** © 2026 dddai-bulk-apikey contributors

</div>