# dev-workflow

A minimal, Skill-native development workflow for coding agents.

It coordinates existing capabilities instead of building another agent framework.
You run one explicit bootstrap, `development-workflow init`, and from then on every
new session in that repository routes development work to the right existing
Skills on its own:

- **Superpowers** for clarifying, planning, implementing, testing, and verifying.
- **Tallyback** for accountability when work is actually delegated.
- **Earn-it** for reviewing the finished implementation.
- **Knowledge Skills** for verified, reusable project or service understanding.

There is no CLI, daemon, orchestrator, or state database. The whole workflow is
one Skill ([`development-workflow/SKILL.md`](development-workflow/SKILL.md)) and an
`AGENTS.md` template.

## Install (Claude Code)

### 1. Required dependencies

All three are required. `init` reports **BLOCKED** if any one is missing.

```bash
# Superpowers (Claude Code plugin)
claude plugin install superpowers@claude-plugins-official

# Tallyback (CLI; needs Node.js 20+ and Git)
git clone https://github.com/FoFxjc/tallyback.git ~/.local/share/tallyback
cd ~/.local/share/tallyback && npm ci && npm run build && npm link

# Earn-it (user Skill)
git clone https://github.com/FoFxjc/earn-it.git ~/.local/share/earn-it
mkdir -p ~/.claude/skills/earn-it
cp ~/.local/share/earn-it/SKILL.md ~/.claude/skills/earn-it/SKILL.md
```

Check that Tallyback works with `tallyback handshake`.

### 2. development-workflow (global, once per machine)

```bash
git clone https://github.com/FoFxjc/dev-workflow.git ~/.local/share/dev-workflow
mkdir -p ~/.claude/skills
ln -sfn ~/.local/share/dev-workflow/development-workflow ~/.claude/skills/development-workflow
```

This installs the Skill at user level, so every new Claude Code session in
every project can find it. Nothing is copied into individual projects.

To update: `git -C ~/.local/share/dev-workflow pull`.
To uninstall: `rm ~/.claude/skills/development-workflow`.

## Initialize a project (once per repository)

Open Claude Code in the repository root and say:

```text
development-workflow init
```

Init will:

1. Check Superpowers, Tallyback, and Earn-it from the current session. If any is
   missing, it reports **BLOCKED**, names what is missing, and changes nothing.
2. Create the root `AGENTS.md` from
   [`references/agents-template.md`](development-workflow/references/agents-template.md).
3. If `AGENTS.md` already exists, show a diff and ask first. The original is
   backed up to `AGENTS.md.bak-<timestamp>`, and its project rules are kept
   verbatim under `## Project-specific rules`. If you decline, nothing changes.
4. Read the file back and report **READY**.

Re-running init on an initialized repository is a no-op: no write, no backup.
Commit the new `AGENTS.md`.

## Daily use

After init, just describe the work in a new session. You do not need to name
`development-workflow` or any sub-Skill. `AGENTS.md` tells the agent to load the
workflow, and the workflow picks the Skills:

| Situation | What the workflow does |
| --- | --- |
| Any task | Reads relevant Knowledge Skills first, explores only the gaps |
| Unclear or new behavior | `superpowers:brainstorming` |
| Multi-step work | `superpowers:writing-plans` |
| Implementing, testing, debugging | `superpowers:test-driven-development`, `systematic-debugging`, `verification-before-completion` |
| Work actually delegated to a subagent or worker | Tallyback ledger: declare, dispatch, claim, verify, explicit settle |
| Implementation finished | `earn-it` review, then targeted fixes and re-verification only for real findings |
| Verified, reusable project or service knowledge learned | `superpowers:writing-skills` → project Skill in `.claude/skills/<name>/` |

Simple tasks skip planning, delegation, and knowledge capture.

### Example

```text
$ cd my-project && claude
> development-workflow init
  ... Superpowers ✓  Tallyback ✓  Earn-it ✓ → AGENTS.md created → READY

$ claude        # new session
> Add a --shout flag to app.py that prints the greeting in upper case.
  → development-workflow → app-knowledge (existing Knowledge Skill)
  → superpowers:test-driven-development (failing test, then implementation)
  → earn-it review: PASS → report with the commands that were run
```

## Component boundaries

- **development-workflow** only chooses and hands off. It does not restate other
  Skills' procedures.
- **Superpowers** owns the development method and its own invocation rules.
- **Tallyback** owns the delegated-work ledger. It is used only for real
  delegation, never for fake workers. A worker's "done" is a Claim, not proof.
- **Earn-it** owns implementation review. Its `PASS` does not replace tests and
  never settles a Tallyback task; settlement is an explicit `tallyback settle`.
- **Knowledge Skills** hold verified, reusable understanding. They do not belong
  in `AGENTS.md`, and not every task produces one.

Evidence order: repository facts and executable checks > Git facts > Tallyback
records > agent claims.

## Non-goals

No CLI, daemon, scheduler, orchestrator, agent hierarchy, runtime, or state
database. No automatic dependency installation. No forced delegation or
Knowledge Skill creation. No duplicated Superpowers, Tallyback, or Earn-it logic.
No SpecLock. No endless review/rewrite loops.
