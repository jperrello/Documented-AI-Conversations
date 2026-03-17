# Unified Personal Knowledge Graph — Vision & Research
**Date:** Tuesday, March 17, 2026
**Time:** 3:54 PM PDT
---

## User

spawn research subagents to scower this moons codebase, steve yegge's beads: https://github.com/steveyegge/beads and google's workspace cli: https://github.com/googleworkspace/cli . I have a vision to take this moons skill and directory to a whole new level. currently we are just a slash command. but imagine if we turned moons into a cli, and a perfect harmony between beads, moons, and google's workspace cli. let me explain this to you but I want you to tell me if I am misunderstanding ANY of the projects or if I am reinventing a system that already exists. Do not praise my idea is the beginning of something much greater and isnt perfect yet. here is my vision:
beads serves as a memory database for agents. it exists so agents do not forget the tasks they have been working on in between sessions. moons follows a similar approach, but it is more for information retrieval when i mention something. i typically say something like "what was my main argument for doing x? check moons" and you had a graph to trace down my original plan markdown and jump to related things about it because you have all my transcripts. Moons is also universal for all documents. I could throw random notes in, powerpoints, academic articles, anything. it all stayed in moons as the humans content. so we have beads the agent memory, and moons a massive knowledgebase of the human's plans and thinking process for a project. Now we have google's cli so an agent can connect to my notes that I have shared on google. it is a whole ecosystem. The thing is I dont know what the ecosystem is for.
We need a user story, where is this applicable? I personally want a way to give an agent access to any of my notes related to any question i have about anything because it has memories and a memory trail. But what is the audience for this. How can this help me apply for jobs? I can have moons of all my previous resumes, the projects i have done, and maybe old work experience. How will this make me a better harness engineer? I have now upgraded typically memory agent problems by connecting users to enviroments they are comfortable using (google docs) and they can talk to any agent as if they have seen all their notes anyways?
What can you find from research? What am i missing? what is the glue that makes these pieces whole. Is there more than glue? Tape?

## Claude

Documenting. Spawning research agents now.

Four research agents deployed:
1. **beads-research** — deep-diving steveyegge/beads architecture and data model
2. **gws-research** — analyzing googleworkspace/cli capabilities and limitations
3. **moons-research** — full exploration of the local moons codebase as it exists today
4. **landscape-research** — scanning for existing systems that already unify agent memory + human knowledge + cloud docs

```tool
[Tool: Agent] beads-research — deep dive steveyegge/beads repo architecture, data model, persistence, CLI, limitations...
```

```tool
[Tool: Agent] gws-research — deep dive googleworkspace/cli capabilities, auth, APIs, agent integration...
```

```tool
[Tool: Agent] moons-research — full exploration of local moons codebase end-to-end...
```

```tool
[Tool: Agent] landscape-research — scan existing agent memory systems, personal knowledge graphs, Google Docs + AI integrations...
```

All four agents completed. Here is the full analysis:

---

## Where You're Misunderstanding the Projects

**Beads is not a memory database.** It's an issue tracker with memory features bolted on. The core is a Dolt-backed (version-controlled MySQL) task tracking system with 50+ column Issue tables, dependency resolution (`bd ready` returns the next unblocked task in ~10ms), and bidirectional sync to Jira/Linear/GitHub. The memory part — `bd remember`/`bd recall` — is a key-value store. A small feature, not the product. The "graph" in beads is dependency edges between work items (blocks, depends_on, replies_to), not a semantic knowledge graph. It doesn't do document ingestion, embeddings, vector search, or RAG. 19K stars, actively developed, but narrowly scoped to coding agent workflows.

**Moons doesn't do what you described it as.** You said "universal for all documents — throw random notes in, powerpoints, academic articles, anything." Currently moons builds knowledge graphs of *codebases*. It uses chomp to ingest repo source files, extracts nodes/edges via LLM, and produces a D3 visualization. The graph schema is code-centric: node types are `code`, `config`, `test`, `script`, `style`, `data`, `note`, `concept`. You can't throw a PowerPoint in. The "all my transcripts" and "trace down my original plan" use case is aspirational, not current.

