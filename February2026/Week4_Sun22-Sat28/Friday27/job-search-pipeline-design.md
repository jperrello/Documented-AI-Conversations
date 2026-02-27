# Job Search Pipeline Design
**Date:** Friday, February 27, 2026
**Time:** 3:21 PM PST
---

## User

I do not like the current applypivot strategy of finding jobs on the web. however, i think your standard web search tool may not be adequate enough as a replacement. we have so much information in this repository about me, we can make highly informed searches online. I am sure we can write a program to find recent, verifiable (not posted 30 days ago), real urls (there needs to be a 200 confirmation with the url, not a 400. the urls have to link to a job), job postings that are in some way matching my experiences. on top of that we should design it to have feature flags that you claude can use to modify depending on information in moons. ask me some questions, but what features do you see existing in this program? do you want to know why i dont like applypivot?

## Claude

```tool
[Tool: Read] /Users/jperr/Documents/Jobs/experience_inventory.md — full file
```

```tool
[Tool: Read] /Users/jperr/Documents/Jobs/graph.json — first 100 lines
```

```tool
[Tool: Glob] **/applypilot/** in /Users/jperr/Documents/Jobs
```

```tool
[Tool: Read] /Users/jperr/Documents/Jobs/applypilot/searches.yaml — full file
```

```tool
[Tool: Read] /Users/jperr/Documents/Jobs/applypilot/profile.json — full file
```

Yes — tell me why you don't like ApplyPilot. Understanding what's broken helps me design what's better.

Here's what I see as the feature set, informed by everything in your repo:

**Core search engine:**
- **Graph-driven query generation** — Instead of static search strings like `searches.yaml`, the program reads `graph.json` and builds queries from your actual skill/project/experience nodes. "mDNS service discovery engineer" is a real query that matches you. "AI engineer new grad" matches everyone.
- **Multi-source scraping** — Direct company career pages (Anthropic, OpenAI, Vercel, etc.), job board APIs, and aggregators. Not just Indeed/LinkedIn/Glassdoor.
- **URL verification** — Every result gets an HTTP request. 200 or discard. No dead links, no redirects to "this posting has expired."
- **Freshness enforcement** — Parse actual posting dates from page content, not just "posted 3 weeks ago" metadata that job boards lie about. Configurable max age (default: 7 days).

**Feature flags (Claude-modifiable):**
- `skill_emphasis` — Weight toward AI/ML, systems, backend, devtools, or research roles
- `experience_ceiling` — Max years required (currently filtering 7+, but configurable)
- `location_mode` — Remote only, Bay Area, hybrid, or expand radius
- `company_stage` — Startup, growth, FAANG, research lab
- `salary_floor` — Minimum compensation threshold
- `board_sources` — Which sources to query
- `archetype` — Maps directly to your `role_archetype` nodes in `graph.json`
- `exclude_companies` — Blacklist
- `freshness_days` — How recent

**Integrity checks:**
- **Deduplication** — SQLite store of seen URLs, never resurface the same job
- **Posting validation** — Confirm the page actually contains a job posting (not a blog post or company about page)
- **Company verification** — Basic check that the company exists and the careers page is real

**Integration with existing workflow:**
- Output compatible with `/evaluate` and `tracker.json`
- Each found job writes a structured record (title, company, URL, match score, posting date, skills matched)

Questions for you:

1. You said "feature flags that Claude can use to modify depending on information in moons" — did you mean "moods" or something else? I want to make sure I understand the modifier source.

2. What language do you want this in? Python seems natural given your stack, but TypeScript is also viable.

3. Should this replace the ApplyPilot database (`applypilot.db`) entirely, or write to its own store that `/evaluate` can also read from?

4. Do you have specific companies or career pages you want to always check, separate from the general search?

## User

I didnt like applypivot because it was slow, had no knowldge of my background, and often just scraped any job board in the hopes of quantity over quality. I want something that knows about ME and the entire pipeline has ME in mind while searching. This is why i think it is important we make the pipeline now. what is the algorithm or psuedocode for this. I tell you go find me some jobs today. you should go to moons and gain knowledge of my background, then perform multiple web searches ( maybe some teaching job searches, some ai job searches, some basic programming too, game dev, ...) dont try to force all these skills i have into one search, the filter will be too great to actually find good jobs. Then how can i trust the jobs you scrape are one you have read and actually scraped. should we convert the html pages to md first? how do we make sure when we do find a job we can say okay this is good and here is a url joey can see for himself. show me the pipeline!

## Claude

That criticism is clear and valid. ApplyPilot treats you as a generic input to a generic scraper. Here's the pipeline:

