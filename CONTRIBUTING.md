# Contributing

This plugin is part of the [Foundry Collection](https://github.com/V-Songbird/foundry) and is maintained by a single author. Contributions are welcome in the form of bug reports, suggestions, and pull requests.

## Before opening a PR

- Check existing issues first — the problem may already be tracked or intentionally deferred.
- For substantial changes (new skills, significant refactors), open an issue first to align on direction before writing code.

## Structure

One package serves Claude Code, Codex and Antigravity. [`AGENTS.md`](AGENTS.md) lists where each part lives and the rules that outrank everything else; read it first.

**Skills are instruction files read by every host.** A change to a `SKILL.md` changes how Claude Code, Codex and Antigravity read the skill. Be precise, and try the affected skill in a real session of each host before a release.

**Templates are copied into every project that mounts collet.** A change under `templates/` changes each later mount; cover it with a test and a changelog line.

**Hooks run on matching tool calls and session events.** Keep them fast, keep network calls out, and test on Windows, Linux and macOS. A new hook event needs an entry in `hooks/hooks.json`, `hooks/codex-hooks.json` and, when Antigravity has the event, the root `hooks.json`. Codex users must review and trust a new or changed hook in `/hooks`; say so in the changelog.

## Tests

Run the suite before submitting:

```
node --test
```

It needs Node.js 22 or later, Git and no dependency installation. CI runs it on Windows, Linux and macOS with Node.js 22. Include regression coverage for changed script, template or hook behavior.

The suite feeds each hook the events a host sends and grades the eval cases without a model. It does not establish that a Claude Code, Codex or Antigravity installation has enabled or trusted the hooks; only a session on that host shows that. The eval cases under `evals/` start paid model sessions; [Running the evals](docs/knowledge/harness-workflow.md#running-the-evals) lists what they need.

## Git hooks

Run this once after cloning:

```
git config core.hooksPath scripts/git-hooks
```

- `pre-commit` runs `node --test tests/*.test.js` and blocks the commit on failure. It also blocks a commit that stages a Markdown file while the root `README.md` lacks its navigation line, or while a link to a heading anchor in any tracked Markdown file no longer resolves. The two `.cjs` test files run under `node --test` and in CI, not at commit time.
- The files under `scripts/git-hooks/`, except its `package.json`, are byte-identical copies shared with the other Foundry plugins; change them together.
- Keep credentials and personal session data out of commits.

## Changelog

Add an entry at the top of [`docs/knowledge/changelog.md`](docs/knowledge/changelog.md) for every user-visible change. The version lives in `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json` and `plugin.json`. Change them together; the suite fails when they differ or when the changelog's newest entry is another version.

Installation, marketplace changes and migration are separate from source changes. A pull request must not update a contributor's installed plugins or personal Claude Code, Codex or Antigravity configuration.