**The Google Workspace CLI is an API wrapper, not a knowledge connector.** It dynamically generates CLI commands from Google's Discovery Service at runtime. Rust-based, 100+ agent skills, structured JSON output, MCP server built in. It gives agents *access* to your Google Docs — but access is not understanding. Fetching a doc and stuffing it into context is not the same as having an agent that *knows* your career arc because it synthesized 200 of your documents into a structured model.

---

## What Already Exists (You Should Know About These)

**Khoj** — Open-source, self-hostable. Indexes PDFs, Markdown, Org-mode, Notion pages, GitHub repos. Uses RAG to answer questions from your documents. Browser, Obsidian plugin, Emacs, desktop, mobile, WhatsApp. Custom agents with specific knowledge bases. This is the closest existing answer to "AI that knows my stuff." But it's retrieval-based (find the right chunk), not graph-based (understand the relationships).

**Obsidian + MCP-Vault** — Bridges any MCP-compatible agent to an Obsidian vault. Agents can read notes, update content, manage tags, search across files. Version 0.8.2, March 2026. Claude can already interact with your Obsidian vault through this.

**Mem0 with Graph Memory** — Added graph memory January 2026. Extracts "memories" from interactions, stores them, retrieves for personalization. Moving toward structured personal memory, but it's agent-to-agent (learns from conversations, not from your pre-existing documents).

**Graphiti (Zep)** — Temporal knowledge graphs. Tracks how facts change over time. Combines graph-based memory with vector search. Could theoretically model a person, but designed for enterprise agent memory.

**Notion AI 3.0** — Autonomous agent that reads hundreds of pages, comments, version history, referenced pages. Works autonomously for up to 20 minutes. Genuine agentic access to a knowledge base — but locked in Notion's ecosystem.

---

## The Actual Gap

The landscape research confirmed something important. Here's what exists vs. what doesn't:

**Exists:**
- Agent-to-agent session memory (Mem0, Letta, Zep, Beads)
- Document-level AI access to notes (Obsidian MCP, Khoj, Notion AI)
- Document-level AI access to Google Docs (Workspace CLI MCP)
- AI resume tailoring from a single document (Jobright, Teal, Rezi)

**Does NOT exist:**
- A system that builds a structured knowledge graph of *a person* — skills, experiences, preferences, career arc, network — from their documents across multiple sources
- An agent memory layer that combines "what I've learned about this user from their documents" with "what I've learned from interacting with them"
- A career AI tool that draws on a rich personal knowledge graph rather than a single resume file
- A unified platform connecting personal document stores (Obsidian, Google Docs, local files) into a single knowledge graph that agents query and update

Existing tools do **retrieval** (find the right document chunk). What's missing is **synthesis** (build a model of the person from all their documents and interactions).

---

## What You're Actually Missing

**1. MCP is your glue, not code.** The Model Context Protocol is literally the interoperability layer designed for this. Google Workspace CLI already has an MCP server. Obsidian has one. Beads has one. If moons exposed its graph as an MCP server, any agent (Claude, Gemini, ChatGPT) could query your personal knowledge graph alongside your Google Docs alongside your beads task state. You don't need to build a monolithic CLI that wraps all three — you need moons to be a good MCP citizen that composes with the others.

**2. You need an ingestion pipeline, not a CLI.** The hard problem isn't "how do I invoke this from the command line." The hard problem is: how do you take a Google Doc, a resume PDF, a set of project notes, and a collection of transcripts, and extract a structured knowledge graph from them? Moons currently delegates this to chomp (which is LLM-based extraction from source code). Extending this to arbitrary documents — with entity resolution, relationship extraction, temporal tracking — is the actual engineering challenge.

**3. The user story you're looking for is "contextual agent."** Not "memory," not "knowledge base," not "CLI." The story is: *I talk to an agent. The agent knows everything I know — my projects, my career, my ideas, my notes — because it has access to a structured model of my knowledge that it can query and update. When I switch agents or start a new session, nothing is lost.* The audience is anyone who works with AI agents regularly and is frustrated that every new session starts from zero.

