# HVAC Static — Live Support Line

Single-operator voice/video call and text chat, running entirely as static files on GitHub Pages. Same pattern as GamePug — no server to host or maintain.

## Files
- `operator.html` — your console. Claims a fixed address the moment it loads and waits for a customer to call in.
- `index.html` — the customer-facing page. Requests camera/mic access, dials the operator, and connects.
- `style.css` — shared control-panel styling for both pages.

## How it works
There's no signaling server of your own. Both pages use **PeerJS's free public cloud broker** to handle the WebRTC handshake — the same role a custom Node/Socket.io server would play, just hosted by PeerJS instead of you. Once the connection is made, video/audio/chat all flow peer-to-peer directly between the two browsers.

`operator.html` registers a fixed Peer ID:
```js
const OPERATOR_ID = 'hvactech-operator-live';
```
`index.html` dials that exact ID. This ID is just an address, not a password — anyone who knows it (or finds it in the page source) can call in. Change it to something less guessable if that matters for your use case, and keep both files' `OPERATOR_ID` in sync if you do.

## Deploying
1. Push `index.html`, `operator.html`, and `style.css` into a folder in your GitHub Pages repo (e.g. `HVACTech/`).
2. Enable GitHub Pages on that repo if it isn't already.
3. Customer link: `https://<yourusername>.github.io/<folder>/index.html`
   Operator link: `https://<yourusername>.github.io/<folder>/operator.html`

## Testing before you rely on it
- Open both pages in separate windows locally first and confirm video, audio, and chat all work.
- Test from an actual cell connection, not just home wifi. This setup only ships a public STUN server by default — most connections traverse fine, but some carrier-grade NAT setups won't. If calls consistently fail to connect only on cellular, that's the sign you need a TURN server added to the `iceServers` list in both files' PeerJS config.
- Keep `operator.html` open and the device awake to stay reachable — closing the tab drops your line.

## Known limitations (by design, for a single operator)
- **No queue.** If you're already on a call, the next customer gets a "no answer" message after ~20 seconds rather than being held in line.
- **No call history or recording.** Chat log lives only in the browser tab for that session.
- **No authentication.** The operator ID is an address, not a login — treat the link as something you control distribution of.

## If you outgrow this version
A real queue, multiple operators, or persistent chat history all require a small server again (Render, Railway, or similar) — the version before this one (`HVACTECH`, using Socket.io) is the starting point for that if the need comes up.
