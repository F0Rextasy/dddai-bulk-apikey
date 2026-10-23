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

**Массовое создание аккаунтов DDD AI с API-ключами — OpenAI-совместимый инференс + коннектор 9Router.**

[![Node](https://img.shields.io/badge/Node-%E2%89%A518-3DA639?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-blue?style=for-the-badge)](#)
[![License](https://img.shields.io/badge/license-MIT-6ea8fe?style=for-the-badge)](LICENSE)
[![Models](https://img.shields.io/badge/models-2%20served-412991?style=for-the-badge)](https://api.dddai.dev/v1/models)
[![9Router](https://img.shields.io/badge/9Router-connector-8B5CF6?style=for-the-badge)](#-connect-to-9router)
[![OmniRoute](https://img.shields.io/badge/OmniRoute-connector-8B5CF6?style=for-the-badge)](#-connect-to-omniroute)
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 Быстрый старт](#-quick-start) · [🎁 Бесплатный тариф](#-free-tier--how-much-you-get) · [🤖 Модели](#-models--all-2-of-them) · [🧪 Использование API](#-how-to-use-the-api) · [✅ Проверено](#-verified-working--every-key-is-tested) · [🧭 Как это работает](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [🔌 OmniRoute](#-connect-to-omniroute) · [⚙️ Опции](#️-cli-options) · [🛰️ Прокси](#️-behind-a-proxy) · [🐞 Проблемы](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ Зачем это нужно

DDD AI работает на **форке new-api с `turnstile_check: false`** — весь барьер регистрации сводится к адресу электронной почты, паролю и 6-символьному коду, который панель присылает вам на почту. Именно это делает процесс пакетным: никаких решателей капчи, никакого браузера. Каждый аккаунт начинается с собственной квоты, API совместим с OpenAI, и **модель вызывает инструменты** (измерено: `tool_calls` с реальными аргументами), так что ключи пригодны для клиентов агентного типа, а не только для обычного чата.

## 🎁 Бесплатный тариф — сколько вы получаете

| Что | Значение |
| --- | --- |
| План | бесплатный аккаунт, по одному на каждый созданный ключ |
| Квота при регистрации | ~**7 300** единиц панели на аккаунт (измерено на двух уже существующих аккаунтах) |
| Сколько стоит вызов | живой шлюз работает на `deepseek-v4.1-flash` (`ratio 37.5`, `completion 1`, `quota_type 0`), так что короткий чат сжигает несколько единиц |
| Ключей на аккаунт | любое количество (`POST /api/token/`), но **каждый ключ на аккаунте расходует одну и ту же квоту аккаунта** — именно поэтому этот инструмент создаёт **один аккаунт на ключ** |
| Группа | токен должен находиться в группе `deepseek`; группа `default` панели отвечает `503 No available channel for model deepseek-v4.1-flash under group default (distributor)` |

> 💡 Бесплатный тариф устанавливается DDD AI и может измениться. Проверяйте актуальные цифры на [dddai.dev](https://dddai.dev) — и пусть решение принимает собственный шлюз инструмента:

```bash
node src/cli.js --verify-only     # every key in keys.txt is re-tested with a real chat call
```

## 🤖 Модели — все 2 из них

`GET https://api.dddai.dev/v1/models` вернул **2 идентификатора**, оба проверены вживую вызовом чата (код выхода зафиксирован, реальный контент):

| ID модели | Чат | Инструменты |
| --- | --- | --- |
| `deepseek-v4.1-flash` | **200** — шлюзовая модель для каждого ключа, который создаёт этот инструмент | **200**, `finish_reason: tool_calls` с аргументами |
| `deepseek-v4-pro` | 200 — тот же эндпоинт, тот же ключ | наследует ту же поддержку инструментов |

> Шлюз перебирает короткий список кандидатов, а не один идентификатор: канал может быть недоступен для одной модели, пока сам аккаунт в порядке.

## 🧪 Как использовать API

Базовый URL **`https://api.dddai.dev/v1`** — совместим с OpenAI.

```python
from openai import OpenAI
c = OpenAI(base_url="https://api.dddai.dev/v1", api_key="<your key>")
print(c.chat.completions.create(model="deepseek-v4.1-flash",
      messages=[{"role": "user", "content": "hi"}]).choices[0].message.content)
```

Вызов инструментов работает так же — передайте обычный массив `tools` и читайте `choices[0].message.tool_calls`.

## 🛠 Быстрый старт

```bash
cd dddai-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # optional — without it the tool runs direct
node src/launcher.js        # interactive: mint / verify / screen / settings

# or directly:
node src/cli.js --count 3 --pool proxies.txt
node src/cli.js --verify-only
```

Почтовый канал — это **Emailnator**, который выдаёт обычный `@gmail.com` на каждый аккаунт — панель применяет политику домена/синтаксиса на стороне сервера (измерено: `EMAIL_ADDRESS_REJECTED` для `user@example.com`, gmail с точками и outlook/hotmail/mailtrap), так что обычный gmail — единственное семейство адресов, которое она принимает. В панели **нет капчи**, поэтому токен или решатель никогда не нужны, а вызов почтового канала идёт через тот же пул выходов, что и вызовы панели.

### 🖱️ Лаунчер

Лаунчер запрашивает список прокси (необязательно), количество и шлюзовую модель, запоминает их в `launcher.config.json` и запускает CLI — создание, проверку, отбор выходов, настройки, справку, подключение к 9Router.

## ✅ Проверено в работе — каждый ключ протестирован

| Проверка | Результат |
| --- | --- |
| `GET /api/status` | `turnstile_check: false` · `email_verification: true` · `password_register: true` |
| `GET /api/verification?email=…` | **200** `{"success":true}` — панель отправляет 6-символьный код |
| `POST /api/user/register` | **200** с той же банкой cookie, что запрашивала код |
| `POST /api/user/login` | **200** → `access_token`; каждый последующий вызов также несёт `New-Api-User: <id>` |
| `POST /api/token/` затем `POST /api/token/<id>/key` | список маскирует ключ; эндпоинт раскрытия возвращает открытый текст |
| `POST /v1/chat/completions` | **200** с реальным контентом — ключ попадает в `keys.txt` только после этого |
| проба `tools` (схема `get_time`) | **200** `finish_reason: tool_calls`, один вызов с `{"timezone":"Europe/Istanbul"}` |

> **Контроль качества.** Каждый ключ здесь прошёл живой `chat-200` прежде чем был записан в `keys.txt`
> (мёртвые ключи никогда не поставляются). Комбинации роутера следуют тому же правилу на уровне модели: запись
> добавляется **только если модель отвечает `chat-200` и умеет вызывать инструменты** — модель без поддержки инструментов,
> мёртвый ключ или мёртвый провайдер отбрасывается, а не добавляется. Повторяйте шаг проверки в любое время;
> сбои сообщаются, а не скрываются.

## 🧭 Как это работает

```
inbox (one address per account)                                  a fresh address per account
   │
   ├─▶ sign-up: DDD AI page → e-mail address → code / link from the inbox
   ├─▶ session: the account's cookies or access token in hand
   ├─▶ key mint: the account's key endpoint → plaintext key returned once (written to disk immediately)
   └─▶ live proof: POST https://api.dddai.dev/v1/chat/completions → real content = the key is live
```

| Шаг | Вызов | Примечания |
| --- | --- | --- |
| почтовый ящик | Emailnator `POST /api/generate-email` | свежий обычный `@gmail.com` на каждый аккаунт — политика панели отклоняет псевдонимы с `+`/точками и домен любого другого канала |
| код | `GET /api/verification?email=…` | письмо с кодом приходит с собственного релейного домена панели |
| регистрация | `POST /api/user/register` | `{username,password,email,verification_code}` — та же банка cookie, что и у запроса кода |
| сессия | `POST /api/user/login` | `access_token` + id аккаунта; `/api/*` также требует `New-Api-User: <id>` |
| токен | `POST /api/token/` `{group:'deepseek'}` → `POST /api/token/<id>/key` | группа важна: у `default` нет канала для этой модели |
| шлюз | `POST /v1/chat/completions` | 200 с реальным контентом = оставить; закрытая группа повторяется на другой группе из `/api/pricing` |

## ⚙️ Опции CLI

```bash
node src/cli.js --count N        # mint N keys (each stops at the live chat gate)
node src/cli.js --verify-only    # re-prove every key in keys.txt
node src/cli.js --screen         # exit pool self-test against api.dddai.dev
node src/cli.js --connect-9router
node src/cli.js --help
```

| Переменная | По умолчанию | Значение |
| --- | --- | --- |
| `DDDAI_PROXIES` | `proxies.txt` | пул выходов, по одной строке на прокси — необязательно: без пула CLI подключается напрямую |
| `DDDAI_MODEL` | `deepseek-v4.1-flash` | модель для живой проверки |

| Флаг | Значение |
| --- | --- |
| `--pool <file>` | список прокси (имеет приоритет над переменной окружения) |
| `--proxy <url>` | закрепить один выход для всех вызовов панели |
| `--group <name>` | группа токена (по умолчанию `deepseek`) |
| `--exits K` | число кандидатов выходов, перебираемых на аккаунт (по умолчанию 6) |

## 🛰️ За прокси

**Необязательно — рекомендуется (лимит регистраций привязан к IP выхода).** Инструмент работает и без прокси.

- **С пулом:** передайте `--pool <file>` (или задайте `DDDAI_PROXIES`), и каждый запрос пойдёт через выход из этого списка; мёртвая строка проваливает эту попытку, и пробуется следующий выход.
- **Без пула:** CLI печатает `DIRECT (no proxy)` и подключается напрямую. Регистрация, создание ключей и шлюз чата не меняются — меняется только то, откуда уходят пакеты.
- Сначала запустите `--screen`: он тестирует каждый выход из вашего списка против `api.dddai.dev/api/status` и печатает OK/DEAD для каждой строки.

## 📄 Файлы

| Путь | Содержимое |
| --- | --- |
| `keys.txt` | один проверенный вживую ключ на строку — игнорируется git |
| `accounts.json` | email / имя пользователя / пароль / группа / статус шлюза — игнорируется git |
| `proxies.txt` | ваши выходы — игнорируется git |

## 🔌 Подключение к 9Router

[9Router](https://9router.com) — это самостоятельно размещаемый, OpenAI-совместимый роутер:

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

По умолчанию: `--prefix dddai`, `--base-url https://api.dddai.dev/v1`, `--model deepseek-v4.1-flash`. Модели подаются как `dddai/<model id>`.

## 🔌 Подключение к OmniRoute

[OmniRoute](https://omniroute.online) — это самостоятельно размещаемый, OpenAI-совместимый роутер:

```bash
node src/connect-omniroute.js --base http://localhost:20128 --password 123456
```

По умолчанию: `--prefix dddai`, `--base-url https://api.dddai.dev/v1`, `--model deepseek-v4.1-flash`. Модели подаются как `dddai/<model id>`.

## 🔑 Что даёт один ключ

| | |
| --- | --- |
| 🏢 Провайдер | **DDD AI** — [dddai.dev](https://dddai.dev) |
| 🎁 Что даёт провайдер | бесплатный аккаунт с собственной квотой (без капчи) |
| 🔑 Что даёт один ключ | один ключ на собственном аккаунте: оба идентификатора `https://api.dddai.dev/v1`, проверенные для чата и инструментов |
| 🌐 Форма API | совместим с OpenAI: `POST https://api.dddai.dev/v1/chat/completions` |
| 🔒 Ваши данные | связь только с `dddai.dev` / `api.dddai.dev`, Emailnator и настроенным вами почтовым бэкендом |

## 🐞 Проблемы

Если DDD AI изменит свои эндпоинты регистрации, добавит Turnstile, ужесточит политику почтовых ящиков или переместит рабочую группу, инструмент остановится на этом шаге и сообщит об этом. **[Откройте issue](../../issues)** с командой и точным выводом.

## 📄 Лицензия

MIT © 2026 — см. [LICENSE](LICENSE).

<div align="center">

**[MIT](LICENSE)** © 2026 dddai-bulk-apikey contributors

</div>