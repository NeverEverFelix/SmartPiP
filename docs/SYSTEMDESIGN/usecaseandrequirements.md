# Smart PiP — Use Cases & Feature Requirements
**Version:** 1.0 (Requirements-Only)  
**Date:** September 05, 2025

> **Scope note:** This document defines user use-cases and feature requirements only. It intentionally avoids any implementation details, internal architecture, or API references.

---

## 1. Product Overview
Smart PiP is a Chrome extension designed to make Picture‑in‑Picture (PiP) genuinely useful for everyday browsing. It focuses on three experiences:
1) Frictionless PiP for major video sites (YouTube, Vimeo, Twitch).  
2) Simple, predictable **media-key** controls using standard keyboard keys (F7/F8/F9) and F1 (to create playlist *youtube only * ) where available.  
3) An optional “Smart Suggest” experience that recommends a best‑match YouTube video based on the page the user is reading (e.g., studying, instructions).

**Primary Success Criteria**
- Users can keep videos visible and controllable while multitasking.  
- Controls feel familiar (standard media keys) and do not require user configuration.  
- Suggestions are relevant, non-intrusive, and fully optional.

**Out of Scope (for this document)**
- How any feature is implemented.  
- Background services, APIs, or data-models.  
- Non-Chrome browsers and mobile platforms.

---

## 2. Target Platforms
- **Browser:** Google Chrome (desktop).  
- **Operating Systems:** Windows, macOS, Linux.  
- **Supported Video Sites:** YouTube, Vimeo, and Twitch 

---

## 3. Personas
- **The Multitasker:** Keeps a lecture or stream on-screen while coding or reading. Wants quick controls and minimal friction.  
- **The Learner:** Lands on “how to …” pages and wants the best corresponding video tutorial surfaced quickly.  
- **The Casual Viewer:** Watches long-form content “on the side” and expects media keys to “just work.”

---

## 4. Core User Use‑Cases

### UC‑01: Toggle and Maintain PiP on Major Video Sites
**Goal:** Keep the current video in PiP and continue watching while navigating other tabs/apps.  
**Preconditions:** A supported site page with an active or visible video.  
**Flow:**
1. User initiates PiP from the extension UI or standard site controls.  
2. A floating PiP window appears and remains visible while the **source tab remains open**.  
3. User can move to another tab or application; PiP remains on top.  
4. PiP closes when the source video/tab is closed or when the user explicitly exits PiP.  
**Postconditions:** User returns to the source tab with the video state intact (if the tab remains open).

### UC‑02: Media‑Key Controls (F7/F8/F9)
**Goal:** Control PiP playback with familiar, predetermined keyboard keys.  
**Preconditions:** System keyboard provides F7/F8/F9 (or equivalent media keys). A PiP video is active.  
**Flow:**
1. **F8** toggles Play/Pause of the active PiP video.  
2. **F7** performs a “Back” action:  
   - If a previous item exists in the current context (e.g., prior video in a sequence), return to it;  
   - Otherwise, perform a short skip‑back within the current video (e.g., 10 seconds).  
3. **F9** performs a “Forward” action:  
   - If a next item exists in the context, advance to it;  
   - Otherwise, perform a short skip‑forward within the current video (e.g., 10 seconds).  
4. If the keyboard lacks these keys, the user can use on‑screen controls instead (see UC‑03).  
5. Drag the top left to increase/decrease size of video.
**Postconditions:** Playback state reflects the action (paused/playing/position changed) without losing PiP.

### UC‑03: On‑Screen PiP Controls
**Goal:** Provide visible, mouse‑accessible controls when keyboard media keys are unavailable or not preferred.  
**Preconditions:** A PiP video is active.  
**Flow:**
1. User reveals minimal on‑screen controls near or within the PiP window.  
2. Controls include: Play/Pause, Skip Back (e.g., 10s), Skip Forward (e.g., 10s), Close/Exit PiP.  
3. Controls are discoverable, non‑obtrusive, and can be repositioned if they block content.  
**Postconditions:** Video responds immediately to user input; PiP remains active unless explicitly closed.

