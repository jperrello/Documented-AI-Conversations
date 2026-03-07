# Thesis Rewrite Agent Team Plan
**Date:** Saturday, March 07, 2026
**Time:** 1:56 PM

---

## User

Let's start an agent team to rewrite my thesis. They all should have the ability to message each other so they know what other agents are currently writing about and discussing in their sections. So we do not repeat anything or be redundant. These agent teams will each correspond to one different section of my essay. That being the abstract, the introduction, the background, and so on. The only sections you will not redo is Appendix B.
All of these agents will use the academic writing skill with their associated sections. So one will read the academic writing skill and look at the abstract section of the writing skill. The introduction agent will do the same thing but with the introduction section instead. Keep in mind these agents are still communicating with each other as this all happens.

They also have access to the structured writing skill. That way they know how to structure their arguments and claims within their sections. I also want to note that conciseness has become a new guideline. These agents should still operate within their page links but rather than shortening arguments for brevity, they should rather pick one central claim for each section or paragraph and revolve around explaining that.

So the introduction should first engage the reader. Then it should explain our motivations. Then it should go into how we discovered the Saturn writing process. I could start an initial draft for each of these sections and I will upload it here. : @Joey-topic-for-each-section.md This information should also be given to the agents. We are basically starting from zero. So I imagine one agent, their job will just be to clear out the thesis before all the other agents begin working. That way the agents don't have our prior work to influence their behavior. So that first agent just deletes thesis dot text, resets the thesis.

All the other knowledge in this code base still exists. Does this plan make sense to you? lets interview eachother, make a plan, stop the documentation, then begin the plan

## Claude

Documenting.

The plan makes sense at a high level. Here's what I understand and where I have questions:

**What I understand:**
- Agent team, one per section: Abstract, Introduction, Background, Design, Security, Implementation, Evaluation, Discussion, Conclusion
- Skip Appendix B
- A "reset agent" runs first to clear thesis.tex back to a skeleton
- Each agent uses the academic-writing skill scoped to their section + structured-arguments skill
- Agents communicate via messaging to avoid redundancy
- Conciseness guideline: one central claim per paragraph, explain it well rather than cramming multiple points
- Joey-topic-for-each-section.md provides the authorial direction for each section

**My questions for you:**

1. **The moons knowledge graph and code analysis files** — those stay untouched and agents read from them as source material, correct?

2. **Ordering** — some sections depend on others. Background defines terms that Design references. Design defines protocol specs that Implementation builds on. Do you want agents to work in waves (e.g., Background finishes before Design starts), or all simultaneously with messaging to coordinate?

3. **The reset** — should the reset agent preserve the LaTeX preamble/document structure (packages, `\begin{document}`, chapter includes) and just empty the chapter files? Or literally delete everything?

4. **Page limits** — you mentioned agents should stay within page limits. What are the current targets per section?

5. **Security** — your topic doc mentions security as part of Design. Is security its own chapter or a subsection of Design?

6. **Appendix A** — you said skip Appendix B. What about Appendix A?

7. **Grading** — do you still want the grader agent from RALPH to evaluate after all sections are written, or is this a fresh process without that?

Let me hear your answers and I'll ask follow-ups. Then we lock in the plan.

## User

1. yes they are untouched.
2. messaging to coordinate.
3. yes, which reminds me that the agents need to adhere to the formatting instructions as well.
4. The optimal target is 40 pages with an absolute max of 50, not including appendix b.
5. subsection of design
6. appendix a is also rewritten
7. fresh process without

## Claude

Good, that clarifies a lot. Let me read the current thesis structure so I understand what the reset agent needs to preserve and what page distribution looks like.

```tool
[Tool: Read] file_path: /Users/jperr/Documents/Saturn-Thesis/thesis.tex
```

```tool
[Tool: Bash] command: ls /Users/jperr/Documents/Saturn-Thesis/chapters/
```

Now I have the full picture. Follow-up questions:

**8. Chapter mapping** — The current thesis has these chapter files: ch1-introduction, ch2-background, ch3-design, ch4-implementation, ch6-evaluation, ch7-discussion, ch8-conclusion, plus appendix-a and appendix-b. There's no ch5 (scenarios) included in thesis.tex. The agents I'd spin up would be:

