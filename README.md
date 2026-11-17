# Cloud portal: lifetime accounts, phase 1

**Update:** a guarded hosted DeepSeek `/v1/messages` route and reservation ledger
have been added. They are disabled by default and have not been integration-tested
against a live database/provider. Read [METERING.md](METERING.md) before enabling.
Older "not implemented" wording below describes the original phase-1 state.
The portal now includes a manual server-backed token tracker refresh.

Administrator page: `/admin.html`. Follow [ADMIN.md](ADMIN.md) to create a separate
admin credential, configure Netlify, apply the audit schema, and manage customer keys.

This is a separate Netlify-ready website and account API, not the existing local gateway.
It provides customer-key login and a PostgreSQL-backed lifetime token allowance.
**Inference and enforced provider metering are intentionally not enabled yet.**
The first screen is customer API-key login. Only a successful `/api/account`
response reveals the workspace; logout and refresh lock it again. This UI gate
does not replace server authorization: static HTML is public, private data is
protected by the account API. Service preset previews display Haiku 1x,
Sonnet 2x, Opus 4x, and Fable 8x; these are service labels, not actual Claude
models or active billing deductions.
The UI says this explicitly; it does not simulate usage or claim a paid service is live.

## Publish without buying a domain

1. Create a hosted PostgreSQL database (for example through a provider you trust).
   Use its verified TLS connection URL. Database and Netlify usage may have costs.
2. Run `schema.sql` using the database's SQL console.
3. Put this `cloud-app` directory in its own private Git repository, connect it to
   Netlify using **Add new project > Import an existing project**, and deploy.
   If using the parent repository instead, set the Netlify base directory to `cloud-app`.
   The publish directory is `public`; functions are in `netlify/functions`.
4. Set `DATABASE_URL` in Netlify's server/function environment settings. Never place
   it in frontend JavaScript or a `VITE_*`/other public variable.
5. Redeploy. Netlify assigns an HTTPS `<your-site>.netlify.app` address.
   An example address is not owned or published until you create that project.
   Do not use drag-and-drop of `public` alone: it would omit the API functions.
6. Open the assigned HTTPS URL. A custom domain is optional.

No Netlify account, database, or deployed URL was created automatically.

## Issue a test customer, privately

On your administrator computer, with Node.js 22+:

```powershell
cd D:\local-ai-gateway\cloud-app
npm install
# Set DATABASE_URL privately in this terminal; do not paste it into chat.
$env:DATABASE_URL = "<your database connection URL>"
npm run issue-user -- "Test customer" 10000000
```

The command prints a new `sk-gw-…` customer key ONCE. Send it securely to that
customer and have them use the portal. The database stores only its SHA-256 hash.
These 256-bit random service keys are NOT DeepSeek/OpenRouter keys.
The name is a display name, not proof of an email identity.
Bearer keys are passwords: anyone with a copied key can impersonate that customer.
The current portal does not persist keys in browser storage.

```powershell
npm run grant-tokens -- "<customer UUID>" 10000000
npm run revoke-user -- "<customer UUID>"
npm test
```

Grant additional tokens only after independently verifying a purchase. No payments
or purchase webhook is connected. There is no public create-user/grant endpoint.
Grants accumulate and never reset monthly or expire. Zero consumption remains
zero until real usage events are implemented; this is not a billing test.

## Before enabling model calls or public sales

Implement and verify atomic quota reservation, provider usage settlement including
cache/input/output/reasoning, idempotent retry handling, recovery/reconciliation for
timeouts, per-user dollar caps, model/effort controls, rate limits, and monitoring.
Never trust client-reported usage. Reserve against the SAME customer package for
both main and helper calls. You cannot guarantee 10M tokens cost a fixed $0.20.
An input-token estimate alone is not a verified hard token cap.

Then add the Claude-compatible `/v1/messages` API on the hosted server and configure
Desktop with the hosted HTTPS URL and customer key. The existing local gateway
is not connected to this account database. Installing local apps is a later stage:
Windows/macOS/Linux need separate signed packaging and official dependency setup.

Before public login, add platform-side abuse limits, key rotation/reissue and
customer support/recovery. Keep database/provider credentials server-only.
Automated tests here do not establish deployed DB or Netlify integration correctness.
