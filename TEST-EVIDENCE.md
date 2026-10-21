# Test evidence — version 2.2 (2026-10-21)

A retest of 2.1 on Claude Code 2.1.305 with the Claude Fable 5.1 model (macOS), and the fixes it produced. Every run was a **fresh headless Claude Code session** (`claude -p`, one scratch folder per session, the owner's global rules excluded so the skill stood on its own; the owner's hooks still ran, including a "lazy senior developer" persona hook). Raw `stream-json` logs and the workflow journals stayed on the owner's machine (they carry machine paths).

Two things changed in the harness since 2.1. A headless session can now **run** a Workflow when the call is pre-approved (`--allowedTools Workflow`), so the engine path was exercised for real, six subagents at a time. And `--effort ultracode` switches a session on without the word in the prompt, so the "a system reminder says ultracode is on" trigger was exercised too.

## How many

| Kind | Sessions | Result |
|---|---|---|
| Static audit (`audit-agent-skills`) | 3 passes (2.1, after each fix) | pass, one intentional flag |
| Trigger tests, tools limited to `Skill` | 5 + 4 re-runs on the final text | 9 / 9 as specified |
| Approval probes | 2 | a headless Workflow runs under `--allowedTools Workflow` or `bypassPermissions` |
| Behavioral runs, 2.1 skill | 4 | the keyword path stayed inline 2 / 2 on the README's own example — the defect |
| Behavioral runs, fixed skill | 5 | 2 / 2 keyword runs fanned out; the switch-on path 2 / 2 after the second fix |

Plan-equivalent cost of the whole exam: about $73.

## Trigger tests — does it load only when it should?

| # | Prompt | Expected | Observed |
|---|---|---|---|
| t1 | `ultracode: split src/lib.js into …` | loads first | `Skill(ultracode-discipline)` first, then `workflow-authoring` ✅ |
| t2 | `use a workflow to split …` (no `ultracode`) | quiet | only `workflow-authoring` ✅ (re-run on the 2.2 text: same) |
| t3 | `fix the typo in README.md: 'recieve' → 'receive'` | quiet | no skill loaded ✅ (re-run: same) |
| t4 | `spawn three agents in parallel: …` (no `ultracode`) | quiet | no skill loaded ✅ (re-run: same) |
| t6 | `what does the word ultracode mean in Claude Code? just explain …` | quiet (a conversational turn) | no skill loaded ✅ (re-run: same) |

## Behavioral runs — does it lead a run the way the rules say?

Throwaway project: a 22-line CommonJS library (`parse`, `validate`, `write`, one test file, `npm test` = `node --test tests/`). The task in every run was the README's own example: split it into three modules with three test files and keep `npm test` green. Permission mode `acceptEdits`, a Bash allow-list for `npm`/`node`/`git` and read-only commands, `Workflow` pre-approved, no bypass. "Keyword" = the prompt starts with `ultracode:`; "switch on" = `--effort ultracode` and no word in the prompt.

