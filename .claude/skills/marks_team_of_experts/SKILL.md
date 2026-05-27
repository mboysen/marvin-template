---
name: marks_team_of_experts
description: |
  Spawns 5 PhD-level subagent reviewers in parallel for a Pareto-style
  multi-perspective review of a system, decision, codebase, or finding.
  Each agent gets the same shared context plus a distinct expert lens;
  they return ranked 3-5 recommendations; the skill synthesizes a unified
  top 5-10 with a single "if you do ONE thing this week" closer.
  Triggers on: "team of experts review", "PhD-level review", "Pareto
  review of X", "what's the 20% that gives 80%", "multi-agent review".
  Use proactively when the user faces an ambiguous architectural choice,
  a complex post-mortem, or a "what should we do next" question that
  spans multiple domains.
---

# Mark's Team of Experts

A multi-agent Pareto review pattern. Born from the 2026-05-27 AI Precise
session that needed perspective across perception, mechatronics, software
architecture, NVIDIA edge platforms, and software engineering process —
no single agent had the right lens for all of it.

## When to invoke

- User explicitly says "team of experts", "PhD-level review", "Pareto review", "multi-expert review", or invokes `/marks_team_of_experts`
- User asks "what would give us 80% of the lift with 20% of the effort"
- After a substantial body of work, when picking next priorities across domains
- Post-mortem on a multi-domain incident
- Architectural decision with cross-cutting implications

## When NOT to invoke

- Simple single-domain question (use a focused agent or answer directly)
- User just wants more depth in ONE area (use the appropriate single specialist)
- Quick question with a clear answer

## Default expert lenses (5 PhD agents)

| # | Lens | What they cover |
|---|---|---|
| 1 | **AI Perception Engineer** | Models, detection, tracking, calibration, threshold strategy, train/test gap |
| 2 | **Mechatronics Engineer** | Physical/control/safety correctness, geometry, actuation, sensor fusion, E-stop, bus utilization |
| 3 | **Software Architecture** | System boundaries, drift control, single source of truth, deploy patterns, technical debt |
| 4 | **NVIDIA Edge Computing** | Jetson runtime, TRT/CUDA, camera capture, power/clocks, container base, per-stage latency budget |
| 5 | **Software Engineering Process** | Test discipline, CI/CD, defect prevention, contract enforcement, cache invalidation |

**Customize when appropriate.** If reviewing a content strategy, swap in "Editorial Director" + "Audience Researcher" + "Distribution Strategist" etc. Always 3-5 distinct lenses, never overlapping.

## Workflow

### Step 1 — Confirm scope (one round of clarification max)

Briefly confirm with the user:
- **What system/decision/codebase to review?** (file paths, version, scope)
- **What lenses?** (default 5 PhDs, or user-specified)
- **What's the Pareto target?** (default: "5-10 items that would deliver 80% of the achievable improvement with 20% of the engineering effort")

Skip clarification if the user's request makes scope unambiguous.

### Step 2 — Gather shared context

Identify the files every expert needs to read:
- Project handoff / state docs (HANDOFF.md, README.md, state/current.md)
- Decision log entries (state/decisions.md, ADRs, design docs)
- Recent test/measurement data (operator runs, benchmarks, JSONL logs)
- Git log overview (`git log --oneline -25`)

List the **files most relevant to each individual lens** so each agent reads the right code without you having to specify mid-stream.

### Step 3 — Spawn agents in parallel (background)

Use the `Agent` tool with:
- `subagent_type: general-purpose`
- `model: opus`
- `run_in_background: true`
- Send all spawn calls in **a single message** so they actually parallelize
- Each prompt includes:
  1. Identity ("You are a PhD-level X engineer")
  2. Shared context (read these files, ground-truth numbers from latest test)
  3. Lens-specific file pointers (~5-10 paths)
  4. **Constraint to output: 3-5 ranked picks**, each with: `What` / `Why` / `Effort (small/medium/large)` / `Expected Impact (quantitative if possible)` / `What this displaces`
  5. **End with one sentence**: "If I could only do ONE thing this week, it would be ___"
  6. Anti-flattery directive: "Push back if today's choices were wrong. No flattery. Be honest about tradeoffs."

### Step 4 — Surface progress, wait for all completions

After spawning, briefly tell the user "5 agents running in parallel; will synthesize once all return." Do NOT poll output files. The harness notifies on each completion automatically. Continue any non-overlapping work in the meantime.

### Step 5 — Synthesize once all 5 return

Build a unified top 5-10 list:

1. **Cross-reference**: items mentioned by 2+ experts get priority (high-confidence). Items mentioned by 1 expert with strong rationale also qualify.
2. **Deduplicate**: experts often phrase the same recommendation differently. Group into single entries.
3. **Surface contradictions honestly**: if Perception says "do X" and Architecture says "don't do X yet", call it out.
4. **Rank by impact/effort ratio**: not by expert vote count.
5. **Tag each pick with its source lens(es)** so the user knows the provenance.
6. **End with**: "If you do ONE thing this week, it's ___" — your synthesized pick, not just whichever expert spoke loudest.

Output structure:

```
## Synthesized top N picks

### #1 — <short title>  [Source: Lens-A, Lens-B]
- What:
- Why (cross-cutting):
- Effort:
- Expected impact:
- Notes / contradictions:

### #2 — ...
```

End with a "What I'd explicitly NOT recommend" subsection capturing any
anti-recommendations the experts agreed on (e.g., "don't retrain", "don't
add CI yet").

## Lessons learned (from the canonical 2026-05-27 run)

- **Heredoc the prompts when invoking the Agent tool**. Plain `ssh host 'cmd'` patterns can lose leading shell content; the same risk exists if you craft prompts via Bash. Use the Agent tool directly with the full prompt string.
- **All 5 spawn calls in one message** to actually parallelize. Sequential `Agent` calls execute one at a time.
- **Give each agent specific file paths**, not "look around the codebase". Saves their context budget for the analysis, not the search.
- **Make the output format mandatory** (the 5-bullet ranked picks + ONE-thing closer). Without it agents return prose that's harder to synthesize.
- **Don't read the agent output file directly via Read/tail** while the agent runs — it's the JSONL transcript and will overflow your context. Wait for the notification, then process the result block in that notification.
- **Honor the user's Pareto framing**. The goal is 5-10 actionable items, not exhaustive analysis. Push back on agents that return 12 items "ranked".
- **Synthesis is YOUR job, not a 6th agent's.** The cross-cutting view is what the user is paying for; an agent doing it would just be another lens.

## Anti-patterns to avoid

- **Recursive expert spawning** (agents that themselves spawn agents). Keep it one level deep.
- **Overlapping lenses** (two "AI experts" with different specialties). Pick distinct, non-overlapping perspectives.
- **More than 5 agents**. Diminishing returns; 5 is the sweet spot between coverage and synthesis tractability.
- **Asking agents to converge with each other**. They run independently; convergence happens in the synthesis step.
- **Sycophantic prompts**. Explicitly tell each agent to push back. Reviews where everyone agrees with everyone produce no signal.

## Output to user

Once all 5 return + synthesis is written, present:
1. The unified top N picks (with source lens tags)
2. Cross-expert contradictions surfaced honestly
3. The single "ONE thing this week" pick
4. The "explicitly NOT recommend" anti-list

Keep the synthesis tight. The agents produced ~5000-10000 words combined;
the user should get ~800-1500 words of synthesis, not a digest of the
raw output.
