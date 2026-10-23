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

**API anahtarlarıyla toplu DDD AI hesabı oluşturun — OpenAI uyumlu çıkarım + 9Router bağlayıcısı.**

[![Node](https://img.shields.io/badge/Node-%E2%89%A518-3DA639?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-blue?style=for-the-badge)](#)
[![License](https://img.shields.io/badge/license-MIT-6ea8fe?style=for-the-badge)](LICENSE)
[![Models](https://img.shields.io/badge/models-2%20served-412991?style=for-the-badge)](https://api.dddai.dev/v1/models)
[![9Router](https://img.shields.io/badge/9Router-connector-8B5CF6?style=for-the-badge)](#-connect-to-9router)
[![OmniRoute](https://img.shields.io/badge/OmniRoute-connector-8B5CF6?style=for-the-badge)](#-connect-to-omniroute)
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 Hızlı başlangıç](#-quick-start) · [🎁 Ücretsiz katman](#-free-tier--how-much-you-get) · [🤖 Modeller](#-models--all-2-of-them) · [🧪 API'yi kullanma](#-how-to-use-the-api) · [✅ Doğrulandı](#-verified-working--every-key-is-tested) · [🧭 Nasıl çalışır](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [🔌 OmniRoute](#-connect-to-omniroute) · [⚙️ Seçenekler](#️-cli-options) · [🛰️ Proxy'ler](#️-behind-a-proxy) · [🐞 Sorunlar](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ Bu neden var

DDD AI, **`turnstile_check: false` ile yeni-api fork'u** çalıştırıyor — tüm kayıt kapısı bir e-posta adresi, bir parola ve panelin size gönderdiği 6 karakterli koddan ibaret. İşte bu yüzden toplu hale getirilebilir: captcha çözücü yok, tarayıcı yok. Her hesap kendi kotasıyla başlar, API OpenAI uyumludur ve **model araçları çağırır** (ölçüldü: gerçek argümanlarla `tool_calls`), böylece anahtarlar yalnızca düz sohbet için değil, ajan tarzı istemciler tarafından da kullanılabilir.

## 🎁 Ücretsiz katman — ne kadar elde edersiniz

| Ne | Değer |
| --- | --- |
| Plan | ücretsiz hesap, üretilen anahtar başına bir tane |
| Kayıt anındaki kota | hesapta ~**7.300** panel birimi (mevcut iki hesapta ölçüldü) |
| Bir çağrının maliyeti | canlı kapı `deepseek-v4.1-flash` üzerinde çalışır (`ratio 37.5`, `completion 1`, `quota_type 0`), yani kısa bir sohbet birkaç birim yakar |
| Hesap başına anahtar | herhangi bir sayı (`POST /api/token/`), ancak **bir hesaptaki her anahtar aynı hesap kotasını çeker** — bu aracın **anahtar başına bir hesap** üretmesinin tam nedeni de budur |
| Grup | token `deepseek` grubunda olmalıdır; panelin `default` grubu `503 No available channel for model deepseek-v4.1-flash under group default (distributor)` yanıtını verir |

> 💡 Ücretsiz katman DDD AI tarafından belirlenir ve değişebilir. Güncel sayıları [dddai.dev](https://dddai.dev) adresinden kontrol edin — ve kararı aracın kendi kapısına bırakın:

```bash
node src/cli.js --verify-only     # every key in keys.txt is re-tested with a real chat call
```

## 🤖 Modeller — hepsi 2 tane

`GET https://api.dddai.dev/v1/models` **2 id** döndürdü, her ikisi de bir sohbet çağrısıyla canlı olarak doğrulandı (çıkış sabitlendi, gerçek içerik):

| Model ID | Sohbet | Araçlar |
| --- | --- | --- |
| `deepseek-v4.1-flash` | **200** — bu aracın yazdığı her anahtar için kapı modeli | **200**, argümanlarla `finish_reason: tool_calls` |
| `deepseek-v4-pro` | 200 — aynı uç nokta, aynı anahtar | aynı araç desteğini devralır |

> Kapı tek bir id değil, kısa bir aday listesini gezer: bir kanal tek bir model için kapalı olabilirken hesap sağlam olabilir.

## 🧪 API nasıl kullanılır

Temel URL **`https://api.dddai.dev/v1`** — OpenAI uyumlu.

```python
from openai import OpenAI
c = OpenAI(base_url="https://api.dddai.dev/v1", api_key="<your key>")
print(c.chat.completions.create(model="deepseek-v4.1-flash",
      messages=[{"role": "user", "content": "hi"}]).choices[0].message.content)
```

Araç çağırma da aynı şekilde çalışır — normal bir `tools` dizisi geçirin ve `choices[0].message.tool_calls` değerini okuyun.

## 🛠 Hızlı başlangıç

```bash
cd dddai-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # optional — without it the tool runs direct
node src/launcher.js        # interactive: mint / verify / screen / settings

# or directly:
node src/cli.js --count 3 --pool proxies.txt
node src/cli.js --verify-only
```

Posta kutusu hattı **Emailnator**'dır ve hesap başına düz bir `@gmail.com` verir — panel sunucu tarafında bir alan adı/sözdizimi politikası uygular (ölçüldü: `user@example.com`, noktalı gmail'ler ve outlook/hotmail/mailtrap için `EMAIL_ADDRESS_REJECTED`), bu yüzden düz bir gmail kabul ettiği tek adres ailesidir. Panelde **captcha yoktur**, dolayısıyla hiçbir zaman token veya çözücü gerekmez ve hat çağrısı panel çağrılarıyla aynı çıkış havuzunu kullanır.

### 🖱️ Başlatıcı

Başlatıcı proxy listesini (isteğe bağlı), sayıyı ve kapı modelini sorar, bunları `launcher.config.json` içinde hatırlar ve CLI'yi çalıştırır — üret, doğrula, çıkışları tara, ayarlar, yardım, 9Router'a Bağlan.

## ✅ Çalıştığı doğrulandı — her anahtar test edilir

| Kontrol | Sonuç |
| --- | --- |
| `GET /api/status` | `turnstile_check: false` · `email_verification: true` · `password_register: true` |
| `GET /api/verification?email=…` | **200** `{"success":true}` — panel 6 karakterli bir kod gönderir |
| `POST /api/user/register` | kodu isteyen aynı cookie kavanozuyla **200** |
| `POST /api/user/login` | **200** → `access_token`; sonraki her çağrı ayrıca `New-Api-User: <id>` taşır |
| `POST /api/token/` ardından `POST /api/token/<id>/key` | liste anahtarı maskeler; açığa çıkarma uç noktası düz metni döndürür |
| `POST /v1/chat/completions` | gerçek içerikle **200** — bir anahtar `keys.txt` dosyasına ancak bundan sonra ulaşır |
| `tools` sondası (`get_time` şeması) | **200** `finish_reason: tool_calls`, `{"timezone":"Europe/Istanbul"}` ile bir çağrı |

> **Kalite kapıları.** Buradaki her anahtar `keys.txt` dosyasına yazılmadan önce canlı bir `chat-200` geçti
> (ölü anahtarlar asla gönderilmez). Yönlendirici kombinasyonları model düzeyinde aynı kuralı izler: bir giriş
> **yalnızca model `chat-200` yanıtlıyorsa ve araçları çağırabiliyorsa** eklenir — araç desteği olmayan bir model,
> ölü bir anahtar veya ölü bir sağlayıcı eklenmek yerine çıkarılır. Doğrulama adımını istediğiniz zaman yeniden çalıştırın;
> hatalar bildirilir, asla gizlenmez.

## 🧭 Nasıl çalışır

```
inbox (one address per account)                                  a fresh address per account
   │
   ├─▶ sign-up: DDD AI page → e-mail address → code / link from the inbox
   ├─▶ session: the account's cookies or access token in hand
   ├─▶ key mint: the account's key endpoint → plaintext key returned once (written to disk immediately)
   └─▶ live proof: POST https://api.dddai.dev/v1/chat/completions → real content = the key is live
```

| Adım | Çağrı | Notlar |
| --- | --- | --- |
| posta kutusu | Emailnator `POST /api/generate-email` | hesap başına taze düz bir `@gmail.com` — panelin politikası `+`/nokta içeren takma adları ve diğer tüm hatların alan adını reddeder |
| kod | `GET /api/verification?email=…` | kod postası panelin kendi aktarma alan adından gelir |
| kayıt | `POST /api/user/register` | `{username,password,email,verification_code}` — kod isteğiyle aynı cookie kavanozu |
| oturum | `POST /api/user/login` | `access_token` + hesap id'si; `/api/*` ayrıca `New-Api-User: <id>` ister |
| token | `POST /api/token/` `{group:'deepseek'}` → `POST /api/token/<id>/key` | grup önemlidir: `default` bu model için kanala sahip değildir |
| kapı | `POST /v1/chat/completions` | gerçek içerikle 200 = tut; kapalı bir grup `/api/pricing` içinden başka bir grupta yeniden denenir |

## ⚙️ CLI seçenekleri

```bash
node src/cli.js --count N        # mint N keys (each stops at the live chat gate)
node src/cli.js --verify-only    # re-prove every key in keys.txt
node src/cli.js --screen         # exit pool self-test against api.dddai.dev
node src/cli.js --connect-9router
node src/cli.js --help
```

| Ortam değişkeni | Varsayılan | Anlamı |
| --- | --- | --- |
| `DDDAI_PROXIES` | `proxies.txt` | çıkış havuzu, proxy başına bir satır — isteğe bağlı: havuz olmadan CLI doğrudan bağlanır |
| `DDDAI_MODEL` | `deepseek-v4.1-flash` | canlı doğrulama için model |

| Bayrak | Anlamı |
| --- | --- |
| `--pool <file>` | proxy listesi (ortam değişkenini geçersiz kılar) |
| `--proxy <url>` | her panel çağrısı için tek bir çıkış sabitle |
| `--group <name>` | token grubu (varsayılan `deepseek`) |
| `--exits K` | hesap başına denenen çıkış adayları (varsayılan 6) |

## 🛰️ Proxy arkasında

**İsteğe bağlı — önerilir (kayıt sınırı çıkış IP'si başınadır).** Araç proxy olmadan da çalışır.

- **Havuzla:** `--pool <file>` geçin (veya `DDDAI_PROXIES` ayarlayın) ve her istek bu listeden bir çıkış üzerinden gider; ölü bir hat o denemeyi başarısız kılar ve bir sonraki çıkış denenir.
- **Havuz olmadan:** CLI `DIRECT (no proxy)` yazdırır ve doğrudan bağlanır. Kayıt, anahtar üretimi ve sohbet kapısı değişmez — yalnızca paketlerin nereden çıktığı değişir.
- Önce `--screen` çalıştırın: listenizdeki her çıkışı `api.dddai.dev/api/status` karşısında test eder ve satır başına OK/DEAD yazdırır.

## 📄 Dosyalar

| Yol | İçerik |
| --- | --- |
| `keys.txt` | satır başına bir canlı doğrulanmış anahtar — git tarafından yok sayılır |
| `accounts.json` | e-posta / kullanıcı adı / parola / grup / kapı durumu — git tarafından yok sayılır |
| `proxies.txt` | çıkışlarınız — git tarafından yok sayılır |

## 🔌 9Router'a Bağlan

[9Router](https://9router.com) kendi sunucunuzda barındırılan, OpenAI uyumlu bir yönlendiricidir:

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

Varsayılanlar: `--prefix dddai`, `--base-url https://api.dddai.dev/v1`, `--model deepseek-v4.1-flash`. Modeller `dddai/<model id>` olarak sunulur.

## 🔌 OmniRoute'a Bağlan

[OmniRoute](https://omniroute.online) kendi sunucunuzda barındırılan, OpenAI uyumlu bir yönlendiricidir:

```bash
node src/connect-omniroute.js --base http://localhost:20128 --password 123456
```

Varsayılanlar: `--prefix dddai`, `--base-url https://api.dddai.dev/v1`, `--model deepseek-v4.1-flash`. Modeller `dddai/<model id>` olarak sunulur.

## 🔑 Bir anahtar size ne kazandırır

| | |
| --- | --- |
| 🏢 Sağlayıcı | **DDD AI** — [dddai.dev](https://dddai.dev) |
| 🎁 Sağlayıcının verdikleri | kendi kotasına sahip ücretsiz bir hesap (captcha kapısı yok) |
| 🔑 Bir anahtarın size kazandırdıkları | kendi hesabında bir anahtar: `https://api.dddai.dev/v1` adresinin her iki id'si, sohbet ve araç doğrulamalı |
| 🌐 API şekli | OpenAI uyumlu: `POST https://api.dddai.dev/v1/chat/completions` |
| 🔒 Verileriniz | yalnızca `dddai.dev` / `api.dddai.dev`, Emailnator ve yapılandırdığınız posta kutusu arka ucuyla iletişim kurulur |

## 🐞 Sorunlar

DDD AI kayıt uç noktalarını değiştirirse, Turnstile eklerse, posta kutusu politikasını sıkılaştırırsa veya çalışan grubu taşırsa, araç o adımda durur ve bunu bildirir. Komut ve tam çıktıyla birlikte **[bir sorun açın](../../issues)**.

## 📄 Lisans

MIT © 2026 — bkz. [LICENSE](LICENSE).

<div align="center">

**[MIT](LICENSE)** © 2026 dddai-bulk-apikey katkıda bulunanlar

</div>