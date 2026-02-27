# Saturn Feasibility Breakdown
**Date:** February 27, 2026
**Time:** Current session
---

## User

❯ we need to break down Saturn's feasability into concrete requirements. what is the evidence that saturn works? what things (like ARP) prevent Saturn from working? can people use all types of ai (chat, agentic coding, photo analysis) with saturn?

## Claude

Documenting.

Let me start by reading the knowledge graph to understand Saturn's architecture and then analyze feasibility.

```tool
[Tool: Read] moons/graph.json
```

```tool
[Tool: Read] moons/claims/claim-1.md, claim-2.md, claim-3.md, evidence-matrix.md, concepts/beacons.md, openai-api.md, ephemeral-keys.md
```

```tool
[Tool: Read] moons/code/python_package_claims.md, saturn-router.md, ai-sdk-provider-saturn.md, saturn-mcp.md, vlc-extension.md, owui-saturn.md, concepts/mdns.md
```

```tool
[Tool: Read] moons/concepts/voice-typing.md
```

Here's the feasibility breakdown across your three questions:

---

## 1. Evidence That Saturn Works

**Existence proof across 5 independent implementations:**

| Component | Language | mDNS Library | What it proves |
|-----------|----------|-------------|----------------|
| Core Python | Python | `zeroconf` | Server + client discovery works. One command (`saturn run ollama`) starts broadcasting |
| saturn-router | Rust | `mdns-sd` | Runs on a $20 GL.iNet router with 128MB RAM alongside DHCP/DNS. Saturn as literal network infrastructure |
| ai-sdk-provider | TypeScript | `multicast-dns` | Cross-language interop. `import { saturn }` — zero API keys, zero env vars |
| owui-saturn | Python | `Zeroconf` | Real web app (Open WebUI) discovers Saturn services via native plugin system |
| saturn-mcp | Python | `zeroconf` (via core) | AI coding assistants (Claude Code, Cursor) discover services through MCP |
| VLC extension | Lua/Python | `dns-sd` subprocess | Consumer app. Copy a folder, click a menu item, talk to AI about your movie |

All six consume the identical `_saturn._tcp.local.` service type. Five different mDNS libraries in three languages interoperate without any bridging layer. This is the strongest evidence: **the protocol works regardless of implementation language or mDNS library**.

**The protocol itself is thin and proven:**
- PTR record → service browsing
- SRV record → host + port
- TXT record → `api_base`, `api_type`, `priority`, `ephemeral_key`, `rotation_interval`
- This is exactly how printer discovery has worked for 20+ years (Guttman 2001, Kim & Reeves 2020)

**The security model works within its constraints:**
- Ephemeral keys: 10-min lifetime, 5-min rotation, auto-delete on shutdown
- saturn-router implements full lifecycle: generate → broadcast → rotate → delete
- ai-sdk-provider auto-refreshes on 401 (expired key)
- Spending limits cap financial exposure

---

## 2. What Prevents Saturn From Working

### Hard blockers (Saturn cannot work at all)

**AP isolation / client isolation** — The biggest. Enterprise and institutional WiFi networks (eduroam, UCSC-guest) block multicast traffic between clients. You confirmed this in a Jan 2026 experiment. mDNS queries go to `224.0.0.251:5353` — if the AP doesn't forward multicast between clients, discovery fails silently. This directly undermines the campus deployment scenario that motivates the thesis.

**VLANs without multicast bridging** — If the admin's Saturn beacon is on VLAN 10 and students are on VLAN 20, mDNS doesn't cross the boundary. mDNS is link-local by design (RFC 6762).

**Firewall rules blocking UDP 5353** — Some corporate firewalls block mDNS outright.

### Soft blockers (Saturn works but with degraded experience)

**No service identity verification** — Any device can broadcast `_saturn._tcp.local.`. A rogue service on a coffee shop WiFi appears alongside legitimate ones. Health checks only verify HTTP 200 on `/v1/health` — they don't validate identity or authenticity. This is the Kaiser & Waldvogel 2014a threat model.

**Plaintext TXT records** — Ephemeral keys are broadcast in cleartext. Anyone on the same network segment can passively read them and use the key until it expires (10-min window). HTTP traffic to discovered services is also unencrypted.

**Router RAM-only deployment** — On the GL.iNet router, the binary runs from `/tmp/saturn` because flash is too small. Lost on reboot. Institutional deployments need persistent storage or auto-download from GitHub (which the init script supports, but requires internet).

**ARP is NOT a blocker** — ARP operates at Layer 2 for IP-to-MAC resolution. It doesn't interfere with mDNS. The confusion might be with ARP spoofing as an *attack vector* (man-in-the-middle), but that's a general network security issue, not Saturn-specific. Saturn's actual network-layer blocker is multicast filtering, not ARP.

### Architectural limits

**Single network segment** — mDNS is link-local. Saturn doesn't work across subnets, across the internet, or between a home and office network. This is by design (the security model depends on the network trust boundary), but it limits reach.

**OpenAI API as the only endpoint standard** — Saturn only advertises and routes to `/v1/chat/completions`, `/v1/models`, `/v1/health`. This constrains what AI modalities Saturn can serve (see next section).

---

## 3. Can People Use All Types of AI?

### What works today

**Chat completions (text-in, text-out)** — Fully supported. This is Saturn's core. All six components demonstrate it. Any backend exposing the OpenAI chat completions API works: Ollama, OpenRouter, DeepInfra, LiteLLM, vLLM.