**4. You're conflating two problems.** Problem A: "Give agents access to my documents" (solved by MCP + Google Workspace CLI + Obsidian MCP). Problem B: "Build a persistent, structured understanding of me from my documents" (not solved by anyone). Moons is positioned for Problem B. Beads solves neither — it solves Problem C: "Help agents remember their own tasks."

**5. The career angle is real but niche.** A personal knowledge graph that synthesizes your resume, projects, transcripts, and notes into a structured model of your skills and experience *would* be genuinely useful for job applications. No existing tool does this. But "job search tool" is a narrow market and not where the real leverage is. The leverage is: this is useful for *any* context where an agent needs to understand you — not just job search.

---

## The Hard Questions

1. **Is this a product or a personal tool?** If it's a product, you need to solve ingestion at scale (hundreds of document types, multiple cloud sources, entity resolution). If it's personal, you can hand-curate the graph and it's fine.

2. **Why a knowledge graph specifically vs RAG?** Khoj uses RAG and works well enough for "answer questions about my docs." A knowledge graph is more powerful (typed relationships, traversal, reasoning) but dramatically harder to build and maintain. What query do you need a graph for that RAG can't answer?

3. **Where does the structure come from?** If the LLM extracts entities and relationships, it's expensive, slow, and lossy. If the user maintains it manually, nobody will. The ingestion quality problem is the hardest unsolved piece.

4. **What's the update story?** Documents change. Your career evolves. The graph needs to stay current. Beads solves this with compaction. Graphiti solves it with temporal versioning. Moons currently overwrites on each run.

The "glue" is MCP. The "tape" is the ingestion/synthesis layer that turns raw documents into structured knowledge. That tape doesn't exist yet, and building it well is the actual hard problem worth solving.

## User

perhaps you should have a research agent look through this local repo of mine: /Users/jperr/Documents/Jobs because it kinda explains how I am trying to give ai more guidance on me. I want a system that knows about all my projects and me specifically, for now it is a personal tool. later I will see if i can make it a tool for everyone

## Claude

```tool
[Tool: Agent] Explore /Users/jperr/Documents/Jobs — directory structure, CLAUDE.md, graph.json, skills, experience_inventory.md, automation, formats, git history...
```

This changes things. You've already built more than I think you realize.

Your Jobs repo has a `graph.json` with 149 lines of skill/project/experience/role_archetype nodes connected by evidence edges. You have 8 Claude Code skills that all read from `experience_inventory.md` as a single source of truth. You have role archetypes that define what a "good AI engineer job" looks like in terms of required skills. You have evidence-path tracing — when `/evaluate` scores a job, it shows *why* through graph traversal, not keyword matching.

**You've already built the personal knowledge graph that doesn't exist anywhere else.** The Jobs repo *is* the prototype of what you're describing. You just didn't realize it because you built it for one domain (job search) and you're now asking "how do I make this work for everything."

Here's what shifts in my analysis:

**The vision is clearer now.** You're not trying to glue three random tools together. You're trying to take what you built in Jobs — a structured model of yourself that agents can reason over — and make it:
1. **Universal** — not just career, but all your knowledge (projects, ideas, transcripts, notes)
2. **Connected to your cloud docs** — via Google Workspace CLI
3. **Persistent across agent sessions** — the beads angle
4. **Queryable by any agent** — via MCP

**What you actually have today across both repos:**

| Repo | Graph | Agent Skills | Source of Truth | Scope |
|------|-------|-------------|-----------------|-------|
| Jobs | `graph.json` (skills, projects, experience, role archetypes, evidence edges) | 8 skills (find, tailor, evaluate, track, etc.) | `experience_inventory.md` | Career |
| Moons | `graph.json` (code nodes, import/reference edges) | 1 skill (/moons) | chomp dump | Codebase structure |

Two separate graphs, two separate schemas, no connection between them. But both follow the same pattern: structured JSON graph + agent skills that traverse it + a source-of-truth document.

**What's actually missing — and it's not what you thought:**

1. **A unified graph schema.** Your Jobs graph has `skill`, `project`, `experience`, `role_archetype` nodes. Your moons graph has `code`, `config`, `test`, `concept` nodes. These need to merge. A project node in your career graph should link to the code nodes in the moons graph for that project. Saturn-the-project should connect to saturn-the-codebase.

