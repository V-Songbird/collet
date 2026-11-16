<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/banner-dark.png" />
    <img src="assets/banner-light.png" alt="collet" width="900" />
  </picture>
  <h1>collet</h1>
  <p><strong>Keep a coding session inside one task, its files and the command that finishes it.</strong></p>
</div>

<p align="center"><strong>Available on</strong></p>
<table align="center">
  <tr>
    <td align="center"><a href="#codex"><img src="assets/edition-codex.svg" alt="" width="80" height="80" /><br />Codex</a></td>
    <td align="center"><a href="#claude-code"><img src="assets/edition-claude.svg" alt="" width="80" height="80" /><br />Claude Code</a></td>
    <td align="center"><a href="#antigravity"><img src="assets/edition-antigravity.svg" alt="" width="80" height="80" /><br />Antigravity</a></td>
  </tr>
</table>

<p align="center"><a href="#install"><strong>Get started</strong></a> · <a href="#what-is-this">What is this?</a> · <a href="#how-it-works">How it works</a> · <a href="#what-you-can-do">What you can do</a> · <a href="#the-numbers">Evidence</a></p>

## What is this?

You ask for a fix in the cart module. The session also edits two files nobody mentioned, then says it is done without running the tests. This example illustrates the problem; it is not a recorded run.

collet constrains a coding session to one open task, its writable files, and the command required to finish it. Closing the task fails while a change sits outside its files. Hooks refuse such a write as it happens only when the project asks for that.

Use it when agents exceed the task's scope or finish without verification. It constrains already planned work; it does not plan the work.

## Why you'd want it

- Every session starts knowing its task, its files and how the task ends.
- A change outside the task's files is named, and the task cannot close while it stands.
- Finished means a real command exited zero, not that a summary says so.
- Add a check for a mistake that keeps happening.
- No dependencies beyond Node.

## How it works

Mounting creates `.collet/` in your project and adds a rules block to `AGENTS.md`. A task names its files and its acceptance command. `node .collet/task.mjs close` checks the working tree against the task, then runs that command.

A write outside the open task goes ahead with a note that names the path. `close` then fails until the task is widened or the change is undone. With `refuse_out_of_scope_writes` set to `true` in `.collet/config.json`, the guard refuses that write instead. The harness's own files and an edit a check catches are refused either way.

With hooks active, the session receives its task at startup, recognized writes are guarded, and a handoff is written before compaction. Antigravity wires only the write guard. Its rules block and `node .collet/task.mjs status` provide task context.

## What you can do

| Task | Request or command |
| --- | --- |
| Mount the harness | `/collet:task-harness` |
| Inspect the task | `node .collet/task.mjs status` |
| Open a task | `node .collet/task.mjs add --title "..." --why "..." --scope "src/cart.mjs,test/**"` |
| Close the task | `node .collet/task.mjs close --left-out "..." --unverified "..."` |
| Check changed files | `node .collet/checks/run.mjs --live` |
| Add a check for a recurring mistake | `/collet:check-writer help me catch skipped tests` |

The skill examples use Claude Code syntax. On Codex use `$task-harness` and `$check-writer`; on Antigravity use `/task-harness` and `/check-writer`.
Optional bundles cover JavaScript/TypeScript, Python, Go, Rust, JVM and .NET source patterns without installing their toolchains.

### Mount the harness

Ask the session to mount the harness using your host's invocation:

| Host | Request |
| --- | --- |
| Codex | `$task-harness` |
| Claude Code | `/collet:task-harness` |
| Antigravity | `/task-harness` |

The skill establishes your project details, first task, scope, and real acceptance command before mounting.
It creates `.collet/`, adds the rules block to `AGENTS.md`, opens the task, and reports the fixture check results.
The rules block also goes into `CLAUDE.md` and `.cursor/rules/collet.md` when the project already has `CLAUDE.md` or a `.cursor` folder.
A `CLAUDE.md` that imports `AGENTS.md`, such as with an `@AGENTS.md` line, already loads the rules from there and gets no copy.
A copy an earlier mount left in it is emptied down to its markers.
On Claude Code, it can also add the permission ask rules you confirm for your ask-first list to the project's `.claude/settings.json`.
It stops before performing the task itself.

