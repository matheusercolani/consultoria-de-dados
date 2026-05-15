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

> Este projeto está sempre vinculado à conta Cloudflare do Matheus Ercolani com o projeto Pages chamado `matheusdados`. Não criar projetos Pages com outros nomes para este repositório.

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
Para integrar com CRM/planilha, adicionar webhook no `handleSubmit` em index.html.
