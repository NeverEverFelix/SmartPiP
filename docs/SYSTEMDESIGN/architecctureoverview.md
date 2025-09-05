# Smart PiP — Architecture Overview

## 0. Document Metadata

* **Doc Type:** High-Level Architecture (HLA)
* **Owner:** Smart PiP Core Team
* **Reviewers:** Tech Leads, Product, SRE
* **Status:** Draft v1.0
* **Related Docs:** Tech Stack, Goals, Reliability Requirements, Use Cases & Requirements

---

## 1. System Context

* **Actors:**

  * End User (browser)
  * Chrome Web Store (distribution)
  * Google Identity (OIDC)
  * YouTube Data API v3
  * Smart PiP Backend (NestJS)
  * PostgreSQL (RDS)
  * Redis (ElastiCache)
  * Observability stack (Prometheus, Grafana, Sentry)

* **Trust Boundaries:**

  * Browser extension sandbox ↔ Backend API
  * Backend ↔ External APIs (Google, YouTube)
  * Data/Secrets handled by AWS KMS & Secrets Manager

---

## 2. Components Overview

### Browser Extension (MV3)

* **Content Scripts:** site adapters (YouTube, Vimeo, Twitch) detect video players.
* **Service Worker:** routes commands (F7/F8/F9/F1-F3), handles cross-tab messaging.
* **UI:** popup/options + PiP overlay controls.
* **PiP Controller:** uses `HTMLVideoElement.requestPictureInPicture()`.

### Backend API (NestJS/Fastify)

* Endpoints:

  * `/auth/*` — Google OIDC
  * `/history` — last 5 watched videos
  * `/suggest` — proxy to YouTube search
  * `/healthz` & `/metrics`
* Auth: Google OIDC → short-lived JWT

### Data & Infra

* **PostgreSQL (with Prisma):**

  * Tables: `users`, `video_history`, `settings`, `oauth_accounts`, `audit_logs`
  * Max 5 video history rows per user
* **Redis:** cache suggest results, manage background jobs (BullMQ)

### External Services

* Google OIDC
* YouTube Data API v3 (search, playlist ops if enabled)

### Observability

* OpenTelemetry → Prometheus/Tempo/Loki or Datadog
* Sentry for client + backend errors

### Deployment

* AWS ECS Fargate + ALB
* RDS PostgreSQL
* ElastiCache Redis
* Cloudflare for TLS/WAF/CDN

---

## 3. Responsibilities & Interfaces

* **Site Adapters:** detect media elements, bind to commands.
* **Command Router:** handle media keys & route to PiP session.
* **Suggest Service:** server-side YouTube search proxy with rate limiting.
* **History Service:** store/retrieve last 5 videos with durability.

---

## 4. Key User Flows

* **UC-01:** PiP toggle & persistence (works as long as source tab open)
* **UC-02/03:** Media key + overlay controls (Back/Play/Pause/Skip)
* **UC-04:** Auto-PiP when tab loses focus (toggleable)
* **UC-05:** Smart Suggest → open recommended video in PiP
* **UC-06:** (Optional) Create YouTube playlist with user consent

---

## 5. Data Model (Logical)

* **Users**: `id`, OIDC subject, created\_at
* **VideoHistory**: `user_id`, `provider`, `video_id`, `title`, `watched_at`
* **Settings**: per-site flags (e.g., Auto-PiP enabled)
* **OAuthAccounts**: encrypted refresh tokens (if stored)
* **AuditLogs**: user actions + errors for support/debug

---

## 6. Non-Functional Requirements

* **Performance:**

  * Extension bundle <500KB gzipped
  * Command→action latency p95 <150ms
* **Reliability:**

  * PiP success ≥99.5% weekly
  * Backend API p95 <200ms
* **Accessibility:**

  * Keyboardable overlay
  * WCAG contrast 4.5:1
* **Privacy:**

  * Least-privilege permissions
  * No unnecessary PII collection

---

## 7. Security Model

* MV3 CSP hardened, no remote code execution.
* Minimal host permissions (YouTube/Vimeo/Twitch only).
* Backend secured with CORS allowlist, Helmet, and KMS-encrypted secrets.

---

## 8. Deployment & Release

* **Environments:** dev → beta → prod
* **Rollout:** staged Chrome rollout (5% → 25% → 100%)
* **Targets:** ECS Fargate, RDS, ElastiCache, Cloudflare TLS/WAF
* **Store Readiness:** permissions rationale, screenshots, privacy policy

---

## 9. Observability & Ops

* **Metrics:** PiP events, latency, errors, site adapter type
* **Dashboards:** Grafana/Datadog for metrics
* **Alerts:** PagerDuty on SLO violations
* **Tracing & Logging:** OTel + Sentry

---

## 10. Failure Modes & Graceful Degradation

* **Site DOM drift:** nightly E2E smoke tests; user-friendly error messaging.
* **Media-key conflicts:** fallback to on-screen controls.
* **Suggest/Playlist outage:** core PiP continues unaffected.

---

## 11. Risks & Mitigations

* **Permissions scare-off:** clear store description & user-facing explanations.
* **MV3 cold-start latency:** pre-warm with alarms/events.
* **YouTube API quota issues:** server-side key, Redis caching, per-user rate limits.

---

## 12. Open Questions

* Default skip interval (5s/10s/30s)?
* Auto-PiP restore on tab return?
* How much metadata shown in Suggest panel?
* Should notes/export feature live in-extension or external?
