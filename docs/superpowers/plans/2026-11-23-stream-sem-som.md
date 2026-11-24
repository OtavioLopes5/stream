# Stream de tela sem som — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** A single static page that streams the PC screen (video only, 30 fps) to a viewer over the internet, with no audio anywhere.

**Architecture:** One `index.html`. Without a query string it is the host (captures the screen with `getDisplayMedia`). With `?v=<id>` it is the viewer. PeerJS handles signaling through its free public cloud server. The viewer opens a data connection to the host, and the host calls back with the screen stream.

**Tech Stack:** Plain HTML/CSS/JS, PeerJS 1.5.4 UMD (`https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js`). Local test server: `python -m http.server 8080`. Hosting: GitHub Pages.

**Spec:** `DESIGN.md`

## Global Constraints

- Zero audio: `getDisplayMedia({video:{frameRate:{ideal:30,max:30}}, audio:false})`. The viewer's `<video>` is `autoplay muted playsinline`, has no `controls`, and drops any audio track it receives.
- 30 fps: the capture asks for 30 fps, and the video track gets `contentHint = 'motion'` (Chrome then keeps the framerate and lowers resolution).
- The host ID is fixed per browser: created once with `crypto.randomUUID()` and stored in `localStorage['stream-id']`. Every storage access is wrapped in try/catch.
- Viewer link format: `location.origin + location.pathname + '?v=' + id`.
- UI copy in Portuguese (pt-BR).
- No build step and no dependency besides the PeerJS script.

## Review Focus

1. The viewer opens the link before the host starts → shows "Aguardando transmissão…" and connects by itself when the host comes online (retry every 3 s).
2. The host stops sharing from Chrome's bar, then clicks "Transmitir" again → connected viewers get the new stream without reloading.
3. The host reloads its page → same ID; if the PeerJS server still holds it (`unavailable-id`), retry every 3 s until it frees up.
4. The viewer receives a stream with an audio track (should never happen) → the track is removed before it reaches the `<video>`.
5. The PeerJS signaling socket drops mid-session (`disconnected`) → `peer.reconnect()`.

---

### Task 1: Streamer + viewer page

**Files:**
- Create: `index.html`
- Create: `.claude/launch.json` (local preview server: python http.server on 8080)

**Interfaces:**
- Produces: `index.html` served as a static file. Host mode at `/`, viewer mode at `/?v=<id>`.

- [ ] **Step 1: Host mode**
  - Peer is `new Peer(id)`; show the viewer link with a "Copiar link" button.
  - "Transmitir" button → `getDisplayMedia` per Global Constraints.
  - On `peer.on('connection', conn)`, keep `conn` in a `Set`. If the stream is live, `peer.call(conn.peer, stream)`. On close, drop it from the set.
  - When the stream starts, call every open connection. When the track ends, close all calls and show the button again.
  - Show how many people are watching (open connections).
  - `unavailable-id` → status message and retry in 3 s. `disconnected` → `peer.reconnect()`.
- [ ] **Step 2: Viewer mode**
  - Peer is `new Peer()`; on `open`, `peer.connect(hostId)`.
  - `peer-unavailable` or the connection closing → status "Aguardando transmissão…", retry in 3 s.
  - `peer.on('call')` → `call.answer()` with no stream. On `stream`, remove audio tracks, set `video.srcObject`, hide the status. On call close → status "Transmissão pausada".
  - Fullscreen button only when `requestFullscreen` exists.
- [ ] **Step 3: End-to-end check in the browser (localhost:8080)**
  - Viewer tab first (`/?v=<host id>`): expect "Aguardando transmissão…".
  - Host tab: override `navigator.mediaDevices.getDisplayMedia` to return an animated `canvas.captureStream(30)`, then click "Transmitir".
  - Expected in the viewer, within 15 s: `video.videoWidth > 0`, `video.muted === true`, `video.srcObject.getAudioTracks().length === 0`, `video.controls === false`.
  - Host: watcher count shows 1, and the capture options seen by the override include `audio:false` and `frameRate` 30.
  - Stop the fake track (`track.stop()` + dispatch `ended`) → viewer shows "Transmissão pausada"; click "Transmitir" again → viewer gets video again (Review Focus 2).
- [ ] **Step 4: Commit** — `git init`, then commit `index.html`, `DESIGN.md`, `docs/`, `.claude/launch.json`.

### Task 2: GitHub Pages

**Files:** none new (repo push only).

- [ ] **Step 1:** The user creates a public repo (e.g. `stream`) on github.com.
- [ ] **Step 2:** `git remote add origin <url>` and `git push -u origin main` (Git Credential Manager opens the user's own GitHub login).
- [ ] **Step 3:** Repo Settings → Pages → Deploy from branch `main` / root.
- [ ] **Step 4: Verify** — open `https://<user>.github.io/stream/`: the page loads and shows a link. The real-device check (a phone playing music while watching) is done by the user.