```
┌─────────────────────────────────────────────────────────────┐
│                    JOEY SAYS "FIND JOBS"                     │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  PHASE 1: KNOW JOEY                                         │
│                                                              │
│  Read experience_inventory.md                                │
│  Read graph.json                                             │
│  Read feature_flags.yaml (Claude-modifiable config)          │
│                                                              │
│  Extract SKILL CLUSTERS (not one mega-query):                │
│    cluster_ai      = [llm-engineering, prompt-engineering,   │
│                       structured outputs, multi-model]       │
│    cluster_systems  = [mDNS, networking, protocol design,    │
│                       distributed systems, cross-compilation]│
│    cluster_backend  = [Python, FastAPI, asyncio, PostgreSQL,  │
│                       API design, Docker]                    │
│    cluster_teaching = [TA, instructor, curriculum, tutoring]  │
│    cluster_gamedev  = [game AI, MCTS, Unity, procedural gen] │
│    cluster_devtools = [TypeScript, npm, Vercel AI SDK,        │
│                       package publishing, open source]       │
│    cluster_research = [MS thesis, computational media,        │
│                       evaluation frameworks]                 │
│                                                              │
│  Each cluster gets its OWN search pass.                      │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  PHASE 2: GENERATE QUERIES                                   │
│                                                              │
│  For each cluster, generate 2-4 targeted search queries:     │
│                                                              │
│  cluster_ai →                                                │
│    "AI engineer entry level remote 2026"                     │
│    "LLM infrastructure engineer new grad"                    │
│    "ML engineer Python OpenAI API"                           │
│                                                              │
│  cluster_teaching →                                          │
│    "computer science instructor Bay Area"                    │
│    "coding bootcamp teacher Python"                          │
│    "CS teaching position university 2026"                    │
│                                                              │
│  cluster_gamedev →                                           │
│    "game AI engineer"                                        │
│    "procedural generation developer"                         │
│                                                              │
│  ... etc for each cluster                                    │
│                                                              │
│  Apply feature flags:                                        │
│    if flags.location_mode == "remote_only":                  │
│      append "remote" to all queries                          │
│    if flags.freshness_days == 3:                             │
│      append date filters where supported                     │
│    if flags.exclude_companies:                               │
│      note for later filtering                                │
│                                                              │
│  Total: ~15-25 distinct searches per run                     │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  PHASE 3: SEARCH (parallel)                                  │
│                                                              │
│  For each query:                                             │
│    results = web_search(query)                               │
│    collect raw URLs + titles + snippets                      │
│                                                              │
│  Aggregate all results into candidate_urls[]                 │
│  Deduplicate against seen_jobs.db (SQLite)                   │
│  Remove URLs matching exclude patterns                       │
│    (blog posts, "about us" pages, news articles)             │
│                                                              │
│  Output: ~50-150 candidate URLs after dedup                  │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  PHASE 4: FETCH + VERIFY (parallel, batched)                 │
│                                                              │
│  For each candidate URL:                                     │
│    response = HTTP GET(url)                                  │
│                                                              │
│    ❌ DISCARD if:                                             │
│      - status != 200                                         │
│      - redirects to generic careers page (not a specific job)│
│      - domain is a known aggregator re-posting stale jobs    │
│                                                              │
│    ✅ KEEP if:                                                │
│      - status == 200                                         │
│      - page contains job-posting signals                     │
│        (apply button, requirements section, salary info)     │
│                                                              │
│    Convert HTML → Markdown (strip nav, footer, ads)          │
│    Store as:                                                 │
│      {                                                       │
│        url: "https://company.com/jobs/12345",                │
│        status: 200,                                          │
│        markdown: "# Software Engineer\n\n## About...",      │
│        fetched_at: "2026-02-27T15:30:00Z"                   │
│      }                                                       │
│                                                              │
│  Output: verified_jobs[] with markdown content               │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  PHASE 5: READ + SCORE (this is where Claude proves it)      │
│                                                              │
│  For each verified job, Claude READS the markdown and:       │
│                                                              │
│  1. Extracts structured fields:                              │
│     - title, company, location, remote?, salary_range        │
│     - required_skills[], preferred_skills[]                  │
│     - years_required, education_required                     │
│     - posting_date (parsed from page content)                │
│                                                              │
│  2. Scores against graph.json:                               │
│     match_score = 0                                          │
│     for skill in required_skills:                            │
│       if skill in joey_skills:                               │
│         match_score += weight(skill.proficiency)             │
│     for skill in preferred_skills:                           │
│       if skill in joey_skills:                               │
│         match_score += weight(skill.proficiency) * 0.5       │
│     penalty if years_required > 3                            │
│     penalty if location doesn't match flags                  │
│     bonus if company_stage matches flags                     │
│                                                              │
│  3. Writes a PROOF record:                                   │
│     {                                                        │
│       url: "https://...",        ← Joey clicks this          │
│       title: "AI Engineer",                                  │
│       company: "Acme Corp",                                  │
│       match_score: 0.82,                                     │
│       matched_skills: ["Python", "LLM", "FastAPI"],          │
│       missing_skills: ["Kubernetes"],                        │
│       posting_date: "2026-02-25",                            │
│       summary: "3-sentence summary of the role",             │
│       why_match: "Saturn demonstrates their core reqs...",   │
│       source_cluster: "cluster_ai"                           │
│     }                                                        │
│                                                              │
│  ❌ DISCARD if:                                               │
│    - match_score < threshold (default 0.4)                   │
│    - posting_date older than flags.freshness_days             │
│    - years_required > flags.experience_ceiling               │
│                                                              │
│  Output: scored_jobs[] — only jobs Claude actually read      │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  PHASE 6: PRESENT TO JOEY                                    │
│                                                              │
│  Sort scored_jobs by match_score DESC                         │
│  Group by source_cluster                                     │
│                                                              │
│  Output format (per job):                                    │
│  ┌────────────────────────────────────────────────────┐      │
│  │ ⬛ AI Engineer — Acme Corp (0.82)                  │      │
│  │ 🔗 https://acme.com/jobs/12345  ← VERIFIED LIVE   │      │
│  │ 📍 Remote | Posted: Feb 25, 2026                  │      │
│  │ ✅ Matched: Python, LLM, FastAPI, API design       │      │
│  │ ❓ Missing: Kubernetes (nice-to-have)              │      │
│  │ 💬 "Saturn directly demonstrates their need for    │      │
│  │    multi-model routing and service discovery..."   │      │
│  └────────────────────────────────────────────────────┘      │
│                                                              │
│  Write results to jobs_found.json                            │
│  Insert new URLs into seen_jobs.db                           │
│  Log run metadata (queries used, counts, timestamp)          │
└─────────────────────────────────────────────────────────────┘
```

