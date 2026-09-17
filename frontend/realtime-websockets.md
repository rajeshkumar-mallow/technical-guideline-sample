# Realtime & WebSockets

**Read this first.** A WebSocket/SSE connection is long-lived state the browser has to babysit through network drops, tab backgrounding, and server restarts. Most realtime bugs are reconnection bugs, not protocol bugs — design for disconnection from the start.

**Applies to:** any frontend consuming WebSocket or Server-Sent Events (SSE) streams. React projects wrap these patterns in a hook or provider — see [`react/hooks.md`](react/hooks.md).

---

## Mandatory

### Connection lifecycle
- Wrap the raw `WebSocket`/`EventSource` in a small client that exposes connection state (`connecting` / `open` / `closed` / `reconnecting`) — components should render against that state, not against raw socket events.
- Close sockets explicitly on unmount/navigation away from the feature that needs them. Leaked open connections accumulate across route changes in an SPA.

### Reconnection with backoff
- Auto-reconnect on unexpected close, with **exponential backoff + jitter**, capped at a sane max interval (e.g. 30s) — never a fixed-interval retry loop, which thundering-herds the server on a shared outage.
- Resubscribe to any per-connection state (rooms/channels) after reconnect — a fresh socket has no memory of what the old one was subscribed to.

```ts
function backoffDelay(attempt: number): number {
  const base = Math.min(30_000, 1000 * 2 ** attempt);
  return base / 2 + Math.random() * (base / 2); // jitter
}
```

### Authenticating the handshake
- **Do not put auth tokens in the WebSocket URL query string** — query strings land in server access logs, browser history, and proxy logs. Treat a token-in-URL the same as leaking a password.
- Prefer **cookie-based auth** for the handshake (the httpOnly session cookie set per [Auth](auth.md) is sent automatically on the upgrade request) validated server-side before the connection is accepted.
- If a token must be sent explicitly (cross-origin socket to a service that can't read your cookie), send it as the **first message after connection** over the already-established (WSS) encrypted channel, not in the URL.

### Message schema versioning
- Every message carries a `type` and a schema `version` (or is versioned at the connection/subscription level) so the client can reject or gracefully handle a shape it doesn't understand instead of crashing.
- Validate incoming messages against a schema (zod or similar) before acting on them — treat the socket payload as untrusted input, same as any HTTP response.

```ts
const MessageSchema = z.discriminatedUnion("type", [
  z.object({ type: z.literal("order.updated"), version: z.literal(1), payload: OrderSchema }),
  z.object({ type: z.literal("order.updated"), version: z.literal(2), payload: OrderV2Schema }),
]);
```

### Always use WSS
- **`wss://` only** in any environment that isn't local dev — plaintext `ws://` exposes every message (including the auth handshake) to network-level interception.

## Recommended (opt-in)

- **Heartbeat/ping-pong** at the application layer to detect a half-open connection faster than the OS-level TCP timeout.
- Fall back to **SSE** (or long-polling) automatically when WebSocket connection fails repeatedly, for environments with restrictive proxies — record the decision via ADR since it adds client complexity.
- Pause/resume subscriptions on tab visibility change (`document.visibilitychange`) to reduce unnecessary server load for backgrounded tabs.

## Anti-Patterns (do not ship)

- Auth token in the WebSocket URL query string.
- Fixed-interval reconnect loop with no backoff (hammers the server during an outage).
- Trusting incoming socket messages without schema validation.
- Plaintext `ws://` outside local development.
- Leaving sockets open after the component/feature using them unmounts.

## Quick Reference

```
✓ Connection-state-driven client wrapper, not raw socket events in components
✓ Exponential backoff + jitter on reconnect, capped max interval
✓ Resubscribe after reconnect
✓ Cookie-based (or post-connect message) auth — never token in URL
✓ wss:// always outside local dev
✓ Validate every incoming message against a versioned schema
✗ Token in query string
✗ Fixed-interval reconnect
✗ Unvalidated message payloads
```

---
*Section version: 0.1 — initial draft*