| Agent | File | Content |
|-------|------|---------|
| Reset | thesis.tex + all chapter files | Clears content, preserves LaTeX skeleton |
| Abstract | thesis.tex (inline) | Abstract block |
| Introduction | ch1-introduction.tex | Engage reader, motivations, Saturn discovery |
| Background | ch2-background.tex | Service discovery survey, mDNS, DHCP |
| Design | ch3-design.tex | Goals, concepts, protocol spec, security subsection |
| Implementation | ch4-implementation.tex | VLC, router, OpenCode — progressive narrative |
| Evaluation | ch6-evaluation.tex | Three claims, relation to motivations |
| Discussion | ch7-discussion.tex | "What now" — university model, future adoption |
| Conclusion | ch8-conclusion.tex | Summary, contributions, future directions |
| Appendix A | appendix-a.tex | Whatever Appendix A covers |

Does that mapping look right? Is ch5-scenarios dead, or does any of that content belong somewhere?

**9. Page budget** — 40 pages optimal, 50 max. How would you roughly distribute that? My suggestion:

- Abstract: 1 page
- Introduction: 3-4 pages
- Background: 6-8 pages
- Design: 8-10 pages (including security)
- Implementation: 6-8 pages
- Evaluation: 6-8 pages
- Discussion: 3-4 pages
- Conclusion: 1-2 pages
- Appendix A: 2-3 pages

That's roughly 36-48 pages. Does this feel right, or do you want to shift weight?

**10. Tone** — Your topic doc reads conversationally. Should the agents write in formal academic third person ("this work presents...") or a more personal first person ("we designed...")?

**11. What is Appendix A currently about?** I need to know what it covers so the Appendix A agent gets the right direction.

## User

8. that looks good to me.
9. sure
10. Formal academic first person. use I instead of we, write from my (joey) perspective
11. its all the important specs for saturn

## Claude

Good. Now my turn to ask — do you have questions for me about how this will work technically?

Here's what I'm thinking for the inter-agent communication and execution model:

**Execution flow:**
1. **Reset agent** runs first (solo). Clears all chapter files to empty stubs, preserves the thesis.tex preamble/structure and formatting.
2. **All section agents launch in parallel** as a TeamCreate group. Each agent can SendMessage to the others to share what they're covering, flag overlaps, and coordinate terminology.
3. **Abstract agent writes last** — it needs to summarize what all other sections actually say. So it waits for the others to finish, reads their output, then writes.

**Each agent's context includes:**
- The academic-writing skill, scoped to their section's reference file
- The structured-arguments skill
- Joey-topic-for-each-section.md (your authorial direction)
- The moons knowledge graph (graph.json + relevant content files)
- The conciseness guideline: one central claim per paragraph, explain it thoroughly
- Tone: formal academic first person ("I"), Joey's perspective
- Their page budget
- UCSC formatting requirements from thesis.tex preamble

**Questions for you:**
1. After all agents finish, do you want me to compile the PDF automatically so you can review?
2. Do you anticipate this being a single pass, or do you want to review and then send agents back for revisions?
3. Any specific terminology preferences? (e.g., "Saturn protocol" vs "Saturn" vs "the Saturn system")

## User

1. yes
2. i have plans, i cannot say. write this as a final draft.
3. Saturn, but Saturn needs to be clearly defined. The definiton of Saturn is (which can be rewritten but generally) the process of specifically sending mDNS packets containing information to access AI service API endpoints, discovering them, and transferring data back and fourth

## Claude

Understood. Final draft quality, Saturn defined early and clearly, compile PDF at the end.

A few more things I want to nail down:

**12. The Generative AI Acknowledgement** — that block in thesis.tex is hand-written by you. The reset agent should leave it untouched, correct?

**13. Bibliography** — references.bib stays as-is? Agents cite from existing entries and don't invent new ones?

**14. Figures and tables** — the current chapters likely reference figures. Should agents create/reference figures, or write prose-only for now and you'll add figures later?

