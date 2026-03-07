# Prose Agent and RALPH Team Architecture
**Date:** Saturday, March 7, 2026
**Time:** Conversation start

---

## User

I just realized that the grader agent in my Ralph loop does not grade based on prose. And I'm starting to wonder how big of a mistake that is. And let me explain why. There is an art and a science to writing, and that is conducted through prose.

I fear that in attempts for this Ralph loop to be concise and short and meet very specific guidelines, it'll end up truncating the thesis rather than rewriting it, if that makes sense. We'll lose the style and the art of writing through such a rigid truncation process. And I almost want to instill a prose agent to help us with this as well.

Let's have a conversation about maybe how this could be implemented. But I want you to do some research on the science of writing and illustrating your points. writing a beautiful piece of work

## Claude

Documenting.

Let me read the current grader agent and do some research on the craft of academic prose before we discuss implementation.

```tool
[Tool: Read] /Users/jperr/Documents/Saturn-Thesis/.claude/agents/grader.md
```

```tool
[Tool: Read] /Users/jperr/Documents/Saturn-Thesis/academic-advisor-prompt.md
```

```tool
[Tool: Read] /Users/jperr/Documents/Saturn-Thesis/rewrite_notes.md
```

```tool
[Tool: WebSearch] Gopen Swan "Science of Scientific Writing" principles stress position topic position
```

```tool
[Tool: WebSearch] Helen Sword "Stylish Academic Writing" principles zombie nouns nominalization
```

```tool
[Tool: WebSearch] Joseph Williams "Style Lessons in Clarity and Grace" principles cohesion coherence known-new
```

```tool
[Tool: WebSearch] academic writing craft prose rhythm sentence variety paragraph flow research
```

```tool
[Tool: WebSearch] Steven Pinker "Sense of Style" academic writing principles classic style
```

```tool
[Tool: WebSearch] LLM AI generated academic writing problems flat prose lack of voice repetitive patterns 2025 2026
```

```tool
[Tool: WebSearch] rhetorical moves academic writing CARS model Swales creating research space argumentation
```

```tool
[Tool: Read] /Users/jperr/Documents/Saturn-Thesis/chapters/ch1-introduction.tex (first 80 lines)
```

Your instinct is right, and the problem is more specific than you might think. Let me walk through it.

## The Gap in the Current Grader

The grader says it on line 11: *"You evaluate structural quality — not prose polish, not formatting, not LaTeX mechanics."* The seven criteria (C1–C7) are all structural: completeness, contribution/claim distinction, ordering, design intent, trade-offs, value density, conciseness. The rules section even says: *"Never suggest prose rewrites. Describe structural problems and fix directions."*

This means the RALPH loop optimizes for one dimension while being completely blind to another. And here's the problem with optimization loops: **what gets measured gets managed, and what doesn't get measured gets destroyed.**

## What the Science of Writing Actually Says

There are four foundational works that together define the "science" side of good prose. Each identifies properties that are *measurable* and *learnable* — not subjective taste.

### 1. Gopen & Swan — Reader Expectation Theory

Their 1990 paper "The Science of Scientific Writing" established that readers have hard-wired expectations about where information should appear in a sentence:

- **Topic position** (beginning): old information that links backward, giving the reader context
- **Stress position** (end): new information the writer wants the reader to emphasize

Your grader partially captures this in C6 ("stress position waste"), but only as a binary: *is there new info there or not?* It doesn't ask: does the topic position link backward? Does the sentence *connect* to what came before? That's the difference between a collection of individually correct sentences and a flowing paragraph.

### 2. Joseph Williams — Cohesion and Coherence

Williams' *Style: Lessons in Clarity and Grace* formalized two distinct properties:

- **Cohesion**: old-to-new flow *between* sentences. Each sentence begins where the last one ended.
- **Coherence**: consistent topic strings *across* paragraphs. The reader always knows what the paragraph is "about."

These are not aesthetic preferences. They are comprehension mechanics. Empirically, readers process cohesive text faster and retain more. A structurally perfect section with broken cohesion reads like a bulleted list disguised as paragraphs.

### 3. Helen Sword — Zombie Nouns

