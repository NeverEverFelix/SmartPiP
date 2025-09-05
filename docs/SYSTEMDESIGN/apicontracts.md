# Smart PiP — APIs & Contracts

This document specifies **message types**, **payload schemas**, and the **site‑adapter interface** for the Smart PiP extension. It is written to be implementation‑ready for MV3 with content scripts, a service worker (SW), and optional backend calls. All examples are **TypeScript**.

> **Versioning**
>
> * Semantic versioning for contracts using `contractVersion: 'x.y'`.
> * Discriminated unions with a stable `type` field.
> * **Never** reuse a `type` value for a different shape; add a new type instead.

---

## 1) Core Message Envelope

```ts
/** Common envelope for all extension-internal messages. */
export interface MessageBase {
  /** Contract semver. Increment minor for backward‑compatible additions; major for breaking. */
  contractVersion: '1.0';
  /** Discriminator for union narrowing. */
  type: MessageType;
  /** Monotonic (uuid v4 recommended) for correlating req/resp and telemetry. */
  requestId: string;
  /** Actor origin, used for routing/permissions. */
  source: 'content-script' | 'service-worker' | 'popup' | 'options' | 'overlay';
  /** Optional tab context; SW will add if absent during routing. */
  tabId?: number;
}

/** Common success/err shape attached to responses. */
export interface ResponseBase {
  contractVersion: '1.0';
  type: ResponseType;
  requestId: string; // mirrors request
  ok: boolean;
  error?: ErrorPayload;
}

export interface ErrorPayload {
  code: ErrorCode;
  message: string;      // user-safe summary
  details?: unknown;    // internal debugging info (redacted from logs if sensitive)
}
```

### Enums

```ts
export type MessageType =
  | 'PIP_TOGGLE'
  | 'PIP_STATUS_GET'
  | 'MEDIA_PLAY_PAUSE'
  | 'MEDIA_SKIP'
  | 'MEDIA_PREV_NEXT'
  | 'HISTORY_GET_LAST'
  | 'HISTORY_PUSH'
  | 'SUGGEST_QUERY'
  | 'ADAPTER_ATTACH'
  | 'ADAPTER_DETACH';

export type ResponseType = `${MessageType}_RESPONSE` | 'STREAM_CHUNK' | 'STREAM_END';

export enum ErrorCode {
  UNKNOWN = 'UNKNOWN',
  NO_VIDEO = 'NO_VIDEO',
  ADAPTER_NOT_FOUND = 'ADAPTER_NOT_FOUND',
  PIP_UNSUPPORTED = 'PIP_UNSUPPORTED',
  PERMISSION_DENIED = 'PERMISSION_DENIED',
  COMMAND_INTERCEPTED = 'COMMAND_INTERCEPTED',
  RATE_LIMITED = 'RATE_LIMITED',
  BACKEND_UNAVAILABLE = 'BACKEND_UNAVAILABLE',
  INVALID_ARGUMENT = 'INVALID_ARGUMENT',
}
```

---

## 2) Message Payload Schemas (Requests & Responses)

### 2.1 PIP\_TOGGLE

```ts
export interface PipToggleRequest extends MessageBase {
  type: 'PIP_TOGGLE';
  /** Whether to force a mode; if omitted, toggle current. */
  action?: 'enter' | 'exit' | 'toggle';
  /** Preferred mode; native PiP or overlay mini-player. */
  mode?: 'native' | 'overlay';
}

export interface PipToggleResponse extends ResponseBase {
  type: 'PIP_TOGGLE_RESPONSE';
  ok: boolean;
  state?: PipState; // present if ok
}

export interface PipState {
  active: boolean;
  mode: 'native' | 'overlay' | 'none';
  provider?: Provider;  // 'youtube' | 'vimeo' | 'twitch'
  videoId?: string;
  tabId?: number;
}
```

### 2.2 PIP\_STATUS\_GET

```ts
export interface PipStatusGetRequest extends MessageBase {
  type: 'PIP_STATUS_GET';
}

export interface PipStatusGetResponse extends ResponseBase {
  type: 'PIP_STATUS_GET_RESPONSE';
  ok: boolean;
  state?: PipState;
}
```

### 2.3 MEDIA\_PLAY\_PAUSE

```ts
export interface MediaPlayPauseRequest extends MessageBase {
  type: 'MEDIA_PLAY_PAUSE';
  /** If provided, force state; otherwise toggle. */
  targetState?: 'play' | 'pause' | 'toggle';
}

export interface MediaPlayPauseResponse extends ResponseBase {
  type: 'MEDIA_PLAY_PAUSE_RESPONSE';
  ok: boolean;
  playing?: boolean;
}
```

