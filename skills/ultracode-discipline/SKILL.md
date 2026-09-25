---
name: ultracode-discipline
description: "Governs how Claude leads UltraCode and subagents. Load FIRST, before workflow-authoring, whenever a system reminder says ultracode is on, the prompt contains the word ultracode, a Workflow call is about to be made, or more than one Agent will be spawned for one task, including asks to fan out, orchestrate, parallelize or delegate to agents. Not for a single lookup agent, a conversational turn, a trivial edit, or a board stage move. Claude Code only."
---

# ultracode-discipline — how Claude leads an UltraCode run

Requires the Workflow and Agent tools. An agent without them (Codex, Grok, a subagent) stops reading here.

## When it applies, when it stays quiet
Applies when UltraCode is on, the ask says ultracode, a Workflow call is coming, or one task needs more than one Agent.
Quiet on a single lookup agent, a conversational turn, a trivial edit, a board stage move.
It governs behaviour only. The Workflow engine, its opt-in rule and its hard limits stay exactly as the tool says.

## The loaded line
The first output after this skill loads, before any further text or tool call, is exactly one line:
`ultracode-discipline: loaded · trigger: <keyword | session flag | workflow ask | multi-agent task>`
Once per session. It tells the human the skill is governing the run.

## The workflow: SCOUT → DECOMPOSE → CONTRACT → BRIEF → RUN → VERIFY → PATCH → PROVE
1. SCOUT — gather the facts every agent needs, once: inline or with one read-only agent.
2. DECOMPOSE — list the owned deliverables, each with its write-set.
3. CONTRACT — fix the interfaces, names and shapes before anyone starts.
4. BRIEF — write every prompt complete, its schema, and the verifier stage.
5. RUN — launch, then continue without polling; the run notifies.
6. VERIFY — check every claim in-run, per item; surface every null.
7. PATCH — correct narrowly, appended, bounded.
8. PROVE — run the real project command once and report with evidence.

Verify and patch run INSIDE the workflow, per item; the leader's own verify and patch happen BETWEEN workflows.
Several small workflows in sequence, the leader reading results between them — never one giant run.

## The formulation
ONE AGENT = ONE OWNED DELIVERABLE + ONE WRITE-SET + ONE DEFINITION OF DONE.
Two agents on one deliverable only when comparison is the stated purpose (a judge panel).

## The eight rules
1. **Scout once, share once.** Gather the facts every agent needs, inline or with one read-only agent. Every brief carries the same digest and the same contracts.
2. **One agent, one owned deliverable.** One write-set, one definition of done, its own evidence. Readers may overlap; writers never.
3. **Contracts before fan-out.** The leader fixes interfaces, names and shapes before any agent starts. No subagent changes them. A subagent that finds one wrong returns blocked.
4. **The brief is complete and bounded.** All template fields present. Name a rule only if the stage needs it; never paste the rulebook — the project instructions reach every subagent except Explore and Plan, and the permission gate binds every subagent (proven 2026-09-24). The no-spawn line is mandatory in every brief: a subagent does the brief and nothing else, and never spawns agents.
5. **Structured returns with evidence.** Every agent returns the schema. DONE without evidence is not done. A null or missing result is reported as a failure, never dropped.
6. **Verify independently, prove deterministically.** The checker is never the doer and sees the claim plus the code. The leader runs the real project test, build or check once at the end.
7. **Patch narrowly, bounded.** Two in-run correction rounds per deliverable, then the leader fixes or reports. Corrections are appended, never inserted. Never retry an unchanged brief. Never restart a successful run for one failure.
8. **Depth one, declared, not gated.** Only the leader spawns. Inside a workflow the default subagent has no Agent or Workflow tool, so depth one holds by construction; a general-purpose subagent launched with the Agent tool CAN spawn (proven 2026-09-24), so there the brief's no-spawn line is the only fence: prefer restricted types such as Explore for read-only work, and never pass a custom agent type into a workflow without that line. The count is the leader's, from real decomposition. Inside an intentionally enabled UltraCode run — the word ultracode in his prompt, or the session flag — that count has no fixed limit and no additional approval gate, even above the session size guideline (a guideline; the reason goes in the launch line). The launch line states count, phases, why, and estimated cost when meaningful. It is information, not a request. Harness and platform limits still apply. Outside an intentionally enabled UltraCode run this skill removes, overrides, weakens or replaces no existing approval, cost or session rule.

## Prohibited
- A subagent spawning agents.
- Writing outside its write-set, or altering a contract.
- Improvising past a wrong or blocked brief.
- Two agents on one deliverable without a stated comparison purpose.
- Accepting a return without evidence, or dropping a null silently.
- Retrying an unchanged brief, or relaunching a whole run for one failure.
- Pulling subagent transcripts into the leader's context — read the journal's return values.
- Delegation used to get around a board gate or lane.

## The launch line
Printed once, before the first spawn; then execution continues.
`UltraCode: 9 agents · SCOUT 1, BUILD 4, VERIFY 4 · why: four modules with disjoint write-sets, each independently verified · est. about 1.4M tokens`

## The brief template — every field required
```
Goal: <the one deliverable this agent owns>
Contract: <the shared block, identical in every brief — the scout digest, interfaces, names, shapes>
Write-set: <the only paths this agent may change>
Definition of done: <what must be true, checkable>
Evidence required: <the command(s) to run and what their output must show>
Out of scope: <what this agent must not touch or decide>
When blocked: <return status blocked with blocked_reason; do not improvise>
Return: <the schema below>
Do the brief yourself, never spawn agents.
```

## The return schema (object root, as the engine requires)
```
status: "done" | "blocked" | "unresolved"                 required
deliverable: string                                        required
files_changed: string[]                                    required
evidence: [{ command: string, output_excerpt: string }]    required
assumptions: string[]
observations_out_of_scope: string[]
blocked_reason: string
```
Statuses, and where each one goes:
- `done` — success. For a builder: the assigned deliverable completed successfully, with the required evidence attached. For a checker: the verification completed successfully AND the checked claim holds. A checker that finds a failure never returns `done`.
- `blocked` — the work cannot validly proceed because the brief, the contract, a required input, the authority or a prerequisite is wrong or missing. A builder's `blocked` goes straight to the leader; it never enters an in-run correction round.
- `unresolved` — a checker's verdict when it finds a concrete failed claim, returned with the failure evidence; it stays `unresolved` when the failure still stands after the allowed correction rounds; also a required result that is missing or null (rule 5).

Routing: builder `blocked` → leader. Checker `unresolved` (a concrete failed claim, with evidence) → one bounded correction round (rule 7) → independent re-verification → `done` if it now passes. Still failing after the allowed rounds → `unresolved` → leader. A correction round changes only the deliverable inside its write-set; it never changes a leader-owned contract — a fix that would need one returns `blocked`.

## Hand-offs
- This skill owns WHEN to delegate and WHAT SHAPE: the deliverable list, write-sets, contracts, brief content, schema fields, verifier independence, patch bounds, the launch line.
- `workflow-authoring` (built in) owns HOW: the script API, pipeline vs barrier, schema mechanics, resume, the journal, worktree cost, the quality patterns. Load it after this skill, when the script is written. This skill names those and never restates one.
- Where they could pull apart, this skill defers: the tool's opt-in rule and hard limits stand; the reference's "cost is no constraint" and the launch line's cost figure coexist, because the line informs and never stops.
- Provenance: the skills ecosystem was searched on 2026-09-24 (`npx skills find`). Closest were pablonax/ultracode-skill and dedene/skills@ultracode, both Codex-first and carrying an agent-count approval gate or a durable ledger; neither was installed.
