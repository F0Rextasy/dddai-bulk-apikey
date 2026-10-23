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

**Crie contas DDD AI em massa com chaves de API — inferência compatível com OpenAI + conector 9Router.**

[![Node](https://img.shields.io/badge/Node-%E2%89%A518-3DA639?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-blue?style=for-the-badge)](#)
[![License](https://img.shields.io/badge/license-MIT-6ea8fe?style=for-the-badge)](LICENSE)
[![Models](https://img.shields.io/badge/models-2%20served-412991?style=for-the-badge)](https://api.dddai.dev/v1/models)
[![9Router](https://img.shields.io/badge/9Router-connector-8B5CF6?style=for-the-badge)](#-connect-to-9router)
[![OmniRoute](https://img.shields.io/badge/OmniRoute-connector-8B5CF6?style=for-the-badge)](#-connect-to-omniroute)
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 Início rápido](#-quick-start) · [🎁 Nível gratuito](#-free-tier--how-much-you-get) · [🤖 Modelos](#-models--all-2-of-them) · [🧪 Use a API](#-how-to-use-the-api) · [✅ Verificado](#-verified-working--every-key-is-tested) · [🧭 Como funciona](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [🔌 OmniRoute](#-connect-to-omniroute) · [⚙️ Opções](#️-cli-options) · [🛰️ Proxies](#️-behind-a-proxy) · [🐞 Problemas](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ Por que isto existe

O DDD AI roda um **fork do new-api com `turnstile_check: false`** — todo o portão de cadastro é um endereço de e-mail, uma senha e o código de 6 caracteres que o painel envia por e-mail. É isso que o torna automatizável em lote: sem solucionador de captcha, sem navegador. Cada conta começa com sua própria cota, a API é compatível com OpenAI e **o modelo chama ferramentas** (medido: `tool_calls` com argumentos reais), então as chaves são utilizáveis por clientes no estilo agente e não apenas para chat simples.

## 🎁 Nível gratuito — quanto você recebe

| O quê | Valor |
| --- | --- |
| Plano | conta gratuita, uma por chave gerada |
| Cota no cadastro | ~**7.300** unidades do painel na conta (medido nas duas contas pré-existentes) |
| Quanto custa uma chamada | o portão ativo roda em `deepseek-v4.1-flash` (`ratio 37.5`, `completion 1`, `quota_type 0`), então um chat curto consome algumas unidades |
| Chaves por conta | qualquer número (`POST /api/token/`), mas **toda chave de uma conta consome a mesma cota da conta** — que é exatamente por isso que esta ferramenta gera **uma conta por chave** |
| Grupo | o token deve estar no grupo `deepseek`; o grupo `default` do painel responde `503 No available channel for model deepseek-v4.1-flash under group default (distributor)` |

> 💡 O nível gratuito é definido pelo DDD AI e pode mudar. Confira os números atuais em [dddai.dev](https://dddai.dev) — e deixe o próprio portão da ferramenta decidir:

```bash
node src/cli.js --verify-only     # every key in keys.txt is re-tested with a real chat call
```

## 🤖 Modelos — todos os 2 deles

`GET https://api.dddai.dev/v1/models` retornou **2 ids**, ambos verificados ao vivo com uma chamada de chat (saída fixada, conteúdo real):

| ID do modelo | Chat | Ferramentas |
| --- | --- | --- |
| `deepseek-v4.1-flash` | **200** — o modelo do portão para toda chave que esta ferramenta grava | **200**, `finish_reason: tool_calls` com argumentos |
| `deepseek-v4-pro` | 200 — mesmo endpoint, mesma chave | herda o mesmo suporte a ferramentas |

> O portão percorre uma pequena lista de candidatos, não um único id: um canal pode estar fora do ar para um único modelo enquanto a conta está bem.

## 🧪 Como usar a API

URL base **`https://api.dddai.dev/v1`** — compatível com OpenAI.

```python
from openai import OpenAI
c = OpenAI(base_url="https://api.dddai.dev/v1", api_key="<your key>")
print(c.chat.completions.create(model="deepseek-v4.1-flash",
      messages=[{"role": "user", "content": "hi"}]).choices[0].message.content)
```

A chamada de ferramentas funciona da mesma forma — passe um array `tools` normal e leia `choices[0].message.tool_calls`.

## 🛠 Início rápido

```bash
cd dddai-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # optional — without it the tool runs direct
node src/launcher.js        # interactive: mint / verify / screen / settings

# or directly:
node src/cli.js --count 3 --pool proxies.txt
node src/cli.js --verify-only
```

O trilho de caixa de correio é o **Emailnator**, que fornece um `@gmail.com` simples por conta — o painel impõe uma política de domínio/sintaxe no lado do servidor (medido: `EMAIL_ADDRESS_REJECTED` para `user@example.com`, gmails com pontos e outlook/hotmail/mailtrap), então um gmail simples é a única família de endereços que ele aceita. O painel **não tem captcha**, então nenhum token ou solucionador é necessário, e a chamada do trilho usa o mesmo pool de saída que as chamadas do painel.

### 🖱️ O launcher

O launcher pede a lista de proxies (opcional), a contagem e o modelo do portão, memoriza-os em `launcher.config.json` e executa a CLI — gerar, verificar, testar saídas, configurações, ajuda, Connect to 9Router.

## ✅ Verificado e funcionando — toda chave é testada

| Verificação | Resultado |
| --- | --- |
| `GET /api/status` | `turnstile_check: false` · `email_verification: true` · `password_register: true` |
| `GET /api/verification?email=…` | **200** `{"success":true}` — o painel envia um código de 6 caracteres por e-mail |
| `POST /api/user/register` | **200** com o mesmo cookie jar que pediu o código |
| `POST /api/user/login` | **200** → `access_token`; toda chamada posterior também carrega `New-Api-User: <id>` |
| `POST /api/token/` e depois `POST /api/token/<id>/key` | a listagem mascara a chave; o endpoint de revelação retorna o texto simples |
| `POST /v1/chat/completions` | **200** com conteúdo real — uma chave só chega a `keys.txt` depois disso |
| Sonda `tools` (esquema `get_time`) | **200** `finish_reason: tool_calls`, uma chamada com `{"timezone":"Europe/Istanbul"}` |

> **Portões de qualidade.** Toda chave aqui passou por um `chat-200` ao vivo antes de ser gravada em `keys.txt`
> (chaves mortas nunca são entregues). As combinações do roteador seguem a mesma regra no nível do modelo: uma entrada só é
> adicionada **se o modelo responder `chat-200` e conseguir chamar ferramentas** — um modelo sem suporte a ferramentas,
> uma chave morta ou um provedor morto é descartado em vez de adicionado. Reexecute a etapa de verificação a qualquer momento;
> as falhas são relatadas, nunca ocultadas.

## 🧭 Como funciona

```
inbox (one address per account)                                  a fresh address per account
   │
   ├─▶ sign-up: DDD AI page → e-mail address → code / link from the inbox
   ├─▶ session: the account's cookies or access token in hand
   ├─▶ key mint: the account's key endpoint → plaintext key returned once (written to disk immediately)
   └─▶ live proof: POST https://api.dddai.dev/v1/chat/completions → real content = the key is live
```

| Etapa | Chamada | Observações |
| --- | --- | --- |
| caixa de correio | Emailnator `POST /api/generate-email` | um `@gmail.com` simples e novo por conta — a política do painel rejeita aliases com `+`/pontos e o domínio de todos os outros trilhos |
| código | `GET /api/verification?email=…` | o e-mail com o código chega do próprio domínio de retransmissão do painel |
| registro | `POST /api/user/register` | `{username,password,email,verification_code}` — mesmo cookie jar da requisição do código |
| sessão | `POST /api/user/login` | `access_token` + o id da conta; `/api/*` também exige `New-Api-User: <id>` |
| token | `POST /api/token/` `{group:'deepseek'}` → `POST /api/token/<id>/key` | o grupo importa: `default` não tem canal para este modelo |
| portão | `POST /v1/chat/completions` | 200 com conteúdo real = manter; um grupo fechado é tentado novamente em outro grupo de `/api/pricing` |

## ⚙️ Opções da CLI

```bash
node src/cli.js --count N        # mint N keys (each stops at the live chat gate)
node src/cli.js --verify-only    # re-prove every key in keys.txt
node src/cli.js --screen         # exit pool self-test against api.dddai.dev
node src/cli.js --connect-9router
node src/cli.js --help
```

| Env | Padrão | Significado |
| --- | --- | --- |
| `DDDAI_PROXIES` | `proxies.txt` | pool de saída, uma linha por proxy — opcional: sem um pool a CLI se conecta diretamente |
| `DDDAI_MODEL` | `deepseek-v4.1-flash` | modelo para a verificação ao vivo |

| Flag | Significado |
| --- | --- |
| `--pool <file>` | lista de proxies (tem prioridade sobre a env) |
| `--proxy <url>` | fixa uma saída para toda chamada do painel |
| `--group <name>` | grupo do token (padrão `deepseek`) |
| `--exits K` | candidatos de saída tentados por conta (padrão 6) |

## 🛰️ Atrás de um proxy

**Opcional — recomendado (o limite de cadastro é por IP de saída).** A ferramenta também funciona sem proxy.

- **Com um pool:** passe `--pool <file>` (ou defina `DDDAI_PROXIES`) e toda requisição passa por uma saída dessa lista; uma linha morta faz aquela tentativa falhar e a próxima saída é tentada.
- **Sem um pool:** a CLI imprime `DIRECT (no proxy)` e se conecta diretamente. O cadastro, a geração de chaves e o portão de chat permanecem inalterados — só muda de onde os pacotes saem.
- Execute `--screen` primeiro: ele testa cada saída da sua lista contra `api.dddai.dev/api/status` e imprime OK/DEAD por linha.

## 📄 Arquivos

| Caminho | Conteúdo |
| --- | --- |
| `keys.txt` | uma chave verificada ao vivo por linha — ignorado pelo git |
| `accounts.json` | email / username / password / group / status do portão — ignorado pelo git |
| `proxies.txt` | suas saídas — ignorado pelo git |

## 🔌 Connect to 9Router

[9Router](https://9router.com) é um roteador auto-hospedado e compatível com OpenAI:

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

Padrões: `--prefix dddai`, `--base-url https://api.dddai.dev/v1`, `--model deepseek-v4.1-flash`. Os modelos são servidos como `dddai/<model id>`.

## 🔌 Connect to OmniRoute

[OmniRoute](https://omniroute.online) é um roteador auto-hospedado e compatível com OpenAI:

```bash
node src/connect-omniroute.js --base http://localhost:20128 --password 123456
```

Padrões: `--prefix dddai`, `--base-url https://api.dddai.dev/v1`, `--model deepseek-v4.1-flash`. Os modelos são servidos como `dddai/<model id>`.

## 🔑 O que uma chave oferece

| | |
| --- | --- |
| 🏢 Provedor | **DDD AI** — [dddai.dev](https://dddai.dev) |
| 🎁 O que o provedor oferece | uma conta gratuita com sua própria cota (sem portão de captcha) |
| 🔑 O que uma chave oferece | uma chave em sua própria conta: ambos os ids de `https://api.dddai.dev/v1`, verificados para chat e ferramentas |
| 🌐 Formato da API | compatível com OpenAI: `POST https://api.dddai.dev/v1/chat/completions` |
| 🔒 Seus dados | apenas `dddai.dev` / `api.dddai.dev`, o Emailnator e o backend de caixa de entrada que você configurar são contatados |

## 🐞 Problemas

Se o DDD AI mudar seus endpoints de cadastro, adicionar Turnstile, apertar a política de caixa de correio ou mover o grupo funcional, a ferramenta para nessa etapa e avisa. **[Abra uma issue](../../issues)** com o comando e a saída exata.

## 📄 Licença

MIT © 2026 — veja [LICENSE](LICENSE).

<div align="center">

**[MIT](LICENSE)** © 2026 dddai-bulk-apikey contributors

</div>