### 2.4 MEDIA\_SKIP (seek)

```ts
export interface MediaSkipRequest extends MessageBase {
  type: 'MEDIA_SKIP';
  seconds: number; // positive = forward, negative = backward
}

export interface MediaSkipResponse extends ResponseBase {
  type: 'MEDIA_SKIP_RESPONSE';
  ok: boolean;
  positionSeconds?: number; // current time after seek if available
}
```

### 2.5 MEDIA\_PREV\_NEXT

```ts
export interface MediaPrevNextRequest extends MessageBase {
  type: 'MEDIA_PREV_NEXT';
  direction: 'prev' | 'next';
}

export interface MediaPrevNextResponse extends ResponseBase {
  type: 'MEDIA_PREV_NEXT_RESPONSE';
  ok: boolean;
  changedTrack?: boolean; // true if adapter switched playlist item
}
```

### 2.6 HISTORY\_GET\_LAST

```ts
export interface HistoryGetLastRequest extends MessageBase {
  type: 'HISTORY_GET_LAST';
}

export interface HistoryGetLastResponse extends ResponseBase {
  type: 'HISTORY_GET_LAST_RESPONSE';
  ok: boolean;
  item?: HistoryItem;
}

export interface HistoryItem {
  provider: Provider; // 'youtube' | 'vimeo' | 'twitch'
  videoId: string;
  title?: string;
  url: string;
  watchedAt: string; // ISO8601
}
```

### 2.7 HISTORY\_PUSH

```ts
export interface HistoryPushRequest extends MessageBase {
  type: 'HISTORY_PUSH';
  item: HistoryItem; // client trusted only if from adapter
}

export interface HistoryPushResponse extends ResponseBase {
  type: 'HISTORY_PUSH_RESPONSE';
  ok: boolean;
}
```

### 2.8 SUGGEST\_QUERY

```ts
export interface SuggestQueryRequest extends MessageBase {
  type: 'SUGGEST_QUERY';
  /** Extracted text signals from the current page. */
  queryText: string;
  /** Optional context for better ranking. */
  context?: {
    pageUrl?: string;
    pageTitle?: string;
    h1?: string;
    selection?: string;
  };
  /** Stream results as they arrive. */
  stream?: boolean;
}

export interface SuggestQueryResponse extends ResponseBase {
  type: 'SUGGEST_QUERY_RESPONSE';
  ok: boolean;
  results?: SuggestResult[]; // present if not streaming
}

export interface SuggestResult {
  provider: Extract<Provider, 'youtube'>; // currently YouTube, extendable later
  videoId: string;
  title: string;
  url: string;
  channel?: string;
  viewCount?: number;
  publishedAt?: string; // ISO8601
  score?: number;       // internal ranking score
}

/** Streaming chunk */
export interface StreamChunk {
  contractVersion: '1.0';
  type: 'STREAM_CHUNK';
  requestId: string;
  partial: SuggestResult[];
}

/** Stream end */
export interface StreamEnd {
  contractVersion: '1.0';
  type: 'STREAM_END';
  requestId: string;
}
```

---

## 3) Site Adapter Interface (per‑site shim)

Adapters are content‑script modules that normalize control of different video players under a common contract.

```ts
export type Provider = 'youtube' | 'vimeo' | 'twitch';

/** Events surfaced by adapters (broadcast to SW). */
export interface AdapterEventMap {
  'video-attached': { provider: Provider; videoId?: string; title?: string; url?: string };
  'video-detached': { provider: Provider };
  'playback-status': { playing: boolean; positionSeconds?: number; durationSeconds?: number };
  'pip-status': { active: boolean; mode: 'native' | 'overlay' | 'none' };
  'progress': { positionSeconds: number; percent?: number };
}

/**
 * Unified adapter contract. One instance per eligible <video> in the page; the manager selects the active one.
 */
export interface SiteAdapter {
  /** Identify if the current page has a controllable video. */
  detect(): Promise<boolean>;

  /** Provider id (static). */
  readonly provider: Provider;

  /** Return a stable video id if derivable (e.g., YouTube videoId). */
  getVideoIdentity(): Promise<{ videoId?: string; url?: string; title?: string }>;

  /** Playback controls */
  play(): Promise<void>;
  pause(): Promise<void>;
  /** Positive = forward, negative = backward */
  seekBy(seconds: number): Promise<void>;
  /** Next / previous within playlist contexts when available */
  next?(): Promise<boolean>;
  prev?(): Promise<boolean>;

  /** Status queries */
  getStatus(): Promise<{
    playing: boolean;
    positionSeconds?: number;
    durationSeconds?: number;
  }>;

  /** PiP control */
  isPiPSupported(): Promise<boolean>;
  enterPiP(prefer: 'native' | 'overlay'): Promise<'native' | 'overlay'>;
  exitPiP(): Promise<void>;

  /** Event subscription for SW bridge */
  on<E extends keyof AdapterEventMap>(evt: E, handler: (p: AdapterEventMap[E]) => void): void;
  off<E extends keyof AdapterEventMap>(evt: E, handler: (p: AdapterEventMap[E]) => void): void;
}
```

