# AGENTS.md

## Cursor Cloud specific instructions

### What this is
`nfd` is a single-file **Cloudflare Worker** (`worker.js`) implementing a Telegram
private-message forwarding / anti-fraud bot. It uses the legacy **Service Worker**
format (`addEventListener('fetch', ...)`), so its config bindings are injected as
**globals** (not via a module `env` argument):

- `ENV_BOT_TOKEN`, `ENV_BOT_SECRET`, `ENV_ADMIN_UID` — plain vars
- `nfd` — a Workers KV namespace binding

There is **no build step, no lint config, and no automated tests** in the repo.

### Running it (local dev)
The dev tool is `wrangler` (a devDependency). `npm run dev` (→ `wrangler dev`)
runs the worker locally via Miniflare with an emulated KV store. Config lives in
`wrangler.toml`; scripts in `package.json`.

- Secrets/vars for local dev go in a git-ignored `.dev.vars` file at the repo
  root (`ENV_BOT_TOKEN`, `ENV_BOT_SECRET`, `ENV_ADMIN_UID`). `wrangler.toml` ships
  placeholder `[vars]`; `.dev.vars` overrides them. Without `.dev.vars` the
  placeholders are used (fine for smoke-testing routing/auth).
- HTTP routes: `GET /` (fallback), `POST /endpoint` (the Telegram webhook;
  requires header `X-Telegram-Bot-Api-Secret-Token` == `ENV_BOT_SECRET`),
  `GET /registerWebhook`, `GET /unRegisterWebhook`.
- Inspect the local KV store with
  `npx wrangler kv key list|get|put --binding nfd --local`.

### Important gotchas (non-obvious)
- **Outbound `fetch()` inside `event.waitUntil()` does NOT complete under local
  `wrangler dev`.** The worker returns its HTTP response immediately and does the
  real work (forwarding, admin replies, fraud-notify, and the Telegram API calls
  themselves) inside `waitUntil`, which includes network `fetch`. Locally those
  fetches are abandoned after the response is sent, so message delivery and any
  KV writes gated on a fetch result are not observable. Plain (no-fetch)
  `waitUntil` KV writes do persist. A `fetch` in the request/`respondWith` path
  (e.g. `/registerWebhook`) works fine.
- Because of the above, **true end-to-end forwarding cannot be exercised with
  local `wrangler dev` alone.** Full testing needs a real Telegram bot token +
  admin chat id and typically deploying the worker (`wrangler deploy`), where
  `waitUntil` behaves normally and Telegram can reach the public webhook URL.
- At runtime the worker fetches `startMessage.md`, `notification.txt`, and
  `fraud.db` from the upstream repo
  `raw.githubusercontent.com/LloydAsp/nfd/main/data/*` — **not** the local copies
  of those files in this repo. Editing the local files does not change bot
  behavior.

### Files
- `worker.js` — the entire application.
- `startMessage.md`, `shuoming.txt`, `caidanshuoming.txt`, `fraud.db` — local
  copies of data/text (reference only; runtime uses upstream GitHub copies).
