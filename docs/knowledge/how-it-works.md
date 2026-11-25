---
type: knowledge
summary: "How Collet keeps a session inside one task: the guard, scope by imports, mounting, opening and closing tasks, the Foreman link and the optional language checks; read to predict what Collet will allow, refuse or report."
related_files:
  - README.md
  - hooks/guard.js
  - hooks/session-start.js
  - hooks/handoff.js
  - scripts/mount.mjs
  - templates/task.mjs
---

# How Collet works

This page says what Collet does to a coding session, part by part. It is for a user who wants to predict what Collet will allow, refuse or report. The short form is in the README, under [How it works](../../README.md#how-it-works).

## Contents

- [The mechanism](#the-mechanism)
- [Private records and the mount's own writes](#private-records-and-the-mounts-own-writes)
- [Mounting the harness](#mounting-the-harness)
- [Permission prompts on Claude Code](#permission-prompts-on-claude-code)
- [Open and close tasks](#open-and-close-tasks)
- [Alongside a planning tool](#alongside-a-planning-tool)
- [Optional language checks](#optional-language-checks)

## The mechanism

Mounting creates `.collet/` in your project and adds a rules block to `AGENTS.md`. A task names its files and its acceptance command. `node .collet/task.mjs close` checks the working tree against the task, then runs that command.

- **Finished is not an opinion.** Closing runs the accept command itself. A non-zero exit leaves the task open.
- **Drift is caught at close, or refused as it happens when the project asks for that.** A write outside the open task goes ahead with a note that names the path. `close` then fails until the task is widened or the change is undone. With `refuse_out_of_scope_writes` set to `true`, the guard refuses that write instead. The harness's own files and an edit a check catches are refused either way.
- **The guard reads what it can read with certainty.** Write tools, patches and these shell forms: redirects, `cp`, `mv`, `rm`, `tee`, `sed -i` and the common PowerShell cmdlets. It also reads `rm -rf` on a directory.
- **JavaScript and TypeScript imports expand the list.** Name the module that owns the behaviour, and Collet adds its relative imports, one level deep, with a reason for each. It stops after 12 added files, so check a module with more imports yourself. For other languages, derive the scope by reading the code.
- **Covered means caught.** Every check ships with a planted mistake and a near miss, a lookalike it must leave alone. A failed pair is reported and never counted as coverage.
- **Closing needs the scope held and the command green.** It first checks the changed files against the task, including untracked ones, against the Git baseline, then runs the accept command. Either half failing leaves the task open, and a check that cannot run also prevents closing.

With hooks active, the session receives its task at startup, recognized writes are guarded, and a handoff is written before compaction so a compacted session keeps the open task. Antigravity wires only the write guard; its rules block and `node .collet/task.mjs status` provide task context.

## Private records and the mount's own writes

The mount's own writes are not task changes: `.collet/` and the rules block between its markers in `AGENTS.md`, `CLAUDE.md` or `.cursor/rules/collet.md`. The first task can therefore close before you commit the mount. A planning tool's `ROADMAP.jsonl` and `.foreman/` do not count either. Any other change outside the scope keeps the task open, including your own text in those files.

A write under `docs/tasks/`, `docs/decisions/`, `docs/knowledge/private/` or `docs/apis/private/` is a private record, not a task change, when Git ignores the path and holds no tracked file there. Some names stay refused ignored or not: any path with a name that starts with a dot, any `AGENTS*.md` or `CLAUDE*.md`, and any name that holds a colon or ends in a dot or space.

## Mounting the harness

Ask the session to mount the harness using your host's invocation:

| Host | Request |
| --- | --- |
| Claude Code | `/collet:task-harness` |
| Codex | `$task-harness` |
| Antigravity | `/task-harness` |

The skill establishes your project details, first task, scope and real acceptance command before mounting. It creates `.collet/`, adds the rules block, opens the task and reports the fixture check results. It stops before performing the task itself. Then fill in `.collet/config.json`: a task cannot be opened while the `project` line or the `accept` command is still a placeholder.

The rules block always goes into `AGENTS.md`. It also goes into `CLAUDE.md` when that file exists, and into `.cursor/rules/collet.md` when `.cursor/` exists. The mount never creates a `CLAUDE.md`. A `CLAUDE.md` that imports `AGENTS.md` with an `@AGENTS.md` or `@./AGENTS.md` line outside a code block or code span gets no copy, because it already loads the block from there. A block an earlier mount left in it is emptied down to its markers, and the text around them is kept.

The mount reads Markdown approximately. An `@AGENTS.md` inside a code block it does not recognize, such as a fence nested in a list item or indented code behind blockquote markers, still counts as the import. An unclosed fence in a list item, or a fence inside an HTML block, can hide an import that the host would count.

A second mount refreshes Collet's own scripts: `task.mjs`, `state.mjs`, the check runner and the built-in scope check with its examples. Your `config.json`, `unverified.md` and your own checks are kept, and so is a `source.mjs` you edited. To take this version's fixes to an edited `source.mjs`, remove it and mount again. The rules block is replaced and your text around it is left alone.

Commit what lands in `.collet/`: its `.gitignore` already leaves out what is derived or belongs to your machine. The mount installs no commit or push hook and no CI workflow.

On Claude Code, the skill can also offer permission ask rules for each `ask_first` item with a clear command, such as `Bash(npm publish *)` and `PowerShell(npm publish *)`. A rule guards only the shell tool it names, so both are offered and you confirm each one. The mount adds the rules you confirm to `permissions.ask` in the project's `.claude/settings.json`, keeping everything already there. With no confirmed rule it writes no settings file. It never touches your user settings, `.claude/settings.local.json` or another host's configuration. A headless run cannot answer the prompt such a rule raises, so the host refuses the matching command there.

A project that already mounted Collet keeps its own copy of the scripts until you mount again, also after you update the plugin.

## Permission prompts on Claude Code

Claude Code's Auto permission mode lets a classifier answer most permission prompts. To answer each tool call yourself, including `widen` and `close`, start with `claude --permission-mode manual`. Answering No to a prompt ends the turn at once until you send a message.

## Open and close tasks

Task commands run from the project directory. For a project containing `src/cart.mjs`:

```shell
node .collet/task.mjs add --title "Round the cart total" --why "Prices show three decimals" --scope "src/cart.mjs"
node .collet/task.mjs close --left-out "currency formatting" --unverified "nobody ran this against real prices"
```

`add` prints the open task, its scope and the accept command. Reads outside the scope are always allowed. `close` requires the `--left-out` and `--unverified` lines. It records them in the task ledger and appends them to `.collet/unverified.md`.

When a guard refuses a write, it names the path and the remedy that fits the refusal:

- For a file the stated task needs, widen the task with a recorded reason. Ask the person first when that file goes beyond their request. A file the task does not need stays out, even when the person asks for it: the session leaves it unchanged and names it in its summary.
- For a harness file, change it between tasks. While a task is open, writes under `.collet/` are refused, except `.collet/unverified.md`, and widening does not lift that. Close the task, write the check, let `node .collet/checks/run.mjs` admit it, then open the next task.
- For an edit that a check caught, fix the edit.

## Alongside a planning tool

A planning tool that keeps `ROADMAP.jsonl` or `.foreman/` owns the plan. Collet owns the session boundary: the open task's files and the accept command. A project with a roadmap mounts like any other, and Collet never reads the roadmap or enforces the files an entry lists.

To tie a task to an entry of [Foreman](https://github.com/V-Songbird/foreman), the roadmap plugin from the same marketplace, open it with `--foreman <id>`. The id has 1 to 64 letters, digits, `.`, `_` or `-`, and starts with a letter or digit. The id is only a label, shown by `status` and `list`. If the entry's handoff names a verification command, pass it as `--accept`.

When that task closes with its checks and accept command passing, `close` prints the Foreman step that marks the entry `awaiting_acceptance`. It carries `<commit sha>` and `<Foreman plugin root>` for you to fill in after committing. Collet does not run it and does not change the roadmap files. A failing close, or a task opened without `--foreman`, prints no step.

For a task with an id, what nobody checked belongs in the entry's `unverified:` note lines. A passing close prints the `--unverified` text and then the `--left-out` text for you to add there, and writes neither to `.collet/unverified.md`. The scope check never refuses a write to `ROADMAP.jsonl` or `.foreman/`.

## Optional language checks

Optional bundles catch patterns that weaken the command that finishes a task, such as skipped tests, empty test bodies, blanket suppressions and commands that hide test failures. They read source text and need no toolchain.

| Language | Checks | Detected at the project root by any of |
| --- | --- | --- |
| JavaScript and TypeScript | 11, plus 1 opt-in | `package.json`, `tsconfig.json` or a lockfile (`package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`, `bun.lock`) |
| Python | 8 | `pyproject.toml`, `requirements.txt`, `setup.cfg`, `tox.ini` or a lockfile (`uv.lock`, `poetry.lock`, `pdm.lock`) |
| Go | 5 | `go.mod`, `go.sum` or `go.work` |
| Rust | 7 | `Cargo.toml`, `Cargo.lock` or `rust-toolchain.toml` |
| JVM | 6 | `build.gradle.kts`, `build.gradle`, `settings.gradle.kts`, `settings.gradle`, `pom.xml` or `gradlew` |
| .NET | 9 | `*.csproj`, `*.sln`, `*.slnx`, `Directory.Build.props`, `Directory.Packages.props`, `global.json` or `nuget.config` |

Ask the skill to mount with the checks for tests and verification settings. From a clone of this repository the same mount is:

```shell
node scripts/mount.mjs "<project-directory>" --accept "npm test" --checks
```

It selects every language detected at the project root, proves each example pair before writing, then runs the pairs in the mounted project. A failed pair exits non-zero. Failures are recorded in `.collet/checks/discarded.json`, which does not disable a check. If no supported bundle is detected, mounting with `--checks` refuses before writing anything.

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
