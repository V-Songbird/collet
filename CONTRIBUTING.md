# Contributing

This plugin is part of the [Foundry Collection](https://github.com/V-Songbird/foundry) and is maintained by a single author. Contributions are welcome in the form of bug reports, suggestions, and pull requests.

---

## Before opening a PR

- Check existing issues first — the problem may already be tracked or intentionally deferred.
- For substantial changes (new skills, significant refactors), open an issue first to align on direction before writing code.

---

## Structure

One package serves Claude Code, Codex and Antigravity.

```
.claude-plugin/
└── plugin.json        # Claude Code metadata and version
.codex-plugin/
└── plugin.json        # Codex metadata and version; points Codex at its hook file
plugin.json             # Antigravity metadata and the same version
hooks.json              # Antigravity event wiring
LICENSE                 # MIT
README.md               # the user guide: how it works, settings, install, results
skills/                 # skills read by every host
├── task-harness/       # mounts the harness and reports on it
└── check-writer/       # adds a check
hooks/
├── hooks.json          # Claude Code event wiring
├── codex-hooks.json    # Codex event wiring
├── guard.js            # the write guard
├── session-start.js    # puts the open task into a session
└── handoff.js          # writes the handoff before compaction
scripts/                # mount.mjs, catalogue.mjs and scripts/git-hooks/ (the commit gates)
templates/              # what a mount copies into a project's .collet/
catalogue/              # optional language check bundles
evals/                  # eval cases for `claude plugin eval`, with fixture scripts and graders
tests/                  # the node:test suite
```

The README is text only and names the host beside every host-specific instruction and result. Each result identifies its host, model, date and sample size; keep losses and limits, show `Not measured` where evidence is absent, and never present a Claude Code result as a Codex result. Keep the README in step with the code: a limit leaves it only with the change that resolves it.

---

## What to keep in mind

**Skills are instruction files read by every host.** A change to a `SKILL.md` changes how Claude Code, Codex and Antigravity read the skill. Be precise, and try the affected skill in a real session of each host before a release.

**Templates are copied into every project that mounts collet.** A change under `templates/` changes each later mount; cover it with a test and say so in the release notes.

**Hooks run on matching tool calls and session events.** Keep them fast, keep network calls out, and test on Windows, Linux and macOS. A new hook event needs an entry in `hooks/hooks.json`, `hooks/codex-hooks.json` and, when Antigravity has the event, the root `hooks.json`. Codex users must review and trust a new or changed hook in `/hooks`; say so in the release notes. Each Codex hook has one `command` and no `commandWindows`: Codex fills in `${PLUGIN_ROOT}` itself, and a Windows override broke the guard when it was tried.

**The root `plugin.json` is Antigravity's manifest.** It carries no `$schema` and no `com.openai` extension, so Codex keeps reading `.codex-plugin/plugin.json` and finds its hooks.

**Public text names another project only where a figure was measured,** and states only numbers that were measured.

---

## Tests

Run the suite before submitting:

```
node --test
```

It needs Node.js 22 or later, Git and no dependency installation. CI runs it on Windows, Linux and macOS with Node.js 22. Include regression coverage for changed script, template or hook behavior.

The suite feeds each hook the events a host sends and grades the eval cases without a model. It does not establish that a Claude Code, Codex or Antigravity installation has enabled or trusted the hooks; only a session on that host shows that. `claude plugin validate .` and `agy plugin validate .` check the package shape; the second warns that `hooks/hooks.json`, Claude Code's file, is never loaded.

Things that have bitten before:

- `package.json` makes every `.js` file an ES module. CommonJS files carry `.cjs`, and `scripts/git-hooks/` has its own `package.json` that says `commonjs`, because its files are shared with other plugins.
- `tests/eval-harness.cjs` has a twin: anneal keeps the same file as `tests/eval-harness.js`, and `templates/state.mjs` holds a copy of anneal's invisible-character pattern. Change each pair together; no test compares the two repositories.
- A test helper that isolates Git config with `GIT_CONFIG_NOSYSTEM` loses Git for Windows' `core.longpaths`; pass `-c core.longpaths=true`.
- On Node.js 22 a suite that fails outside its tests can still exit 0: a `describe` callback or an `after` hook that throws is reported but left out of the exit code.
- Passing subprocess tests do not prove installed-host behavior. Hook discovery, trust and prompts need a session on the host.

### Eval cases

The three cases under `evals/` each drive a real Claude Code session in a small price-formatting library and start paid model sessions; `tests/eval-graders.test.js` replays them without a model. A live run needs a sandbox, which native Windows lacks, so run it under WSL2 or Linux with `bubblewrap` and `socat` installed, from the repository root, writing results outside it:

```
claude plugin eval . --tag smoke --tag safety --scaffold --no-publish \
  --allow-tools Bash Edit Write --output-dir <results>
```

Pass `--case <name>` to run one case. With no terminal, add `--trust-plugin`. Every scaffold runs `evals/price-project.sh`, which finds the plugin as the folder above `evals/`, so the target must be the whole repository. That script lists the entries the harness leaves in the workspace in `.git/info/exclude`, because collet's close-time scope check counts them; when a harness version adds an entry, add it there. The `check-admitted` and `accept-command-passes` graders are regex heuristics over the trace.

---

## Git hooks

Run this once after cloning:

```
git config core.hooksPath scripts/git-hooks
```

This enables the following commit checks:

- `pre-commit` runs `node --test tests/*.test.js` and blocks the commit on failure. It also blocks a commit that stages a Markdown file while the root `README.md` lacks its navigation line, or while a link to a heading anchor in any tracked Markdown file no longer resolves. The two `.cjs` test files run under `node --test` and in CI, not at commit time.
- The files under `scripts/git-hooks/`, except its `package.json`, are byte-identical copies shared with the other Foundry plugins; change them together.
- Public source names and attribution are allowed in documentation and commit messages. Keep credentials and personal session data out of commits.

---

## Release notes

Release notes live on the [releases page](https://github.com/V-Songbird/collet/releases).
Describe every user-visible change in your pull request so it can go into the
next release's notes. The version lives in `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json`
and `plugin.json`. Change them together; the test suite fails when they differ.

Installation, marketplace changes, and migration are separate from source
changes. A pull request must not update a contributor's installed plugins or
personal Claude Code, Codex or Antigravity configuration.
