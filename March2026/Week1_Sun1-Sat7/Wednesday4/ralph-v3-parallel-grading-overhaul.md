# RALPH v3: Parallel Grading and 0-100 Scale Overhaul
**Date:** Wednesday, March 4, 2026
**Time:** 4:48 PM PST
---

## User

lets think about the ralph loop i have setup in this repository. what comes to your mind while exploring the system, and the skills/agents associated with it. assume i cant answer the question either

## Claude

Let me explore the RALPH system across the repository.

```tool
[Tool: Glob] pattern: **/*ralph*
[Tool: Glob] pattern: .claude/agents/*.md
[Tool: Glob] pattern: .claude/skills/**/*.md
[Tool: Grep] pattern: ralph|RALPH, glob: *.md
```

```tool
[Tool: Read] RALPH_PROMPT.md
[Tool: Read] ralph.sh
[Tool: Read] .claude/agents/grader.md
[Tool: Read] .claude/skills/academic-writing/SKILL.md
[Tool: Read] rewrite_notes.md
```

```tool
[Tool: Read] .claude/skills/structured-arguments/SKILL.md
[Tool: Read] moons/meta/writing-toolchain.md
```

Here's what I see in the RALPH system:

## What it is

A **headless autonomous revision loop** for your thesis. `ralph.sh` is a 4-line bash script that pipes `RALPH_PROMPT.md` into Claude in a `while true` loop with `--dangerously-skip-permissions`. Each invocation is stateless — the filesystem is the only persistence between runs.

## The architecture

```
ralph.sh (infinite loop)
  └── Fresh Claude session each iteration
        ├── Phase 0: Read state (rewrite_notes.md, rubric, moons)
        ├── Phase 1: Spawn grader agent (read-only, Opus, scores 7 sections × 6 criteria)
        ├── Phase 2: Decide (pick worst non-CLEAN section)
        ├── Phase 3: Revise (invoke academic-writing skill, one section only)
        ├── Phase 4: Log (append to rewrite_notes.md, update status table)
        └── Phase 5: Exit (shell script re-invokes)
```

## The pieces

- **`RALPH_PROMPT.md`** — the master agent prompt. Defines five phases, chapter contracts (what each chapter inherits/establishes/hands off), and hard rules (one cycle per invocation, never self-evaluate, maintain truth with moons).

- **`grader.md`** (`.claude/agents/`) — a read-only subagent with only Read/Grep/Glob tools. Scores C1-C6 (structural completeness, contribution vs. claim, ordering, design intent, trade-off honesty, value density). Returns a score matrix, priority ranking, deep-dive on the worst section, and cross-cutting issues. It has plateau detection — if a score hasn't moved in 3+ passes, it flags the structural root cause.

- **`academic-writing` skill** — 19 writing principles (concreteness, old-to-new, topic strings, one canonical home per concept, stress positions) plus section-specific reference files. Mandatory draft→evaluate→revise loop.

- **`structured-arguments` skill** — labeled argument scaffold (V1/V2/V3, A-series, AV, CA, B) with a registry in moons. Maps your three claims to formal views.

- **`rewrite_notes.md`** — the inter-session memory. Contains a section status table (scores + CLEAN/NEEDS ANOTHER PASS) and a detailed run log. This is the only thing connecting one invocation to the next.

- **`moons/`** — ground truth. The grader validates claims against it; the revision agent must never contradict it.

## What's interesting about the design

**Separation of evaluation and revision.** The grader is a distinct agent that can't edit files. The revision agent can't score itself. This prevents the common failure mode of an agent marking its own work as good.

**Chapter contracts** are the most architecturally distinct piece. Each section has an Inherits/Establishes/Hands-off contract that prevents the #1 problem in AI-written academic work — concept re-introduction across chapters. The grader checks for it (C6), and the revision agent is explicitly told to delete or cross-reference anything from the Inherits column.

**Evolution is documented.** RALPH v1 targeted AI detection scores (Sapling API). After 8 runs, you abandoned that signal as noisy and pivoted to structural quality rubrics. The writing principles in Run 0 of `rewrite_notes.md` are what survived from the detection-era runs — lessons learned, not wasted work.

**Termination condition.** All sections 5/5 on all criteria with no cross-cutting issues → appends `## RALPH COMPLETE` → next invocation reads it and stops. The current state shows scores in the 22-28 range (out of 30), all sections marked CLEAN, meaning the system has converged but hasn't hit perfection.