### UC‑04: Auto‑PiP on Tab Blur (Optional)
**Goal:** Enter PiP automatically when the user switches tabs or the source tab loses focus.  
**Preconditions:** Feature toggled **on** for the current site; a video is present.  
**Flow:**
1. User enables “Auto‑PiP on Tab Blur” per site or globally.  
2. When the source tab loses focus, PiP appears automatically.  
3. When returning to the source tab, PiP can remain or close based on the user’s preference.  
**Postconditions:** Users can multitask without manual PiP toggling.

### UC‑05: Smart Suggest (Optional)
**Goal:** Suggest a best‑match YouTube video related to the page the user is viewing (e.g., study material or instructions).  
**Preconditions:** Smart Suggest is enabled; the current page is readable and relevant.  
**Flow:**
1. User opens the suggestion panel from the extension UI while reading an article or documentation.  
2. The panel displays a small set of relevant YouTube videos (e.g., top 3–5).  
3. User selects a video; it opens and enters PiP.  
**Postconditions:** User is watching a relevant video without needing to search manually.

### UC‑06: YouTube Playlist Creation (Optional)
**Goal:** Allow users to create and manage YouTube playlists for later viewing.  
**Preconditions:** User explicitly connects their YouTube account and consents to playlist access; Smart PiP provides clear disclosure.  
**Flow:**
1. User selects “Create playlist” and provides a playlist name (e.g., “Study Later”).  
2. User can add the current or suggested videos to the playlist.  
3. User can reorder or remove items and view the updated list.  
**Postconditions:** The playlist reflects the user’s changes and is accessible for later viewing.

### UC‑07: Transcripts & Notes (When Available)
**Goal:** Help learners follow along by viewing transcripts (if provided by the source) and saving time‑stamped notes.  
**Preconditions:** The active video offers transcripts or captions; the user opens the transcript/notes panel.  
**Flow:**
1. Transcript (when present) is displayed with time markers.  
2. User clicks a timestamp to jump playback.  
3. User can add a note that includes the playback time; notes can be reviewed later.  
**Postconditions:** Notes and transcript references persist for future sessions.

### UC‑08: Site Controls (Whitelist/Blacklist)
**Goal:** Give users control over where Smart PiP is active.  
**Preconditions:** User opens the extension’s site settings.  
**Flow:**
1. User adds sites to a whitelist for Auto‑PiP or Smart Suggest.  
2. User adds sites to a blacklist to prevent any Smart PiP behavior on those domains.  
3. User can quickly toggle a site’s status from the main UI.  
**Postconditions:** Smart PiP respects the user’s per‑site preferences.

### UC‑09: Auto‑Resume Within a Site
**Goal:** Maintain PiP continuity while navigating within the same video site (e.g., moving between watch pages).  
**Preconditions:** PiP is active; the user navigates within YouTube/Vimeo/Twitch.  
**Flow:**
1. As the user navigates, PiP continues playing as long as a valid source remains available and the originating tab remains open.  
2. If the content changes to a new video, PiP switches to that new video where appropriate.  
**Postconditions:** Minimal disruptions while browsing within the same site.

### UC‑10: Close & Exit Behavior
**Goal:** Provide a clear way to exit PiP and return focus to the source tab/video.  
**Preconditions:** PiP is active.  
**Flow:**
1. User clicks the close/exit control on the PiP window or uses a designated close command.  
2. PiP closes; the source video remains in its last known state in the source tab.  
**Postconditions:** No orphaned floating elements; user regains context.

---

## 5. Feature Requirements (Functional)

> Requirements use MoSCoW prioritization: **Must**, **Should**, **Could**, **Won’t** (for now). Each requirement includes acceptance criteria and key constraints where user‑visible.

### FR‑001: PiP Toggle on Supported Sites — **Must**
**Description:** Users can start/stop PiP on YouTube, Vimeo, and Twitch.  
**Acceptance Criteria:**
- A visible control allows entering/exiting PiP on supported sites.  
- PiP remains active while the source tab is open.  
- Exiting PiP returns control to the source video without losing the page context.  
**Constraints (User‑Visible):** DRM or site changes may limit availability; a brief, friendly notice explains when PiP isn’t possible.

### FR‑002: Media‑Key Controls (F7/F8/F9) — **Must**
**Description:** Playback control via standard keyboard media keys where available.  
**Acceptance Criteria:**
- **F8:** toggles play/pause of the active PiP video.  
- **F7:** performs “Back” (previous item if available; otherwise skip‑back within the current video).  
- **F9:** performs “Forward” (next item if available; otherwise skip‑forward within the current video).  
- If media keys are unavailable on the device, on‑screen controls (FR‑003) provide equivalent functionality.  
**Constraints (User‑Visible):** Some devices or focused apps may intercept media keys; the extension should clearly communicate when keys aren’t available.

