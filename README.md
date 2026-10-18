# dev-workflow

A Skill-native way to make coding agents follow repeatable development
processes instead of improvising a new one for every task.

## Why it exists

Coding agents can write working code and still drift from the agreed process:
skip a review, change the route mid-task, or treat a delegated worker's
"done" as proof. Repeating instructions every session does not make the
process reliable.

**The goal is process stability, not lower token usage or cheaper runs.**
Required checks may cost more up front; skipped stages and rework can cost
more later. `development-workflow` makes the intended route explicit,
reuses it for the same kind of problem, and reports what actually ran.
It is instruction-based guidance, not an enforcement or acceptance engine.

## How it works

`development-workflow` is one
[Agent Skill](development-workflow/SKILL.md) that connects existing Skills
into an ordered **route**. A repository can store approved routes as
`.claude/skills/workflow-<type>/SKILL.md`, each scoped to a specific kind
of problem.

- **Matching workflow:** reuse its approved route without approving the
  route again. The task's design still needs its normal approval.
- **No matching workflow:** compose a one-off **single-point route** from
  available Skills and approve it with the design.
- **After execution:** if the problem recurs and the boundary is stable,
  propose saving or updating a workflow, with separate approval. Otherwise
  keep the route one-off.

Workflow matching is based on problem type, scope, constraints, and
acceptance evidence—not simply whether two routes invoke the same Skills.

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

`AGENTS.md` is the single source of the project's development rules, and agents
that read `AGENTS.md` use it directly. Init also makes sure the root
`CLAUDE.md` contains the import line `@AGENTS.md`, so Claude Code loads the
same rules through its native `CLAUDE.md` mechanism. The rules are never
copied into `CLAUDE.md`.

## What it is not

- **Not a CLI.** `development-workflow init` is an instruction you give the
  agent inside a session. No executable is installed.
- Not an agent framework, orchestrator, scheduler, runtime, or state database.
- Not a replacement for Superpowers, Tallyback, Earn-it, or any other Skill.
  It hands off to them and follows their own instructions.
- Not a dependency installer. It checks dependencies and tells you what is
  missing.

## Quick start (Claude Code)

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

- Update: `git -C ~/.local/share/dev-workflow pull`, then run
  `development-workflow init` again in each repository. On an initialized
  repository this only adds what is missing, such as the `CLAUDE.md` import.
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
   missing, and does not touch `AGENTS.md` or `CLAUDE.md`.
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
5. **Connects `CLAUDE.md`.** If there is no root `CLAUDE.md`, it creates one
   containing only `@AGENTS.md`. If `CLAUDE.md` already has a line that is
   exactly `@AGENTS.md`, it leaves the file alone. Otherwise it shows the two
   lines it would append (a blank line and `@AGENTS.md`) and asks for explicit
   confirmation. On confirmation, the original is backed up to
   `CLAUDE.md.bak-<YYYYmmddHHMMSS>` and every existing line is kept. Content
   already in `CLAUDE.md` stays; it applies to Claude Code only, so rules
   meant for every agent belong in `AGENTS.md`.
6. **Verifies** by reading both files back. It reports **READY** only if the
   dependencies passed, `AGENTS.md` starts with the current template, and
   `CLAUDE.md` contains the import.

If you decline either change, init writes neither file and reports
**BLOCKED (declined)**. Without the import it is never READY. If `AGENTS.md`
or `CLAUDE.md` is a symlink, a directory, or not writable, init reports
**BLOCKED** with the reason and changes nothing.

Init is idempotent. If `AGENTS.md` already starts with the current template and
`CLAUDE.md` already has the import, it writes nothing and makes no backup. Init
does not create any routes. Commit the resulting `AGENTS.md` and `CLAUDE.md`.

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
   workflow is approved at the plan's existing approval point (the in-chat
   design, or the `writing-plans` execution handoff). With one reasonable
   route, one "yes" approves the design and route together. Only meaningfully
   different routes are offered as 2–3 options, each with its stages, Skills,
   cost, and whether Tallyback is used. An approved workflow is reused without
   asking again.
4. **Run the route.** Each stage's named Skill must be invoked; a passing
   command does not replace a required Skill call. Code changes require an
   `earn-it` review, but invoking the Skill alone does not prove the review
   produced a verdict. The report lists each stage as executed, skipped,
   blocked, or substituted, with supporting evidence.
5. **Update or solidify.** If the run showed a workflow should change, the agent
   proposes the edit. The edit is applied with an `Updated:` date once you
   approve. Saving a single-point route as a workflow is suggested only after
   the run, when you say the problem will recur or history shows it recurring. Workflows are
   compared by problem type, scope, constraints, and acceptance evidence;
   identical steps alone don't make them the same. A problem that belongs to
   an existing workflow reuses it. A distinct, stable problem type may get its
   own workflow. Otherwise the single-point route is kept. Narrow workflows are
   never widened into catch-alls.

