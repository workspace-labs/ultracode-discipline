---
name: ultracode-discipline
description: "How Claude leads an UltraCode run. Load it first — before any file is read or written and before workflow-authoring — when a system reminder says ultracode is on or the prompt contains the word ultracode. Not for an ordinary Agent spawn, a parallel read-only lookup, a Workflow the user asked for without ultracode, a conversational turn, a trivial edit, or a board stage move. Claude Code only."
---

# ultracode-discipline

Requires the Workflow tool. The built-in `workflow-authoring` reference owns the script mechanics
(schemas, pipeline vs barrier, resume, journal, budget, quality patterns) — load it after this and
never restate it. This file adds only what the reference does not say.

## Six rules
1. **One agent = one deliverable + one write-set.** Writers never overlap; readers may.
   Two agents on one deliverable only for a stated comparison (judge panel).
   Count write-sets from the decomposition, never from the size of the change: parts that can
   be built without reading each other's output are separate write-sets, however few lines
   each one is. Three or more → fan out; that is what the human typed `ultracode` for, and
   "it would cost more than it saves" is never a reason to stay inline. Fewer than three →
   say so (rule 6) and do it yourself.
2. **Contracts before fan-out.** The leader fixes interfaces, names and shapes first.
   No subagent changes one; a subagent that finds one wrong returns `blocked`. The leader's own
   edits to anything the agents read land before the fan-out or after the run, never during it.
3. **Checker ≠ doer.** Every *write* deliverable gets a verifier that did not build it and sees
   the claim plus the code. Read-only deliverables get none. For audits use the reference's
   adversarial-verify pattern instead of one checker.
4. **Blocked means stop.** A wrong or incomplete brief is returned as `blocked` with a reason,
   never improvised past. A null return is a failure, reported, never dropped.
5. **Correct by appending.** A failed check gets at most two in-run correction rounds, each
   appended to the script and re-verified. Never retry an unchanged brief. Never relaunch a run
   for one failure.
   The leader runs the project's real test/build once at the end.
6. **Declare the launch; a refusal is a no.** The script's `meta.description` IS the launch
   line — the approval dialog shows it to the human before anything runs — and reads exactly:
   `UltraCode: <n> agents · <phases> · why: <the decomposition>`
   A fan-out through direct Agent calls writes that same line as text before the first call;
   fewer than three write-sets get `UltraCode: 0 agents · <n> write-set(s) — inline` as text
   and no fan-out.
   A refused Workflow call is
   the human declining that fan-out: never re-run the same agents through the Agent tool or a
   smaller script — ask, or do the work inline and say the fan-out was declined.

## Every brief
```
Goal: <the one deliverable this agent owns>
Contract: <identical block in every brief — scout digest, interfaces, names, shapes>
Write-set: <the only paths this agent may change>
Done when: <checkable>
Evidence: <command(s) to run; what the output must show>
Out of scope: <must not touch or decide>
Blocked: <return status blocked + blocked_reason; do not improvise>
Return: <the schema below, nothing else — no report, no handoff>
Do the brief yourself; never spawn agents.
```

## Return schema
```
status: "done" | "blocked" | "unresolved"           required
deliverable: string                                  required
files_changed: string[]                              required — [] for a checker
evidence: [{ command, output_excerpt }]              required — [] only when blocked
findings: string[]                                   checker only — its failed claims: file:line — what — what the contract requires
blocked_reason: string
observations_out_of_scope: string[]
```
`done` = succeeded with evidence (a checker: the claim holds). `blocked` → leader.
`unresolved` = a checker found a failure (in `findings`), or still fails after the correction
rounds → leader.

## Scope
Inside an UltraCode run the agent count comes from the real decomposition and needs no extra
approval; the size guideline is a guideline. Outside an UltraCode run this skill changes nothing.
Never use delegation to get around a board gate or lane.
Keep the leader's normal context to structured returns / journal values, not subagent transcripts.
Use transcripts only as the reference's diagnostic fallback when the journal is unavailable.