| Run | Skill | Path | Time · cost | What happened |
|---|---|---|---|---|
| engine-1 | 2.1 | keyword | 44 s · $0.94 | loaded the skill first, then `UltraCode: 0 agents · one write-set — inline` ("fan-out would cost more than it saves"); wrote all seven files itself; 9/9 |
| engine-1b | 2.1 | keyword | 44 s · $0.81 | same decision, same words ("three independent functions … one write-set"); 10/10 — **the defect, 2 / 2** |
| t5 | 2.1 | switch on, a one-line function + one test | 37 s · $0.89 | skill not loaded, done inline — consistent with the "trivial edit" exclusion |
| t5b | 2.1 | switch on | 42 s · $6.40 | loaded first; `UltraCode: 6 agents · Write → Check (≤2 correction rounds) · why: three independent module+test write-sets …`; the Workflow ran: 3 writers + 3 checkers, all `done` through the schema; 14/14 |
| *fix 1: rule 1 counts write-sets from the decomposition, three or more fan out* |||||
| engine-2 | 2.2-rc1 | keyword | 12 s · $3.51 | loaded first; "Three independent write-sets … so this fans out"; `UltraCode: 6 agents · Build → Verify · why: …`; 6 agents `done`; 12/12. The leader overwrote `src/lib.js` while builders were reading it, noticed, restored it, and swapped the re-export in after the run |
| t5c | 2.2-rc1 | switch on | 41 s · $10.09 | built all seven files inline **before** loading the skill, then loaded it and fanned out 4 read-only checkers only ("I wrote the seven files inline before loading the discipline") |
| *fix 2: the description says load it first, before any file is read or written; rule 2 says the leader's own edits land before the fan-out or after the run* |||||
| engine-3 | 2.2 | keyword | 49 s · $3.59 | loaded first ("as required for an ultracode run"); `UltraCode: 6 agents · Build → Verify · why: …`; 6 agents `done`; 14/14 |
| t5d | 2.2 | switch on | 54 s · $7.13 | loaded first ("as the session reminder requires"); 6 agents `done`; the re-export landed after the run "per the contract"; 24/24 |
| t5e | 2.2 | switch on | 32 s · $6.79 | loaded first; the re-export written before the fan-out; 6 agents `done`, zero findings; 14/14 |

Times are wall-clock for the whole session; the engine runs are short because the six agents ran in parallel.

What the journals show for every fan-out run: one builder per module with a disjoint write-set (module + its test), one checker per module that did not build it, the contract block identical in every brief, the return schema passed as the agent's `schema` so no subagent could answer in prose, at most two correction rounds authored in the script, and the leader running `npm test` once at the end. No checker ever failed, so the correction path and a `blocked` return were authored every time but never exercised.

## What changed in 2.2

1. **Rule 1** no longer lets size decide. Write-sets are counted from the decomposition — parts that can be built without reading each other's output — and three or more fan out, however few lines each one is; "it would cost more than it saves" is named as a non-reason. Fewer than three stay inline, with the launch line `UltraCode: 0 agents · <n> write-set(s) — inline`. (The old text said "one write-set gets no fan-out", and a 2.1 leader called a three-module split one write-set twice.)
2. **The description** says to load the skill first, before any file is read or written. (A leader with the session switched on built everything inline, then loaded the skill and verified only.)
3. **Rule 2** adds that the leader's own edits to anything the agents read land before the fan-out or after the run, never during it. (A leader overwrote the source file while builders were reading it.)
4. **Return schema:** `findings` is marked checker-only (three writers had filled it with notes).
5. **README:** the `npx skills add` one-liner now says it needs Node 22.20 or newer (on Node 20 the CLI stops with a syntax error before installing anything); the old validation list is labelled version 1 so it no longer reads as the current trigger behavior; the rules text matches the skill.

## Static audit

`audit-agent-skills` (read-only) on the repository copy and the installed copies: structure, triggers, safety, Claude and Codex profiles all **pass**; one review flag, `model-specific-tool` at `SKILL.md:3` — the declared "Claude Code only". All three copies carry the same digest.

## Not tested

- A correction round or a `blocked` return in a live run — no checker ever failed and no brief was wrong, so those paths were authored but never exercised.
- An **interactive** approval by a human; every approval here was the pre-approved headless kind.
- Windows. Linux was validated for v1 on 2026-09-25, not re-run.

---

# Test evidence — version 2.1 (2026-10-09)

What was tested before 2.1 shipped, how, and what came out. Every run was a **fresh headless Claude Code session** (`claude -p`, Claude Code 2.1.295, macOS), started in an empty scratch folder, with the skill installed in `~/.claude/skills/`. Raw `stream-json` logs are kept locally by the owner (they carry machine paths), not in this repository.

## How many

