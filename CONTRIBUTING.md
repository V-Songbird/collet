# Contributing

Collet is part of the [Foundry marketplace](https://github.com/V-Songbird/foundry) and has a single maintainer. Contributions are welcome as bug reports, suggestions and pull requests.

One package serves Claude Code, Codex and Antigravity.

---

## Before opening a PR

- Check existing issues first. The problem may already be tracked or deliberately deferred.
- For a substantial change, such as a new skill or a significant refactor, open an issue first to agree on direction.
- Branch from `main` and open the pull request against `main`.

---

## Structure

```text
.claude-plugin/
└── plugin.json          # Claude Code metadata and version
.codex-plugin/
└── plugin.json          # Codex metadata and version; points Codex at its hook file
plugin.json              # Antigravity metadata and the same version
hooks.json               # Antigravity event wiring
package.json             # makes every .js file an ES module
.github/
├── dependabot.yml       # weekly updates of the pinned actions
└── workflows/
    └── test.yml         # CI: runs the test suite
CONTRIBUTING.md          # this file
SECURITY.md              # how to report a vulnerability
LICENSE                  # MIT
assets/                  # the plugin icons the manifests name, the banners, host cards, Ember scene and hero graphic the README shows
README.md                # the front page: what it is, install, limits, results
docs/
└── knowledge/           # the detail pages: how it works, settings, limits, benchmarks
skills/                  # skills read by every host
├── task-harness/        # mounts the harness and reports on it
└── check-writer/        # adds a check
hooks/
├── hooks.json           # Claude Code event wiring
├── codex-hooks.json     # Codex event wiring
├── guard.js             # the write guard
├── session-start.js     # puts the open task into a session
└── handoff.js           # writes the handoff before compaction
scripts/                 # mount.mjs and catalogue.mjs
└── git-hooks/           # the commit gates and the README nav check
templates/               # what a mount copies into a project's .collet/
catalogue/               # optional language check bundles
evals/                   # eval cases for `claude plugin eval`, with fixture scripts and graders
tests/                   # the node:test suite
```

The README is the front page. The pages under `docs/knowledge/` hold the detail it links, and a fact lives in one place: the README states the short form and links the page. The README and the pages name the host beside every host-specific instruction and result.

---

## What to keep in mind

**Skills are instruction files read by every host.** A change to a `SKILL.md` changes how Claude Code, Codex and Antigravity read the skill. Be precise, and try the affected skill in a real session of each host before a release.

**Templates are copied into every project that mounts Collet.** A change under `templates/` changes each later mount; cover it with a test and say so in the release notes.

**Hooks run on matching tool calls and session events.** Keep them fast, keep network calls out, and test on Windows, Linux and macOS. A new hook event needs an entry in `hooks/hooks.json`, `hooks/codex-hooks.json` and, when Antigravity has the event, the root `hooks.json`. Codex users must review and trust a new or changed hook in `/hooks`; say so in the release notes. Each Codex hook has one `command` and no `commandWindows`, because Codex fills in `${PLUGIN_ROOT}` itself.

**The root `plugin.json` is Antigravity's manifest.** It carries no `$schema` and no `com.openai` extension, so Codex keeps reading `.codex-plugin/plugin.json` and finds its hooks.

**Keep the README in step with the code.** A limit leaves the README only with the change that resolves it.

**Public text names another project only where a figure was measured,** and states only numbers that were measured.

**Published figures follow one rule.** Each result names its host, model, date and sample size. Losses and limits stay, a host with no results shows `Not measured`, and a result from one host never stands for another.

---

## Tests

Run these with Node.js 22 or later and Git on your PATH. No dependency installation is needed:

```bash
node --test
node scripts/git-hooks/check-readme-nav.js
```

CI runs the suite on Windows, Linux and macOS with Node.js 22. The suite feeds each hook the events a host sends and grades the eval cases without a model.

A pull request that changes behavior without updating its tests is not merged. Include regression coverage for changed script, template or hook behavior.

The suite does not establish that a Claude Code, Codex or Antigravity installation has enabled or trusted the hooks; only a session on that host shows that.

Validate the package for each host when you can:

- **Claude Code:** `claude plugin validate .`
- **Codex:** the installed plugin-creator validator for `.codex-plugin/plugin.json`. It lives in the Codex installation, not in this repository.
- **Antigravity:** `agy plugin validate .` with the Antigravity CLI installed. It warns that `hooks/hooks.json`, Claude Code's file, is never loaded.

### Pitfalls in the suite

- `package.json` makes every `.js` file an ES module. CommonJS files carry `.cjs`, and `scripts/git-hooks/` has its own `package.json` that says `commonjs`, because its files are shared with other plugins.
- `tests/eval-harness.cjs` has a twin: anneal keeps the same file as `tests/eval-harness.js`, and `templates/state.mjs` holds a copy of anneal's invisible-character pattern. Change each pair together; no test compares the two repositories.
- A test helper that isolates Git config with `GIT_CONFIG_NOSYSTEM` loses Git for Windows' `core.longpaths`; pass `-c core.longpaths=true`.
- On Node.js 22 a suite that fails outside its tests can still exit 0: a `describe` callback or an `after` hook that throws is reported but left out of the exit code.
- Passing subprocess tests do not prove installed-host behavior. Hook discovery, trust and prompts need a session on the host.

### Eval cases

The three cases under `evals/` each drive a real Claude Code session in a small price-formatting library and start paid model sessions; `tests/eval-graders.test.js` replays them without a model. A live run needs a sandbox, which native Windows lacks, so run it under WSL2 or Linux with `bubblewrap` and `socat` installed, from the repository root, writing results outside it:

```bash
claude plugin eval . --tag smoke --tag safety --scaffold --no-publish \
  --allow-tools Bash Edit Write --output-dir <results>
```

Pass `--case <name>` to run one case. With no terminal, add `--trust-plugin`. Every scaffold runs `evals/price-project.sh`, which finds the plugin as the folder above `evals/`, so the target must be the whole repository. That script lists the entries the harness leaves in the workspace in `.git/info/exclude`, because collet's close-time scope check counts them; when a harness version adds an entry, add it there. The `check-admitted` and `accept-command-passes` graders are regex heuristics over the trace.

---

## Git hooks

Run this once after cloning:

```bash
git config core.hooksPath scripts/git-hooks
```

This enables a `pre-commit` hook that runs `node --test tests/*.test.js` and blocks the commit on failure. When a commit stages a Markdown file, it also blocks the commit while the root `README.md` lacks its navigation line, or while a link to a heading anchor in any tracked Markdown file no longer resolves. The two `.cjs` test files run under `node --test` and in CI, not at commit time.

Every Foundry plugin keeps byte-identical copies of the commit gates under `scripts/git-hooks/` and of the tests that cover them. A change to one goes into all of them. Collet's own `scripts/git-hooks/package.json`, which declares CommonJS, is the one file there that is not shared.

Public source names and attribution are allowed in documentation and commit messages. Keep credentials and personal session data out of commits.

---

## Release notes

Release notes live on the [releases page](https://github.com/V-Songbird/collet/releases). Describe every user-visible change in your pull request, so it can go into the next release's notes.

The version lives in `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json` and `plugin.json`. Change them together; the test suite fails when they differ.

A pull request never changes a contributor's installed plugins or personal configuration.

---

## Code of conduct

This project follows the [Contributor Covenant 2.1](https://www.contributor-covenant.org/version/2/1/code_of_conduct/). To report behavior that goes against it, contact the maintainer privately through the address in [SECURITY.md](SECURITY.md).
