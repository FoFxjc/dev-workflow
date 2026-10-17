# dev-workflow

A minimal, Skill-native development workflow for coding agents.

## What it is

`development-workflow` is a manual for using Skills: one
[Agent Skill](development-workflow/SKILL.md) that turns each development
request into a **route**, an ordered chain of existing Skills connected by
Claude Code's native capabilities (the Skill tool, subagents,
`AskUserQuestion`, plan approval, worktrees, the Workflow tool). Routes you
approve are stored as **workflow Skills** in the repository, one per kind of
problem (`.claude/skills/workflow-<type>/SKILL.md`), so a dynamic workflow
becomes a fixed one. Workflow first, single-point second: when no workflow fits,
a route is composed for that one task and can later be saved as a new workflow.

Capabilities come from any installed Skill: built-in, plugin, user, or
project. Three are required:

- **[Superpowers](https://github.com/obra/superpowers)** for understanding the
  requirement, planning, implementing, testing, and verifying.
- **[Tallyback](https://github.com/FoFxjc/tallyback)** for accountability when
  work is actually delegated to another agent.
- **[Earn-it](https://github.com/FoFxjc/earn-it)** for reviewing the finished
  implementation for unjustified complexity and unsupported claims.

Your own **Knowledge Skills** hold verified, reusable understanding of a
repository or service.

You set up a repository once with `development-workflow init`. That writes a
short root `AGENTS.md` that makes every future session **workflow-first**:
understand the requirement, run the matching workflow (or a single-point route),
get any new or changed route approved as part of the plan, then run it.

## What problem it solves

Good Skills exist, but each session has to discover them, chain them in the
right order, and do it again next time. Agents tend to jump straight into
editing code. Delegated work comes back as a worker's "done" rather than
evidence. Review tends to be skipped or turned into an endless rewrite loop.
This Skill puts understanding and a route before code, keeps approved routes
in one file, and connects the pieces without reimplementing any of them.

## What it is not

- **Not a CLI.** `development-workflow init` is an instruction you give the
  agent inside a session. No executable is installed.
- Not an agent framework, orchestrator, scheduler, runtime, or state database.
- Not a replacement for Superpowers, Tallyback, Earn-it, or any other Skill.
  It hands off to them and follows their own instructions.
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
4. **Upgrades a v0.1 `AGENTS.md`** (one that starts with the previous
   template) by replacing only that leading template text, after the same
   diff, confirmation, and backup. Your other content stays as it is.
5. **Verifies** by reading the file back and reports **READY**.

Init is idempotent. If `AGENTS.md` already starts with the current template, it
writes nothing and makes no backup. Init does not create any routes. Commit the
resulting `AGENTS.md`.

## Daily use

Open a new session and describe the work. You do not need to name
`development-workflow` or any sub-Skill. Each task goes through these steps:

1. **Understand the requirement** with `superpowers:brainstorming`. The agent
   looks facts up itself (in the repo, Knowledge Skills, or commands, using an
   Explore subagent for broad searches). It asks you only for decisions, one
   at a time, each with a recommended answer (`AskUserQuestion`).
2. **Select a workflow** by comparing the requirement with each
   `workflow-*` Skill's "Use when" and "Not for". If several fit, the most
   specific wins, or you are asked. If none fits, it composes a
   **single-point route** for this task from the Skills available in the
   session, reading their descriptions broadly and opening only the
   candidates.
3. **Approve the route as part of the plan.** A single-point route or a changed
   workflow is offered as 2–3 options at the plan's existing approval point
   (the in-chat design, or the `writing-plans` execution handoff). Each option
   lists its stages, Skills, native capabilities, cost, and whether Tallyback
   is used. An approved workflow is reused without asking again.
4. **Run the route.** Every code change gets an `earn-it` review. Only findings
   with evidence are fixed, then re-verified.
5. **Update or solidify.** If the run showed a workflow should change, the agent
   proposes the edit. The edit is applied with an `Updated:` date once you
   approve. If a single-point route worked and the problem will recur, it
   proposes saving it as a new workflow, or extending an overlapping one.

Example workflow Skills in one repository: `workflow-feature`,
`workflow-bugfix`, `workflow-small-change`, `workflow-db-migration`.

A route step looks like this
([template](development-workflow/references/workflow-template.md)):

```text
1. Understand — superpowers:brainstorming — via AskUserQuestion — done when the design is approved
2. Implement — superpowers:test-driven-development — via Skill tool — done when tests pass
3. Review — earn-it — via Skill tool — done when VERDICT is PASS or findings are fixed
```

Small tasks get a short requirement check and a short route, not ceremony.

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

- **development-workflow** only chooses, connects, and hands off.
- **Workflow Skills** (`workflow-*`, per repository) hold approved routes only,
  one kind of problem each: which capabilities, in which order, connected how.
  No procedures, no knowledge.
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
- **Workflow tool steps:** Claude Code requires the user's explicit opt-in for
  the Workflow tool. Picking a route option that names a Workflow step is meant
  as that opt-in, but this has not been verified.
- **Headless sessions** (`claude -p`) cannot ask interactive questions. The
  agent presents route options as text and stops.
- **No automated test suite.** Behavior was verified with live Claude Code
  sessions in throwaway repositories.
- **Tallyback ships from source only** and needs Node.js 20+.

## License

[MIT](LICENSE)
