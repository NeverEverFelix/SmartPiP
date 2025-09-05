# Smart PiP — Tech Stack (with Backend & PostgreSQL)

Production-grade, Chromium‑first extension with a secure backend and PostgreSQL storage.

---

## 1) Client (Chrome Extension)

* **Manifest V3** (Chromium-first; Edge/Brave compatible)
* **TypeScript** (strict)
* **Content scripts**, **Service worker**, **Offscreen Documents API**
* Chrome APIs: **commands**, **scripting**, **tabs**, **storage**, **identity** (for Google sign‑in/YouTube), **activeTab**, **offscreen**
* **React** (popup/options) + **CSS Modules**
* PiP: `HTMLVideoElement.requestPictureInPicture()`
* Site connectors: **YouTube / Vimeo / Twitch** DOM adapters
* Shortcut handling: **Chrome Commands API** (F1–F9)

## 2) Backend API (Our Service)

* **Node 20 + NestJS (Fastify)**

  * Validation: **zod** (or class‑validator)
  * API style: **REST** with **OpenAPI/Swagger** docs
  * Authn: **Google OAuth (OpenID Connect)** via **passport** or **oidc-provider**
  * Authz: **JWT** (short‑lived access, refresh rotation)
* **Prisma ORM** (strict schema & migrations)
* **YouTube Search proxy** endpoint (keeps API key secret for F3 Suggest)
* **Playlist ops**: client-side via `chrome.identity` (server optional; if server-side, store refresh tokens encrypted — see Security)

## 3) Database & Storage

* **PostgreSQL 15+** (managed: **Neon** or **AWS RDS Postgres**)
* Pooling: **PgBouncer**
* Migrations: **Prisma Migrate**
* Primary tables: **users**, **video\_history** (max 5 recent), **settings**, **oauth\_accounts**, **audit\_logs**
* Indexing: `(user_id, watched_at DESC)` on history; **JSONB** for provider metadata

## 4) Caching & Jobs

* **Redis** (Upstash / ElastiCache)

  * Caching: suggest results & user settings
  * Queue: **BullMQ** for background jobs (pre‑fetch metadata, link unfurling)

## 5) Authentication & Identity

* **Google Sign‑In (OIDC)** for app login (email/profile)
* **YouTube OAuth** scopes:

  * Client: `youtube.readonly` (default), `youtube` (only when user uses F2) via `chrome.identity`
  * Server (optional): If centralizing playlist ops, store **encrypted** refresh tokens with **AES‑GCM** using **AWS KMS**/**GCP KMS**

## 6) YouTube Integration

* **YouTube Data API v3**

  * Backend‑proxied `search.list` (F3 Suggest)
  * Client‑side `playlists.insert` / `playlistItems.insert` (F2)
* Quotas: enforce per‑user & global rate limits (Redis sliding window)

## 7) Observability & Ops

* **OpenTelemetry** (traces/metrics/logs) → **Grafana Tempo/Prometheus/Loki** or **Datadog**
* **Sentry** (client + server error tracking)
* **Structured logging**: pino (server), console JSON (client worker)

## 8) Security & Privacy

* MV3 **CSP** hardened; **no remote code**
* **Least‑privilege host\_permissions** (YouTube/Vimeo/Twitch only; expand when Suggest enabled)
* **CORS**: strict allowlist; **Helmet** headers
* **Secrets**: **AWS Secrets Manager**/**Doppler**
* **Token encryption**: server‑side OAuth tokens sealed with **KMS** (only if server stores them)
* **Data retention**: last 5 videos per user (configurable), explicit opt‑in for analytics

## 9) Build, Test, Release

* **Vite** build for extension; **ESLint + Prettier + Husky**
* Server tests: **Vitest/Jest**; E2E: **Playwright** (YouTube/Vimeo/Twitch flows)
* **GitHub Actions** CI/CD: lint → test → build → sign/package →

  * **Chrome Web Store** upload (Dev/Beta/Prod)
  * **Backend deploy** to **AWS ECS Fargate** (or **Cloud Run**/**Fly.io** for speed)

## 10) Deployment & Networking

* Backend: **AWS ECS Fargate + ALB** (or **Cloud Run**)
* DB: **RDS Postgres** (or **Neon** for serverless dev)
* Cache: **ElastiCache Redis** (or **Upstash**)
* TLS & DNS: **Cloudflare** (WAF, rate limiting, CDN for static assets)

## 11) Browser Permissions (MV3 Manifest)

* `permissions`: `storage`, `scripting`, `tabs`, `commands`, `identity` (gated)
* `host_permissions`: `https://*.youtube.com/*`, `https://*.vimeo.com/*`, `https://*.twitch.tv/*` (plus optional `*://*/*` when Suggest is enabled)
* `optional_permissions`: expand only on explicit user opt‑in
* `offscreen`: for F1 flow (controlled)

## 12) Optional Phase‑2 (Nice‑to‑Have)

* **Cross‑device history sync** (server‑backed)
* **Firefox MV3** / **Safari Web Extension** builds
* **Lightweight re‑ranker** for Suggest (BM25‑style)
* **Feature flags** via **ConfigCat**/**Unleash**
