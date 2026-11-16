# collet

**Keep a coding session inside one task, its files and the command that finishes it.**

Available on [Claude Code](#claude-code), the primary host, and on the compatibility hosts [Codex](#codex) and [Antigravity](#antigravity).

[Get started](#install) · [What is this?](#what-is-this) · [How it works](#how-it-works) · [What you can do](#what-you-can-do) · [Evidence](#the-numbers)

## What is this?

You ask for a fix in the cart module. The session also edits two files nobody mentioned, then says it is done without running the tests. This example illustrates the problem; it is not a recorded run.

collet constrains a coding session to one open task, its writable files, and the command required to finish it. Closing the task fails while a change sits outside its files. Hooks refuse such a write as it happens only when the project asks for that.

Use it when agents exceed the task's scope or finish without verification. It constrains already planned work; it does not plan the work.

## Why you'd want it

- Every session starts knowing its task, its files and how the task ends.
- A change outside the task's files is named, and the task cannot close while it stands.
- Finished means a real command exited zero, not that a summary says so.
- Add a check for a mistake that keeps happening.
- No dependencies beyond Node.js.

## How it works

Mounting creates `.collet/` in your project and adds a rules block to `AGENTS.md`. A task names its files and its acceptance command. `node .collet/task.mjs close` checks the working tree against the task, then runs that command.

- **Finished is not an opinion.** Closing runs the accept command itself. A non-zero exit leaves the task open.
- **Drift is caught at close, or refused as it happens when the project asks for that.** A write outside the open task goes ahead with a note that names the path. `close` then fails until the task is widened or the change is undone. With `refuse_out_of_scope_writes` set to `true`, the guard refuses that write instead. The harness's own files and an edit a check catches are refused either way.
- **The guard reads what it can read with certainty.** Write tools, patches and these shell forms: redirects, `cp`, `mv`, `rm`, `tee`, `sed -i` and the common PowerShell cmdlets. It also reads `rm -rf` on a directory.
- **JavaScript and TypeScript imports expand the list.** Name the module that owns the behaviour, and collet adds its relative imports, one level deep, with a reason for each. It stops after 12 added files, so check a module with more imports yourself. For other languages, derive the scope by reading the code.
- **Covered means caught.** Every check ships with a planted mistake and a near miss, a lookalike it must leave alone. A failed pair is reported and never counted as coverage.
- **Closing needs the scope held and the command green.** It first checks the changed files against the task, including untracked ones, against the Git baseline, then runs the accept command. Either half failing leaves the task open, and a check that cannot run also prevents closing.

With hooks active, the session receives its task at startup, recognized writes are guarded, and a handoff is written before compaction so a compacted session keeps the open task. Antigravity wires only the write guard; its rules block and `node .collet/task.mjs status` provide task context.

The mount's own writes are not task changes: `.collet/` and the rules block between its markers in `AGENTS.md`, `CLAUDE.md` or `.cursor/rules/collet.md`. The first task can therefore close before you commit the mount. A planning tool's `ROADMAP.jsonl` and `.foreman/` do not count either. Any other change outside the scope keeps the task open, including your own text in those files.

A write under `docs/tasks/`, `docs/decisions/`, `docs/knowledge/private/` or `docs/apis/private/` is a private record, not a task change, when Git ignores the path and holds no tracked file there. Some names stay refused ignored or not: any path with a name that starts with a dot, any `AGENTS*.md` or `CLAUDE*.md`, and any name that holds a colon or ends in a dot or space.

## What you can do

| Task | Request or command |
| --- | --- |
| Mount the harness | `/collet:task-harness` |
| Inspect the task | `node .collet/task.mjs status` |
| List every task, open and closed | `node .collet/task.mjs list` |
| Open a task | `node .collet/task.mjs add --title "..." --why "..." --scope "src/cart.mjs,test/**"` |
| Add a file the open task needs | `node .collet/task.mjs widen --add <path> --why "<reason>"` |
| Close the task | `node .collet/task.mjs close --left-out "..." --unverified "..."` |
| Prove the checks still catch what they claim | `node .collet/checks/run.mjs` |
| Check the working tree against the open task | `node .collet/checks/run.mjs --live` |
| Add a check for a recurring mistake | `/collet:check-writer help me catch skipped tests` |

The skill examples use Claude Code syntax. On Codex use `$task-harness` and `$check-writer`; on Antigravity use `/task-harness` and `/check-writer`.

### Mount the harness

Ask the session to mount the harness using your host's invocation:

| Host | Request |
| --- | --- |
| Claude Code | `/collet:task-harness` |
| Codex | `$task-harness` |
| Antigravity | `/task-harness` |

The skill establishes your project details, first task, scope and real acceptance command before mounting. It creates `.collet/`, adds the rules block, opens the task and reports the fixture check results. It stops before performing the task itself. Then fill in `.collet/config.json`: a task cannot be opened while the `project` line or the `accept` command is still a placeholder.

The rules block always goes into `AGENTS.md`. It also goes into `CLAUDE.md` when that file exists, and into `.cursor/rules/collet.md` when `.cursor/` exists. The mount never creates a `CLAUDE.md`. A `CLAUDE.md` that imports `AGENTS.md` with an `@AGENTS.md` or `@./AGENTS.md` line outside a code block or code span gets no copy, because it already loads the block from there; a block an earlier mount left in it is emptied down to its markers and the text around them is kept. The mount reads Markdown approximately, so an `@AGENTS.md` inside a code block it does not recognize, such as a fence nested in a list item or indented code behind blockquote markers, still counts as the import. An unclosed fence in a list item, or a fence inside an HTML block, can hide an import that the host would count.

A second mount refreshes collet's own scripts, `task.mjs`, `state.mjs`, the check runner and the built-in scope check with its examples. Your `config.json`, `unverified.md` and your own checks are kept, and so is a `source.mjs` you edited; to take this version's fixes to an edited `source.mjs`, remove it and mount again. The rules block is replaced and your text around it is left alone. Commit what lands in `.collet/`: its `.gitignore` already leaves out what is derived or belongs to your machine. The mount installs no commit or push hook and no CI workflow.

On Claude Code, the skill can also offer permission ask rules for each `ask_first` item with a clear command, such as `Bash(npm publish *)` and `PowerShell(npm publish *)`. A rule guards only the shell tool it names, so both are offered and you confirm each one. The mount adds the rules you confirm to `permissions.ask` in the project's `.claude/settings.json`, keeping everything already there. With no confirmed rule it writes no settings file, and it never touches your user settings, `.claude/settings.local.json` or another host's configuration. A headless run cannot answer the prompt such a rule raises, so the host refuses the matching command there.

### Open and close tasks

Task commands run from the project directory. For a project containing `src/cart.mjs`:

```shell
node .collet/task.mjs add --title "Round the cart total" --why "Prices show three decimals" --scope "src/cart.mjs"
node .collet/task.mjs close --left-out "currency formatting" --unverified "nobody ran this against real prices"
```

`add` prints the open task, its scope and the accept command. Reads outside the scope are always allowed. `close` requires the `--left-out` and `--unverified` lines; it records them in the task ledger and appends them to `.collet/unverified.md`.

When a guard refuses a write, it names the path and the remedy that fits the refusal:

- For a file the stated task needs, widen the task with a recorded reason. Ask the person first when that file goes beyond their request. A file the task does not need stays out, even when the person asks for it: the session leaves it unchanged and names it in its summary.
- For a harness file, change it between tasks. While a task is open, writes under `.collet/` are refused, except `.collet/unverified.md`, and widening does not lift that. Close the task, write the check, let `node .collet/checks/run.mjs` admit it, then open the next task.
- For an edit that a check caught, fix the edit.

### Alongside a planning tool

A planning tool that keeps `ROADMAP.jsonl` or `.foreman/` owns the plan. collet owns the session boundary: the open task's files and the accept command. A project with a roadmap mounts like any other, and collet never reads the roadmap or enforces the files an entry lists.

To tie a task to an entry of [Foreman](https://github.com/V-Songbird/foreman), the roadmap plugin from the same marketplace, open it with `--foreman <id>`: 1 to 64 letters, digits, `.`, `_` or `-`, starting with a letter or digit. The id is only a label, shown by `status` and `list`. If the entry's handoff names a verification command, pass it as `--accept`. When that task closes with its checks and accept command passing, `close` prints the Foreman step that marks the entry `awaiting_acceptance`, with `<commit sha>` and `<Foreman plugin root>` for you to fill in after committing. collet does not run it and does not change the roadmap files. A failing close, or a task opened without `--foreman`, prints no step. For a task with an id, what nobody checked belongs in the entry's `unverified:` note lines: a passing close prints the `--unverified` text and then the `--left-out` text for you to add there, and writes neither to `.collet/unverified.md`. The scope check never refuses a write to `ROADMAP.jsonl` or `.foreman/`.

### Optional language checks

Optional bundles catch patterns that weaken the command that finishes a task, such as skipped tests, empty test bodies, blanket suppressions and commands that hide test failures. They read source text and need no toolchain.

| Language | Checks | Root marker |
| --- | --- | --- |
| JavaScript and TypeScript | 11, plus 1 opt-in | `package.json` |
| Python | 8 | `pyproject.toml` |
| Go | 5 | `go.mod` |
| Rust | 7 | `Cargo.toml` |
| JVM | 6 | `build.gradle.kts` or `pom.xml` |
| .NET | 9 | `*.csproj` or `*.sln` |

Ask the skill to mount with the checks for tests and verification settings. From a clone of this repository the same mount is `node scripts/mount.mjs "<project-directory>" --accept "npm test" --checks`. It selects every language detected at the project root, proves each example pair before writing, then runs the pairs in the mounted project; a failed pair exits non-zero. Failures are recorded in `.collet/checks/discarded.json`, which does not disable a check. If no supported bundle is detected, mounting with `--checks` refuses before writing anything.

The mount takes these options:

| Option | Effect |
| --- | --- |
| `--accept "<command>"` | The command a task closes with |
| `--checks` | Add the language bundles detected at the project root |
| `--edition <id>` | Pick a language instead of automatic selection; repeat it for several. Needs `--checks`. Ids: `javascript-typescript`, `python`, `go`, `rust`, `jvm`, `dotnet` |
| `--with <edition>.<class>` | Add an opt-in check, such as `javascript-typescript.type-widened-to-any` |
| `--remove <edition>.<class>` | Delete that check and keep it out of every later mount. Needs `--checks` |
| `--restore <edition>.<class>` | Take a removed check back |
| `--exclude <glob>` | Path globs no bundle check reads |
| `--ask-first "<item>"` | Something the person is asked about before it happens |
| `--ask-rule "<rule>"` | A Claude Code permission ask rule to add to `.claude/settings.json`; needs an `ask_first` list |

A fresh mount without `--checks` adds only the scope check. A later mount preserves each installed check and its examples, including your edits; new catalogue patterns do not upgrade those preserved files. A check missing its module or either kind of example stops the mount before writes.

### Settings

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

The session start states `project`, `conventions`, `ask_first` and the open task's accept command before the session reads a file. The rules block states the `ask_first` list from the last mount, and the session start reads the config each time. With a list, both also say that every other step goes ahead until the accept command exits zero, or until a missing input or a broken environment means it cannot pass, which ends the work as a blocker named in the summary with the task still open.

Kill switch: create `.collet/off` and every session guard goes silent. Committed checks keep running.

With `refuse_out_of_scope_writes` on, a refused write reads like this:

```text
src/checkout.mjs is outside the open task (t1). Writable: src/cart.mjs. Widen the task
only for a file the stated task needs: node .collet/task.mjs widen --add <path>
--why "<reason>". A file the task does not need stays out, even when the person asks
for it. If a needed file goes beyond what the person asked for, ask them first. When the
file is not needed, or nobody can be asked, leave it alone and name it in your summary.
```

## Install

collet is one package for Claude Code, Codex and Antigravity. Install it in the one you use. Every host needs Node.js 22 or later, with no other dependencies; the checks also run directly with Node.

Your project needs a Git repository with a committed baseline readable by the host. Scope checks use `git diff` and `git ls-files`.

### Claude Code

Requirements: Node.js 22 or later, Git and Claude Code 2.1.323 or later.

Inside Claude Code:

```text
/plugin marketplace add V-Songbird/foundry
/plugin install collet@foundry
```

Start a new session to load it. In a terminal, `claude plugin list` names `collet` once it is installed.

Claude Code's Auto permission mode lets a classifier answer most permission prompts. To approve or refuse each tool call yourself, including `widen` and `close`, start with `claude --permission-mode manual`. Answering No to a prompt ends the turn at once until you send a message.

### Codex

Requirements: Node.js 22 or later, Git and a Codex host with plugin support.

```text
codex plugin marketplace add V-Songbird/foundry
codex plugin add collet@foundry
```

Before guarded work, open `/hooks` in Codex CLI and trust the plugin's `SessionStart`, `PreToolUse` and `PreCompact` hooks. A changed hook definition or hook script needs another review; see [Codex hook trust](https://learn.chatgpt.com/docs/hooks#review-and-trust-hooks). Then start a new Codex session.

### Antigravity

Requirements: Node.js 22 or later, Git and the Antigravity CLI. Antigravity has no marketplace for third-party plugins, so install collet from a clone of Foundry, which checks collet out at the commit the Claude Code and Codex catalogs pin. Replace `<path-to-clone>` with the directory to clone into:

```shell
git clone --recurse-submodules https://github.com/V-Songbird/foundry.git <path-to-clone>
agy plugin install <path-to-clone>/collet
agy plugin list
```

The list names `collet`. If it does not, the install did not register: run the install command again with the clone path. Start a new conversation to load it. Use CLI installation; workspace-directory discovery is unverified. Antigravity wires only the write guard here: no session-start or compaction hook runs, so the open task reaches a session through the rules block and `node .collet/task.mjs status`. A call that omits workspace context needs an absolute tool path for the guard to find the harness; relative arguments cannot establish it.

### Update

In Claude Code, refresh the marketplace, update the plugin and start a new session:

```text
/plugin marketplace update foundry
/plugin update collet@foundry
```

In Codex, refresh the marketplace, add the plugin again and start a new session. Review and trust any changed hook in `/hooks`:

```text
codex plugin marketplace upgrade
codex plugin add collet@foundry
```

An Antigravity install is a copy, so it stays at the commit it was installed from. Update the clone, install again and start a new conversation:

```shell
git -C <path-to-clone> pull
git -C <path-to-clone> submodule update --init
agy plugin install <path-to-clone>/collet
```

A project that already mounted collet keeps its own copy of the scripts until you mount again.

## Good to know

> **Experimental.** No support or stability promise. It can change or disappear without a migration path.

- No open task means no enforcement. That is deliberate.
- Without `refuse_out_of_scope_writes`, a write outside the task is caught at close, not as it happens. On Antigravity the note does not reach the session, because its hook answer has no field for one.
- Shell analysis recognizes specific forms. A path built from a variable, a dynamic expression, or a file written by a program the command invoked goes through; this narrows the hole and does not close it. Four path starts are read: `$PWD`, `${PWD}` and cmd's `%CD%` as the call's working directory, and `~` as the home directory. A call whose working directory the host does not name leaves them unread.
- Optional checks inspect `Write`, `Edit` and `MultiEdit` text at write time, and only for files matching a check's paths. Shell writes, Codex patches, Antigravity file text and unsupported notebook edits wait for `close` or `run.mjs --live`. On Codex CLI 0.155.1 a skipped test added by a patch was allowed at the call and refused at `close`.
- JavaScript skip and focus checks recognize Jest and Vitest calls and simple native `node:test` inline options such as `{ skip: true }` and `{ only: true }`. Dynamic calls, quoted option keys and complex option objects are outside that coverage.
- Bundle checks compare each detector's match count in changed files against `HEAD`; untracked files start at zero. An increase fails unless the matches moved: one removed from another changed file and added with the same text, ignoring whitespace, is not new. Replacing one existing match with another can leave the count unchanged, and removing an identical line from an unrelated file reads as a move.
- The check at the moment of a write compares an edit only with that file's own text, so it refuses matches moved into a new file even though closing accepts them. To split a file, copy it with a shell command or `git mv`, then trim each copy with edits that only remove lines.
- Closing treats the text between the collet markers as the harness's own, so it does not report an edit made there. While a task is open, the guard still refuses writes to a rules file the task does not list.
- Closing requires the `--left-out` and `--unverified` lines, strict live checks and the accept command to pass. An unavailable `HEAD` or unreadable source is reported as skipped, and skipped checks prevent closing; custom checks need a working-tree check to allow it. The scope check covers only the mounted directory, so a mount below the Git root sees only its own directory.
- A host cuts a hook short at its timeout, 10 seconds here, and discards its answer. Claude Code documents that the tool call then goes ahead; Codex and Antigravity are not checked. While a task is open the guard logs each call in `.collet/guard-log.jsonl`, and the next guard call, `status` and `close` report a call that was cut short. A session start or handoff that was cut short is reported at the next session start; a cut-short handoff means `.collet/handoff.md` may be missing or stale. Hook errors fail open. None of this refuses anything.
- Text collet states to the session or records (project line, conventions, `ask_first`, task fields, handoff) leaves out invisible characters a model reads and a person does not see, and `task.mjs` and `mount.mjs` refuse them where that text is written. The message names the field and the code points.
- Fixture success does not measure false alarms on your code, and a green accept command means one command exited zero and the writes stayed inside a list someone drew. It does not mean the work is correct.
- Mounting and check writing have run in Claude Code, in headless Codex CLI 0.155.1 sessions and in headless Antigravity CLI 1.2.9 sessions without `--sandbox`. Both are unverified in the Antigravity IDE.
- A collet refusal and a verified close have been observed in long sessions: interactive on Claude Code 2.1.278 with a script answering the prompts, interactive in the Antigravity IDE 2.17.0 across two conversations with a person answering, and headless on Codex CLI 0.155.1 with a script answering.
- The open task carries over a manual `/compact` on Claude Code 2.1.278 and an automatic compaction on headless Codex CLI 0.155.1, observed with a script answering and the context window lowered per run. Automatic compaction on Claude Code, and compaction in an interactive Codex session, are unverified.
- These results come from runs on specific host versions, some with a script rather than a person answering the prompts.
- Installing from the Foundry marketplace has not been observed on any host.
- Unit tests supply host-shaped events to the hooks. They do not prove live host integration.

## The numbers

No session with collet has been compared against one without it, on any host. Each host stays marked as not measured until such a run exists.

The one recorded score is not a session measurement. In a mounted project, `node .collet/checks/run.mjs` runs the scope check on its own planted fixtures: it catches 5 of 5 violations and leaves 6 of 6 near misses alone, and prints `ok   scope — 5 violation(s) caught, 6 near miss(es) left alone`. Fixture success does not measure false alarms on your code.

### Claude Code results

<!-- foundry:evidence {"platform":"Claude","status":"pending","reason":"No Claude Code session measurement of collet exists."} -->
| Model | Setup | Tasks closed in scope |
| --- | --- | --- |
| Not measured | Not measured | Not measured |

No Claude Code session has been measured yet.

### Codex results

<!-- foundry:evidence {"platform":"Codex","status":"pending","reason":"No Codex session measurement of collet exists."} -->
| Model | Setup | Tasks closed in scope |
| --- | --- | --- |
| Not measured | Not measured | Not measured |

No Codex session has been measured yet.

### Antigravity results

<!-- foundry:evidence {"platform":"Antigravity","status":"pending","reason":"No Antigravity session measurement of collet exists."} -->
| Model | Setup | Tasks closed in scope |
| --- | --- | --- |
| Not measured | Not measured | Not measured |

No Antigravity session has been measured yet.

## What changed

Release notes for every version are on the [releases page](https://github.com/V-Songbird/collet/releases).

## License

MIT — see [LICENSE](LICENSE).