**Agentic coding** — Works through saturn-mcp. Claude Code, Cursor, Windsurf can discover Saturn services and use them for code generation, explanation, etc. But the underlying API is still chat completions — the "agentic" part is the assistant's tool-use loop, not a Saturn feature.

**Vision / photo analysis** — **Partially works, not advertised.** The OpenAI chat completions API supports multimodal inputs (images as base64 or URLs in the messages array). If the backend model supports vision (e.g., `llava` on Ollama, `gpt-4o` via OpenRouter), the request passes through Saturn's proxy transparently. **But Saturn's TXT records don't advertise capabilities like "vision" or "multimodal."** A client has no way to discover which services support image input without trying. The `features` field exists conceptually but isn't standardized for modality advertisement.

### What doesn't work today

**Image generation** — The OpenAI API uses `/v1/images/generations`. Saturn doesn't advertise or route to this endpoint. A Saturn service could expose it, but clients wouldn't discover it through the current protocol.

**Audio / speech-to-text / TTS** — `/v1/audio/transcriptions`, `/v1/audio/speech`. Same situation — these endpoints exist in the OpenAI API but aren't part of Saturn's endpoint spec (`/v1/health`, `/v1/models`, `/v1/chat/completions`).

**Embeddings** — `/v1/embeddings`. Not part of Saturn's current spec.

**Real-time / streaming beyond SSE** — WebSocket-based real-time APIs (OpenAI's Realtime API) don't fit Saturn's HTTP-based discovery model. Saturn supports SSE streaming for chat completions, but not persistent WebSocket connections.

### The gap

Saturn's feasibility is currently scoped to **text chat completions with optional vision passthrough**. This covers the most common AI use case (chatting with an LLM) and the most valuable agentic use case (coding assistants), but it doesn't cover the full spectrum of AI modalities. The protocol could be extended — add `capabilities` to TXT records, add more endpoint types to the spec — but today it's a text-chat protocol that happens to pass through multimodal requests if the backend supports them.

This is worth being honest about in the thesis. The claim is that zero-config AI provisioning is *feasible*, not that Saturn provisions every possible AI modality. The text-chat scope covers the use cases that matter most for the equity argument (students accessing LLMs for writing, coding, learning).

---

Want me to create a structured feasibility requirements document in the moons knowledge base from this analysis?

## User

the only thing i dont like about what you said is "  OpenAI API as the only endpoint standard — Saturn only advertises and routes to
  /v1/chat/completions, /v1/models, /v1/health. This constrains what AI modalities Saturn can serve
   (see next section)." This is not necessarily true as ollama and the scoped jwt tokens from deepinfra also worked. I think the rest of the information you gave me should be preserved within moons. use the AskUserQuestion tool to proactively talk to me and ask questions about the design of feasability in moons as you create it

## Claude

You're right — I overstated that constraint. Saturn discovers services and hands the client a URL + credentials via TXT records (`api_base`, `api_type`, `ephemeral_key`). What the client does with that endpoint is between the client and the backend. Ollama exposes `/api/generate`, `/api/embeddings`, its own chat endpoint — all reachable once Saturn provides the address. DeepInfra's scoped JWTs work through the same mechanism. Saturn's discovery layer is API-agnostic; the `api_type` field lets clients know what they're talking to.

Let me ask some design questions before building this out in moons.

```tool
[Tool: AskUserQuestion] Structure: single file vs split. AP isolation: hard limitation vs router solves it. Scope: current state vs roadmap.
```

User answers: Split by topic, Hard limitation, Current state only.

```tool
[Tool: AskUserQuestion] Location: claims/feasibility/ vs top-level. API evidence: specific vs general.
```

User answers: moons/claims/feasibility/, Specific evidence.

```tool
[Tool: Write] moons/claims/feasibility/evidence.md — 6 implementations, API-agnostic discovery, protocol layer, security model, hardware deployment
```

```tool
[Tool: Write] moons/claims/feasibility/blockers.md — AP isolation, VLANs, firewalls, plaintext TXT, rogue services, RAM-only deployment
```

```tool
[Tool: Write] moons/claims/feasibility/modalities.md — chat completions, agentic coding, vision passthrough, embeddings passthrough, what doesn't work
```

```tool
[Tool: Edit] moons/graph.json — added feasibility-evidence, feasibility-blockers, feasibility-modalities nodes with edges
```

```tool
[Tool: Bash] python3 moon_visualization/sync.py — Synced 72 nodes into viewer.html
```

```tool
[Tool: AskUserQuestion] Backends: three cover it? Working environments: add to blockers?
```

User answers: Those three cover it. Yes, add working environments.

```tool
[Tool: Edit] moons/claims/feasibility/blockers.md — added "Where Saturn works reliably" section (home WiFi, small office, ad-hoc, lab/classroom, wired Ethernet)
```

Created three files in `moons/claims/feasibility/`:
- **evidence.md** — 6 implementations across 3 languages/5 mDNS libraries, API-agnostic discovery (Ollama native, DeepInfra JWT, OpenRouter ephemeral), thin protocol layer, security model, hardware deployment
- **blockers.md** — working environments listed first, then hard blockers (AP isolation confirmed Jan 2026, VLANs, firewalls), soft blockers (rogue services, plaintext TXT, RAM-only router), ARP clarified as non-blocker, architectural limits (link-local scope, discovery latency)
- **modalities.md** — fully supported (chat, agentic coding), passthrough (vision, embeddings, audio), not supported (image gen, WebSocket realtime, TTS)

Updated graph.json with three new nodes and edges to claims, concepts, and components. Synced viewer.html (72 nodes).

## User

I am done documenting.