### 3.1 Adapter Manager (content script)

```ts
export interface AdapterManager {
  /** select/instantiate the appropriate adapter for this page */
  attach(): Promise<SiteAdapter | null>;
  detach(): Promise<void>;
  current(): SiteAdapter | null;
}
```

---

## 4) Routing Contracts (SW ↔ Content Script)

```ts
// Send a message; the router enforces type safety at compile time.
export async function sendMessage<TReq extends MessageBase, TRes extends ResponseBase>(
  tabId: number,
  msg: TReq
): Promise<TRes> {
  return chrome.tabs.sendMessage<TRes>(tabId, msg);
}

// Example: play/pause toggle
await sendMessage<MediaPlayPauseRequest, MediaPlayPauseResponse>(tabId, {
  contractVersion: '1.0',
  type: 'MEDIA_PLAY_PAUSE',
  requestId: crypto.randomUUID(),
  source: 'service-worker',
  tabId,
  targetState: 'toggle',
});
```

---

## 5) Validation (optional but recommended)

Use `zod` to validate at boundaries. Keep schemas adjacent to types.

```ts
import { z } from 'zod';

export const zMediaSkipRequest = z.object({
  contractVersion: z.literal('1.0'),
  type: z.literal('MEDIA_SKIP'),
  requestId: z.string().uuid(),
  source: z.enum(['content-script','service-worker','popup','options','overlay']),
  tabId: z.number().optional(),
  seconds: z.number().int(),
});
```

---

## 6) Error Semantics & Fallbacks

* **NO\_VIDEO**: adapter couldn’t find an attached `<video>` → show tip to start a video.
* **PIP\_UNSUPPORTED**: native PiP not supported → offer overlay mode.
* **COMMAND\_INTERCEPTED**: OS/app captured key → persist overlay controls longer and hint settings.
* **BACKEND\_UNAVAILABLE**: degrade Suggest to client heuristic; queue telemetry.

All errors must return a **user-safe** `message` and a machine‑readable `code`.

---

## 7) Telemetry Event Contracts (minimal)

```ts
export type TelemetryEvent =
  | { name: 'command_issued'; data: { type: MessageType; latencyMs?: number } }
  | { name: 'pip_started'; data: { mode: 'native' | 'overlay'; provider?: Provider } }
  | { name: 'error'; data: { code: ErrorCode } };
```

---

## 8) Security & Privacy Notes

* Only the SW may perform privileged actions; content scripts must go through the router.
* History writes accepted only from trusted adapters (origin check + runtime id).
* Suggest requests strip PII and send only text signals required for ranking.
* OAuth tokens (if used) never leave the device unless the user opts into backend sync.

---

## 9) Backward Compatibility Strategy

* New optional fields only add **backward‑compatible** behavior.
* To remove/rename required fields or change semantics, introduce **new message types** (e.g., `PIP_TOGGLE_V2`).
* Maintain a **compat matrix** in tests to ensure older SW/newer content‑script combinations still function for core PiP.

---

## 10) Worked Examples

### Example A — Toggle PiP overlay

```ts
const req: PipToggleRequest = {
  contractVersion: '1.0',
  type: 'PIP_TOGGLE',
  requestId: crypto.randomUUID(),
  source: 'popup',
  mode: 'overlay',
};
```

### Example B — Seek forward 30s

```ts
const req: MediaSkipRequest = {
  contractVersion: '1.0',
  type: 'MEDIA_SKIP',
  requestId: crypto.randomUUID(),
  source: 'service-worker',
  seconds: 30,
};
```

### Example C — Smart Suggest (streaming)

```ts
const req: SuggestQueryRequest = {
  contractVersion: '1.0',
  type: 'SUGGEST_QUERY',
  requestId: crypto.randomUUID(),
  source: 'content-script',
  queryText: 'mount a tv to drywall safely',
  context: { pageUrl: location.href, pageTitle: document.title },
  stream: true,
};
```

---

## 11) Open Extensions (future)

* Add `MEDIA_SET_SPEED`, `MEDIA_MUTE_UNMUTE`.
* Introduce `PIP_AUTO_POLICY_SET` for per‑site blur/restore behavior.
* Extend `Provider` union to include Dailymotion, Wistia.
