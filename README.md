<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/banner-dark.png" />
    <img src="assets/banner-light.png" alt="Collet" width="900" />
  </picture>
  <h1>Collet</h1>
  <p><strong>Keep a coding session inside one task, its files and the command that finishes it.</strong></p>
</div>

<p align="center"><strong>Available on</strong></p>
<table align="center">
  <tr>
    <td align="center"><a href="#claude-code"><img src="assets/edition-claude.svg" alt="" width="80" height="80" /><br />Claude Code, primary host</a></td>
    <td align="center"><a href="#codex"><img src="assets/edition-codex.svg" alt="" width="80" height="80" /><br />Codex</a></td>
    <td align="center"><a href="#antigravity"><img src="assets/edition-antigravity.svg" alt="" width="80" height="80" /><br />Antigravity</a></td>
  </tr>
</table>

<p align="center"><a href="#install"><strong>Get started</strong></a> · <a href="#what-is-this">What is this?</a> · <a href="#how-it-works">How it works</a> · <a href="#what-you-can-do">What you can do</a> · <a href="#the-numbers">Evidence</a></p>

## What is this?

Collet is a plugin for Claude Code, an AI coding assistant. It also runs on Codex and Antigravity, two other coding assistants.

You ask for a fix in the cart module. The assistant also edits two files you never named, then says it is done without running the tests.

Collet ties a session to one open task. The task names the files it may change and the command that must pass before it closes. Collet does not plan the work.

<p align="center"><img src="assets/mascot.svg" alt="Ember works inside the boundary of one task. An edit to a file outside it is flagged. Once that edit is gone, the check passes and the task closes." width="700"></p>

*Ember is the mascot of the Foundry plugins. The scene is an illustration, not a measurement.*

**You ask:** open a task for the cart file, then close it.

**You get:** two commands, shown as an illustration:

```shell
node .collet/task.mjs add --title "Round the cart total" --why "Prices show three decimals" --scope "src/cart.mjs"
node .collet/task.mjs close --left-out "currency formatting" --unverified "nobody ran this against real prices"
```

`add` prints the open task, its files and its closing command. `close` fails while a change sits outside `src/cart.mjs`, or while that command fails.

## Why you'd want it

- **Every session knows its task.** It knows its files and how the task ends.
- **Drift is named.** The task cannot close while a change outside its files stands.
- **Finished means a command passed.** A summary saying so does not count.
- **Recurring mistakes get a check.** Add one when a mistake repeats.

## How it works

You set Collet up once per project. Collet calls this mounting. It creates a `.collet/` folder and adds a block of rules to `AGENTS.md`, the file assistants read for instructions.

| Moment | What Collet does |
| --- | --- |
| The assistant writes a file | A guard reads the write. The guard is a hook, a small program the assistant's host runs before each write. Outside the task, it notes the path, or refuses when `refuse_out_of_scope_writes` is `true`. |
| You close the task | `node .collet/task.mjs close` compares your changes with the task. Then it runs the accept command, the command that must pass. A failure of either leaves the task open. |
| You add a check | Every check ships with two test files: one holds the mistake, one comes close without making it. A check that fails either never counts. |

[How Collet works](docs/knowledge/how-it-works.md) covers the whole mechanism.

## What you can do

| You want to | What to type |
| --- | --- |
| Inspect the task | `node .collet/task.mjs status` |
| List every task | `node .collet/task.mjs list` |
| Open a task | `node .collet/task.mjs add --title "..." --why "..." --scope "src/cart.mjs,test/**"` |
| Add a file the open task needs | `node .collet/task.mjs widen --add <path> --why "<reason>"` |
| Close the task | `node .collet/task.mjs close --left-out "..." --unverified "..."` |
| Prove the checks still catch what they claim | `node .collet/checks/run.mjs` |
| Check the working tree against the open task | `node .collet/checks/run.mjs --live` |

These two go to the assistant:

| You want to | Claude Code | Codex | Antigravity |
| --- | --- | --- | --- |
| Set Collet up in a project | `/collet:task-harness` | `$task-harness` | `/task-harness` |
| Add a check for a recurring mistake | `/collet:check-writer help me catch skipped tests` | `$check-writer` | `/check-writer` |

### Settings

Values live in `.collet/config.json`. After mounting, fill in the two required ones: a task cannot open while either is a placeholder.

| Setting | Values | What it does |
| --- | --- | --- |
| `project` | Text, required | One line on what the repository is and what it runs on. |
| `accept` | A command, required | The command a task closes with, unless the task names its own. |
| `refuse_out_of_scope_writes` | `true`, set by hand | The guard refuses a write outside the open task instead of noting it. |

Create `.collet/off` to silence every session guard. Committed checks keep running.

[Collet settings](docs/knowledge/settings.md) lists all seven.

## Install

Every host needs Node.js 22 or later and Git. Your project needs a Git repository with one commit, so Collet can compare changes.

### Claude Code

You need Claude Code 2.1.323 or later.

Inside Claude Code, run:

```text
/plugin marketplace add V-Songbird/foundry
/plugin install collet@foundry
```

Then start a new session.

To check that it worked, run `claude plugin list` in a terminal. The list names `collet`.

### Codex

You need a Codex host with plugin support.

In a terminal, run:

```text
codex plugin marketplace add V-Songbird/foundry
codex plugin add collet@foundry
```

Open `/hooks` in Codex CLI and trust the plugin's `SessionStart`, `PreToolUse` and `PreCompact` hooks. A changed hook registration needs another review; a changed hook script does not. See [Codex hook trust](https://learn.chatgpt.com/docs/hooks#review-and-trust-hooks). Then start a new Codex session.

