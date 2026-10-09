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
