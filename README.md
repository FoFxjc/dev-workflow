# dev-workflow

A minimal, Skill-native development workflow for coding agents.

**The goal is to coordinate existing capabilities, not build another agent framework.** The workflow has one explicit bootstrap operation, `development-workflow init`, and then guides agents to use existing Skills and tools for development.

> Status: **V0 implemented** as a Skill in [`development-workflow/`](development-workflow/SKILL.md). There is no CLI.

## What it does

1. **Initialize a repository** with a small, durable `AGENTS.md` entry point.
2. **Verify required dependencies** before declaring the project ready.
3. **Route development work** to the appropriate existing Skills, without reimplementing them.
4. **Capture verified reusable knowledge** about a project or integrated service as Knowledge Skills, when worthwhile.
5. **Preserve accountability** for delegated work through Tallyback and review implementations with Earn-it.

## Required dependencies

All three are required for a successful V0 initialization:

| Dependency | Purpose | Initialization check |
| --- | --- | --- |
| [Superpowers](https://github.com/obra/superpowers) | Exploration, design, planning, implementation, testing, and Skill authoring | Discover and read the relevant installed Skills, including `using-superpowers`, `brainstorming`, `writing-plans`, and `writing-skills`; report missing capabilities |
| [Tallyback](https://github.com/FoFxjc/tallyback) | Git-native accountability ledger for delegated work | Confirm the actual CLI is available and responds to a safe, read-only help/version check; do not infer installation from a same-named Skill |
| [Earn-it](https://github.com/FoFxjc/earn-it) | Evidence-oriented implementation review: DELETE / SIMPLIFY / TRACE / PROVE | Discover and read its installed `SKILL.md` |

Check availability **from the current agent host**, rather than assuming a single global Skill directory. Do not install or modify dependencies without explicit user authorization. Missing any required dependency means initialization is **BLOCKED**, not successful.

**Not included in V0:** SpecLock. Knowledge Skills are created or discovered as needed, not a preinstalled dependency.

## First step: `development-workflow init`

The user explicitly invokes this operation once after installing the `development-workflow` Skill. It is a **Skill instruction**, not a requirement to ship a new executable.

### Initialization contract

1. **Preflight:** Identify the repository root and current agent host. Check all three required dependencies, including whether the needed Skills are actually discoverable and readable.
2. **Inspect:** Read the existing root `AGENTS.md`, if present, and identify important project-specific rules that replacement could remove. Detect existing project/service Knowledge Skills without generating new ones.
3. **Confirm:** Show dependency results and the proposed `AGENTS.md` content or diff. If replacing an existing file, obtain explicit confirmation; do not silently discard repository constraints.
4. **Write:** Only after successful preflight and confirmation, back up the previous `AGENTS.md` (when present) and write the standard entry point. Preserve relevant project-specific constraints or explicitly surface their removal for approval.
5. **Verify:** Read back the written file, confirm it points to `development-workflow`, and report **READY** only when all required dependencies and file checks pass. Otherwise report **BLOCKED** with actionable reasons.

If preflight fails or the user declines replacement, **do not modify `AGENTS.md`**. Re-running `init` on an already-correct file should be idempotent: avoid unnecessary writes and backups.

The root `AGENTS.md` is an **entry point**, not a second workflow specification or a dumping ground for project knowledge. Its purpose is to tell future agent sessions to load and follow `development-workflow` for development tasks.

### Intended generated `AGENTS.md`

```markdown
# Agent Development Instructions

This repository uses the `development-workflow` Skill.

For software development tasks, load and follow `development-workflow`
before planning or implementation.

The workflow coordinates:
- Superpowers for exploration, planning, implementation, and verification.
- Tallyback for accountability when work is delegated.
- Earn-it for implementation review.
- Knowledge Skills for verified, reusable project or service understanding.

Follow each selected Skill's own instructions. Do not duplicate their
procedures here. Treat repository facts and executable checks as stronger
evidence than agent claims.
```

The implementation may refine this template, but must keep it short and avoid duplicating the underlying Skills.

## Normal development workflow

```text
User problem
   |
   v
Discover relevant Skills and existing project/service knowledge
   |
   v
Clarify the problem when necessary (Superpowers brainstorming)
   |
   v
Plan when warranted (Superpowers writing-plans)
   |
   v
Implement, test, debug, and verify (Superpowers)
   |                          |
   |                    If work is delegated:
   |                    Tallyback accountability
   v
Earn-it implementation review
   |
   +-- Concrete issue -> targeted correction and re-verification
   |
   +-- Evidence sufficient -> report outcome and stop
```

This is **routing guidance, not a mandatory stage machine**. Skip unnecessary steps on small tasks. Reuse existing plans, tests, and evidence; do not manufacture artifacts just to satisfy the diagram.

### Component boundaries

- **Superpowers** owns its own workflows and invocation rules. `development-workflow` chooses and hands off; it does not clone Superpowers instructions.
- **Tallyback** is a required installed capability, but its ledger is used according to its **delegated-work accountability** semantics. Do not create fake workers or delegation merely to populate a ledger. Distinguish Task, Attempt, Claim, Evidence, Verdict, and Settlement; a claim is not proof, and a review result is not a settlement.
- **Earn-it** reviews the actual implementation for unnecessary complexity and strength of evidence. Its verdict does not replace tests, independent acceptance, Git facts, or Tallyback settlement. `Leave alone` is a valid review outcome; do not invent findings.
- **Knowledge Skills** capture verified reusable understanding of this repository or an integrated service (interfaces, invariants, contracts, pitfalls). Use Superpowers `writing-skills` when authoring or improving them. Verify facts against code, tests, docs, or real service behavior, and verify that another agent can use the Skill. Do not put this knowledge into `AGENTS.md`. Do not create a Knowledge Skill for every task.

## V0 non-goals

- No new CLI, daemon, scheduler, orchestrator, agent hierarchy, runtime, or state database.
- No automatic dependency installation.
- No forced delegation or forced Knowledge Skill creation.
- No duplicated Superpowers, Tallyback, or Earn-it implementation.
- No SpecLock integration.
- No endless review/rewrite loops; stop when evidence is sufficient or the remaining blocker is explicit.
- No licensing work required for this initial private-use workflow.

## Suggested repository layout

```text
dev-workflow/
├── README.md
└── development-workflow/
    ├── SKILL.md
    └── references/
        └── agents-template.md
```

The implementation can adjust placement for the target host's Skill discovery rules; the **Skill name and init operation** should remain `development-workflow` and `development-workflow init`.

## V0 acceptance criteria

1. **Clean bootstrap:** Given an installed Skill, a new repository, and all three dependencies, an explicit `development-workflow init` creates a valid root `AGENTS.md` and reports READY.
2. **Missing dependency:** If any required dependency is unavailable or unreadable, initialization reports BLOCKED, identifies the missing dependency, and does not modify `AGENTS.md`.
3. **Existing file:** When `AGENTS.md` exists, initialization presents the proposed replacement, requires confirmation, backs up the original, and does not silently erase project-specific rules.
4. **Idempotence:** Re-running init on an already-initialized, unchanged repository makes no unnecessary changes.
5. **Session routing:** In a fresh session, the generated `AGENTS.md` directs the agent to the installed `development-workflow` Skill for development tasks.
6. **Appropriate restraint:** A trivial fix does not force a plan, new Knowledge Skill, delegated worker, or ledger entry. A genuinely delegated task uses Tallyback.
7. **Review boundary:** Earn-it review findings trigger only justified corrections; a review PASS does not silently settle a Tallyback task.

Test these with realistic clean/missing-dependency/existing-file cases and at least one fresh-session routing test. Report observed results rather than assuming that the generated `AGENTS.md` guarantees compliance.

## Implementation guidance for Claude

Implement the smallest viable Skill that satisfies the acceptance criteria. Start by inspecting the actual installed Superpowers, Tallyback, and Earn-it interfaces and their authoritative instructions; do not guess command syntax or Skill discovery behavior. Reuse their conventions. Prefer declarative Skill instructions and a template; add a helper script **only if** deterministic verification proves instructions alone are insufficient. Do not broaden scope without a concrete failure and an explicit reason.
