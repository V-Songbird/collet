# collet

collet is one plugin for Claude Code, Codex and Antigravity: a harness it mounts into a project,
which holds a session inside one open task, the files that task may touch and the command that
ends it. Its hooks, scripts and templates are dependency-free Node.js and run with Node.js 22 or
later and Git; there is no install or build step. This repository is mounted by the Foundry
marketplace as a submodule and ships from `main`.

## Start here

- Before changing the mount, the scope check or task closure, read [runtime contracts](docs/knowledge/runtime-contracts.md).
- Before changing a task command, a config key, a catalogue or an eval case, read [the harness workflow](docs/knowledge/harness-workflow.md).
- Before a release, read [the changelog](docs/knowledge/changelog.md) and [CONTRIBUTING.md](CONTRIBUTING.md).

## Rules that outrank everything

- `templates/` is what a mount copies into a project's `.collet/`. A change there changes every later mount, so it needs a test and a changelog line.
- The three manifests, `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json` and `plugin.json`, carry the same name, version, description and author, changed together; `tests/manifests.test.js` fails when they differ or when the changelog's newest entry is another version.
- The root `plugin.json` is Antigravity's manifest. It carries no `$schema` and no `com.openai` extension, so Codex keeps reading `.codex-plugin/plugin.json` and finds its hooks.
- Each Codex hook has one `command` and no `commandWindows`: Codex fills in `${PLUGIN_ROOT}` itself, and a Windows override broke the guard when it was tried.
- A public file never names another project except where it was measured, and states only numbers that were measured.

## Commands

Run from the repository root.

| Command | Purpose | Cost |
| --- | --- | --- |
| `node --test` | The suite: mount, task commands, scope, hooks, catalogues, manifests and eval graders, with no model session | Local temporary fixtures |
| `node scripts/git-hooks/check-readme-nav.js` | The root README has its nav line, and every heading anchor linked from a tracked Markdown file resolves | Local |
| `node scripts/mount.mjs "<project-directory>" --accept "<command>"` | Mount the harness into a project | Writes `.collet/` and the rules block in that project |
| `claude plugin validate .` | Claude Code package shape | Local; needs the `claude` CLI |
| `agy plugin validate .` | Antigravity package shape; it warns that `hooks/hooks.json`, Claude Code's file, is never loaded | Local; needs the Antigravity CLI |
| `git config core.hooksPath scripts/git-hooks` | One-time: the pre-commit hook runs `tests/*.test.js` and the nav check | Local |

Live eval sessions bill a model and need a sandbox; their commands are in [Running the evals](docs/knowledge/harness-workflow.md#running-the-evals).

## Where things live

| Path | Content |
| --- | --- |
| `skills/` | `task-harness`, which mounts and reports, and `check-writer`, which adds a check |
| `scripts/` | `mount.mjs` and `catalogue.mjs`; `scripts/git-hooks/` holds the commit gates |
| `templates/` | The task CLI, the state and source helpers, the check runner, the scope check with its fixtures, the config and the rules block |
| `catalogue/` | Optional language check bundles: JavaScript and TypeScript, Python, Go, Rust, JVM and .NET |
| `hooks/` | The write guard, the session start and the compaction handoff; `hooks/hooks.json` is Claude Code's wiring and `hooks/codex-hooks.json` is Codex's |
| `hooks.json`, `plugin.json` | Antigravity's hook wiring and manifest, at the root |
| `evals/` | Eval cases for `claude plugin eval`, with their fixture script and graders |
| `tests/` | The `node:test` suite |
| `docs/knowledge/` | The harness workflow, the runtime contracts and the changelog |
| `assets/` | Copies of the brand kit's images; change them in the kit, never here |

## Conventions

`package.json` makes every `.js` file here an ES module. CommonJS files carry `.cjs`, and
`scripts/git-hooks/` has its own `package.json` that says `commonjs`, because its files are
copies shared with other plugins. `CLAUDE.md` imports this file; keep shared contributor facts
here.

## Pitfalls

- **The pre-commit hook runs `tests/*.test.js` only.** `tests/readme_nav.test.cjs` and `tests/pre_commit.test.cjs` run under `node --test` and in CI, not at commit time.
- **`tests/eval-harness.cjs` has a twin.** anneal keeps the same file as `tests/eval-harness.js`, and `templates/state.mjs` holds a copy of anneal's invisible-character pattern; change each pair together, since no test compares the two repositories.
- **Isolated Git config drops long paths on Windows.** A test helper that sets `GIT_CONFIG_NOSYSTEM` loses Git for Windows' `core.longpaths`; pass `-c core.longpaths=true`.
- **On Node.js 22 a suite that fails outside its tests can still exit 0.** A `describe` callback or an `after` hook that throws is reported but left out of the exit code.
- **Antigravity wires only the write guard.** It has no session-start or compaction hook here, so the open task reaches a session through the rules block.
- **Passing subprocess tests do not prove installed-host behavior.** Hook discovery, trust and prompts need a session on the host.