**15. The "too cheap to meter" framing** — your topic doc opens with this. Should it appear only in the Introduction, or is it a thesis-wide motif that recurs in Discussion/Conclusion too?

**16. Cross-references** — should agents use LaTeX \label/\ref to cross-reference each other's sections (e.g., "as discussed in Chapter 3"), or keep sections self-contained and you'll add cross-refs later?

## User

12. yep
13. sure
14. ill add figures later
15. only intro. you can pull from the report as much as u want
16. add the refs
im ready for the plan, i want that to be documented though.

## Claude

Here is the plan.

---

# Saturn Thesis Rewrite — Agent Team Plan

## Overview
A team of 10 agents rewrites the Saturn thesis from scratch. One reset agent clears the slate, then 8 section agents write in parallel with inter-agent messaging, and finally 1 abstract agent writes after all others finish. Target: 40 pages (50 max), final draft quality.

## Constraints (All Agents)
- **Tone:** Formal academic first person ("I"), Joey's perspective
- **Saturn definition:** Must be clearly defined early — the process of sending mDNS packets containing information to access AI service API endpoints, discovering them, and transferring data between client and service
- **Conciseness:** One central claim per paragraph, explained thoroughly. No cramming multiple points.
- **Sources:** Read from moons/ knowledge graph, Joey-topic-for-each-section.md, structured-arguments skill, and existing references.bib entries only
- **Formatting:** UCSC thesis format (12pt Times New Roman, double-spaced, margins per thesis.tex preamble)
- **Figures:** Prose only. Leave figure placement for later.
- **Cross-references:** Use \label and \ref across chapters
- **Bibliography:** Cite from existing references.bib. Do not invent citations.
- **Do not touch:** Generative AI Acknowledgement, Appendix B, thesis.tex preamble/structure

## Phase 1: Reset Agent (solo, blocks all others)

**Job:** Clear all chapter files to empty LaTeX stubs. Each file keeps its \chapter{} declaration and \label{} but all content is removed.

**Files touched:**
- chapters/ch1-introduction.tex
- chapters/ch2-background.tex
- chapters/ch3-design.tex
- chapters/ch4-implementation.tex
- chapters/ch6-evaluation.tex
- chapters/ch7-discussion.tex
- chapters/ch8-conclusion.tex
- chapters/appendix-a.tex

**Also:** Clear the abstract block in thesis.tex (lines 147-154), replacing with a placeholder comment.

**Does NOT touch:** thesis.tex preamble, Generative AI Acknowledgement, appendix-b.tex, references.bib, moons/

## Phase 2: Section Agents (parallel, with messaging)