### FR‑003: On‑Screen Controls — **Must**
**Description:** Minimal, discoverable PiP controls accessible via mouse/trackpad.  
**Acceptance Criteria:**
- Controls include Play/Pause, Skip Back, Skip Forward, and Close.  
- Controls don’t obstruct critical content and can be repositioned.  
- Controls remain responsive and clearly indicate state (e.g., paused).

### FR‑004: Auto‑PiP on Tab Blur — **Should**
**Description:** Optionally enter PiP when the source tab loses focus.  
**Acceptance Criteria:**
- Per‑site toggle is available (e.g., enabled for YouTube but not for Twitch).  
- When enabled, switching away triggers PiP; switching back respects the user’s preference to remain in PiP or restore inline playback.  
**Constraints (User‑Visible):** Requires prior user consent for a site; a clear setting explains behavior.

### FR‑005: Smart Suggest Panel — **Should**
**Description:** Suggest a small set of relevant YouTube videos based on the current page (optional).  
**Acceptance Criteria:**
- When enabled, users can open a suggestion panel on any readable page.  
- The panel shows 3–5 video results with titles and basic context.  
- Selecting a result opens the chosen video and presents it in PiP.  
**Constraints (User‑Visible):** Suggestions may not be available on all pages; panel communicates when no relevant results are found.

### FR‑006: YouTube Playlist Creation — **Could**
**Description:** Create and manage playlists for later viewing (YouTube only).  
**Acceptance Criteria:**
- Users can opt‑in and connect their account with clear consent and scope descriptions.  
- Users can create a named playlist, add/remove items, and reorder.  
- Users can view a playlist summary (title, size) within the extension.  
**Constraints (User‑Visible):** Requires account connection and explicit consent; feature remains fully optional.

### FR‑007: Transcripts & Notes — **Could**
**Description:** Display transcripts (when available) and allow time‑stamped notes.  
**Acceptance Criteria:**
- A transcript panel displays when the video provides captions/transcript.  
- Clicking a time marker navigates playback accordingly.  
- Notes save with a timestamp and can be reviewed later.  
**Constraints (User‑Visible):** Not all videos provide transcripts; UI states this clearly.

### FR‑008: Site Whitelist/Blacklist — **Should**
**Description:** Per‑site control over Smart PiP behavior.  
**Acceptance Criteria:**
- Users can add or remove sites from a whitelist for Auto‑PiP and/or Smart Suggest.  
- Users can add sites to a blacklist to disable all Smart PiP behavior.  
- The current site’s status is clearly shown and easy to toggle.

### FR‑009: Auto‑Resume Within a Site — **Should**
**Description:** Maintain PiP continuity while navigating within the same site.  
**Acceptance Criteria:**
- While moving between relevant pages on YouTube, Vimeo, or Twitch, PiP continues without unnecessary interruption.  
- When the content changes (e.g., a new video), the PiP experience updates accordingly with minimal disruption.  
**Constraints (User‑Visible):** Some site behaviors may limit continuity; the UI communicates when continuity isn’t possible.

### FR‑010: Close & Exit — **Must**
**Description:** Provide a predictable way to exit PiP.  
**Acceptance Criteria:**
- A clear Close/Exit control is always available in the PiP context.  
- Closing PiP does not unexpectedly navigate away from the source page.  
- The source video state (time, play/pause) remains consistent when PiP closes.

---

## 6. Non‑Functional Requirements (NFRs)

### NFR‑A: Usability & Learnability
- Media‑key mapping uses familiar conventions (F7/F8/F9).  
- On‑screen controls are discoverable within 2–3 seconds by a first‑time user.  
- No required configuration to achieve core value.

### NFR‑B: Performance & Smoothness
- Entering/exiting PiP feels instantaneous to the user.  
- On‑screen controls react immediately and do not feel sluggish.  
- Resource usage does not noticeably degrade browsing or video playback.

### NFR‑C: Accessibility
- On‑screen controls include accessible labels and clear focus indicators.  
- The suggestion and transcript/notes panels are keyboard navigable.  
- Visual states (playing, paused, disabled) are communicated with sufficient contrast.

