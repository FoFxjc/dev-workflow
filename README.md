# dev-workflow

**Build the workflows you need. Reuse the skills you already have.**

`development-workflow` is one [Agent Skill](development-workflow/SKILL.md): a
small, development-focused Workflow Core for coding agents. It selects,
composes, invokes and reuses existing Skills, and it does not reimplement
them.

## Why it exists

Coding agents can write working code and still drift from the agreed process:
- skip a review;
- change the route mid-task;
- treat a delegated worker's "done" as proof.

Repeating instructions every session does not make the process reliable.

The goal is **stable, reusable execution procedures**, not lower token usage
or cheaper runs. Required checks may cost more up front. Skipped stages and
rework usually cost more later.

Principles:

- **Reuse first:** use existing Skills and the host's native capabilities
  whenever they cover the need.
- **Bridge second:** connect existing capabilities only where a real
  integration gap exists. The `CLAUDE.md` → `AGENTS.md` import is one example.
- **Build last:** add a new mechanism only when reuse and bridging cannot
  solve the problem.
- **Workflow fidelity:** once an execution procedure is validated, stabilize
  it as a workflow instead of improvising it again.
- **Minimal complexity:** every additional abstraction has to justify its
  existence.

## How the Workflow Core works

1. **Understand the requirement first,** with `superpowers:brainstorming`.
2. **Select a workflow.** A repository can store approved routes as workflow
   Skills, `.claude/skills/workflow-<type>/SKILL.md`. Each one is scoped by a
   precise "Use when" and "Not for".
   - Matching uses specific problem semantics, constraints and acceptance
     evidence, not generic task categories.
   - Two problems that happen to invoke the same Skills are not the same
     workflow.
   - A matching workflow is reused without approving its route again. The
     task's design still gets its normal approval.
3. **No workflow fits?** Compose a one-off **single-point route** from the
   available Skills. It is approved together with the design: one decision
   when there is one reasonable route, or 2–3 options when routes genuinely
   differ.
4. **Run the route.** Each stage invokes its named Skill. The report lists
   every stage as executed, skipped, blocked or substituted, with evidence.
5. **After the run:**
   - If the run showed a saved workflow should change, the agent proposes the
     edit.
   - A single-point route may be proposed as a new workflow only when you say
     the problem will recur, or the repository history shows it recurring.
   - Either change needs your **separate** approval.
   - Narrow workflows are never widened into catch-alls.

Three things are kept apart:
- **Skill invocation:** the Skill was loaded.
- **Stage completion:** the Skill's required output exists, for example an
  Earn-it verdict.
- **Independent verification:** tests or checks you can run yourself.

None of them stands in for another.

The Core only coordinates. It holds no procedures of its own, and it is not a
CLI, runtime, orchestrator, scheduler or state database.

### Project instructions: `AGENTS.md`, bridged into `CLAUDE.md`

`AGENTS.md` is the single source of the project's development instructions.
Agents that read `AGENTS.md` use it directly. The root `CLAUDE.md` contains
the import line `@AGENTS.md`, so Claude Code loads the same instructions
through its native mechanism. They are never copied into `CLAUDE.md`.

## Existing Skills and their responsibilities

Capabilities come from any installed Skill: built-in, plugin, user or project.
Three are required.