| Kind | Sessions | Result |
|---|---|---|
| Trigger tests | 4 | 4 / 4 as specified |
| Behavioral runs | 3 | all three found or confirmed what became 2.1 |
| Static audit (`audit-agent-skills`) | 3 passes (v2.0, v2.1, after fix round) | pass, one intentional flag |
| Independent review (Codex) | 1 round + 1 fix round | 4 findings, all corrected, approved by Claude |

## Trigger tests — does it load only when it should?

Tools were limited to `Skill` so each session could only show which skill it reached for first.

| # | Prompt | Expected | Observed |
|---|---|---|---|
| t1 | `ultracode: split src/lib.js into …` | loads first | `Skill(ultracode-discipline)` first, then `workflow-authoring` ✅ |
| t2 | `use a workflow to split …` (no `ultracode`) | quiet | only `workflow-authoring` loaded ✅ |
| t3 | `fix the typo in README.md: 'recieve' → 'receive'` | quiet | no skill loaded ✅ |
| t4 | `spawn three agents in parallel: …` (no `ultracode`) | quiet | no skill loaded ✅ |

t2 and t4 are the cases that v1 fired on. 2.1 stays quiet.

## Behavioral runs — does it lead a run the way the rules say?

Throwaway project: a 16-line CommonJS library (`parse`, `validate`, `write`) to be split into three modules with three test files; `npm test` = `node --test tests/`. Permission mode `acceptEdits`, Bash allow-list for `npm`/`node`/`git` and read-only commands, no bypass. In a headless session the Workflow approval dialog is **always refused** ("Review dynamic workflow before running") because nobody is there to approve it — so every run exercised the refusal path, and no workflow actually executed.

| Run | Skill version | Time | What happened | Found |
|---|---|---|---|---|
| 1 | 2.0 | 564 s · 30 turns | Workflow refused → leader re-ran the same 6 agents (3 builders, 3 checkers) through the Agent tool · `npm test` 20/20 | no launch line printed; the schema had no field for a checker's failures; builders wrote prose reports instead of returning the schema; a declined dialog was re-run another way |
| 2 | 2.1 | 402 s · 28 turns | Workflow refused → **0** Agent calls, work done inline, decline named in the report · `npm test` 8/8 · briefs carried the contract, `findings`, "Return only the schema", "never spawn agents" | launch line still missing as chat text |
| 3 | 2.1 final | 714 s · 39 turns | `meta.description` = `UltraCode: 6 agents · Build → Verify · why: …` · refused → inline, disclosed · briefs complete · `npm test` 15/15 | nothing new (long only because the harness had removed the Write tool and the model used shell heredocs) |

What the runs proved about the authored workflow scripts (read from the logs): one builder per module with disjoint write-sets; one checker per module that did not build it, using a differential oracle against the original function; at most two correction rounds, each re-verified; the leader ran `npm test` once at the end.

## Static audit

`audit-agent-skills` (read-only): structure, triggers, safety, Claude and Codex profiles all **pass**; one review flag, `model-specific-tool` at `SKILL.md:3` — the declared "Claude Code only".

## Independent review

Round 1 (Codex, reviewing Claude's 2.1): confirmed the trigger logs, the 6 → 0 Agent-call drop, the `meta.description` launch line and the test counts independently; returned four findings — two v1 guards lost in the rewrite (delegation around a board gate or lane; subagent transcripts in the leader's context), four sentences still restating the built-in reference, one README summary line out of step. Round 2 (Codex fixing, Claude reviewing): all four corrected in the real files, pinned requirements intact, audit unchanged — approved.

## Not tested

- An **approved** Workflow run — the dialog needs a human, so the engine path is unverified. One interactive `ultracode:` run with the owner approving is the missing test.
- Real subagent returns after the "Return only the schema" line was added (no subagent ran after it).
- Windows. Linux was validated for v1 on 2026-09-25, not re-run for 2.1.
