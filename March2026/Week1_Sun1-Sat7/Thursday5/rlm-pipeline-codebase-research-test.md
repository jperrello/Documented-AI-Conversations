# RLM Pipeline Test: Codebase Research via Saturn.md
**Date:** Thursday, March 05, 2026
**Time:** 2:35 PM
---

## User

/document /rlm

## Claude

Documenting.

The RLM skill is loaded. I need two things to proceed:

1. **Context file path** — the large file to process
2. **Query** — what you want to extract or answer from it

What are they?

## User

1. @kunai/Saturn.md 2. How do I implement saturn into a lua script so i can run it and an ai says hello with the time of day and a funny joke

## Claude

```tool
[Tool: Bash] python3 rlm_repl.py init /Users/jperr/Documents/omlx-saturn/kunai/Saturn.md — initialized 1,854,560 chars
```

```tool
[Tool: Bash] python3 rlm_repl.py exec — peek beginning/end of file, grep for lua/chat/completions/v1/ terms
```

```tool
[Tool: Agent] rlm-subcall x3 — extract Lua VLC extension code, server API details, README overview (parallel)
```

```tool
[Tool: Bash] python3 rlm_repl.py exec — peek README.md, send_message, http_get, http_post implementations
```

Here's how to implement Saturn into a Lua script:

## What Saturn Is

Saturn is a zero-config AI service discovery protocol. Services announce themselves via mDNS as `_saturn._tcp.local.` and expose **OpenAI-compatible endpoints**: `/v1/health`, `/v1/models`, `/v1/chat/completions`. Any language that can do HTTP can use it.

## Lua Implementation

Saturn's own VLC extensions (`saturn_chat.lua`) show the pattern. For a standalone Lua script, you need:

1. **Discover Saturn** — find a running Saturn service on your network
2. **POST to `/v1/chat/completions`** — standard OpenAI-compatible request
3. **Parse the JSON response**

Here's a complete script:

```lua
-- saturn_hello.lua
-- Requires: lua-socket, lua-cjson (or dkjson)

local http = require("socket.http")
local ltn12 = require("ltn12")
local json = require("cjson")  -- or dkjson

-- Step 1: Connect to Saturn
local SATURN_URL = "http://127.0.0.1:8000"

-- Step 2: Pick a model (query /v1/models)
local function models()
  local body = {}
  http.request{
    url = SATURN_URL .. "/v1/models",
    sink = ltn12.sink.table(body)
  }
  return json.decode(table.concat(body))
end

-- Step 3: Chat completions
local function chat(model, messages)
  local payload = json.encode({
    model = model,
    messages = messages,
    max_tokens = 300
  })

  local body = {}
  http.request{
    url = SATURN_URL .. "/v1/chat/completions",
    method = "POST",
    headers = {
      ["Content-Type"] = "application/json",
      ["Content-Length"] = #payload
    },
    source = ltn12.source.string(payload),
    sink = ltn12.sink.table(body)
  }

  local response = json.decode(table.concat(body))
  return response.choices[1].message.content
end

-- Step 4: Build the greeting
local hour = tonumber(os.date("%H"))
local greeting = "morning"
if hour >= 12 and hour < 17 then greeting = "afternoon"
elseif hour >= 17 then greeting = "evening" end

-- Get available models and pick the first one
local model_list = models()
local model = model_list.data[1].id

-- Ask the AI to say hello with a joke
local reply = chat(model, {
  {
    role = "system",
    content = "You are a friendly, funny AI. Greet the user with the time of day and tell a short, original joke."
  },
  {
    role = "user",
    content = "Good " .. greeting .. "! Say hello and tell me a funny joke."
  }
})

print(reply)
```

## Prerequisites

1. **Saturn running on your network:**
   ```bash
   pip install saturn-ai
   saturn openrouter    # or saturn ollama, etc.
   ```

2. **Lua dependencies:**
   ```bash
   luarocks install luasocket
   luarocks install lua-cjson   # or: luarocks install dkjson
   ```

3. **Run it:**
   ```bash
   lua saturn_hello.lua
   ```

## Key Details

| Detail | Value |
|--------|-------|
| Protocol | `_saturn._tcp.local.` via mDNS |
| API format | OpenAI-compatible |
| Health check | `GET /v1/health` |
| Models list | `GET /v1/models` |
| Chat | `POST /v1/chat/completions` |
| Request body | `{model, messages: [{role, content}], max_tokens}` |
| Response path | `response.choices[1].message.content` |

The script auto-detects time of day, queries Saturn for available models, and asks the AI for a greeting + joke. No API keys needed if you're on the same network as a Saturn service.

## User

you know why I made you do this? i wanted to test the pipeline if i could give you a specific codebase and tell you how to fix it. i want to use this to research a codebase so that when i am working inside of it, I can ask questions about the codebase md file rather than telling you to manually search the codebase

## Claude

That makes sense. Use the RLM pipeline as a pre-indexed knowledge layer — dump a codebase into a `.md` file, chunk it once, then query against it repeatedly without Claude having to re-explore the source tree each time.

Faster than live codebase search, and you control exactly what context is included.

Want me to note this pattern in memory so future sessions know that's how you intend to use RLM + codebase `.md` files?

## User

I am done documenting.
