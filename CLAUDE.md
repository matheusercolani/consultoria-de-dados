# Projeto: Consultoria de Dados — Matheus Ercolani

Landing page de captação de leads para serviços de Marketing Science e rastreamento avançado com IA para e-commerces de cosméticos.

## Repositório GitHub

- **Usuário:** matheusercolani
- **Repo:** consultoria-de-dados
- **URL:** https://github.com/matheusercolani/consultoria-de-dados
- **Branch principal:** main

## Deploy — Cloudflare Pages

- **Conta Cloudflare:** conta pessoal do Matheus Ercolani
- **Nome do projeto Pages:** `consultoria-de-dados`
- **URL de produção:** https://consultoria-de-dados.pages.dev
- **Método de deploy:** GitHub conectado (push na branch `main` → redeploy automático)
- **Framework:** nenhum (HTML puro, sem build step)
- **Build command:** *(vazio)*
- **Build output directory:** `/`

> Este projeto está sempre vinculado à conta Cloudflare do Matheus Ercolani com o projeto Pages chamado `consultoria-de-dados`. Não criar projetos Pages com outros nomes para este repositório.

## Cloudflare CLI — Gerenciamento de Contas

O sistema `cf-*` está instalado em `~/.bashrc`. Comandos disponíveis:

| Comando | O que faz |
|---|---|
| `cf-on matheus-consultoria` | Ativa a conta Cloudflare deste projeto |
| `cf-wrangler <cmd>` | Roda wrangler autenticado com o profile ativo |
| `cf-off` | Desativa o profile |
| `cf-list` | Lista todos os profiles cadastrados |
| `cf-status` | Mostra o profile ativo |

**Sempre rodar `cf-on matheus-consultoria` antes de qualquer comando wrangler neste projeto.**

Profile registrado em `~/.config/cloudflare/` com Account ID `e4a039cfcd4e8160e4a12883e63b36c5`.

## Stack

- HTML + CSS + JS puro (sem framework)
- Fontes: Anton (Google Fonts) + Satoshi (Fontshare)
- Design: Brutalist-lite SaaS — paleta carvão (#171e19) + amarelo (#ffd000)

## Captação de Leads

Os formulários (hero e CTA final) capturam: **nome**, **e-mail** e **WhatsApp**.
O `handleSubmit` dispara: Meta Pixel (browser) + `/tracker` (server-side CAPI + GA4).

## Sistema de Tracking (krob-tracking-stack)

Stack completa de rastreamento server-side integrada ao Cloudflare Pages Functions + D1.
Documentação completa: `docs/TRACKING.md`.

### D1 Database
- **Nome:** `consultoria-tracking`
- **ID:** `4011dd66-4380-4da1-80bd-cba6992d74b6`
- **Migrations:** `migrations/` (0001–0015, aplicar com `cf-wrangler d1 migrations apply consultoria-tracking --remote`)

### Funções (Cloudflare Pages Functions)
| Arquivo | Rota | Função |
|---|---|---|
| `functions/_middleware.js` | `/*` | Cookie de sessão 400 dias + captura UTM/fbclid |
| `functions/tracker.js` | `/tracker` | SHA-256 PII → Meta CAPI v25 + GA4 MP |
| `functions/scripts/[[path]].js` | `/scripts/` | Proxy first-party do script GA4 |
| `functions/api/leads.js` | `/api/leads` | API de leads para o dashboard |
| `functions/api/events.js` | `/api/events` | API de tracking health para o dashboard |
| `dash/index.html` | `/dash` | Dashboard analytics (autenticado por DASH_KEY) |

### Variáveis de Ambiente (Cloudflare Pages → Settings → Environment Variables)
| Variável | Valor |
|---|---|
| `META_PIXEL_ID` | `26796486073311252` |
| `META_ACCESS_TOKEN` | token do Conversions API |
| `GA4_MEASUREMENT_ID` | `G-MCHVNR5E5V` |
| `GA4_API_SECRET` | API secret do Measurement Protocol |
| `DASH_KEY` | senha do `/dash` |
| `DEFAULT_COUNTRY_CODE` | `55` |
| `META_TEST_EVENT_CODE` | só durante testes |