## What stands out as potential concerns

1. **Score ceiling/plateau.** Several criteria sit at 3-4 across multiple passes. The grader has plateau detection, but the revision agent may lack the structural insight to break through — surface edits can't fix deep organizational problems.

2. **Single-section-per-cycle granularity.** Cross-cutting issues (redundancy between chapters, citation role duplication) require coordinated edits across multiple files, but the loop only targets one section per invocation. The prompt handles this ("if every section is CLEAN but cross-cutting issues remain, target those"), but it's the harder case.

3. **No rollback mechanism.** If a revision makes things worse, the next grader run will catch it, but there's no automatic `git revert`. The loop trusts forward progress.

4. **`--dangerously-skip-permissions`** means the agent has full autonomy. The safety rails are entirely in the prompt (no human present, conservative choices, never contradict moons). This is deliberate — the whole point is unattended operation.

## User

I wanted to run in dangerously skip missions mode. Yeah, the score ceiling and the plateau is actually the worst thing. I actually have an agent working on a solution right now where the grader reads the notes that the previous agents left. My goal with this was to understand if it's getting stuck. Opus is powerful enough to realize, oh, this is my eighth pass.

I should grade a little harsher than I did before. With that being said, I am probably going to start my next Ralph session with a completely fresh rewrite notes. That way we could truly start this Ralph loop from the beginning and see how well it modifies. I think this should solve the plateau because then Opus could analyze a paper that's been read so much and not just be exhausted that the Ralph loop has been going on so long, but look at it as a more thought out draft.

And it's already tried so many different things and you need to look at it under a different type of microscope. Perhaps a score from one to five is too generous. Perhaps we should use floating numbers or a total of 100 to convey minor changes or little unique things that go above and beyond.

Be harsh in the grading. An A needs to be earned. An A plus is not something that is just handed out. It is something that is earned and impresses humans. Yeah, the single section thing also kind of sucks. Like, I don't really know why that's there. Do you have a better idea for a solution? Rollback isn't necessary.

## Claude

The single-section constraint exists because of context window management. Each invocation is a fresh Claude session — it has to read the thesis, spawn the grader (which also reads the thesis), get the report back, then do the actual editing. If you let it touch multiple sections, you're asking it to hold the full grader report plus multiple sections' worth of edits in one session. The prompt was being conservative about what fits.

But that's a solvable problem. Two approaches:

**Option A: Multi-section with priority cap.** Remove the "pick one section" rule. Instead, let the revision agent work through the priority ranking top-to-bottom until it's addressed all HIGH issues across all sections. The grader already ranks sections — the revision agent just stops too early. The risk is a single session running long and degrading in quality toward the end. You could cap it at 2-3 sections per cycle instead of 1.

**Option B: Parallel section agents.** After the grader returns, the orchestrator spawns separate revision agents per section — each gets only its own deep-dive issues and the relevant chapter file. They work in isolation (worktrees or just non-overlapping files). The orchestrator logs all results. This is the cleaner architecture — it matches how the thesis actually works, since chapter contracts already define what each section owns. Cross-cutting issues get their own dedicated pass.

Option B is better. The single-section rule was solving a context problem, not a writing problem. Parallel agents don't have that constraint.

On scoring — the 1-5 scale isn't just coarse, it's psychologically anchoring the grader toward the middle. A 3 feels "adequate" and a 4 feels "good enough." Two changes that work together:

1. **Switch to 0-100 with explicit anchor points.** 90+ means a human reviewer would have no structural feedback. 70-89 means clear issues but functional. Below 70 means structural problems that undermine the section's purpose. This gives the grader room to distinguish between "two minor issues" (82) and "one minor issue" (87) instead of both landing at 4.

2. **Grade to a standard, not relative to previous runs.** The current grader prompt says "score relative to what remains, not what a first draft would look like." That's the plateau machine — it's literally told to grade on a curve. Kill that line. The rubric should describe what 90+ looks like in absolute terms, and the section either meets it or doesn't. The revision history in `rewrite_notes.md` is useful for plateau detection, not for calibration.

The fresh `rewrite_notes.md` idea is sound. The current one is 80KB+ of accumulated history. That's not context — that's baggage. The grader reads it for calibration and unconsciously anchors to previous scores. A clean start forces it to evaluate the thesis as-is.

Want me to implement these changes — the 0-100 scale, the multi-section/parallel architecture, and harsher absolute grading — into the prompt and grader agent?

## User