| Component | Responsibility |
| --- | --- |
| **[Superpowers](https://github.com/obra/superpowers)** | The development method: understanding the requirement, planning, test-driven implementation, verification |
| **[Earn-it](https://github.com/FoFxjc/earn-it)** | Reviews the finished change for unjustified complexity and unsupported claims. Its verdict does not replace tests |
| **[Tallyback](https://github.com/FoFxjc/tallyback)** | Accountability ledger, used **only when work is actually delegated**. A worker's "done" is a Claim, not proof. An Earn-it `PASS` never settles a task; settlement is an explicit `tallyback settle` |
| **Workflow Skills** (`workflow-*`, per repository) | Approved routes only, one kind of problem each: which capabilities, in which order, connected how. No procedures, no project knowledge |
| **Knowledge Skills** (yours) | Verified, reusable understanding of a repository or service. Never in `AGENTS.md`, and most tasks produce none |
| **development-workflow** | Chooses, connects and hands off. Each selected Skill's own instructions govern how it runs |

Evidence order: repository facts and executable checks > Git facts >
Tallyback records > agent claims.

## Install and initialize (Claude Code)

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

These follow each project's own README. If they differ, the upstream README
wins.

### 2. development-workflow (once per machine)

```bash
git clone https://github.com/FoFxjc/dev-workflow.git ~/.local/share/dev-workflow
mkdir -p ~/.claude/skills
ln -sfn ~/.local/share/dev-workflow/development-workflow ~/.claude/skills/development-workflow
```

This makes the Skill available to every new Claude Code session in every
project. Nothing is copied into individual repositories. Start a **new**
session after installing, so the Skills are discovered.

- **Update:** run `git -C ~/.local/share/dev-workflow pull`, then run
  `development-workflow init` again in each repository. On an initialized
  repository this only adds what is missing, such as the `CLAUDE.md` import.
- **Uninstall:** `rm ~/.claude/skills/development-workflow`

### 3. Initialize a repository (once per repository)

Open Claude Code in the root of a Git repository and send:

```text
development-workflow init
```

`development-workflow init` is an instruction to the agent; no executable is
installed. The agent then:

1. **Checks dependencies** as the current session sees them.
   - Superpowers' `using-superpowers`, `brainstorming`, `writing-plans` and
     `writing-skills` must be available and readable.
   - `tallyback version` and `tallyback handshake` must succeed.
   - The `earn-it` Skill must be available and readable.

   If anything is missing, it reports **BLOCKED**, names what is missing, and
   does not touch `AGENTS.md` or `CLAUDE.md`.
2. **Creates `AGENTS.md`** from
   [`references/agents-template.md`](development-workflow/references/agents-template.md)
   if the repository has none.
3. **Handles an existing `AGENTS.md`** by showing a diff and asking for
   explicit confirmation first.
   - By default your existing rules are kept verbatim under
     `## Project-specific rules`.
   - Any rule it proposes to drop is named for your approval.
   - On confirmation, the original is backed up to
     `AGENTS.md.bak-<YYYYmmddHHMMSS>` before writing.
4. **Upgrades a v0.1 `AGENTS.md`** (one that starts with the
   [previous template](development-workflow/references/agents-template-v0.1.md))
   by replacing only that leading template text. The same diff, confirmation
   and backup apply, and your other content stays as it is.
5. **Connects `CLAUDE.md`.**
   - No root `CLAUDE.md`: it creates one containing only `@AGENTS.md`.
   - A `CLAUDE.md` with a line that is exactly `@AGENTS.md` (outside code
     blocks) is left alone.
   - Otherwise it shows the two lines it would append (a blank line and
     `@AGENTS.md`) and asks for explicit confirmation. On confirmation, the
     original is backed up to `CLAUDE.md.bak-<YYYYmmddHHMMSS>` and every
     existing line is kept.
   - Other content in `CLAUDE.md` applies to Claude Code only, so rules meant
     for every agent belong in `AGENTS.md`.
6. **Verifies** by reading both files back. It reports **READY** only if the
   dependencies passed, `AGENTS.md` starts with the current template, and
   `CLAUDE.md` contains the import.

If init does not finish:
- **You decline either change:** it writes neither file and reports
  **BLOCKED (declined)**. Without the import it is never READY.
- **`AGENTS.md` or `CLAUDE.md` is a symlink, a directory, or not
  writable:** it reports **BLOCKED** with the reason and changes nothing.

Init is idempotent. If both files are already in place, it writes nothing and
makes no backup. Init does not create any routes. Commit the resulting
`AGENTS.md` and `CLAUDE.md`.

## Daily use

Open a new session and describe the work. You do not need to name
`development-workflow` or any sub-Skill.

- **Facts and decisions.** The agent looks facts up itself (the repo,
  Knowledge Skills, commands, an Explore subagent for broad searches). It asks
  you only for decisions, one at a time, each with a recommended answer.
- **Approval.** A single-point route or a changed workflow is approved at the
  plan's existing approval point: the in-chat design, or the `writing-plans`
  execution handoff.
  - With one reasonable route, one "yes" approves the design and the route.
  - Only meaningfully different routes are offered as 2–3 options. Each shows
    its stages, Skills, cost, and whether Tallyback is used.
- **Execution.**
  - A passing command does not replace a required Skill call.
  - Every code change gets an `earn-it` review. Only findings with evidence
    are fixed, then re-verified.
- **Small tasks** get a short requirement check and a short route, not
  ceremony.

A route step uses the
[workflow template](development-workflow/references/workflow-template.md)
format:

```text
1. Implement — superpowers:test-driven-development — via Skill tool — done when tests pass
2. Verify — superpowers:verification-before-completion — via shell commands — done when fresh output is kept
3. Review — earn-it — via Skill tool — done when a verdict is written and evidence-backed findings are fixed
```

Example workflow names are narrow, for example `workflow-cli-option-change` or
`workflow-iterator-wrong-result`. They are illustrations, not prescribed
categories.

### Example

```text
$ cd my-project && claude
> development-workflow init
  Superpowers ✓  Tallyback ✓  Earn-it ✓ → AGENTS.md created → CLAUDE.md created (@AGENTS.md) → READY

$ claude        # first CLI change: no workflow yet
> Add a --shout flag to app.py that prints the greeting in upper case.
  → development-workflow → superpowers:brainstorming (short design)
  → no workflow fits → design + single-point route, one approval
  → TDD → verification → earn-it verdict → stage-by-stage report

> Add a --sort flag too. We add CLI flags all the time.
  → design + route approval → TDD → verification → earn-it verdict
  → after the run: "Save this route as workflow-cli-option-change?"
  → separate yes (Claude Code asks permission to write .claude/skills/…)

$ claude        # later, a similar change
> Add a --name option so it greets that name.
  → matches workflow-cli-option-change → only the design is approved;
    the route is not asked again

$ claude        # a different kind of problem
> Add a GitHub Actions CI configuration that runs pytest on every push.
  → workflow-cli-option-change says "Not for: CI" → design + single-point route
```

## Evidence and known limitations

### Evidence

- **What was run.** The workflow was exercised in isolated, headless Claude
  Code sessions (2.1.283, Sonnet) on historical `more-itertools` tasks, with
  scripted approvals.
- **Functional results.** Every task implementation passed its independent
  hidden acceptance tests.
- **Workflow reuse:**
  - a bug-fix workflow proposed by the agent after its first run was selected
    in each later bug-fix session;
  - one feature workflow, written by the experimenter rather than saved by an
    agent, was selected in its single reuse check;
  - neighbouring workflows were rejected by their "Not for".
- **CLAUDE.md bridge.** Init behaviour was checked in 11 isolated repository
  scenarios: new or existing files, decline, a repeated run, v0.1 migration,
  a symlink, and a read-only file. These tests found one defect, an
  overwritten read-only file, which was fixed and re-tested. A live check
  confirmed that `@AGENTS.md`
  is imported and that `development-workflow` is invoked through it.
- **What this does not show.** These are small samples. They are **not proof
  that workflows improve functional correctness or reduce cost**, and no
  comparison without the Skill was run. An instruction-only Earn-it A/B test
  (two runs per arm) showed no incremental benefit, and that wording was not
  adopted.

### Known limitations

- **Verified on Claude Code only** (2.1.x, with Superpowers 6.4.1,
  Tallyback 0.1.0 and Earn-it 0.1.0). Other agent hosts that read `AGENTS.md`
  have not been tested and are not claimed to be compatible.
- **Routing is instruction-based, not enforced.** The model can still choose
  differently. For example, it once asked to re-approve an unchanged
  workflow's route.
- **Earn-it verdicts are not always written.** The Skill was always invoked,
  but in some runs no verdict was written before the task was reported
  complete. The agent then reported the review stage as open, rather than
  claiming it ran. Check that the report contains a verdict.
- **Not exercised in testing:**
  - knowledge capture, i.e. creating a new Knowledge Skill;
  - delegated work tracked with Tallyback;
  - the architectural `writing-plans` path.
- **Workflow tool steps.** Claude Code requires your explicit opt-in for the
  Workflow tool. Picking a route option that names a Workflow step is meant as
  that opt-in, but this has not been verified.
- **`CLAUDE.md` and `AGENTS.md` in Claude Code.** Observed in 2.1.283:
  - the built-in AGENTS.md support loads `AGENTS.md` only when the repository
    has no `CLAUDE.md`, so an existing `CLAUDE.md` without the import hides
    `AGENTS.md`;
  - with the import, the instructions are loaded once, not twice.

  Init manages only the root `CLAUDE.md`, not `.claude/CLAUDE.md` or
  `CLAUDE.local.md`. A symlinked `CLAUDE.md` is reported BLOCKED rather than
  rewritten.
- **Headless sessions (`claude -p`)** cannot ask interactive questions. The
  agent presents the design with its route (or route options) as text and
  stops.
- **Interactive prompts not exercised.** That covers interactive
  `AskUserQuestion` and the `.claude/` permission prompt. Saving a workflow
  writes under `.claude/`, which Claude Code guards with its own prompt (part
  of the approval). In headless sessions that write is refused, so the agent
  shows the file content and stops.
- **No automated test suite in this repository.** Behaviour was checked
  through isolated Claude Code sessions. A separate local evaluation harness
  was used for trace-based checks; it is not shipped with this Skill.
- **Tallyback ships from source only** and needs Node.js 20+.

## License

[MIT](LICENSE)
