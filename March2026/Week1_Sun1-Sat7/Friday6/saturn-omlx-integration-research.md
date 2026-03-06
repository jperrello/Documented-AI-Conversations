# Saturn omlx Integration Research
**Date:** Friday, March 6, 2026
**Time:** 09:00 PST
---

## User
/bite I want to integrate Saturn into omlx. Theres two ways we can do this, collect ai services on the network with saturn and use them inside omlx, or we can announce devices running models on omlx and offer their services. I think we should do both

## Claude
Documenting. Now running the `/bite` skill against the `omlx` and `Saturn` chomps.

Research complete. Output written to `chomp/bites/saturn-omlx-integration/research.md`.

```tool
[Tool: Read] chomp/bits/omlx/surface.md, chomp/bits/omlx/patterns.md, chomp/bits/Saturn/surface.md, chomp/bits/Saturn/patterns.md, chomp/bits/omlx/deps.md, chomp/bits/Saturn/deps.md
```

```tool
[Tool: Grep] SaturnAdvertiser|register|unregister|announce in chomp/Saturn.md
```

```tool
[Tool: Grep] EnginePool|engine_pool|load_model|discover_models in chomp/omlx.md
```

```tool
[Tool: Grep] class ServerState, lifespan, SaturnDiscovery, ServerConfig across both chomps
```

```tool
[Tool: Write] chomp/bites/saturn-omlx-integration/research.md — full integration research output
```

**Summary of findings:**

**Direction 1 — Saturn → omlx (consume network services):**
- Use `SaturnDiscovery` to find AI services on the LAN during omlx lifespan startup
- Merge discovered models into `/v1/models` response alongside local MLX models
- Proxy requests for network models through to remote Saturn endpoints via SSE passthrough
- All Saturn services already speak OpenAI-compatible API, so the proxy adapter is thin

**Direction 2 — omlx → Saturn (announce local models):**
- Use `SaturnAdvertiser` to announce omlx's loaded models via mDNS
- omlx already exposes `/v1/models`, `/v1/chat/completions`, `/v1/health` — exactly what Saturn expects
- Hook into model load/unload events to keep advertisement current
- Requires host bound to `0.0.0.0` (not localhost) for network visibility

**Both directions** share the `saturn` Python dependency (optional), toggle independently via settings, and integrate at the FastAPI lifespan + route level.

## User
/bite with @chomp/bites/saturn-omlx-integration/research.md how would we handle multi turn conversations with both directions? how about streaming output and tool calls. THe front end would also need to be updated wit saturn somewhere (despite being zeroconf)

## Claude
Research complete. Updated `chomp/bites/saturn-omlx-integration/research.md` with 4 new sections and created `surface-map.md`.

```tool
[Tool: Grep] stream_chat_completion, SSEFormatter, sendMessage, proxy_sse across both chomps
```

```tool
[Tool: Read] omlx/server.py lines 47147-47345 — full stream_chat_completion() with tool call buffering
```

```tool
[Tool: Read] omlx/admin chat.js lines 14388-14487 — Alpine.js streaming chat UI
```

```tool
[Tool: Agent x7] RLM subcalls against omlx chunks (0002, 0009, 0013) and Saturn chunks (0001, 0007, 0008) for streaming, tool calls, dashboard, proxy SSE
```

**Multi-turn conversations:** Both directions are stateless — client sends full `messages` array each time. Proxying multi-turn to Saturn is trivial: forward the entire body. Saturn's Pipe class already does `{**body, 'model': model_id}`. No session state to synchronize.

**Streaming:** Both sides use identical `data: {OpenAI chunk json}\n\n` SSE format. The proxy doesn't parse or reformat — just pipes bytes through. omlx's `_with_sse_keepalive()` wrapper can wrap the proxy stream for timeout protection.

**Tool calls:** omlx buffers generation when `tools` are present, then parses tool calls from accumulated text post-hoc. For proxied Saturn models, this is irrelevant — the remote service handles parsing. omlx just passes through the `delta.tool_calls` chunks and `finish_reason: "tool_calls"`. Multi-turn tool use works because the client manages tool execution and sends results back in subsequent requests.

**Frontend:** Dashboard needs a "Network" settings section (enable/disable, priority, service name), a "Network Models" section in the models tab showing discovered services, and a badge on network models in the chat dropdown. The chat UI itself needs zero changes — streaming works identically for local and proxied models.

**Key insight:** The integration is simpler than expected because both systems are stateless, speak the same SSE protocol, and handle tool calls on the generation side. The proxy adapter is a thin SSE passthrough.

## User
/bite what about beacons? have we considered the steps necessary for rotating ephemeral keys? seamless failover?

## Claude
Research output written to `chomp/bites/beacons-keys-failover/research.md` and `surface-map.md`.

```tool
[Tool: Grep] BeaconAdvertiser, CredentialManager, rotation_loop, ephemeral_key, waitForKeyRefresh across Saturn.md
```

```tool
[Tool: Read] Saturn.md lines 38135-38315 — CredentialManager + BeaconAdvertiser + rotation_loop
```

```tool
[Tool: Read] Saturn.md lines 3608-3637 — waitForKeyRefresh() mDNS re-query + polling
```

```tool
[Tool: Agent x3] RLM subcalls against Saturn chunks (0003, 0004, 0005) for beacons, credentials, failover
```

**Beacons are fundamentally different from proxy services.** No HTTP server — the beacon only announces an `ephemeral_key` via mDNS. Clients call the upstream API (DeepInfra, OpenRouter) directly with that key. omlx needs two code paths: proxy-through for network services, direct-call for beacons.

**Ephemeral key rotation:** `CredentialManager` creates keys with 10-min TTL, rotates every 5 min (5-min overlap where both keys are valid). `BeaconAdvertiser.re_register()` unregisters/registers to update the immutable mDNS TXT record. `SaturnDiscovery` listener auto-updates cached keys on re-registration.

**401 handling:** On auth failure, wait briefly, re-read key from discovery cache (beacon may have just rotated), retry once. If still 401, surface the error. Streaming key expiry is unlikely due to the 5-min overlap window.

**Seamless failover:** Priority-sorted services, circuit breaker per service (closed → open after 3 failures → half-open after 30s). No mid-stream failover after content has been emitted to the client — same limitation as Saturn TS provider.

**Key gap from previous research:** We assumed all Saturn services are proxies. Beacons require a `SaturnRouter` in omlx that branches on `service.is_beacon` to route requests correctly.

## User
I am done documenting
