# Smart PiP — Service Reliability Plan (Draft v1.0)

## 1. Purpose & Scope

Define **Service Level Indicators (SLIs)**, **Service Level Objectives (SLOs)**, and **Service Level Agreements (SLAs)** for Smart PiP.
Covers: Chrome extension client, backend API (NestJS + Postgres), and YouTube OAuth flows.

---

## 2. Service Level Indicators (SLIs)

* **Availability**

  * % of PiP commands that result in successful PiP activation (`pip_started / command_issued`).
  * % of backend API requests returning 2xx within latency budget.
* **Latency**

  * p95 end-to-end command→action (user hits key → PiP visible).
  * p95 backend response time for Suggest/History endpoints.
* **Correctness**

  * % of media-key actions routed to the correct player.
  * % of Smart Suggest results above confidence threshold.
* **Reliability**

  * Error sessions rate (uncaught client/server errors).
* **Durability**

  * % of last-5 video history entries stored/retrieved correctly.

---

## 3. Service Level Objectives (SLOs)

| Area                          | Target          | Notes                                          |
| ----------------------------- | --------------- | ---------------------------------------------- |
| PiP toggle success            | ≥ 99.5% weekly  | Core product value                             |
| Media-key routing correctness | ≥ 99%           | Across YouTube/Vimeo/Twitch                    |
| Command→action latency (p95)  | < 150 ms        | Includes DOM attach + PiP API                  |
| Backend API availability      | ≥ 99.9% monthly | AWS ECS Fargate + RDS                          |
| Backend API latency (p95)     | < 200 ms        | For Suggest/History endpoints                  |
| Error session rate            | < 0.1%          | Tracked via Sentry                             |
| Data durability               | 100%            | Last-5 video history must persist in Postgres  |
| Privacy compliance            | 100%            | No PII stored; OAuth tokens encrypted with KMS |

**Error Budget Policy:**
If SLOs are breached (e.g., PiP success drops below 99.5%), error budget is consumed → halt new feature rollouts until reliability is restored.

---

## 4. Service Level Agreements (SLAs)

(SLAs are external promises to users; they’re stricter than internal SLOs and include remedies.)

* **Extension Availability SLA:** 99.0% monthly (measured via PiP success rate).
* **Backend API SLA:** 99.5% monthly uptime; 95% of requests under 250 ms.
* **Data SLA:** User’s last-5 videos are never lost; if corruption occurs, restore from daily backups.
* **Support SLA (Post-launch):**

  * Bug reports acknowledged within 48h.
  * Critical reliability issues (e.g., PiP broken on YouTube) hot-fixed within 7 days.

---

## 5. Observability & Monitoring

* **Metrics:** `pip_started`, `pip_latency_ms`, `error_hashed`, `api_request_latency_ms`.
* **Dashboards:** Grafana (Prometheus + Tempo + Loki).
* **Alerting:**

  * PagerDuty on SLO violation (latency >150ms p95, error rate >0.1%).
  * Slack channel alerts for backend API downtime.

---

## 6. Reliability Practices

* **Testing:**

  * E2E Puppeteer/Playwright tests across YouTube/Vimeo/Twitch.
  * CI perf smoke with p95 latency <120 ms.
* **Rollouts:** staged (5% → 25% → 100%) with kill-switch feature flags.
* **Backups:** daily Postgres snapshots (RDS/Neon), Redis ephemeral.
* **Failover:** Graceful degrade when Suggest/Playlist API is unavailable (client still functions in core PiP mode).

---

## 7. Document Classification

This document is a **Service Reliability Plan (SRP)** — also known in some FAANG teams as an **SLO/SLA Specification** or part of an **Operational Readiness Review (ORR)** package.