Example narrowly scoped workflow Skills: `workflow-cli-option-change`,
`workflow-iterator-wrong-result`, `workflow-db-schema-migration`.
These are examples, not prescribed categories.

Understanding the requirement always comes first and is not part of a route.
A route step looks like this
([template](development-workflow/references/workflow-template.md)):

```text
1. Implement — superpowers:test-driven-development — via Skill tool — done when tests pass
2. Review — earn-it — via Skill tool — done when VERDICT is PASS or findings are fixed
3. Verify — superpowers:verification-before-completion — via shell commands — done when fresh output is kept
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
  Superpowers ✓  Tallyback ✓  Earn-it ✓ → AGENTS.md created → CLAUDE.md created (@AGENTS.md) → READY

$ claude        # first CLI change: no workflow yet
> Add a --shout flag to app.py that prints the greeting in upper case.
  → development-workflow → superpowers:brainstorming (bounded: short design)
  → no workflow fits → design + single-point route, one approval
  → TDD → earn-it → verification-before-completion

> Add a --sort flag too. We add CLI flags all the time.
  → design + route approval → implementation → review → verification
  → after the run: "Save this recurring route as workflow-cli-change?"
  → separate yes (Claude Code asks permission to write .claude/skills/…)

$ claude        # later, a similar change
> Add a --name option so it greets that name.
  → matches workflow-cli-change → only the design is confirmed;
    the route is not asked again

$ claude        # a different kind of problem
> Add a GitHub Actions CI configuration that runs pytest on every push.
  → workflow-cli-change says "Not for: CI" → design + single-point route
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

## Evidence and limitations

The workflow has been exercised in isolated Claude Code sessions on
historical `more-itertools` tasks. Five task implementations passed their
independent functional acceptance tests, and two matching saved workflows
were selected correctly in reuse checks. These are small-sample observations,
**not proof that workflows improve functional correctness or reduce cost**.
An Earn-it instruction-only A/B test (two runs per arm) did not establish
an incremental benefit; that experimental wording was not adopted.

## Known limitations

- **Verified on Claude Code only** (2.1.x, with Superpowers 6.4.1,
  Tallyback 0.1.0, and Earn-it 0.1.0). Other agent hosts that read `AGENTS.md`
  have not been tested and are not claimed to be compatible.
- **Routing is instruction-based**, not enforced. It was checked with a small
  number of real headless sessions. The model can still choose differently,
  for example skipping a plan when it judges a task simple.
- **Earn-it verdicts are not always written.** In headless evaluation runs the
  `earn-it` Skill was always invoked, but in some runs no verdict was written
  before the task was reported complete. The agent then reported the review
  stage as open rather than claiming it ran. Check that the report contains a
  verdict.
- **Knowledge capture** (creating a new Knowledge Skill) and **delegated work
  tracked with Tallyback** under workflow-first routing have not been exercised
  in testing.
- **Workflow tool steps:** Claude Code requires the user's explicit opt-in for
  the Workflow tool. Picking a route option that names a Workflow step is meant
  as that opt-in, but this has not been verified.
- **`CLAUDE.md` and `AGENTS.md` in Claude Code.** Observed in 2.1.283:
  - Claude Code's built-in AGENTS.md support loads `AGENTS.md` only when the
    repository has no `CLAUDE.md`. An existing `CLAUDE.md` without the import
    hides `AGENTS.md` from Claude Code, which is why init adds the import.
  - With the import, the rules are loaded once, not twice.

  Init manages only the root `CLAUDE.md`, not `.claude/CLAUDE.md` or
  `CLAUDE.local.md`. A `CLAUDE.md` that is a symlink, for example one pointing
  to `AGENTS.md`, is reported BLOCKED instead of being rewritten.
- **Headless sessions** (`claude -p`) cannot ask interactive questions. The
  agent presents the design with its route (or route options) as text and
  stops.
- **Saving a workflow writes under `.claude/`**, which Claude Code protects
  with its own permission prompt (that prompt is part of the approval). In
  headless sessions the write is refused even with an allow rule, so the agent
  shows the file content and stops. The interactive prompt itself has not been
  exercised in testing; saving was tested by writing the approved file by
  hand.
- **No automated test suite in this repository.** Skill behavior was checked
  through isolated Claude Code sessions. A separate local evaluation harness
  was used for trace-based checks; it is not shipped as part of this Skill.
- **Tallyback ships from source only** and needs Node.js 20+.

## License

[MIT](LICENSE)