To check that it worked, run `codex plugin list --marketplace foundry` in a terminal. The list names `collet`.

### Antigravity

You need the Antigravity CLI. It has no marketplace for third-party plugins, so you install from a copy of the Foundry repository. Replace `<path-to-clone>` with a folder of your choice.

In a terminal, run:

```shell
git clone --recurse-submodules https://github.com/V-Songbird/foundry.git <path-to-clone>
agy plugin install <path-to-clone>/collet
agy plugin list
```

Then start a new conversation.

To check that it worked, run `agy plugin list`. It names `collet`.

### Update

In Claude Code, run these. Then start a new session:

```text
/plugin marketplace update foundry
/plugin update collet@foundry
```

In Codex, run these. Then start a new session. A changed hook stays off until you trust it again in `/hooks`:

```text
codex plugin marketplace upgrade
codex plugin remove collet@foundry
codex plugin add collet@foundry
```

In Antigravity, run these. Then start a new conversation:

```shell
git -C <path-to-clone> pull
git -C <path-to-clone> submodule update --init
agy plugin install <path-to-clone>/collet
```

## Good to know

> **Experimental.** No support or stability promise. It can change or disappear without a migration path.

No open task means no enforcement, on purpose.

| Limit | What it means for you |
| --- | --- |
| Antigravity install | Install through the CLI. Discovery from a workspace folder is unverified. |
| Timing | Without `refuse_out_of_scope_writes`, a write outside the task is caught at close, not as it happens. |
| Antigravity | Only the write guard runs there. Its refusal is checked by tests, not by a live session. |
| Shell writes | The guard recognizes specific forms. A path built from a variable goes through. |
| Optional checks | At write time they read only `Write`, `Edit` and `MultiEdit` text. Everything else waits for `close`. |
| JavaScript skips | Dynamic calls, quoted option keys and complex option objects are not covered. |
| Match counts | Replacing one match with another can leave the count unchanged. |
| Split files | The write-time check refuses matches moved into a new file, although closing accepts them. |
| Rules files | The guard refuses writes to a rules file the open task does not list. |
| Closing | A skipped check prevents closing. A mount below the Git root sees only its own directory. |
| Hook timeout | A host cuts a hook short at 10 seconds. Hook errors fail open. |
| Invisible text | `task.mjs` and `mount.mjs` refuse invisible characters and name the field. |
| Fixture score | It does not measure false alarms on your code. A green accept command does not mean the work is correct. |
| Live runs | Mounting and check writing are unverified in the Antigravity IDE. |
| Hooks that never ran | A host can skip untrusted hooks. `close` then says so. |
| Compaction | Automatic compaction on Claude Code, and compaction in an interactive Codex session, are unverified. |
| Evidence | Unit tests do not prove live host integration. |

[Collet limits](docs/knowledge/limits.md) states each one in full.

## The numbers

Each figure comes from recorded test sessions. The figures describe those runs, not yours, and results vary between runs.

Collet's own check of itself is not such a measurement. `node .collet/checks/run.mjs` catches 5 of 5 planted mistakes and leaves 6 of 6 near misses alone. It prints `ok   scope — 5 violation(s) caught, 6 near miss(es) left alone`.

### Claude Code results

<!-- foundry:evidence {"platform":"Claude","status":"measured","models":["Claude Opus 5.5"],"source":"docs/knowledge/benchmarks.md","date":"2026-11-16","revision":"a70b19e","reviewedAt":"2026-11-16"} -->
| Model | Job | What was checked | With Collet | Without |
| --- | --- | --- | --- | --- |
| Claude Opus 5.5 | Setting up a first task | Settings filled in | 3/3 | 0/3 |
| Claude Opus 5.5 | Setting up a first task | First task opened | 3/3 | 0/3 |
| Claude Opus 5.5 | Setting up a first task | Harness proven by its own checks | 3/3 | 0/3 |
| Claude Opus 5.5 | Setting up a first task | Existing instructions kept | 3/3 | 0/3 |
| Claude Opus 5.5 | Writing a new check | New check admitted | 3/3 | 2/3 |
| Claude Opus 5.5 | Writing a new check | Cart code left untouched | 3/3 | 2/3 |
| Claude Opus 5.5 | Guarding a file outside the task | Guard refused a write | 0/3 | not graded |

<p align="center"><img src="assets/hero.svg" alt="Seven checks, three runs each with and without Collet. Six passed more often with Collet. One lost: the guard refused a write in 0 of 3 runs." width="700"></p>

The run was on 2026-11-16 at commit `a70b19e`. It had three jobs, three runs each way and 18 sessions. It used `claude-opus-5-5` through the Claude Agent SDK, not a Claude Code session. Checks that needed judgement ran on `claude-sonnet-5-5`.

**One result is a loss.** The guard refused a write in 0 of 3 runs with Collet. The sessions never tried the write. Three runs each way are exploratory.

[Collet benchmarks](docs/knowledge/benchmarks.md) holds the method and what the table leaves out.

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

## Going deeper

[How Collet works](docs/knowledge/how-it-works.md) · [Collet settings](docs/knowledge/settings.md) · [Collet limits](docs/knowledge/limits.md) · [Collet benchmarks](docs/knowledge/benchmarks.md)

Release notes are on the [Collet releases page](https://github.com/V-Songbird/collet/releases). [Foundry](https://github.com/V-Songbird/foundry) lists the whole family.

Report a bug or ask a question on [Collet's issue tracker](https://github.com/V-Songbird/collet/issues). Report a vulnerability as [SECURITY.md](SECURITY.md) says. To change Collet, read [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT, see [LICENSE](LICENSE).