Yeah, I want to add in that the parallel agents have the academic writing skill that is actually designed to be sectioned off anyways. So these sub-agents, we could direct the main orchestrator model to give each sub-agent their respected writing skill. I also think that the main agent in the Ralph loop needs to serve as a final pass for what the subagents provide, not just put them all back together. Like, it needs to do the job of formatting, but also making sure that the essay as a whole comes together and there isn't, like, conflicting ideas.

It is basically the final check on the initial grader score. For point one, you brought up a good point of putting guidelines between what different percentages metaphorically require. I want you to go deeper to around the 25 percentile. That way, the grader has more details on what these percentages mean. I mean, even give details down to the individual digits from 70 to 80 or 80 to 90. You know, why would a grader pick an 82 versus an 89?

That's a drastic difference. So I really like what you said about point one, but you just need to dig a little deeper into that idea. Point two is okay. Devise a plan to implement this.

## Claude

```tool
[Tool: EnterPlanMode]
```

Let me explore the current files to understand all the pieces before designing the plan.

```tool
[Tool: Agent] subagent_type: Explore, description: Explore RALPH prompt and grader
```

```tool
[Tool: Agent] subagent_type: Plan, description: Design RALPH v3 implementation plan
```

```tool
[Tool: Read] academic-advisor-prompt.md
[Tool: Glob] pattern: chapters/ch*
```

Good — chapter files are modular (`chapters/ch[N]-[name].tex`), confirming parallel agents can work on separate files without conflicts. The `academic-advisor-prompt.md` has no scale references to update — it just says "structural grade" generically.

Now let me write the final plan.

```tool
[Tool: Write] /Users/jperr/.claude/plans/humming-orbiting-llama.md — full plan for RALPH v3
```

```tool
[Tool: ExitPlanMode]
```

## User

[User approved the plan]

## Claude

Let me implement this. I'll start with the independent pieces in parallel, then do the files that depend on them.

```tool
[Tool: TaskCreate] Create evaluation.md reference file
[Tool: TaskCreate] Rewrite grader.md with 0-100 scale
[Tool: TaskCreate] Rewrite RALPH_PROMPT.md with parallel architecture
[Tool: TaskCreate] Reset rewrite_notes.md and update academic-advisor-prompt.md
[Tool: TaskCreate] Verify all changes end-to-end
```

**Step 1: Created `evaluation.md`** — new section reference file at `.claude/skills/academic-writing/references/evaluation.md` with 4 required elements and 10 rules.

**Step 2: Rewrote `grader.md`** — Complete rewrite with 0-100 absolute scale, granular anchors at every 2-3 points from 70-100, removed relative calibration, multi-section deep-dives, completion check section.

**Step 3: Rewrote `RALPH_PROMPT.md`** — v3 architecture with parallel section agents (Phase 3), integration pass (Phase 3.5), updated logging (Phase 4), chapter file mapping table.

**Step 4: Reset `rewrite_notes.md`** — Backed up old file as `rewrite_notes.v1.md`, wrote clean template with 0-100 columns. Added scoring scale note to `academic-advisor-prompt.md`.

**Step 5: Verified all changes** — Read all 5 modified files end-to-end, confirmed coherence, ran mental dry-run walkthrough of the full loop.

All done. Here's what was implemented:

**5 files changed:**

1. **`grader.md`** — Complete rewrite. 0-100 absolute scale with anchors at every 2-3 point interval from 70-100. Removed relative calibration ("score relative to what remains"). Multi-section deep-dives instead of worst-only. Completion check section added.

2. **`RALPH_PROMPT.md`** — v3 architecture. Phase 3 spawns parallel `general-purpose` agents (one per chapter needing work), each isolated to its own `.tex` file with its own reference file and contract row. Phase 3.5 is the integration pass where you (the orchestrator) read the full thesis and catch cross-chapter breaks. Always compiles PDF on exit.

3. **`evaluation.md`** — New file. Fills the gap in the section routing table. 4 required elements, 10 rules, focused on evidentiary standards (methodology before results, no interpretation, threats state consequences).

4. **`rewrite_notes.md`** — Clean slate. Old one backed up as `rewrite_notes.v1.md`. 0-100 columns with Avg instead of Total.

5. **`academic-advisor-prompt.md`** — Added line 5 to delivery section pointing to grader's scale.

## User

/document this whole conversation before this point and after

## Claude

Documenting. I'll capture the full conversation — everything before this point and everything going forward.

## User

I am done documenting