2. **An ingestion layer that handles more than source code.** Chomp handles repos. You need the equivalent for Google Docs, PDFs, transcripts, notes. Not a CLI — a pipeline. Something that takes a Google Doc, extracts entities (skills, decisions, people, timelines), and adds them to the graph.

3. **A single graph that lives somewhere permanent.** Right now you have two graph.json files in two repos. The "moons" concept should be the *one* graph that holds everything — your career data, your codebase structures, your project plans, your transcripts. One graph, queryable from anywhere.

4. **MCP exposure.** Turn the graph into an MCP server. Then any agent (Claude Code, Gemini, whatever) can query "what does Joey know about distributed systems?" and get back structured evidence paths — not raw document chunks.

**What you do NOT need:**

- Beads. Your use case isn't "agents remembering their tasks." It's "agents understanding me." Beads solves a different problem. If you eventually want agent task persistence, you can add it, but it's not the core of what you're building.
- A CLI. You're already operating through Claude Code skills, and that's working. The power move is MCP, not another binary.
- Google Workspace CLI as a core component. It's a useful *connector* — you'd use it inside your ingestion pipeline to pull docs — but it's not a peer to moons. It's plumbing.

**The real architecture is simpler than you think:**

```
Google Docs ─────┐
Local files ─────┤
Transcripts ─────┤──→ Ingestion Pipeline ──→ Unified Graph (moons/graph.json)
Repo source ─────┤                                    │
Career data ─────┘                                    ├──→ MCP Server
                                                      ├──→ Claude Code skills
                                                      ├──→ D3 Visualization
                                                      └──→ Query CLI (query.py)
```

The graph is the product. Everything else is either feeding it (ingestion) or reading it (skills, MCP, viz).

**How this helps you as a harness engineer:** You've already demonstrated the pattern — structured personal knowledge + agent skills that traverse it — in your Jobs repo. That's a portfolio piece. Generalizing it into a universal personal knowledge graph with MCP exposure would be a genuine contribution to the agent tooling space, because as the landscape research confirmed, nobody has built this yet.

## User

okay so we understand what we want to make but i need you to understand HOW it will be made. you gave me four things that were missing from what i currently have. those four things are huge individual sections that require multiple claude sessions and planning. I ultimately work like this, chat with you and prompt for research until we make a plan with specs. I then feed this information into a ralph loop, like https://ghuntley.com/ and /Users/jperr/Documents/youtube/RALPH_PROMPT.md . This way we in chat create the plan, specs, vision, and final outcome, the ralph loop is simply the ai model that is given small tiny tasks that it does for one session then passes on and it gets removed. that is where i like beads. something like https://simonwillison.net/guides/agentic-engineering-patterns/agentic-manual-testing/ is typically used in the ralph loop too

## Claude

```tool
[Tool: Agent] Read /Users/jperr/Documents/youtube/RALPH_PROMPT.md — extract RALPH loop structure, phases, task passing, human vs AI roles, tooling conventions...
```

```tool
[Tool: Agent] Research ghuntley.com and simonwillison.net agentic-manual-testing pattern...
```

I understand the workflow now.

**Your process:**
1. **This conversation** → Research, define vision, produce specs for each piece
2. **RALPH loop** → Autonomous build agent, one component per session, filesystem as state, `moons/graph.json` as the progress tracker, shell script re-invokes fresh sessions
3. **Beads** fits in RALPH, not in the product. It's the task memory between RALPH sessions — replacing or augmenting the filesystem-as-persistence pattern your current RALPH uses

So beads isn't part of the moons *product*. It's part of your *development workflow*. The RALPH loop currently passes state through `graph.json` + `runs/pass-N.md` + component markdown files. Beads could give that loop dependency-aware task queuing (`bd ready` to pick the next unblocked task) and persistent memory across sessions (`bd remember`/`bd recall`) instead of relying purely on filesystem reads.

**What you need from me right now:** Not implementation. Not code. You need the four missing pieces defined clearly enough that each can become its own RALPH prompt — with components, scoring criteria, interfaces, dependencies, and a completion gate.