A project that plans its work in `ROADMAP.jsonl` or `.foreman/` mounts like any other. That planning tool owns the plan; collet enforces the session boundary.
[Alongside a planning tool](docs/knowledge/harness-workflow.md#alongside-a-planning-tool) describes how the two divide the work.

### Inspect the result

In the mounted project:

```shell
node .collet/task.mjs status
node .collet/checks/run.mjs
```

The first command prints the open task, scope and acceptance command. The second reports whether each check catches its violations and permits its near misses.

When an active guard refuses a write, it names the affected path and the remedy that fits the refusal:

- For a file the stated task needs, widen the task with a recorded reason. Ask the person first when that file goes beyond their request. A file the task does not need stays out, even when the person asks for it.
- For a harness file, change it between tasks.
- For an edit that a check caught, fix the edit.

See the [harness workflow](docs/knowledge/harness-workflow.md) for direct mounting, task commands, optional language checks, and the fixture benchmark.

### Settings

Values live in `.collet/config.json`. There are no environment settings or secrets.

| Name | Required | Default | What it does |
| --- | --- | --- | --- |
| `project` | yes | `REPLACE ME: …` | Describes the project and runtime |
| `conventions` | no | two `REPLACE ME` entries | Constrains project changes |
| `accept` | yes | `REPLACE ME: …` | Command for closing tasks without their own override |
| `ask_first` | no | absent; set by `mount.mjs --ask-first` | Things the person is asked about before they happen |
| `exclude` | no | absent; set by `mount.mjs --exclude` | Path globs no bundle check reads, such as a generated mirror |
| `removed_checks` | no | absent; set by `mount.mjs --remove` | Bundle checks a remount keeps out |
| `refuse_out_of_scope_writes` | no | absent; set by hand to `true` | Makes the guard refuse a write outside the open task instead of noting it |

Replace the `project` and `accept` placeholders before opening a task; leftover `conventions` placeholders are ignored. Creating `.collet/off` silences session guards; committed checks keep running.
The [configuration reference](docs/knowledge/harness-workflow.md#configuration) explains generated files, host rules, and remount behavior.

## Install

collet is one package for Codex, Claude Code and Antigravity. Install it in the one you use. Every host needs Node.js 22 or later, with no other dependencies; the checks also run directly with Node.

Your project needs a Git repository with a committed baseline readable by the host. Scope checks use `git diff` and `git ls-files`.

### Codex

Requirements: Node.js 22 or later, Git and a Codex host with plugin support.

```text
codex plugin marketplace add V-Songbird/foundry
codex plugin add collet@foundry
```

Before guarded work, open `/hooks` in Codex CLI and trust the plugin's `SessionStart`, `PreToolUse` and `PreCompact` hooks. A changed hook definition or hook script needs another review; see [Codex hook trust](https://learn.chatgpt.com/docs/hooks#review-and-trust-hooks). Then start a new Codex session.

### Claude Code

Requirements: Node.js 22 or later, Git and Claude Code.

Inside Claude Code:

```text
/plugin marketplace add V-Songbird/foundry
/plugin install collet@foundry
```

Start a new session to load it. In a terminal, `claude plugin list` names `collet` once it is installed.

### Antigravity

Requirements: Node.js 22 or later, Git and the Antigravity CLI. Antigravity has no marketplace for third-party plugins, so install collet from a clone of Foundry, which checks collet out at the commit the Claude Code and Codex catalogs pin. Replace `<path-to-clone>` with the directory to clone into:

```shell
git clone --recurse-submodules https://github.com/V-Songbird/foundry.git <path-to-clone>
agy plugin install <path-to-clone>/collet
agy plugin list
```

The list names `collet`. If it does not, the install did not register: run the install command again with the clone path. Start a new conversation to load it. Use CLI installation; workspace-directory discovery is unverified.

The install is a copy, so it stays at that commit. To take a new release, update the clone, install again and start a new conversation:

```shell
git -C <path-to-clone> pull
git -C <path-to-clone> submodule update --init
agy plugin install <path-to-clone>/collet
```

## Good to know

> **Experimental.** No support or stability promise. It can change or disappear without a migration path.

- No open task means no enforcement.
- Without `refuse_out_of_scope_writes`, a write outside the task is caught at close, not as it happens. On Antigravity the note does not reach the session.
- Shell analysis recognizes specific forms. Dynamic paths and writes performed by invoked programs can escape the guard.
- Optional checks inspect some edit tools immediately; other writes wait for working-tree checks.
- Closing requires the `--left-out` and `--unverified` lines, strict live checks and the acceptance command to pass. Unavailable verification leaves the task open.
- Fixture success does not measure false alarms on your code, and an acceptance command passing does not establish correctness.
- Mounting and check writing have run in Claude Code, in headless Codex sessions and in headless Antigravity CLI sessions without `--sandbox`. Both are unverified in the Antigravity IDE.
- A collet refusal and a verified close have been observed in long sessions: interactive on Claude Code and the Antigravity IDE, headless on Codex.
- The open task carries over a manual `/compact` on Claude Code and an automatic compaction on headless Codex. Automatic compaction on Claude Code, and compaction in an interactive Codex session, are unverified.
- These results come from runs on specific host versions, some with a script rather than a person answering the prompts. [Host coverage](docs/knowledge/harness-workflow.md#host-coverage) lists each run.
- Installing from the Foundry marketplace has not been observed on any host.
- Unit tests supply host-shaped events to the hooks. They do not prove live host integration.

The [full limits](docs/knowledge/harness-workflow.md#limits) describe host context requirements, pattern coverage, and baseline comparison behavior.

## The numbers

No session with collet has been compared against one without it, on any host. Each host stays marked as unmeasured until such a run exists.

The one recorded score is not a session measurement. In a mounted project, `node .collet/checks/run.mjs` runs the scope check on its own planted fixtures: it catches 5 of 5 violations and leaves 6 of 6 near misses alone. See [Benchmarks](docs/knowledge/harness-workflow.md#benchmarks). Which flows have run on which host is under [host coverage](docs/knowledge/harness-workflow.md#host-coverage).

### Codex results

<!-- foundry:evidence {"platform":"Codex","status":"pending","reason":"No Codex performance measurements are available."} -->
| Model | Setup | Tasks closed in scope |
| --- | --- | --- |
| Not measured | Not measured | Not measured |

No Codex session has been measured yet.

### Claude Code results

<!-- foundry:evidence {"platform":"Claude","status":"pending","reason":"No Claude Code performance measurements are available."} -->
| Model | Setup | Tasks closed in scope |
| --- | --- | --- |
| Not measured | Not measured | Not measured |

No Claude Code session has been measured yet. Three eval cases drive a live Claude Code session; [Running the evals](docs/knowledge/harness-workflow.md#running-the-evals) lists the command, the WSL2 or Linux sandbox requirement and what each case grades. No result of theirs is published.

### Antigravity results

<!-- foundry:evidence {"platform":"Antigravity","status":"pending","reason":"No Antigravity performance measurements are available."} -->
| Model | Setup | Tasks closed in scope |
| --- | --- | --- |
| Not measured | Not measured | Not measured |

No Antigravity session has been measured yet.

## Going deeper

[Harness workflow](docs/knowledge/harness-workflow.md) · [Runtime contracts](docs/knowledge/runtime-contracts.md) · [Configuration](docs/knowledge/harness-workflow.md#configuration) · [Limits](docs/knowledge/harness-workflow.md#limits) · [Contributing](CONTRIBUTING.md) · [Security](SECURITY.md) · [Changelog](docs/knowledge/changelog.md)

[Foundry](https://github.com/V-Songbird/foundry) lists this plugin. Bugs and questions go to the [issue tracker](https://github.com/V-Songbird/collet/issues).

## License

MIT — see [LICENSE](LICENSE).