All agents launch simultaneously. Each uses:
- academic-writing skill (scoped to their section's reference file)
- structured-arguments skill
- Joey-topic-for-each-section.md
- moons/graph.json → follow edges to relevant content files
- Inter-agent messaging to coordinate coverage and avoid redundancy

### Agent 1: Introduction (ch1-introduction.tex)
- **Page budget:** 3-4 pages
- **Central arc:** Engage reader → motivations → Saturn discovery process
- **Key content from topic doc:**
  - Open with "too cheap to meter" — recontextualized from energy to AI services
  - Joey's original motivation: provision AI access to everyone on campus
  - What Saturn contributes (distinct from abstract)
- **Defines Saturn** clearly here — this is where the term gets its formal definition
- **Messages to:** All agents, broadcasting the Saturn definition and core framing so everyone aligns

### Agent 2: Background (ch2-background.tex)
- **Page budget:** 6-8 pages
- **Central arc:** What service discovery exists → why each was insufficient → why mDNS
- **Key content from topic doc:**
  - Survey: NetBIOS, WS-Discovery, UPnP, DLNA — brief coverage of each
  - Deep coverage of mDNS: Avahi, Bonjour, TXT records
  - Where text completions are used: LibreChat, OpenWebUI, coding agents, VS Code mDNS debugging, voice typing
  - DHCP: how it differs from Saturn on the network stack
  - Explain why Saturn was chosen over these methods
- **Messages to:** Design agent (what terms are defined here), Implementation agent (what protocols are explained)

### Agent 3: Design (ch3-design.tex)
- **Page budget:** 8-10 pages (including security subsection)
- **Central arc:** What Saturn achieves architecturally
- **Key content from topic doc:**
  - Goals of the Saturn architecture
  - Audiences: administrator, developer, end user
  - Concepts: endpoints, beacons, priorities, ephemeral keys
  - Protocol spec: _saturn._tcp.local., TXT record details, expected endpoint paths (/health, /v1/models, /v1/chat/completions)
  - Architecture decisions
  - **Security subsection:** Two threat models (corporate data collection vs. untrusted sysadmin)
- **Messages to:** Implementation agent (what specs are established), Evaluation agent (what claims the design enables)

### Agent 4: Implementation (ch4-implementation.tex)
- **Page budget:** 6-8 pages
- **Central arc:** Progressive narrative of how Saturn was built, not just a list of implementations
- **Key content from topic doc:**
  - VLC extension: proving AI can be added to non-AI-native apps
  - Router: proving AI can be provisioned at network level for all connected devices
  - OpenCode: proving Saturn works with complex coding agents (tool calling, multi-turn conversation, file editing, codebase knowledge)
  - Acknowledge OpenCode is natively AI-based, but VLC shows Saturn isn't only for AI apps
- **Messages to:** Evaluation agent (what was built and can be evaluated), Design agent (any implementation-driven design changes)

### Agent 5: Evaluation (ch6-evaluation.tex)
- **Page budget:** 6-8 pages
- **Central arc:** Three claims proven, tied back to Introduction's motivations
- **Key content from topic doc:**
  - Three claims and how each is proven
  - Claims relate to initial motivations — Saturn works, benefits society, worth configuring at scale
- **Reads from:** moons/claims/ for evidence and methodology
- **Messages to:** Discussion agent (what was proven, what limitations emerged), Introduction agent (confirm claims match stated contributions)

### Agent 6: Discussion (ch7-discussion.tex)
- **Page budget:** 3-4 pages
- **Central arc:** "What happens now" — implications of Saturn being proven
- **Key content from topic doc:**
  - Universities: students connect to endpoints, charged per token, no enterprise subscriptions
  - Apps stop worrying about API key config — Saturn servers assumed present
  - Lowers barrier to AI experimentation
  - Users encounter AI without knowing — it happens on the network transparently
- **Messages to:** Conclusion agent (what themes to echo), Introduction agent (verify discussion answers the motivations)

### Agent 7: Conclusion (ch8-conclusion.tex)
- **Page budget:** 1-2 pages
- **Central arc:** What the reader has read, what Saturn contributed, future directions
- **Key content from topic doc:**
  - Restate contributions
  - Restate what was proven
  - Why Saturn exists and why it matters
  - Future directions
  - Primary motivation of the author, one final time
  - One short paragraph feel
- **Messages to:** Abstract agent (signal completion, share summary points)

### Agent 8: Appendix A (appendix-a.tex)
- **Page budget:** 2-3 pages
- **Central arc:** Standalone protocol specification reference
- **Content:** Rewrite the Saturn protocol spec. An implementor reading only this appendix and cited RFCs should produce a conforming system.
- **Messages to:** Design agent (ensure spec here matches design chapter, no contradictions)

## Phase 3: Abstract Agent (runs after all Phase 2 agents complete)

- **Page budget:** 1 page
- **Reads:** All completed chapter files
- **Writes:** The abstract block in thesis.tex
- **Content:** Overview of Saturn protocol, integration approach, evaluation results, acknowledged security limitations
- **No messaging needed** — all other agents are done

## Phase 4: Compile

- Run pdflatex + bibtex + pdflatex + pdflatex to compile thesis.pdf
- Report any compilation errors

## Inter-Agent Messaging Protocol

Agents message each other for three purposes:
1. **Definition broadcast:** Introduction agent sends Saturn definition + core framing first
2. **Coverage claims:** Each agent announces what topics they're covering so others don't duplicate
3. **Handoff signals:** When an agent finishes, it notifies dependent agents

---

Does this plan look right to you? Anything to add or change before we stop documentation and execute?

## User

I am done documenting. give that to me as a plan so i can clear context and bypass permissions
