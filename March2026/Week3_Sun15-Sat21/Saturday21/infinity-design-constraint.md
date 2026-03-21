# Infinity Design Constraint — Character-Mediated Abstraction
**Date:** Saturday, March 21, 2026
**Time:** 4:08 PM PDT
---

## User

I have an idea for the infinity system. I just had a breakthrough. Infinity should be separated between the outside of infinity, where the user can actually interact, and the inside, which is infinity that runs autonomously with no user interaction. The overseer sees everything, sees both of these two.

The user and coach are on the outside, and the garden is on the outside. outside as well.

## Claude

Documenting.

This is a significant architectural shift. Currently INFINITY.md puts everything inside the Overseer's box — User, Coach, the Void, the Garden, all wrapped in one boundary. Your breakthrough splits that into two distinct zones:

**Outside** (user-facing, interactive):
- User
- Coach
- The Garden

**Inside** (autonomous, no user interaction):
- The Void
- Teams
- Players navigating
- Ralph loops grinding

**Overseer** spans both — it sees the outside interactions AND the inside autonomous work.

The Garden being on the outside is interesting. Right now it's drawn at the bottom of the Void. Moving it outside means the Garden is something the user can see and touch — task completion surfaces back to the user's world, not buried inside the autonomous system. A player's journey ends by *exiting* the Void, not by reaching a destination within it.

This also clarifies Coach's role: Coach is a boundary agent. It takes something from the outside (a task) and sends it into the inside (the Void). The Garden is the other boundary — where things come back out.

What does the Overseer do differently now that it explicitly straddles both zones? Does it have different capabilities or behaviors on each side?

## User

The purpose of infinity is to be as abstracted as possible. Current orchestrators like Gastown or OpenClaw are too complex for an average person to understand. A student counselor would never want it. A chemical engineer offers difficult tasks and expects a one-shot solution.

This is an inherent problem with AI orchestrators. And this problem is inherent because natural language is inherently vague. We humans so rely on context. The whole idea behind characters of infinity is to make people understand who they are talking to.

Right now that is confusing. We need to develop an AI orchestrator slash integrated development environment that attempts to be as abstracted for a user as possible.

## Claude

Documenting.

You're identifying the core design constraint for Infinity: abstraction as the primary feature, not power.

The character system in INFINITY.md already points in this direction — Coach as the single point of contact, the void hidden behind it. But you're saying something sharper: the reason for characters isn't just narrative flavor, it's a UX solution to the vagueness problem. When a user talks to "Coach," they understand the contract — I give you a task, you handle it. They don't need to know about teams, ralph loops, mDNS, or Saturn services.

The gap you're pointing at: existing orchestrators expose their complexity. They require the user to understand the system to use the system. Infinity should invert that. The user understands the *character*, and the character understands the system.

This means the abstraction layer isn't just Coach's API — it's Coach's *personality*. The interface IS the character. A student counselor talks to Coach the same way they'd talk to a colleague: "I need to figure out which students are at risk of dropping out this semester." Coach doesn't ask them to configure a pipeline or choose a model. Coach just takes the task and sends a player into the void.

The question this raises: how deep does the abstraction go? Does the user ever see inside the void, or is it fully opaque? And when something fails — a player gets lost — how does Coach communicate that without exposing the machinery?

## User

No, i would rather us write a clear defined design constraint. therefore we can directly make a claim about what inifinity is providing. think about it in the context of current work (use web search sub agents). we need this design goal in order to conduct research on a claim, address threat models, or make arguments

## Claude

```tool
[Tool: Agent] Research AI orchestrator complexity — search for criticism/usability problems with current AI orchestrators (LangChain, CrewAI, AutoGen, etc.), abstraction gap research, mainstream adoption failures, simplified interface attempts
```

```tool
[Tool: Agent] Research NL ambiguity in AI systems — search for natural language ambiguity in agent systems, common ground/grounding theory in human-AI interaction, personas as interface abstractions, mental models in HCI for AI agents
```

Here's what the research supports. Let me propose a design constraint.

---

**Design Constraint:**

> **Infinity provides a character-mediated abstraction layer over autonomous AI orchestration, such that the user never needs to understand the system in order to use it.**

---

This is a testable claim. Here's why it holds up against the research:

**The problem it addresses:** 79% of multi-agent system failures originate from specification and coordination issues — communication, not computation. Existing orchestrators (LangChain, CrewAI, AutoGen) force users to understand the system to use it. No-code platforms (Zapier, Lindy) remove friction but hit hard capability ceilings. Nobody occupies the middle: full orchestration power, zero user-facing complexity.

**Why characters, not just a "simple API":** Research on metaphor-fluid conversation design (2025) shows that personas do real cognitive work — they give users a mental model for what the system can and cannot do without documentation. "Coach" isn't a branding choice. It's an interface contract. The user knows: I give you a task, you handle it. Clark & Brennan's grounding theory (1991) says communication requires shared understanding. Characters *are* the shared understanding.

**Why this is distinct from existing solutions:**
- LangChain/LangGraph: abstractions for developers, not users
- No-code builders: visual workflows that cap out at simple automation
- Anthropic's own guidance: "start with raw APIs, add complexity only when needed" — correct for developers, irrelevant for the student counselor or chemical engineer you mentioned

**What it enables:**
- **Research claims:** "Character-mediated abstraction reduces the grounding gap in human-AI orchestration" — testable via user studies
- **Threat models:** What happens when the character's implied capabilities exceed the system's actual capabilities? (The anthropomorphism trap — users of more humanlike agents try to exploit capabilities that weren't signaled)
- **Arguments:** Against framework complexity (86% failure rate in agentic systems, 52% of developers don't use agents at all per Stack Overflow 2025)

Do you want to refine this constraint, or lock it in and update INFINITY.md?

## User

its also important to think about the 10 usability heuristics too www.nngroup.com/articles/ten-usability-heuristics/ . I want you to lock these in and update infinity.md . do you mind making a shortermd version of the ten usability heuristics in this repo so i can refer to it

## Claude

```tool
[Tool: WebFetch] https://www.nngroup.com/articles/ten-usability-heuristics/ — Extract all 10 usability heuristics with names and one-sentence descriptions
```

Created `HEURISTICS.md` — a short reference of all 10 Nielsen heuristics.

Updated `INFINITY.md` with a new **Design Constraint** section placed between Premise and What Exists. It locks in the constraint statement, the research grounding (Clark & Brennan, metaphor-fluid design, anthropomorphism trap), the stats (79% failure from communication, 86% agentic failure rate, 52% developer non-adoption), and a direct reference to `HEURISTICS.md` with the three most relevant heuristics called out: visibility of system status, match between system and real world, aesthetic and minimalist design.

## User

I am done documenting.
