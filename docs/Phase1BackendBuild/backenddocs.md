# Smart PiP — Phase 1 Backend (Ultra-Cheap Deployment Plan w/ Cloudflare)

**Owner:** Felix Moronge
**Scope:** Backend service for Smart PiP MVP (API + worker + DB + cache)
**Deployment Model:** Ultra-cheap, single EC2 (Graviton) instance running Docker Compose, fronted by Cloudflare
**Target:** Ship MVP with minimal cost (<\$40/mo) and keep a clean scale-out path

---

## 1) What We’re Building (Phase-1 Outcome)

A backend that exposes:

* **Auth (OIDC):** Google sign-in → short-lived JWT.
* **History:** CRUD for **last 5 watched videos per user** (with provider tag).
* **Suggest:** Proxy to YouTube search (API key server-side) + caching/ranking.
* **Playlist (optional):** Server-assisted playlist creation, only if user consents.
* **Health/Readiness:** `/healthz`, `/readyz`.
* **Metrics:** `/metrics` (Prometheus format).
* **Worker:** BullMQ for background jobs (cache warm, cleanup, rate-limit bookkeeping).

**Graceful degrade:** Core PiP works without backend; Suggest/Playlist disable cleanly if backend is down.

---

## 2) Architecture (Ultra-Cheap “One Box” w/ Cloudflare)

```
User browser (Smart PiP extension)
   │
   ▼
Cloudflare (DNS, TLS termination, WAF, CDN for static)
   │
   ▼
EC2 (t4g.small) running Docker Compose:
   - api (NestJS/Fastify)
   - worker (BullMQ jobs)
   - postgres (data, EBS volume)
   - redis (cache, ephemeral)
   - nginx (reverse proxy, origin TLS)
```

**Services (all on one host):**

* `api`: Auth, History, Suggest, Health, Metrics.
* `worker`: BullMQ worker (jobs, caching, rate limiting).
* `postgres`: containerized Postgres 15 with persistent EBS volume.
* `redis`: containerized Redis 7 (no persistence, cache/queue only).
* `nginx`: reverse proxy, optional origin TLS (Cloudflare Full Strict).

**Cloudflare responsibilities:**

* DNS & TLS termination (free SSL certs, auto-renew).
* CDN for static assets (icons, manifests, docs). API endpoints marked as non-cacheable.
* DDoS protection, WAF, and rate-limiting at the edge.
* Automatic HTTPS redirect + HSTS.

---

## 3) API v1 (REST)

* **Auth**

  * `POST /v1/auth/oidc/callback` → exchange Google sign-in → JWT.
* **History**

  * `GET  /v1/history` → return ≤5 items.
  * `POST /v1/history` → upsert entry `{provider, videoId, title, url, watchedAt}` and prune to 5.
  * `GET  /v1/history/last` → return most recent.
  * `DELETE /v1/history` → purge history.
* **Suggest**

  * `GET /v1/suggest?q=<context>` → return ranked suggestions.
  * Cache: Redis, TTL 10–30 min.
* **Playlist (optional)**

  * `POST /v1/youtube/playlist` → add to playlist if user consent + scope.
* **Ops**

  * `GET /healthz`, `GET /readyz`, `GET /metrics`

---

## 4) Data Model (Postgres)

**Tables:**

* `users(id uuid pk, oidc_sub text unique, created_at timestamptz default now())`
* `video_history(id bigserial pk, user_id fk, provider text, video_id text, title text, url text, watched_at timestamptz)`
* `settings(user_id pk, auto_pip bool, skip_seconds int default 10, site_flags jsonb)`
* `oauth_accounts(...)` *(only if server stores refresh tokens)*
* `audit_logs(id bigserial pk, user_id fk, event text, meta jsonb, created_at timestamptz)`

**Constraints:**

* `video_history`: keep only last 5 per user (trigger or app logic).

---

## 5) Suggest Ranking

1. Normalize query terms (strip stopwords).
2. Call YouTube Search API with safe defaults.
3. Rank: `score = log(viewCount+1) + freshnessBoost(publishedAt) + textMatch(query,title,desc)`.
4. Cache result in Redis.

---

## 6) Ops, Security & Privacy

* **Secrets:** Stored in AWS SSM Parameter Store (cheaper than Secrets Manager).
* **TLS:** Cloudflare terminates TLS; origin can run Let’s Encrypt or Cloudflare Origin Cert.
* **Auth:** JWT (15m expiry) issued after OIDC.
* **Data retention:** Last-5 history only; purge on account deletion.
* **Backups:** Nightly `pg_dump` to S3; retention 30–90 days.
* **Edge security:** Cloudflare WAF + bot protection.

---

## 7) Observability

* Logs: pino JSON logs shipped to CloudWatch (7–14d retention).
* Metrics: `/metrics` endpoint scraped by CloudWatch Agent (or Prometheus if self-run).
* Alerts: CPU >80%, disk <15%, error rate >1%.

---

## 8) Deployment & Runbook

* **Provision:** t4g.small EC2 with 20–40GB gp3 EBS.
* **Install:** Docker + Compose.
* **Run:** `docker compose up -d` (api, worker, postgres, redis, nginx).
* **DNS:** `api.smartpip.com` → Cloudflare → EC2 public IP.
* **TLS:** Cloudflare Full Strict; origin cert optional.
* **Backup:** cron job `pg_dump` → S3.
* **Patch:** monthly OS updates; rebuild containers weekly.

---

## 9) Reliability Targets (One-Box MVP)

* Availability: best effort; single point of failure.
* Latency: p95 <200ms for Suggest/History.
* Data durability: nightly dumps; last-5 history must persist.

---

## 10) Cost & Scale Path

* **Estimated Cost:** \~\$20–\$40/mo (t4g.micro/small + EBS + CloudWatch light; Cloudflare free tier).
* **Scale Path:**

  1. Migrate Postgres → RDS single-AZ.
  2. Move app + worker → ECS Fargate (Graviton).
  3. Replace Redis → ElastiCache.
  4. Add ALB + multi-AZ DB when >5–10k DAU.
  5. Cloudflare stays in front as CDN + WAF.
