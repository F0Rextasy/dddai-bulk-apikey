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

**Crea cuentas de DDD AI en masa con claves API — inferencia compatible con OpenAI + conector 9Router.**

[![Node](https://img.shields.io/badge/Node-%E2%89%A518-3DA639?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-blue?style=for-the-badge)](#)
[![License](https://img.shields.io/badge/license-MIT-6ea8fe?style=for-the-badge)](LICENSE)
[![Models](https://img.shields.io/badge/models-2%20served-412991?style=for-the-badge)](https://api.dddai.dev/v1/models)
[![9Router](https://img.shields.io/badge/9Router-connector-8B5CF6?style=for-the-badge)](#-connect-to-9router)
[![OmniRoute](https://img.shields.io/badge/OmniRoute-connector-8B5CF6?style=for-the-badge)](#-connect-to-omniroute)
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 Inicio rápido](#-quick-start) · [🎁 Nivel gratuito](#-free-tier--how-much-you-get) · [🤖 Modelos](#-models--all-2-of-them) · [🧪 Usar la API](#-how-to-use-the-api) · [✅ Verificado](#-verified-working--every-key-is-tested) · [🧭 Cómo funciona](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [🔌 OmniRoute](#-connect-to-omniroute) · [⚙️ Opciones](#️-cli-options) · [🛰️ Proxies](#️-behind-a-proxy) · [🐞 Problemas](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ Por qué existe esto

DDD AI ejecuta un **fork de new-api con `turnstile_check: false`** — toda la barrera de registro es una dirección de correo, una contraseña y el código de 6 caracteres que el panel te envía por correo. Eso es lo que lo hace procesable en lote: sin solucionador de captcha, sin navegador. Cada cuenta comienza con su propia cuota, la API es compatible con OpenAI y **el modelo llama a herramientas** (medido: `tool_calls` con argumentos reales), por lo que las claves son utilizables por clientes de estilo agente y no solo para chat simple.

## 🎁 Nivel gratuito — cuánto obtienes

| Qué | Valor |
| --- | --- |
| Plan | cuenta gratuita, una por cada clave generada |
| Cuota al registrarse | ~**7,300** unidades del panel en la cuenta (medido en las dos cuentas preexistentes) |
| Cuánto cuesta una llamada | la barrera en vivo se ejecuta en `deepseek-v4.1-flash` (`ratio 37.5`, `completion 1`, `quota_type 0`), así que un chat corto consume unas pocas unidades |
| Claves por cuenta | cualquier número (`POST /api/token/`), pero **cada clave de una cuenta consume la misma cuota de la cuenta** — que es exactamente por lo que esta herramienta genera **una cuenta por clave** |
| Grupo | el token debe estar en el grupo `deepseek`; el grupo `default` del panel responde `503 No available channel for model deepseek-v4.1-flash under group default (distributor)` |

> 💡 El nivel gratuito lo define DDD AI y puede cambiar. Consulta los números en vivo en [dddai.dev](https://dddai.dev) — y deja que la propia barrera de la herramienta decida:

```bash
node src/cli.js --verify-only     # every key in keys.txt is re-tested with a real chat call
```

## 🤖 Modelos — los 2 que hay

`GET https://api.dddai.dev/v1/models` devolvió **2 ids**, ambos verificados en vivo con una llamada de chat (salida fijada, contenido real):

| ID del modelo | Chat | Herramientas |
| --- | --- | --- |
| `deepseek-v4.1-flash` | **200** — el modelo de la barrera para cada clave que escribe esta herramienta | **200**, `finish_reason: tool_calls` con argumentos |
| `deepseek-v4-pro` | 200 — mismo endpoint, misma clave | hereda el mismo soporte de herramientas |

> La barrera recorre una lista corta de candidatos, no un solo id: un canal puede estar caído para un único modelo mientras la cuenta está bien.

## 🧪 Cómo usar la API

URL base **`https://api.dddai.dev/v1`** — compatible con OpenAI.

```python
from openai import OpenAI
c = OpenAI(base_url="https://api.dddai.dev/v1", api_key="<your key>")
print(c.chat.completions.create(model="deepseek-v4.1-flash",
      messages=[{"role": "user", "content": "hi"}]).choices[0].message.content)
```

El llamado a herramientas funciona igual — pasa un array `tools` normal y lee `choices[0].message.tool_calls`.

## 🛠 Inicio rápido

```bash
cd dddai-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # optional — without it the tool runs direct
node src/launcher.js        # interactive: mint / verify / screen / settings

# or directly:
node src/cli.js --count 3 --pool proxies.txt
node src/cli.js --verify-only
```

El riel de correo es **Emailnator**, que entrega un `@gmail.com` simple por cuenta — el panel aplica una política de dominio/sintaxis del lado del servidor (medido: `EMAIL_ADDRESS_REJECTED` para `user@example.com`, gmails con puntos y outlook/hotmail/mailtrap) así que un gmail simple es la única familia de direcciones que acepta. El panel **no tiene captcha**, así que nunca se necesita un token ni un solucionador, y la llamada del riel viaja por el mismo pool de salida que las llamadas al panel.

### 🖱️ El lanzador

El lanzador pide la lista de proxies (opcional), la cantidad y el modelo de la barrera, los recuerda en `launcher.config.json` y ejecuta el CLI — generar, verificar, filtrar salidas, ajustes, ayuda, Conectar a 9Router.

## ✅ Verificado y funcionando — cada clave se prueba

| Comprobación | Resultado |
| --- | --- |
| `GET /api/status` | `turnstile_check: false` · `email_verification: true` · `password_register: true` |
| `GET /api/verification?email=…` | **200** `{"success":true}` — el panel envía por correo un código de 6 caracteres |
| `POST /api/user/register` | **200** con el mismo cookie jar que pidió el código |
| `POST /api/user/login` | **200** → `access_token`; cada llamada posterior también lleva `New-Api-User: <id>` |
| `POST /api/token/` y luego `POST /api/token/<id>/key` | la lista enmascara la clave; el endpoint de revelado devuelve el texto plano |
| `POST /v1/chat/completions` | **200** con contenido real — una clave solo llega a `keys.txt` después de esto |
| sonda `tools` (esquema `get_time`) | **200** `finish_reason: tool_calls`, una llamada con `{"timezone":"Europe/Istanbul"}` |

> **Barreras de calidad.** Cada clave aquí pasó un `chat-200` en vivo antes de escribirse en `keys.txt`
> (las claves muertas nunca se entregan). Las combinaciones del router siguen la misma regla a nivel de modelo: una entrada se
> añade **solo si el modelo responde `chat-200` y puede llamar a herramientas** — un modelo sin soporte de herramientas,
> una clave muerta o un proveedor muerto se descarta en lugar de añadirse. Vuelve a ejecutar el paso de verificación cuando quieras;
> los fallos se reportan, nunca se ocultan.

## 🧭 Cómo funciona

```
inbox (one address per account)                                  a fresh address per account
   │
   ├─▶ sign-up: DDD AI page → e-mail address → code / link from the inbox
   ├─▶ session: the account's cookies or access token in hand
   ├─▶ key mint: the account's key endpoint → plaintext key returned once (written to disk immediately)
   └─▶ live proof: POST https://api.dddai.dev/v1/chat/completions → real content = the key is live
```

| Paso | Llamada | Notas |
| --- | --- | --- |
| buzón | Emailnator `POST /api/generate-email` | un `@gmail.com` simple nuevo por cuenta — la política del panel rechaza alias con `+`/puntos y el dominio de cualquier otro riel |
| código | `GET /api/verification?email=…` | el correo con el código llega desde el propio dominio de retransmisión del panel |
| registro | `POST /api/user/register` | `{username,password,email,verification_code}` — mismo cookie jar que la petición del código |
| sesión | `POST /api/user/login` | `access_token` + el id de la cuenta; `/api/*` también quiere `New-Api-User: <id>` |
| token | `POST /api/token/` `{group:'deepseek'}` → `POST /api/token/<id>/key` | el grupo importa: `default` no tiene canal para este modelo |
| barrera | `POST /v1/chat/completions` | 200 con contenido real = conservar; un grupo cerrado se reintenta en otro grupo de `/api/pricing` |

## ⚙️ Opciones del CLI

```bash
node src/cli.js --count N        # mint N keys (each stops at the live chat gate)
node src/cli.js --verify-only    # re-prove every key in keys.txt
node src/cli.js --screen         # exit pool self-test against api.dddai.dev
node src/cli.js --connect-9router
node src/cli.js --help
```

| Env | Predeterminado | Significado |
| --- | --- | --- |
| `DDDAI_PROXIES` | `proxies.txt` | pool de salida, una línea por proxy — opcional: sin un pool el CLI se conecta directamente |
| `DDDAI_MODEL` | `deepseek-v4.1-flash` | modelo para la verificación en vivo |

| Flag | Significado |
| --- | --- |
| `--pool <file>` | lista de proxies (tiene prioridad sobre la variable de entorno) |
| `--proxy <url>` | fija una salida para cada llamada al panel |
| `--group <name>` | grupo del token (predeterminado `deepseek`) |
| `--exits K` | candidatos de salida probados por cuenta (predeterminado 6) |

## 🛰️ Detrás de un proxy

**Opcional — recomendado (el límite de registro es por IP de salida).** La herramienta también funciona sin proxy.

- **Con un pool:** pasa `--pool <file>` (o define `DDDAI_PROXIES`) y cada petición pasa por una salida de esa lista; una línea muerta hace fallar ese intento y se prueba la siguiente salida.
- **Sin un pool:** el CLI imprime `DIRECT (no proxy)` y se conecta directamente. El registro, la generación de claves y la barrera de chat no cambian — solo cambia desde dónde salen los paquetes.
- Ejecuta `--screen` primero: prueba cada salida de tu lista contra `api.dddai.dev/api/status` e imprime OK/DEAD por línea.

## 📄 Archivos

| Ruta | Contenido |
| --- | --- |
| `keys.txt` | una clave verificada en vivo por línea — ignorada por git |
| `accounts.json` | correo / usuario / contraseña / grupo / estado de la barrera — ignorado por git |
| `proxies.txt` | tus salidas — ignorado por git |

## 🔌 Conectar a 9Router

[9Router](https://9router.com) es un router autoalojado y compatible con OpenAI:

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

Predeterminados: `--prefix dddai`, `--base-url https://api.dddai.dev/v1`, `--model deepseek-v4.1-flash`. Los modelos se sirven como `dddai/<model id>`.

## 🔌 Conectar a OmniRoute

[OmniRoute](https://omniroute.online) es un router autoalojado y compatible con OpenAI:

```bash
node src/connect-omniroute.js --base http://localhost:20128 --password 123456
```

Predeterminados: `--prefix dddai`, `--base-url https://api.dddai.dev/v1`, `--model deepseek-v4.1-flash`. Los modelos se sirven como `dddai/<model id>`.

## 🔑 Qué te da una clave

| | |
| --- | --- |
| 🏢 Proveedor | **DDD AI** — [dddai.dev](https://dddai.dev) |
| 🎁 Qué da el proveedor | una cuenta gratuita con su propia cuota (sin barrera de captcha) |
| 🔑 Qué te da una clave | una clave en su propia cuenta: ambos ids de `https://api.dddai.dev/v1`, verificados para chat y herramientas |
| 🌐 Forma de la API | compatible con OpenAI: `POST https://api.dddai.dev/v1/chat/completions` |
| 🔒 Tus datos | solo se contacta con `dddai.dev` / `api.dddai.dev`, Emailnator y el backend de buzón que configures |

## 🐞 Problemas

Si DDD AI cambia sus endpoints de registro, añade Turnstile, endurece la política de buzón o mueve el grupo funcional, la herramienta se detiene en ese paso y lo indica. **[Abre un issue](../../issues)** con el comando y la salida exacta.

## 📄 Licencia

MIT © 2026 — consulta [LICENSE](LICENSE).

<div align="center">

**[MIT](LICENSE)** © 2026 dddai-bulk-apikey contributors

</div>