The four pieces again:
1. **Unified graph schema** — merge Jobs graph + moons graph into one schema that handles skills, projects, experience, code, concepts, documents
2. **Ingestion pipeline** — take Google Docs, PDFs, transcripts, notes, repos and extract entities/edges into the unified graph
3. **Single persistent graph** — one `moons/graph.json` that lives somewhere central, not scattered across repos
4. **MCP server** — expose the graph so any agent can query it

Each of those is a RALPH-sized project. Do you want to start speccing them out one at a time, or do you want to first define how they relate to each other (dependency order, interfaces between them) so the RALPH loops can be sequenced?

## User

okay so we understand what we want to make but i need you to understand HOW it will be made. you gave me four things that were missing from what i currently have. those four things are huge individual sections that require multiple claude sessions and planning. I ultimately work like this, chat with you and prompt for research until we make a plan with specs. I then feed this information into a ralph loop, like https://ghuntley.com/ and /Users/jperr/Documents/youtube/RALPH_PROMPT.md . This way we in chat create the plan, specs, vision, and final outcome, the ralph loop is simply the ai model that is given small tiny tasks that it does for one session then passes on and it gets removed. that is where i like beads. something like https://simonwillison.net/guides/agentic-engineering-patterns/agentic-manual-testing/ is typically used in the ralph loop too

## Claude

The patterns research confirmed the workflow. One correction to note: RALPH isn't an acronym — it's named after Ralph Wiggum (the Simpsons character). Huntley's core loop is literally `while :; do cat PROMPT.md | claude-code ; done`. Each iteration reads state, picks one thing, builds it, validates it. Specs + stdlib files steer outcomes, not step-by-step instructions.

Your RALPH prompt in the youtube repo follows this exactly — 7 phases, filesystem as state, testers/builders/integrators as subagents, scoring criteria per component.

Waiting on your answer: sequence the four pieces together first, or start speccing them individually?

## User

I am done documenting. You are right that is exactly what I need. Develop one ralph prompt that connects to a plan for the four different parts of this pipeline to be implemented sequentially. i only type 1 bash command.

## User

/document I have an idea where we create an lms.txt: llmstxt.org but for youtbube videos it wouldnt be called llms.txt. it would be a feature on coding agents where I can paste a youtube video link and a better understanding of the video would be given to the agent. currently the best option is just giving the transcript, which is flawed. we need a system that gives agents more subtle details about the video, ultimately strengthening the context.

## Claude

Documenting.

This is a compelling idea. The core problem is real — transcripts lose a massive amount of signal. A transcript is just words, but a coding tutorial video carries:

- **Visual context** — what's on screen (code, terminal output, diagrams, browser tabs, file trees)
- **Temporal structure** — the order things are built, what gets deleted/refactored, the "arc" of the explanation
- **Implicit intent** — when someone hovers over code, scrolls to a specific section, highlights text, or switches between files, that's signal a transcript completely drops
- **Error/debug cycles** — the most valuable part of many coding videos is watching someone hit an error, diagnose it, and fix it. Transcripts flatten this into an incoherent sequence of words

The llms.txt analogy is sharp. llms.txt gives agents a structured, machine-readable summary of a website. Your idea would give agents a structured, machine-readable summary of a video — not just what was *said*, but what was *shown* and *done*.

Some initial thoughts on what this could look like:

**What the output format might contain:**
- Timestamped segments with semantic labels (setup, implementation, debugging, explanation)
- Extracted code blocks tied to timestamps
- Screen content descriptions (what file is open, what terminal shows)
- Dependency/tool list derived from what's visible
- A structured summary that preserves the build order and decision points

**Why "just use the transcript" fails:**
- Speaker says "this" or "here" referencing something on screen — meaningless without the visual
- Code is typed live but never fully dictated
- Error messages appear on screen but aren't read aloud
- File paths, configs, and terminal output are visual-only

What's your thinking on the implementation side? Are you imagining this as a pipeline that processes videos ahead of time, or something that runs on-demand when an agent gets a YouTube link?