Sword's concept of "zombie nouns" names a specific failure mode: nominalizations that "cannibalize active verbs, suck the lifeblood from adjectives, and substitute abstract entities for human beings." Compare:

- Zombie: *"The implementation of the protocol facilitated the discovery of services."*
- Alive: *"We implemented the protocol, and clients discovered services."*

This is critical because **LLM revision loops tend to increase nominalization density**. Research from PNAS shows that LLM-generated text is characteristically noun-heavy and informationally dense — exactly the opposite of vivid prose. A grader that rewards conciseness (C7) and value density (C6) could accidentally reward zombie prose, because dense nominalizations pack more "information" per sentence.

### 4. Steven Pinker — Classic Style

Pinker's *The Sense of Style* defines **classic style** as the gold standard for explanatory writing: the writer sees something the reader hasn't noticed yet, and orients the reader's gaze so they can see it for themselves. The anti-patterns he identifies — metadiscourse, excessive signposting, hedging, professional narcissism — are exactly what a structurally-focused loop might *introduce*. When an agent tries to satisfy "ordering serves the reader" (C3), it adds signposting. When it tries to satisfy "design intent" (C4), it adds formulaic "we chose X rather than Y because Z" patterns. Structurally correct. Prosetically dead.

## The Specific Risk for RALPH

Each RALPH pass asks the writer agent to fix structural issues. The writer agent (an LLM) will:

1. **Compress to meet page budgets** → loses sentence rhythm and variety
2. **Add rationale for every design choice** → produces formulaic X-rather-than-Y patterns
3. **Remove "filler" per C6** → may remove the breathing room that gives prose rhythm
4. **Satisfy "every sentence earns its place"** → produces relentlessly dense text that exhausts the reader

The result: structurally excellent, prosetically dead text. Like a building with flawless structural engineering but no architecture.

Here's the thing — your introduction already has great prose. The Strauss nuclear energy opening, the personal "as a graduate student paying $100 per month" narrative, the concrete three-subscriptions-three-API-keys example. That's classic style. That's voice. **The RALPH loop didn't create those qualities. And without a prose dimension, it could destroy them.**

## What a Prose Agent Would Evaluate

Drawing from the research, here are the dimensions the current grader is blind to:

| Dimension | Source | What It Measures |
|---|---|---|
| **Sentence Rhythm** | Gopen/Swan, Pinker | Variation in sentence length; interplay of long exploratory and short declarative sentences |
| **Information Flow** | Williams, Gopen/Swan | Old-to-new contract at the sentence level; does each sentence connect to the last? |
| **Voice and Agency** | Sword, Pinker | Active subjects, minimal nominalizations, preserved personal voice |
| **Rhetorical Moves** | Swales CARS | Is each paragraph *doing* something — establishing, claiming, occupying — or just listing? |
| **Concrete/Abstract Balance** | Pinker | Are abstract ideas grounded in specific examples, or floating in generality? |
| **Paragraph Shape** | Williams | Does a paragraph open with claim, develop through evidence, close with implication? |

## Implementation: How to Build It

I see four options, in order of complexity:

**Option A: Bolt prose criteria onto the existing grader.**
Add C8–C13 for the prose dimensions above. Simple, but it overloads the grader and mixes concerns. Structural issues and prose issues require different kinds of attention.

**Option B: Separate prose agent, runs alongside the structural grader.**
A second evaluation agent with its own criteria, its own scoring scale, its own deep-dives. The RALPH orchestrator gets two reports per pass and synthesizes them for the writer agent. Separation of concerns. This is my recommendation.

**Option C: Prose agent as a polish pass after structural convergence.**
Get structure right first (RALPH runs until structural scores converge), then run a prose-only revision pass. Risk: if prose problems have structural roots, a late polish can't fix them. Paint over rust.

**Option D: Interleaved — both graders inform every revision.**
Most comprehensive, but the two agents may give conflicting directives (structural grader says "compress this paragraph," prose grader says "this paragraph needs room to breathe").

**My recommendation is Option B** — a separate prose grader that runs in parallel with the structural grader, with the RALPH orchestrator responsible for resolving conflicts. The completion threshold would require both structural *and* prose scores to meet their respective bars.

