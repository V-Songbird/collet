---
type: knowledge
summary: "Every collet setting in .collet/config.json, its default and effect, what the session start states, the kill switch and the text of a refused write; read before editing config.json."
related_files:
  - README.md
  - templates/task.mjs
  - hooks/guard.js
  - hooks/session-start.js
---

# Collet settings

This page lists every setting collet reads, with its default and effect. The README shows the three most used ones under [Settings](../../README.md#settings).

Values live in `.collet/config.json`. No environment variable or secret is read by collet's settings; the hooks use only the project directory your host names.

| Name | Required | Default | What it does |
| --- | --- | --- | --- |
| `project` | yes | `REPLACE ME: …` | One line on what this repository is and what it runs on |
| `conventions` | no | two `REPLACE ME` entries | Decisions that constrain what a change may look like; a leftover placeholder is dropped, and an empty list is valid |
| `accept` | yes | `REPLACE ME: …` | The command a task closes with, unless the task names its own |
| `ask_first` | no | absent; set by `--ask-first` | What must never happen without asking the person; the rules block and the session start state it as a fact about the project |
| `exclude` | no | absent; set by `--exclude` | Path globs no bundle check reads, at write time or at close, such as a committed generated mirror |
| `removed_checks` | no | absent; set by `--remove` | Bundle checks the project took out; a remount keeps them out until `--restore` |
| `refuse_out_of_scope_writes` | no | absent; set by hand to `true` | Makes the guard refuse a write outside the open task instead of noting it. Any other value, or none, notes it. Add it while no task is open |

## What the session is told

The session start states `project`, `conventions`, `ask_first` and the open task's accept command before the session reads a file. The rules block states the `ask_first` list from the last mount, and the session start reads the config each time.

With a list, both also say that every other step goes ahead until the accept command exits zero, or until a missing input or a broken environment means it cannot pass. That ends the work as a blocker named in the summary with the task still open.

## Kill switch

Create `.collet/off` and every session guard goes silent. Committed checks keep running.

## A refused write

With `refuse_out_of_scope_writes` on, a refused write reads like this:

```text
src/checkout.mjs is outside the open task (t1). Writable: src/cart.mjs. Widen the task
only for a file the stated task needs: node .collet/task.mjs widen --add <path>
--why "<reason>". A file the task does not need stays out, even when the person asks
for it. If a needed file goes beyond what the person asked for, ask them first. When the
file is not needed, or nobody can be asked, leave it alone and name it in your summary.
```
