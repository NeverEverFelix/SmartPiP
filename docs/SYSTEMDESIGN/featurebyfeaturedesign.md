# Smart PiP Chrome Extension — Feature-by-Feature Design Document

## Conventions

* **Extension surfaces:** Manifest V3 service worker (command router), site adapters (YouTube/Vimeo/Twitch), popup/options, small overlay.
* **Perf/SLO guardrails:** command → action p95 <150 ms; PiP success ≥99.5% weekly.
* **Privacy:** least-privilege host permissions; last-5 history only (purgeable); optional OAuth scopes for playlists.
* **Observability:** `command_issued`, `pip_started`, `pip_latency_ms`, `adapter_name`, `errors` (opt-in).

---

## Feature 1. Cross-Platform Picture-in-Picture

**User Value.** Maintain video visibility across multitasking.

**UX.**

* Toggle PiP via popup button or keyboard.
* Overlay shows play/pause, skip ±N, close; keyboard accessible (ARIA).

**Implementation.**

* Site adapters detect `<video>` and expose control API.
* PiP lifecycle: `video.requestPictureInPicture()` with session-tracked `{tabId, provider, videoId}`.
* Auto-resume on navigation within site.

**Testing.**

* Golden-path E2E per site.
* PiP success ≥99.5%.

---

## Feature 2. Media Keys F7/F8/F9

**User Value.** Standard playback shortcuts.

**Command Routing.**

* Service worker listens to `chrome.commands` and routes to PiP session.
* Actions:

  * **F8:** Play/pause toggle
  * **F7:** Seek −10s (or prev item)
  * **F9:** Seek +10s (or next item)
* Configurable skip interval.

**Fallback.**

* If OS intercepts keys, surface overlay controls.

**SLO.** ≥99% routing correctness; latency <150 ms.

---

## Feature 3. F1 — Resume Last Watched Video

**User Value.** Quick resume.

**Flow.**

* Fetch last item from history.
* Open in new tab and start PiP in current tab.

**Data.**

* History schema: `{provider, videoId, title, url, watchedAt}`.
* Store max 5.

**MVP vs Backend.**

* Local storage (MVP).
* Backend `/v1/history/last` (Phase 2).

---

## Feature 4. Last-5 History with Tags

**User Value.** Lightweight continuity.

**Implementation.**

* On significant progress (>30s or >10%), record entry.
* Server or client prunes to 5.

**Privacy.**

* Options page: "Clear history".
* Disclosure: only last 5 stored.

**Schema.**

* `video_history(user_id, provider, video_id, title, url, watched_at)`.

---

## Feature 5. F2 — YouTube Playlist Creation

**User Value.** One-key "save for later".

**Auth.**

* OAuth via `chrome.identity`.
* Minimal scopes requested at first use.

**Flows.**

* If no playlist: prompt → create → add.
* If playlist exists: add directly.
* Idempotency key = `{playlistId, videoId}`.

**UX.**

* Toast confirmation with undo.
* Settings: connect/disconnect account.

---

## Feature 6. F3 — Smart Suggest

**User Value.** Suggest relevant video based on current page.

**Signals.**

* Extract title, H1, meta, or selected text.

**Backend (recommended).**

* Proxy YouTube search.
* Rank by viewCount, freshness, relevance.
* Cache results for 10–30 minutes.

**MVP (alt).**

* Client-only heuristic suggestion.

**Failure.**

* Silent degrade when quota/outage.
* Empty state message.

---

## Feature 7. Drag-to-Shrink from Top-Left Corner

**User Value.** Flexible resizing.

**Implementation.**

* Native PiP: OS-resizable, we listen for resize events.
* Overlay mini-player: draggable/resizable with top-left handle; CSS transform.

**Accessibility.**

* Handle labeled; keyboard resizing with Alt+Arrows.

---

## Feature 8. Auto-PiP on Tab Blur

**Behavior.**

* Enabled per-site toggle.
* On blur: start PiP (p95 <300 ms).
* On return: keep or restore inline (setting).

**Implementation.**

* Adapter listens to visibility change.
* Debounce duplicate events.

---

## Feature 9. On-Screen Controls

**Placement.**

* Minimal controls adjacent to PiP/mini-player.
* Auto-hide after 1s; never obscure site UI.
* Repositionable.

**Fallback.**

* If media keys intercepted, overlay persists longer.

---

## Feature 10. Settings & Permissions

**Options Page.**

* Auto-PiP per-site toggle.
* Skip interval selection.
* PiP mode: native vs overlay.
* Smart Suggest toggle & disclosure.
* YouTube account connect/disconnect.
* Clear history button.

**Manifest Permissions.**

* `commands`, `storage`, `tabs`, `scripting`, `identity` (gated).
* Host permissions limited to YouTube/Vimeo/Twitch.

---

## Feature 11. Backend API & Data (Phase 2)

**Endpoints.**

* `/v1/history`
* `/v1/history/last`
* `/v1/suggest`
* `/v1/youtube/playlist`
* `/healthz`, `/metrics`

**DB.**

* `users`, `video_history`, `settings`, `oauth_accounts`, `audit_logs`.
* PgBouncer; Redis cache.

**SLOs.**

* API availability ≥99.9%.
* p95 latency <200 ms.

---

## Feature 12. Testing & Release Gating

**Unit.** Adapters, router, suggest ranking, history prune.

**E2E.** Playwright flows for PiP, auto-PiP, media keys, resume, playlist, suggest.

**Perf CI.** Command → action p95 <120 ms.

**Rollout.** Staged: 5% → 25% → 100% with SLO monitors.

---

## Scope Choices

* **MVP:** Client-first; local history; client-only suggest; no backend.
* **Phase 2:** Client + Backend; centralized suggest, durable history, playlist proxy.

**Recommendation:** Phase 2 for production quality and observability, with feature flags and graceful fallback.
