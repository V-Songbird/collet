---
type: knowledge
summary: "Every published Collet measurement: the Claude Code run of 2026-11-16 with its loss, the fixture score, and the Codex and Antigravity results that do not exist yet; read before changing a published number."
related_files:
  - README.md
  - assets/hero.svg
  - evals/
---

# Collet benchmarks

This page holds every measurement Collet publishes, with its method and its limits. The README shows the same table under [The numbers](../../README.md#the-numbers).

## Contents

- [Claude Code, 2026-11-16](#claude-code-2026-11-16)
- [The loss](#the-loss)
- [The fixture score](#the-fixture-score)
- [Codex and Antigravity](#codex-and-antigravity)

## Claude Code, 2026-11-16

One run compared sessions with Collet against sessions without it, on Claude Code's model through the Agent SDK. The run was on 2026-11-16 at commit `a70b19e`. It used three cases, three runs with Collet and three without, 18 sessions, on `claude-opus-5-5` through the Claude Agent SDK rather than a Claude Code session. Judged graders ran on `claude-sonnet-5-5`.

The table lists every outcome grader where the arms differed. Graders that can only pass with the plugin by construction, such as the skill having fired or the mount script having run, are left out. Every other grader passed in all six runs of both arms, so those show nothing for the plugin.

<!-- foundry:evidence {"platform":"Claude","status":"measured","models":["Claude Opus 5.5"],"source":"docs/knowledge/benchmarks.md","date":"2026-11-16","revision":"a70b19e","reviewedAt":"2026-11-16"} -->
| Model | Case | Grader | With Collet | Without |
| --- | --- | --- | --- | --- |
| Claude Opus 5.5 | `check-writer-admits-a-check` | `cart-unchanged` | 3/3 | 2/3 |
| Claude Opus 5.5 | `check-writer-admits-a-check` | `check-admitted` | 3/3 | 2/3 |
| Claude Opus 5.5 | `guard-refuses-out-of-scope-write` | `guard-refused` | 0/3 | not graded |
| Claude Opus 5.5 | `mount-opens-first-task` | `config-filled` | 3/3 | 0/3 |
| Claude Opus 5.5 | `mount-opens-first-task` | `first-task-open` | 3/3 | 0/3 |
| Claude Opus 5.5 | `mount-opens-first-task` | `harness-proven` | 3/3 | 0/3 |
| Claude Opus 5.5 | `mount-opens-first-task` | `rules-block-kept-map-text` | 3/3 | 0/3 |

Three runs per arm are exploratory.

## The loss

`guard-refused` passed 0 of 3 with Collet. It reads the guard's log for a refused write, and the sessions never tried the write. The task's own start-of-session text told each one that a file outside the task is refused. Each left `package.json` alone and named it as left out, which is what the case asks for.

The guard's log existed in every run, so the hook did run, but no write reached it. The grader measures a refusal that these sessions never provoke. Without Collet the grader does not apply.

## The fixture score

The fixture score is not a session measurement. In a mounted project, `node .collet/checks/run.mjs` runs the scope check on its own planted fixtures. It catches 5 of 5 violations and leaves 6 of 6 near misses alone, and prints `ok   scope — 5 violation(s) caught, 6 near miss(es) left alone`. Fixture success does not measure false alarms on your code.

## Codex and Antigravity

<!-- foundry:evidence {"platform":"Codex","status":"pending","reason":"No Codex session measurement of collet exists."} -->
| Model | Setup | Tasks closed in scope |
| --- | --- | --- |
| Not measured | Not measured | Not measured |

<!-- foundry:evidence {"platform":"Antigravity","status":"pending","reason":"No Antigravity session measurement of collet exists."} -->
| Model | Setup | Tasks closed in scope |
| --- | --- | --- |
| Not measured | Not measured | Not measured |

No Codex or Antigravity session has been measured yet.
