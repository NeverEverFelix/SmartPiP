# Smart PiP — Goals & Success Criteria (MVP)

**Status:** Draft v1.0  
**Owner:** Felix Moronge (PM/Eng)  
**Date:** 2025-09-05  
**Related:** Smart-PiP Use Cases & Requirements (PRD), HLD/Tech Spec (MV3), Test Plan, Privacy Policy, Release Plan

---

## 1) Purpose & Scope
Define measurable goals for Smart PiP (Chrome MV3) that reflect a production-grade, user-facing product. These goals drive design, testing, release gating, and post-launch monitoring.

**In scope (MVP):** YouTube, Vimeo, Twitch; PiP toggle; Auto-PiP on blur; media keys; on-screen mini controls; Smart Suggest (light, on-device).  
**Out of scope (v1):** Accounts/sync, heavy ML, server backends, ads/monetization, scraping protected content, broad site support without adapters.

---

## 2) Product Goals (Functional)

### G1 — Reliable PiP Toggle
- **Definition:** One-click PiP (popup or hotkey) attaches to the visible/active player on supported sites.
- **Success Criteria:**
  - **Accuracy:** ≥95% successful attaches on supported pages.
  - **Latency:** command→action dispatch **p95 < 100 ms**.
  - **UX:** No unintended page scrolls or focus loss.

### G2 — Auto-PiP on Blur (Per-site Toggle)
- **Definition:** When the user leaves a tab with a playing video, PiP auto-starts; restore cleanly on return.
- **Success Criteria:**
  - **Latency:** **p95 < 300 ms** from blur to PiP.
  - **Correctness:** No duplicate PiP windows; re-focus restores previous state.

### G3 — Media Keys Routing
- **Definition:** Play/Pause/Next/Prev target the active player or PiP consistently.
- **Success Criteria:**
  - **Correct Targeting:** **≥99%** correctness across YouTube/Vimeo/Twitch.
  - **No Conflicts:** Does not trigger page shortcuts or scroll.

### G4 — On-Screen Mini Controls
- **Definition:** Minimal overlay with Play/Pause, Exit PiP, Next/Prev when applicable.
- **Success Criteria:**
  - **A11y:** Keyboard accessible; labeled controls; no layout shift.
  - **UX:** Overlay never blocks native controls for >1s; hides when not needed.

### G5 — Smart Suggest (Lite, On-Device)
- **Definition:** On obvious “how-to” pages, surface exactly one best tutorial/resource suggestion.
- **Success Criteria:**
  - **Precision:** False-positive rate **<5%**; shows only above confidence threshold.
  - **Graceful Degrade:** Silent when unsure; no network calls required.

---

## 3) Non-Functional Goals (NFRs)

### Performance
- Content scripts **idle by default**; service worker sleeps when not handling events.
- Steady-state memory **< 20 MB**; bundle size **< 500 KB gz** (stretch: < 300 KB).
- Command→action **p95 < 120 ms** under E2E load (CI smoke).

### Reliability (SLOs & Error Budget)
- **SLO1:** PiP command success **≥99.5%** weekly.
- **SLO2:** Uncaught error sessions **<0.1%**.
- **Policy:** Breach consumes error budget → pause rollouts, fix, and/or flag off feature.

### Privacy
- Least-privilege host patterns; only required Chrome perms (`activeTab`, `scripting`, `storage`, `commands`).
- No PII; telemetry is **opt-in** and anonymized; clear disclosures in-product + store listing.

### Accessibility
- Full keyboard navigation; ARIA roles/labels on overlay and popup; **contrast ≥ 4.5:1**.
- Screen reader labels validated (NVDA/VoiceOver spot checks).

### Compatibility
- Chrome MV3 primary; Chromium derivatives best-effort. Degrade safely if PiP API unavailable.

---

## 4) Engineering & Process Goals (FAANG-grade)

### E1 — Observability
- **Events (opt-in):** `command_issued`, `pip_started`, `pip_latency_ms`, `site_adapter`, `error_hashed`, `version`.
- **Dashboards:** Daily health + weekly report (SLOs, latency, adoption, error budget).

### E2 — Testing
- **Unit:** ≥80% coverage on adapters, messaging, heuristics.
- **E2E:** Puppeteer runs for 5 golden paths (YouTube/Vimeo/Twitch: toggle, auto-PiP, media keys).
- **Perf smoke:** command→action **p95 < 120 ms** in CI.
- **Static checks:** Lint + typecheck required to merge.

### E3 — Feature Flags & Rollback
- All new features behind flags; Options includes kill switch.
- Storage migrations are versioned and reversible.

### E4 — Release Hygiene
- Staged rollout: **5% → 25% → 100%** with monitoring gates.
- Signed, reproducible build; tagged releases; CHANGELOG.

### E5 — Store Readiness
- Clear permission rationale; compliant screenshots; short demo video; privacy policy URL; support email; known-issues list.

---

## 5) KPI Scorecard (Launch Targets)

| KPI | Target | How Measured |
|---|---:|---|
| PiP command success (site-weighted) | ≥ 99.5% | `pip_started` ÷ `command_issued` |
| Command→action latency p95 | < 150 ms | `pip_latency_ms` distribution |
| Error sessions | < 0.1% | sessions with `error_hashed` |
| Weekly retention (WAU/DAU) | ≥ 20% | opt-in telemetry / store stats |
| Store rating | ≥ 4.7★ (≥25 reviews) | Chrome Web Store |
| Active installs (30–60d) | ≥ 1,000 | Store console |

---

## 6) Acceptance Criteria by Goal
- **G1:** On each supported site, 20/20 scripted E2E attempts succeed; manual ad-hoc across 5 random videos succeed ≥95%.
- **G2:** 20/20 tab blur tests produce PiP in <300 ms; no duplicates on return.
- **G3:** 50 media-key actions across sites → ≥99% routed correctly; no page scroll/shortcut side-effects.
- **G4:** Keyboard-only flow completes all overlay actions; axe-core/a11y check passes; no CLS from overlay.
- **G5:** On a labeled set of 100 how-to pages, FP <5%, TP ≥70%; silent on non-how-to pages.

---

## 7) Release Readiness Checklist (Gating)
- [ ] Unit coverage ≥80% for adapters/messaging/heuristics.
- [ ] E2E suite green on CI (3 sites × 5 paths).
- [ ] Perf smoke: p95 command→action <120 ms.
- [ ] A11y scan + manual screen reader spot check passed.
- [ ] Privacy copy & permissions rationale finalized; telemetry opt-in implemented.
- [ ] Feature flags + kill switch verified.
- [ ] Staged rollout plan configured; dashboards live.
- [ ] Store assets (screenshots/video) + CHANGELOG ready.

---

## 8) Traceability to PRD
Each goal maps to PRD requirements:  
- G1 → FR-001, FR-002; G2 → FR-010; G3 → FR-020; G4 → FR-030; G5 → FR-040.  
NFRs map to PRD-NFR-P (Performance), PRD-NFR-R (Reliability), PRD-NFR-PR (Privacy), PRD-NFR-A11y.

---

## 9) Risks & Mitigations (MVP)
- **R1:** Site DOM changes break adapters → **Mitigation:** adapter interface + smoke E2E on nightly canary.
- **R2:** Permissions scare off users → **Mitigation:** least-privilege host patterns; clear in-product rationale.
- **R3:** Media key conflicts on some platforms → **Mitigation:** detection + fallback hints; disable per-site if needed.
- **R4:** Latency regressions with MV3 service worker → **Mitigation:** pre-warm via alarms/events; perf smoke in CI.
