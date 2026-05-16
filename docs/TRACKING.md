# Sistema de Tracking — Consultoria de Dados

Rastreamento server-side de leads via Cloudflare Pages Functions + D1, baseado no [krob-tracking-stack](https://github.com/gustavokrob/krob-tracking-stack).

## Arquitetura

```
Visitante → _middleware.js → cria cookie _krob_sid (400 dias) + captura UTM/fbclid/gclid → D1 sessions
Formulário → handleSubmit → /tracker → SHA-256 PII → Meta CAPI + GA4 MP → D1 event_log
Dashboard → /dash?key=DASH_KEY → /api/leads + /api/events
```

## Endpoints

| Rota | Método | Função |
|---|---|---|
| `/*` (HTML) | GET | Middleware: session cookie + UTM capture |
| `/tracker` | POST | Recebe Lead, hasheia PII, dispara Meta CAPI + GA4 |
| `/scripts/gtag.js` | GET | Proxy first-party do script GA4 |
| `/api/leads` | GET | Lista leads com UTMs (autenticado por DASH_KEY) |
| `/api/events` | GET | Logs de tracking health (autenticado por DASH_KEY) |
| `/dash` | GET | Dashboard de analytics |

## Banco D1 — `consultoria-tracking`

| Tabela | Função |
|---|---|
| `sessions` | Uma linha por visitante: UTMs, fbclid, gclid, IP, UA |
| `event_log` | Leads disparados: status Meta/GA4, bot detection, PII presence |
| `checkout_sessions` | (Reservado) Para funil de vendas futuro |
| `purchase_log` | (Reservado) Para compras futuras |
| `purchase_items` | (Reservado) Itens de compra |
| `ad_spend` | (Reservado) Gasto de mídia sincronizado da Meta API |
| `sync_log` | (Reservado) Log de sincronizações |

## Variáveis de Ambiente (Cloudflare Pages)

| Variável | Obrigatória | Descrição |
|---|---|---|
| `META_PIXEL_ID` | ✅ | `1290709392346372` |
| `META_ACCESS_TOKEN` | ✅ | Token do Conversions API (Events Manager) |
| `META_TEST_EVENT_CODE` | Testes | Código de teste do Events Manager — remover em produção |
| `GA4_MEASUREMENT_ID` | ✅ | `G-KK9GFR6KW4` |
| `GA4_API_SECRET` | ✅ | API Secret do Measurement Protocol |
| `DASH_KEY` | ✅ | Senha de acesso ao `/dash` |
| `DEFAULT_COUNTRY_CODE` | ✅ | `55` (Brasil) |

## Como Testar

1. Acessar a página com `?utm_source=teste&utm_medium=cpc` na URL
2. Submeter o formulário
3. Verificar no **Meta Events Manager → Testar Eventos** (com `META_TEST_EVENT_CODE` ativo)
4. Verificar no **GA4 DebugView** o evento `generate_lead`
5. Verificar em `/dash?key=DASH_KEY` → aba **Leads**

## Dados Hasheados Enviados ao Meta CAPI

- `em` → email SHA-256
- `fn` → primeiro nome SHA-256
- `ln` → sobrenome SHA-256
- `ph` → telefone com +55 SHA-256
- `fbp` → cookie _fbp (gerado pelo middleware, 400 dias)
- `fbc` → cookie _fbc (construído a partir do fbclid na URL)
- `client_ip_address` + `client_user_agent` → capturados server-side

## Deduplicação Meta

O mesmo `event_id` (UUID) é enviado tanto pelo Pixel (browser) quanto pelo CAPI (servidor). A Meta usa esse ID para não contar em duplicata — é obrigatório para dual-fire.
