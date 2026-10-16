# dev-workflow

A minimal, Skill-native development workflow for coding agents.

## What it is

`development-workflow` is a single [Agent Skill](development-workflow/SKILL.md) that
tells a coding agent **which existing Skill or tool to use, and when**, for
ordinary software development:

- **[Superpowers](https://github.com/obra/superpowers)** for clarifying,
  planning, implementing, testing, and verifying.
- **[Tallyback](https://github.com/FoFxjc/tallyback)** for accountability when
  work is actually delegated to another agent.
- **[Earn-it](https://github.com/FoFxjc/earn-it)** for reviewing the finished
  implementation for unjustified complexity and unsupported claims.
- **Knowledge Skills** (your own project Skills) for verified, reusable
  understanding of a repository or service.

You set up a repository once with `development-workflow init`. That writes a
short root `AGENTS.md` telling every future agent session to load the workflow,
so you can just describe the task and the agent picks the right Skills.

## What problem it solves

Good Skills exist, but each session has to be told to use them, in the right
order, without overdoing it on small tasks. Delegated work comes back as a
worker's "done" rather than evidence. Review tends to be skipped or turned into
an endless rewrite loop. This Skill is the thin routing and handoff layer that
connects those pieces, without reimplementing any of them.

## What it is not

- **Not a CLI.** `development-workflow init` is an instruction you give the
  agent inside a session. No executable is installed.
- Not an agent framework, orchestrator, scheduler, runtime, or state database.
- Not a replacement for Superpowers, Tallyback, or Earn-it. It hands off to
  them and follows their own instructions.
- Not a dependency installer. It checks dependencies and tells you what is
  missing.

## Install (Claude Code)

### 1. Required dependencies

All three are required. `init` reports **BLOCKED** if any one is missing.

**Superpowers** (Claude Code plugin from the official marketplace):

```bash
claude plugin install superpowers@claude-plugins-official
```

**Tallyback** (CLI, built from source; needs Node.js 20+ and Git):

```bash
git clone https://github.com/FoFxjc/tallyback.git ~/.local/share/tallyback
cd ~/.local/share/tallyback && npm ci && npm run build && npm link
tallyback handshake
```

**Earn-it** (user-level Skill):

```bash
git clone https://github.com/FoFxjc/earn-it.git ~/.local/share/earn-it
mkdir -p ~/.claude/skills/earn-it
cp ~/.local/share/earn-it/SKILL.md ~/.claude/skills/earn-it/SKILL.md
```

These follow each project's own README. If they differ, the upstream README wins.

### 2. development-workflow (once per machine)

```bash
git clone https://github.com/FoFxjc/dev-workflow.git ~/.local/share/dev-workflow
mkdir -p ~/.claude/skills
ln -sfn ~/.local/share/dev-workflow/development-workflow ~/.claude/skills/development-workflow
```

This makes the Skill available to every new Claude Code session in every
project. Nothing is copied into individual repositories.

- Update: `git -C ~/.local/share/dev-workflow pull`
- Uninstall: `rm ~/.claude/skills/development-workflow`

Start a **new** Claude Code session after installing so the Skills are
discovered.

## Initialize a repository (once per repository)

Open Claude Code in the root of a Git repository and send:

```text
development-workflow init
```

The agent then:

1. **Checks dependencies** as the current session sees them. Superpowers'
   `using-superpowers`, `brainstorming`, `writing-plans`, and `writing-skills`
   must be available and readable. `tallyback version` and
   `tallyback handshake` must succeed. The `earn-it` Skill must be available
   and readable. If anything is missing, it reports **BLOCKED**, names what is
   missing, and does not touch `AGENTS.md`.
2. **Creates `AGENTS.md`** from
   [`references/agents-template.md`](development-workflow/references/agents-template.md)
   if the repository has none.
3. **Handles an existing `AGENTS.md`** by showing a diff and asking for explicit
   confirmation first. By default your existing rules are kept verbatim under
   `## Project-specific rules`, and any rule it proposes to drop is named for
   your approval. On confirmation, the original is backed up to
   `AGENTS.md.bak-<YYYYmmddHHMMSS>` before writing. If you decline, nothing
   changes.
4. **Verifies** by reading the file back and reports **READY**.

Init is idempotent. If `AGENTS.md` already starts with the template, it writes
nothing and makes no backup. Commit the resulting `AGENTS.md`.

## Daily use

Open a new session and describe the work. You do not need to name
`development-workflow` or any sub-Skill:

| Situation | Handed off to |
| --- | --- |
| Any task | Relevant existing Knowledge Skills first; explore only the gaps |
| Unclear or new behavior | `superpowers:brainstorming` |
| Multi-step work | `superpowers:writing-plans` |
| Implementing, testing, debugging, verifying | `superpowers:test-driven-development`, `systematic-debugging`, `verification-before-completion` |
| Work actually delegated to a subagent or worker | Tallyback ledger (declare, dispatch, claim, verify, explicit settle) |
| Every completed code change | `earn-it` review; fix only evidenced findings, re-verify, stop |
| Verified, reusable project or service knowledge | `superpowers:writing-skills` → project Skill in `.claude/skills/<name>/` |

Simple tasks skip planning, delegation, and knowledge capture.

**About Tallyback:** it must be installed, but its ledger is used **only when
work is actually delegated**. The workflow never invents workers or delegation
to fill a ledger. A worker's "done" is recorded as a Claim, not proof. An
Earn-it `PASS` never settles a Tallyback task; settlement is a separate,
explicit `tallyback settle`. When delegation is tracked, Tallyback keeps its
ledger in a `.tallyback/` directory in the repository.

### Example

```text
$ cd my-project && claude
> development-workflow init
  Superpowers ✓  Tallyback ✓  Earn-it ✓ → AGENTS.md created → READY

$ claude        # a new session
> Add a --shout flag to app.py that prints the greeting in upper case.
  → development-workflow → app-knowledge (existing Knowledge Skill)
  → superpowers:test-driven-development (failing test, then implementation)
  → earn-it: PASS → report with the commands that were run
```

## Component boundaries

- **development-workflow** only chooses and hands off.
- **Superpowers** owns the development method.
- **Tallyback** owns the delegated-work ledger.
- **Earn-it** owns implementation review. Its verdict does not replace tests.
- **Knowledge Skills** hold verified, reusable understanding. They never go in
  `AGENTS.md`, and not every task produces one.

Evidence order: repository facts and executable checks > Git facts > Tallyback
records > agent claims.

## Known limitations

- **Verified on Claude Code only** (2.1.x, with Superpowers 6.4.1,
  Tallyback 0.1.0, and Earn-it 0.1.0). Other agent hosts that read `AGENTS.md`
  have not been tested and are not claimed to be compatible.
- **Routing is instruction-based**, not enforced. It was checked with a small
  number of real headless sessions. The model can still choose differently,
  for example skipping a plan when it judges a task simple.
- **Knowledge capture** (creating a new Knowledge Skill) has not been exercised
  in testing.
- **No automated test suite.** Behavior was verified with live Claude Code
  sessions in throwaway repositories.
- **Tallyback ships from source only** and needs Node.js 20+.

## License

[MIT](LICENSE)
