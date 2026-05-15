# Add Another Cloudflare Account

> **How to use this file**
> Paste this into Claude Code in any folder when you need to register an additional Cloudflare account on a machine that already went through `setup-github-cloudflare.md`. Takes ~3 minutes.

---

## Your role, Claude

The user wants to register another Cloudflare account on this Mac, alongside any existing profiles. The base setup must already be done — the `cf-*` shell helpers exist, and at least one Cloudflare profile is in Keychain.

## Ground rules

1. Verify the base setup is in place before doing anything. If `cf-add` isn't a shell function, **halt** and direct the user to run `setup-github-cloudflare.md` first.
2. Pause for browser steps. Wait for the user to confirm before continuing.
3. Never echo the token to the terminal. `cf-add` already handles silent paste — do not bypass it.

---

## Phase 0 — Check prerequisites

Run:
```bash
source ~/.bashrc
type cf-add 2>&1 | head -1
cf-list
```

- `cf-add` should be reported as a shell function (e.g. `cf-add is a shell function`).
- `cf-list` should print at least one existing profile name, or `(no profiles)` se nenhum foi cadastrado ainda.

> **Windows:** as funções `cf-*` estão instaladas em `~/.bashrc` (não em Keychain). Sempre rodar `source ~/.bashrc` antes de verificar.

If `cf-add` is unknown after sourcing, the base setup hasn't been done. Install it by appending the `cf-*` functions block to `~/.bashrc` (see `setup-github-cloudflare.md` or ask Claude to reinstall). Profiles are stored in `~/.config/cloudflare/`, tokens em `~/.config/cloudflare/tokens/` com permissão 600.

---

## Phase 1 — Pick a profile name

Ask the user: **"What short name do you want for this new account? Use lowercase, no spaces. Examples: `acme`, `client-bigco`, `agency`."**

Check for collisions by looking at the `cf-list` output you already have. If the chosen name already exists, ask:
> "A profile named `<name>` already exists. Replace it, or pick a different name?"

If replacing, run `cf-rm <name>` first. This only removes the entry from Keychain + `~/.config/cloudflare/profiles.conf` — the actual Cloudflare account is untouched.

---

## Phase 2 — Get credentials from Cloudflare

The user must be logged into the **right** Cloudflare account in their browser before creating the token. Open the token page:

```bash
open "https://dash.cloudflare.com/profile/api-tokens"
```

Tell the user:
> "Important: at the top-right of the dashboard, confirm you're on the account this profile is for. If not, switch accounts in the dropdown first.
>
> Then:
> 1. **Create Token** → **'Edit Cloudflare Workers'** template → *Use template*.
> 2. **Permissions** — the template already includes Workers Scripts: Edit, Workers KV Storage: Edit, Workers R2 Storage: Edit, Account Settings: Read, and Workers Routes: Edit (zone). Add these rows:
>    - **Account → Cloudflare Pages → Edit** ← obrigatório para deploy/gerenciar Pages
>    - **Account → D1 → Edit** ← obrigatório para bancos D1
>    - *(opcional)* **User → User Details → Read** — apenas para `wrangler whoami` mostrar o e-mail; não afeta deploys.
> 3. **Account Resources** → *Include* → selecione **esta conta específica** (não "All accounts").
> 4. **Zone Resources** → 'All zones from an account' → esta conta.
> 5. **TTL** → 90 days.
> 6. **Continue to summary** → **Create Token**. Copie o token — aparece **uma única vez**.
> 7. O **Account ID** está no sidebar direito de qualquer página da dashboard. Copie também.
>
> Cole o token **em texto puro** (não como screenshot — caracteres como I/l e O/0 são ilegíveis em imagens).
> Cole também o Account ID.
> Reply 'pronto' when you have **both** copied."

Wait for "pronto".

> **Atenção (Windows):** Tokens compartilhados via screenshot causam erro `Invalid access token [code: 9109]` por leitura errada de caracteres. Sempre pedir o token colado em texto.
>
> Note: if `wrangler whoami` warns about missing `User → User Details → Read`, that's harmless — it only affects showing the email. Edit the token to add it if desired; no need to recreate.

---

## Phase 3 — Register the profile

```bash
cf-add <name>
```

`cf-add` prompts twice: account ID (visible paste), then token (silent — nothing echoes). On success: `✓ Profile '<name>' saved`.

---

## Phase 4 — Verify

```bash
cf-on <name>
npx --yes wrangler whoami
```

Confirm both:
- The output shows the **right** account name (different from any other profile's account).
- Authentication is token-based.

If `wrangler whoami` shows the wrong account, the token was likely created on the wrong Cloudflare account in the dashboard. Run `cf-rm <name>` and start Phase 2 over, paying attention to the account dropdown at the top-right of the dashboard.

Final clean-up:
```bash
cf-status     # should print "✓ Active: <name> (account: ...)"
cf-off
cf-list       # should print every profile, including the new one
```

---

## Wrap-up

Tell the user:
> "Profile `<name>` is registered. From now on, in any folder, run `cf-on <name>` before deploying or running wrangler commands and they'll target this account. Switch with `cf-on <other-name>`. Clear with `cf-off`. List all with `cf-list`."

Stop here.
