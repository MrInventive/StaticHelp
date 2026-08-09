# Help — Static Video Chat

A fully static, serverless video chat tool. No backend, no database, no Node process to run — just HTML/JS files served from anywhere (GitHub Pages, a USB stick, whatever). Deploy it and it works.

It uses [PeerJS](https://peerjs.com/) and its free public cloud broker to handle the WebRTC handshake between two browsers. Once connected, video/audio/chat flow directly peer-to-peer — the broker's job ends the moment the call connects.

## Two lanes

### 1. Public support line (`index.html` + `operator.html`)

One fixed operator address. Customers call in; the operator answers whoever's first.

- **`operator.html`** — the operator opens this and leaves it open. Registers a fixed PeerJS ID (`help-operator-live`) so customers always have a known address to dial. Only one call at a time — a second caller while the operator's busy gets rejected immediately.
- **`index.html`** — the customer side. Enters a name and a note, then requests help.

**False queue (no server, no real queue):** if the operator's busy, the customer is asked "wait in queue?" If they say yes, their browser tab automatically re-dials the operator every 8 seconds until it connects — no server holds their place, the open tab *is* the queue slot. Closing the tab drops them silently. Multiple people waiting all retry independently; there's no fairness or ordering, first one to land the handshake when the operator frees up wins.

### 2. Private call by code (`code.html` + `join.html`)

For a one-on-one call with someone specific, not the public line.

- **`code.html`** — open this to host. Generates a random 4-character code and registers your PeerJS ID as `help-` + that code. Share the code out of band (text, voice, whatever).
- **`join.html`** — the guest opens this, types the code in, and it dials `help-XXXX` directly.

**Important:** the code is a short public address, not a password. PeerJS's broker will connect anyone who dials the right ID — there's no authentication layer. With only ~1.7 million possible 4-character codes, brute-force guessing is impractical for a call that's live a few minutes, but don't treat it as secure. For anything sensitive, lengthen the code or add a passphrase check over the data channel before answering.

## Files

| File | Role |
|---|---|
| `index.html` | Customer — requests help, waits in false queue if busy |
| `operator.html` | Operator — fixed address, answers one call at a time |
| `code.html` | Host — generates a private call code |
| `join.html` | Guest — joins a private call using a code |
| `style.css` | Shared instrument-panel styling for all four pages |

## Requirements

- A modern browser with camera/mic permissions (getUserMedia)
- Two devices/tabs open at once to test a call
- Internet access to reach the PeerJS public broker (`0.peerjs.com`) — the only "server" involved, and it's not one you run

## Known limitations

- **No real queue ordering.** The false queue is per-customer retry, not a server-tracked list. Fine for one operator and light traffic; unfair under load.
- **NAT traversal isn't guaranteed.** PeerJS uses public STUN by default. If either side is behind strict NAT/CGNAT, the call may fail to connect directly — a TURN server would be needed to relay media, which isn't included here.
- **No persistence.** No chat history, no call logs, nothing saved anywhere. Close the tab, it's gone.
- **If you need a real queue, multiple operators, or saved history** — that requires actual server-side state. A small Node/Socket.io signaling server is the natural next step if this outgrows the static version.