---

What's your reaction? Do you want to dig into the specific criteria for the prose agent, or talk about how the orchestrator would handle conflicts between the two graders?

## User

I dont necessarily like this example, but i get what you are saying:
  - Zombie: "The implementation of the protocol facilitated the discovery of services."
  - Alive: "We implemented the protocol, and clients discovered services."
I was in charge of the whole protocol and this seems like i had people help me or configure Saturn in the process of my project.

Point four is incredible, absolute gold. By the way, we should add this referenced material to moons.

Here are my thoughts (in addition to what i just interrupted):
I agree that option B is most likely the best way to go. The problem I have with this is that I feel like the main writing agent may get too flustered at that point with two graders going on. Remember, we are working with Opus, but the context window is very limited.

It's already gathering all this information from the thesis. It's already gathering all the information from Moons. It's already gathering the skill context and everything. The worst thing we could do is have so many grading criteria that the main writing agent fails.

So what I think we should do instead is implore an agent team. And this agent team could just become the new grader agent. I actually wrote this current thesis using an agent team as well. Each agent was given the academic writing skill and context about their specific section. The context came from moons as well as @Joey-topic-for-each-section.md . The moons, as well as that document, are important because they are direct work from me. They are my voice. The skill reflects this too. This is me injecting my voice into the thesis.

This is what I am trying to get Claude to write. The problem is, is it's just so rigid, fitting within that length, right? So this Ralph loop is designed to basically shorten my paper to be so concise to where it fits within that 40 page limit, or 50 if you want to be generous, while scoring a 90 or above on the grader's harsh grading scale.

Because the grader's grading scale was designed again by me and my advisor. You know, I want that grade to be achieved, and I want the academic writing skill to be maintained, but I want my knowledge to be channeled into this thesis. It's just a lot of hard things to get one coding agent to do.

So we need to find a way to separate the grader and the writing agent by sections into an agent team, if that makes sense. So the introduction has a writer and a grader specifically for the introduction. The background has a writer and a grader specifically for the background.

## Claude

Fair point on the "we" example — you built Saturn solo. A better zombie/alive pair for your thesis would be:

- Zombie: *"The discovery of services was facilitated through the implementation of mDNS beacons."*
- Alive: *"I broadcast mDNS beacons, and clients discovered services automatically."*

That preserves your first-person voice and the agency you actually had.

Now to the architecture point — you've identified the real bottleneck. The context window problem is not hypothetical. One Opus agent trying to hold: the full chapter text, the moons context, the academic-writing skill, the section reference file, a structural grading report, AND a prose grading report — that's too much. The agent doesn't get "flustered" in a human sense, but it does start dropping context. It prioritizes the most recent instructions and loses the nuance from earlier material. Your voice notes from `Joey-topic-for-each-section.md` — the exact material that carries your voice — would be the first thing to get squeezed out.

The section-level agent team solves this cleanly. Here's how I'd structure it:

**Per-section team (7 teams total):**

```
┌─────────────────────────────────────┐
│         RALPH Orchestrator          │
│  (reads rewrite_notes.md, decides   │
│   which sections need work, spawns  │
│   section teams, resolves conflicts)│
└──────────┬──────────────────────────┘
           │ spawns per section
           ▼
┌─────────────────────────────────────┐
│     Section Team (e.g. Design)      │
│                                     │
│  ┌─────────┐  ┌──────────────────┐  │
│  │Structure│  │  Prose Grader    │  │
│  │ Grader  │  │  (new agent)     │  │
│  └────┬────┘  └───────┬──────────┘  │
│       │               │             │
│       ▼               ▼             │
│  ┌─────────────────────────────────┐│
│  │     Section Writer              ││
│  │  (gets BOTH reports +           ││
│  │   voice notes + skill)          ││
│  └─────────────────────────────────┘│
└─────────────────────────────────────┘
```

**Why this works for context:**

