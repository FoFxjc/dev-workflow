---
name: development-workflow
description: Use for any software development task in a repository whose AGENTS.md points here (features, bug fixes, refactors, reviews, delegated work), and when the user runs `development-workflow init` to set up a repository's AGENTS.md entry point.
---

# Development Workflow

Coordinate existing Skills and tools. Do not reimplement them. This Skill only
decides **which** capability to hand off to and **when to stop**; each selected
Skill's own instructions govern how it runs.

Required dependencies (install instructions live in each upstream README):
**Superpowers** (Skills, https://github.com/obra/superpowers), **Tallyback**
(CLI, https://github.com/FoFxjc/tallyback), **Earn-it** (Skill,
https://github.com/FoFxjc/earn-it). SpecLock is not part of this workflow.

## Operation: `development-workflow init`

Run only when the user explicitly asks for `development-workflow init`. Never
install or modify dependencies without explicit user authorization.

### 1. Preflight

Find the repository root (`git rev-parse --show-toplevel`) and name the current
agent host. Check each dependency **as this host sees it** — not a guessed global
directory, not a source checkout:

| Dependency | Passes only if |
| --- | --- |
| Superpowers | `using-superpowers`, `brainstorming`, `writing-plans`, and `writing-skills` are listed as available Skills in this session **and** you actually read each `SKILL.md`. List any other missing Superpowers Skills. |
| Tallyback | `tallyback version` and `tallyback handshake` (both read-only) exit 0 from the shell. A same-named Skill or a source checkout does not count. |
| Earn-it | An `earn-it` Skill is available in this session **and** you read its `SKILL.md`. |

Any failure → **BLOCKED**: report each missing or unreadable dependency with
what you checked and the official install route; touch nothing; stop.

### 2. Inspect

Read the root `AGENTS.md` if present. Compare it with
`references/agents-template.md`:

- **Already initialized:** the file begins with the template text exactly.
  Anything after it is preserved project content. Make no write and no backup;
  go to Verify.
- **Other existing content:** list each project-specific rule (commands,
  constraints, conventions, prohibitions) that replacement would remove.

Note existing project/service Knowledge Skills (for example under
`.claude/skills/`, `.agents/skills/`, or `skills/`). Do not generate new ones.

### 3. Confirm

Show the dependency results and the proposed file content.

- **No `AGENTS.md`:** the explicit `init` request is the confirmation; write
  the template as-is.
- **Existing `AGENTS.md`:** also show a diff. The default proposal is the
  template followed by `## Project-specific rules` holding the existing rules
  verbatim. Name any rule you propose to drop. Then **ask and wait for explicit
  confirmation** of the replacement. If you cannot ask in this turn, stop here
  and report that confirmation is pending.

If the user declines, do not modify `AGENTS.md`; report BLOCKED (declined).

### 4. Write

Only after a passing preflight (and confirmation, when replacing):

1. If `AGENTS.md` exists, copy it to `AGENTS.md.bak-<YYYYmmddHHMMSS>` first.
2. Write the confirmed content to the root `AGENTS.md`.

### 5. Verify

Read back `AGENTS.md`. Confirm it begins with the template text and names
`development-workflow`, and that the backup (if any) matches the original.
Report **READY** only if every dependency and file check passed; otherwise
**BLOCKED** with actionable reasons. Include the backup path when one was made.

## Development tasks: routing

This is routing guidance, not a stage machine. Skip steps a task does not need,
and reuse existing plans, tests, and evidence.

1. **Discover.** Check available Skills (Superpowers' `using-superpowers`
   governs how) and existing Knowledge Skills for this repo or service.
2. **Clarify** unclear or new behavior → `superpowers:brainstorming`.
3. **Plan** multi-step work → `superpowers:writing-plans`. A trivial fix needs
   no plan.
4. **Implement, test, debug, verify** → the matching Superpowers Skill
   (`test-driven-development`, `systematic-debugging`,
   `verification-before-completion`, ...).
5. **Delegated work** → Tallyback (below).
6. **Review** the actual implementation → `earn-it`. Fix only findings backed
   by evidence, re-verify, then stop. Do not loop review/rewrite once evidence is
   sufficient or the remaining blocker is explicit.
7. **Report** the outcome with evidence: what was run, what was not, what is
   unproven.

## Handoff boundaries

- **Superpowers** owns its workflows and invocation rules. Hand off; do not
  restate its procedures.
- **Tallyback** applies only when work is actually delegated (a subagent or
  another worker owns an attempt and returns a claim). Do not create fake
  workers or delegation to fill a ledger. Start with `tallyback handshake` and
  follow Tallyback's own Skill or `tallyback --help`. Keep Task, Attempt, Claim,
  Evidence, Verdict, and Settlement distinct: a worker's "done" is a Claim, not
  proof.
- **Earn-it** reviews complexity and strength of evidence. Its verdict does not
  replace tests, independent acceptance, Git facts, or Tallyback settlement.
  An Earn-it `PASS` never settles a Tallyback task; settlement is a separate,
  explicit `tallyback settle` decision. `Leave it alone.` is a valid outcome.
- **Knowledge Skills** hold verified reusable understanding of this repo or an
  integrated service (interfaces, invariants, contracts, pitfalls). Create or
  improve one only when the knowledge is reusable and verified against code,
  tests, docs, or real behavior, using `superpowers:writing-skills`. Never put
  this knowledge in `AGENTS.md`, and never create one per task.

Evidence order: repository facts and executable checks > Git facts >
Tallyback records > agent claims and memory.
