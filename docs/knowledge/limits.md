---
type: knowledge
summary: "Every current limit of collet in full: what the guard and the checks do not see, what closing needs, what a cut-short hook does, and which host behavior is observed, tested or unverified; read before relying on collet to stop a write, and before changing a guard or a check."
related_files:
  - README.md
  - docs/knowledge/how-it-works.md
---

# Collet limits

This page states each limit of Collet in full. The README names each one in a line, under [Good to know](../../README.md#good-to-know).

Collet is experimental. It has no support or stability promise, and it can change or disappear without a migration path.

No open task means no enforcement. That is deliberate.

| Area | Limit |
| --- | --- |
| Timing | Without `refuse_out_of_scope_writes`, a write outside the task is caught at close, not as it happens. On Antigravity the note does not reach the session, because its hook answer has no field for one. |
| Antigravity guard | It reads shell commands through `run_command` and file writes through `write_to_file`, `replace_file_content`, `multi_replace_file_content`, `write_file` and `edit_file`. Antigravity CLI sent `write_file` for a created file and `edit_file` for an edit in a live payload. The refusal on those two names is checked by tests, not by a live session. Antigravity wires only the write guard: no session-start or compaction hook runs, so the open task reaches a session through the rules block and `node .collet/task.mjs status`. A call without workspace context needs an absolute tool path. |
| Shell writes | The guard recognizes specific forms. A path built from a variable, a dynamic expression, or a file written by a program the command ran goes through. This narrows the hole and does not close it. `$PWD`, `${PWD}` and `%CD%` read as the call's working directory, and `~` as the home directory. A call whose working directory the host does not name leaves them unread. |
| Optional checks | They inspect `Write`, `Edit` and `MultiEdit` text at write time, only for files matching a check's paths. Shell writes, Codex patches, Antigravity file text and unsupported notebook edits wait for `close` or `run.mjs --live`. On Codex CLI a skipped test added by a patch was allowed at the call and refused at `close`. |
| JavaScript skips | The checks recognize Jest and Vitest calls and simple native `node:test` options such as `{ skip: true }` and `{ only: true }`. Dynamic calls, quoted option keys and complex option objects are outside that coverage. |
| Match counts | Bundle checks compare each detector's match count in changed files against `HEAD`; untracked files start at zero. An increase fails unless the matches moved. Replacing one match with another can leave the count unchanged, and removing an identical line from an unrelated file reads as a move. |
| Split files | The write-time check compares an edit only with that file's own text, so it refuses matches moved into a new file although closing accepts them. To split a file, copy it with a shell command or `git mv`, then trim each copy with edits that only remove lines. |
| Rules files | Closing treats the text between the Collet markers as the harness's own. While a task is open, the guard still refuses writes to a rules file the task does not list. |
| Closing | It needs the `--left-out` and `--unverified` lines, strict live checks and a passing accept command. An unavailable `HEAD` or unreadable source is reported as skipped, and skipped checks prevent closing. Custom checks need a working-tree check to allow it. The scope check covers only the mounted directory, so a mount below the Git root sees only its own directory. |
| Hook timeout | A host cuts a hook short at 10 seconds and discards its answer. Claude Code documents that the tool call then goes ahead; Codex and Antigravity are not checked. While a task is open, the guard logs each call in `.collet/guard-log.jsonl`, and the next guard call, `status` and `close` report a cut-short call. A cut-short session start or handoff is reported at the next session start, and `.collet/handoff.md` may be missing or stale. Hook errors fail open. None of this refuses anything. |
| Hooks that never ran | A host can skip a plugin's hooks, for example when they are not trusted or a managed setting turns them off. Collet cannot run them itself. When a task closes with no guard record, `close` says the guard judged no call during the task. The checks at close still ran. |
| Invisible text | Text Collet states to the session or records leaves out invisible characters a model reads and a person does not see. `task.mjs` and `mount.mjs` refuse them where that text is written, and name the field and the code points. |
| Fixture score | It does not measure false alarms on your code. A green accept command means one command exited zero and the writes stayed inside a list someone drew. It does not mean the work is correct. |
| Live runs | Mounting and check writing have run in Claude Code, in headless Codex CLI sessions and in headless Antigravity CLI sessions without `--sandbox`. Both are unverified in the Antigravity IDE. |
| Refusal and close | A refusal and a verified close were observed in long sessions: interactive on Claude Code with a script answering the prompts, interactive in the Antigravity IDE across two conversations with a person answering, and headless on Codex CLI with a script answering. |
| Compaction | The open task carries over a manual `/compact` on Claude Code and an automatic compaction on headless Codex CLI, observed with a script answering and the context window lowered per run. Automatic compaction on Claude Code, and compaction in an interactive Codex session, are unverified. |
| Evidence | These results come from runs on specific host versions, some with a script rather than a person answering. Unit tests supply host-shaped events to the hooks. They do not prove live host integration. |