Each writer agent only loads:
1. One chapter file (~3-5 pages of LaTeX, not seven chapters)
2. That section's moons context (edges from graph.json to the relevant nodes only)
3. That section's voice notes (the relevant paragraph from Joey-topic-for-each-section.md)
4. The academic-writing skill
5. That section's reference file (from .claude/skills/academic-writing/references/)
6. Two focused grading reports — one structural, one prose — for just their section

That's a fraction of what one monolithic agent would need. Each writer has deep context on its section rather than shallow context on everything.

The graders are even lighter. The structural grader reads one chapter + moons for validation. The prose grader reads one chapter + the voice notes. Neither needs the writing skill — they evaluate, they don't write.

The orchestrator stays lean too. It reads rewrite_notes.md to see scores, decides which sections need another pass, spawns the relevant section teams, and collects results. It never reads the full thesis text — it works from the score matrix.

The key insight you're surfacing is that the voice injection has to happen at the writer level, not the grader level. The grader can check whether voice survived. But Joey-topic-for-each-section.md and the moons voice notes need to be in the writer's context so it has something to channel. The current RALPH design feeds the writer structural fix directions. This new design also feeds it your voice as source material and a prose grade that tells it whether it's still capturing that voice.

One question before we go further: do you want the two graders per section to run as separate agents (true Option B), or should we merge them into a single grader agent that has both structural and prose criteria but is section-scoped? Since each grader only reads one chapter now instead of seven, the context window math changes — a combined grader per section might actually fit.

## User

I dont like the orchestrator deciding which sections need work, that is based on grader scores. as long as a section is below 90 it absolutely needs work. I would say the only time a section wouldnt necessarily need work is if it was at a 95. We can make it one grading agent for each writing skill. But what I think we need to have after all of the section agents are done is one final pass agent. You've said that the orchestrator resolves conflicts, but what conflicts is it resolving? Should that really be on the burden of the orchestrator?

The orchestrator is simply watching the teams. I think after the section writer, we should have an agent that reads the full thesis. This agent just notices some things that may have slipped through. Maybe the abstract isn't all on one page, so we would tweak it.

Maybe it realized we said one thing in the introduction that really could have been said better in the background, or even worse, it got repeated in a section because we had section writers in specific areas. So that final pass agent would basically just do one proofread skim for me so it may catch some bugs that slipped through.

I understand that this is a route loop, and that's a lot to run on each call, but I think it is worth it because even on the third iteration of this route loop, that final pass agent is still needed.

Which leads me to ask you one question. I run ralph loops autonomously. I run the ralph loop and then I go on a walk for an hour and I come back and I see what happens. This agent could be running 20 times, it could be running 100 times, maybe 5 times.

What do you think about the scaling of this? Like, are we doing too much work for each pass? Are we doing not enough? Think in the sense that I am completely absent and this is running autonomously on its own.

## Claude

You're right to push back on the orchestrator doing conflict resolution. That was scope creep. The orchestrator's job is simple: spawn teams, collect scores, log results, decide if we loop again. It's a dispatcher, not an editor.

And the final-pass agent is a different role entirely from the orchestrator. The revised architecture:

```
RALPH Orchestrator
  1. Read rewrite_notes.md
  2. Spawn ALL section teams in parallel (every section < 95 gets a team)
  3. Wait for all teams to finish
  4. Spawn Final Pass Agent (every 3rd iteration)
  5. Log scores to rewrite_notes.md
  6. If any section avg < 90 → loop again

Per-section team (7 parallel):
  Section Grader (struct + prose) → Section Writer (skill + voice + grader report)

Final Pass Agent (every 3rd pass):
  Reads full thesis. Catches cross-section repetition, misplaced content,
  abstract formatting, terminology drift, chapter handoff continuity.
  Makes surgical edits only.
```

The math: 15 agent invocations per pass (14 parallel section work + 1 final pass). Wall-clock ~10-15 min per pass. The final-pass agent is the scaling concern — reads full thesis for diminishing returns, hence every-3rd-pass cadence. The bigger risk is oscillation, which plateau detection catches.

## User

Im cool with option 1, we can have a variable that indicates what turn the agent is on. The exit condition should stay 95, idc. I am done documenting. Lets write this new loop and grader agents and I will look at them before kicking it off.
