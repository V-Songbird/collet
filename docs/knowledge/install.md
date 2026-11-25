---
type: knowledge
summary: "How to install Collet in Claude Code, Codex and Antigravity, check that it worked and update it to a new release."
related_files:
  - README.md
  - .claude-plugin/plugin.json
  - plugin.json
---

# Install and update Collet

This page says how to install Collet on each host, how to tell it worked and how to move to a new release. The short form is in the README, under [Install](../../README.md#install).

Every host needs Node.js 22 or later and Git. Your project needs a Git repository with one commit, for Collet's diffs.

## Claude Code

You need Claude Code 2.1.323 or later.

Inside Claude Code, run:

```text
/plugin marketplace add V-Songbird/foundry
/plugin install collet@foundry
```

Then start a new session.

To check that it worked, run `claude plugin list` in a terminal. The list names `collet`.

## Codex

You need a Codex host with plugin support.

In a terminal, run:

```text
codex plugin marketplace add V-Songbird/foundry
codex plugin add collet@foundry
```

Open `/hooks` in Codex CLI and trust the plugin's `SessionStart`, `PreToolUse` and `PreCompact` hooks. A changed hook registration needs another review; a changed hook script does not. See [Codex hook trust](https://learn.chatgpt.com/docs/hooks#review-and-trust-hooks). Then start a new Codex session.

To check that it worked, run `codex plugin list --marketplace foundry` in a terminal. The list names `collet`.

## Antigravity

You need the Antigravity CLI. It has no marketplace for third-party plugins, so you install from a copy of the Foundry repository. Replace `<path-to-clone>` with a folder of your choice.

In a terminal, run:

```shell
git clone --recurse-submodules https://github.com/V-Songbird/foundry.git <path-to-clone>
agy plugin install <path-to-clone>/collet
agy plugin list
```

Then start a new conversation.

To check that it worked, run `agy plugin list`. It names `collet`.

## Update

### Claude Code

In Claude Code, run these. Then start a new session:

```text
/plugin marketplace update foundry
/plugin update collet@foundry
```

### Codex

In Codex, run these. Then start a new session. A changed hook stays off until you trust it again in `/hooks`:

```text
codex plugin marketplace upgrade
codex plugin remove collet@foundry
codex plugin add collet@foundry
```

### Antigravity

In Antigravity, run these. Then start a new conversation:

```shell
git -C <path-to-clone> pull
git -C <path-to-clone> submodule update --init
agy plugin install <path-to-clone>/collet
```