**On your HTML→MD question:** Yes. Converting to markdown is the right move. It strips the noise (nav bars, footers, tracking scripts, cookie banners) and gives Claude clean text to actually parse. Raw HTML is full of garbage that makes extraction unreliable. The markdown becomes the proof artifact — "here is what I read, here is what I extracted from it."

**On trust:** The proof record is the answer. Every job in the output has: the URL you can click, the skills Claude matched against YOUR graph, and a `why_match` explanation referencing YOUR projects. If Claude can't explain why it matched, it doesn't show up.

**On "moons"** — I'm still unclear on this word. Did you mean your memory files / experience docs? Or something else?

One design question: should the program be a standalone Python script you run (`python find_jobs.py`), or should it be a Claude skill (`/find-jobs`) that I execute using my own tools (WebSearch, WebFetch) within conversation? The skill approach means I'm doing the searching and reading live, with no external dependencies. The script approach is more automated but needs API keys for search.

## User

create a plan to implement this and save it as PLAN.md. when you are done just say I made PLAN.md

## Claude

```tool
[Tool: Write] /Users/jperr/Documents/Jobs/PLAN.md — Full implementation plan for /find-jobs skill covering 6 phases, search_config.yaml structure, seen_jobs.json, skill prompt, and integration with existing pipeline
```

I made PLAN.md

## User

does it also include removing any mention of applypivot in our system?

## Claude

No. The plan covers building the new `/find-jobs` skill but doesn't include removing ApplyPilot references. That should be a step. Specifically:

- `CLAUDE.md` — the pipeline section still references `applypilot run discover enrich` and `APPLYPILOT_DIR`
- `applypilot/` directory — `searches.yaml`, `profile.json`, `applypilot.db`, `.env`
- `~/.zshrc` — the `APPLYPILOT_DIR` env var
- Any skills that reference `applypilot.db` (the `/evaluate` skill reads from it)

Want me to add a cleanup step to `PLAN.md` that removes all of this?

## User

yes

## Claude

```tool
[Tool: Read] /Users/jperr/Documents/Jobs/PLAN.md — full file
```

```tool
[Tool: Edit] /Users/jperr/Documents/Jobs/PLAN.md — Added Step 5 (Remove ApplyPilot) and Step 6 (Update CLAUDE.md Key Files), covering deletion of applypilot/ directory, CLAUDE.md cleanup, ~/.zshrc cleanup, /evaluate skill rewiring, and new pipeline docs
```

Done. Added Step 5 (Remove ApplyPilot) and Step 6 (Update CLAUDE.md Key Files) to `PLAN.md`. The cleanup covers:

- Deleting the entire `applypilot/` directory
- Stripping all ApplyPilot references from `CLAUDE.md`
- Removing `APPLYPILOT_DIR` from `~/.zshrc`
- Rewiring or retiring the `/evaluate` skill so it no longer reads `applypilot.db`
- Replacing the pipeline docs with the new `/find-jobs` flow

## User

I am done documenting.