### NFR‑D: Privacy & Transparency
- The extension clearly explains optional features (Smart Suggest, Playlist Creation) and what information they use.  
- Users can opt out of suggestions at any time.  
- Any account connection is explicit and reversible; scopes/permissions are clearly described in human‑readable language.

### NFR‑E: Reliability
- PiP controls consistently reflect the current playback state.  
- Features degrade gracefully on pages where PiP or transcripts are unavailable.  
- Site‑specific quirks are handled with user‑friendly messaging when limits are encountered.

### NFR‑F: Internationalization (Basic)
- UI strings are prepared so they can be translated in the future.  
- Date/time formatting for notes follows system locale.

---

## 7. Settings & Preferences

- **General**
  - Enable/disable Smart Suggest (default: off).  
  - Enable/disable Auto‑PiP on Tab Blur (default: off).  
  - Site Whitelist / Blacklist management.  

- **Controls**
  - Show/hide on‑screen controls.  
  - Choose skip interval values (e.g., 5s/10s/30s) for Back/Forward when no previous/next item exists.  
  - Close behavior preference when returning to the source tab (remain in PiP vs. restore inline).

- **Data**
  - View/export notes (when enabled).  
  - Disconnect linked accounts (when enabled for playlist creation).

---

## 8. Error States & Empty States (User‑Visible)

- **PiP Not Available:** “Picture‑in‑Picture isn’t available on this page.”  
- **No Suggestions Found:** “No matching videos found for this page.”  
- **Media Keys Unavailable:** “Media keys aren’t available on this device. Use on‑screen controls instead.”  
- **Transcript Unavailable:** “This video doesn’t provide a transcript.”  
- **Account Not Connected (Playlist):** “Connect your account to create or manage playlists.”

---

## 9. Compatibility & Constraints (User‑Visible)

- **Supported Sites:** Optimized for YouTube, Vimeo, and Twitch. Behavior on other video sites is not guaranteed.  
- **Media Keys:** Behavior depends on the device/OS. Some apps may intercept media keys; on‑screen controls are always available.  
- **Site Variability:** Some sites may change layouts or restrict features; Smart PiP communicates clearly when a feature is unavailable.

---

## 10. Release Scope (Proposed)

- **MVP (Must):** FR‑001, FR‑002, FR‑003, FR‑010.  
- **V1.0 (Should):** FR‑004, FR‑005, FR‑008, FR‑009.  
- **V1.1 (Could):** FR‑006, FR‑007.

> Note: Release groupings communicate **priority**, not implementation details.

---

## 11. Risks (User‑Visible Impact) & Mitigations
- **Media‑Key Conflicts:** Some devices/apps may capture keys → Provide clear fallback via on‑screen controls and messaging.  
- **Site Changes:** Video sites evolve → Maintain transparent messaging when PiP or controls are limited.  
- **Suggestion Quality:** Irrelevant results frustrate users → Keep panel optional and easy to dismiss; allow quick feedback (“Not relevant”).  
- **Account Linking Friction:** Some users avoid connecting accounts → Keep playlist creation optional and clearly explained.

---

## 12. Open Questions (to be resolved with stakeholders)
1. What should the default skip interval be when no previous/next item exists (e.g., 5s vs 10s vs 15s)?  
2. Should Auto‑PiP close automatically when the user returns to the source tab, or remain in PiP by default?  
3. For Smart Suggest, do we show additional context (duration/channel) or keep it minimal to reduce clutter?  
4. For notes, do we offer exporting to a file or just an in‑extension view?  
5. Do we support basic controls for non‑focus pages (e.g., multiple videos on a page)?

---

## 13. Glossary
- **PiP (Picture‑in‑Picture):** A small, always‑on‑top video window that remains visible while users interact with other tabs/apps.  
- **Media Keys:** Keyboard keys commonly labeled as Back (F7), Play/Pause (F8), and Forward (F9).  
- **Smart Suggest:** A feature that proposes relevant videos based on the page the user is currently viewing.  
- **Transcript:** A text rendering of a video’s speech content, often with timestamps.

---

## 14. Change Log
- **v1.0 (September 05, 2025):** Initial requirements-only draft aligned to predetermined media‑key controls and optional Smart Suggest & Playlist features.
