---
name: development-workflow
description: Use for every software development task in a repository whose AGENTS.md points here (features, bug fixes, refactors, reviews, delegated work), to understand the requirement, then run the matching `workflow-*` Skill (or a single-point route when none fits), and to create and update those workflows; also when the user runs `development-workflow init` to set up the AGENTS.md entry point.
---

# Development Workflow

A manual for using Skills. It turns a development request into a **route**:
an ordered chain of existing Skills, connected by the host's native
capabilities. Approved routes are stored as **workflow Skills**, one per kind of
problem (`.claude/skills/workflow-<type>/SKILL.md`), and reused. Workflow
first, single-point second: a route composed for one task is the fallback
when no workflow fits.

This Skill only chooses, connects, and hands off. Each selected Skill's own
instructions govern how it runs. Superpowers is one supplier of capabilities,
not the only one: built-in, plugin, user, and project Skills are all candidates.

Required dependencies (install instructions live in each upstream README):
**Superpowers** (Skills, https://github.com/obra/superpowers), **Tallyback**
(CLI, https://github.com/FoFxjc/tallyback), **Earn-it** (Skill,
https://github.com/FoFxjc/earn-it). SpecLock is not part of this workflow.

## Operation: `development-workflow init`

Run only when the user explicitly asks for `development-workflow init`. Never
install or modify dependencies without explicit user authorization. Init does
not create routes; routes are composed once a real requirement is understood.

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
`references/agents-template.md` (current) and
`references/agents-template-v0.1.md` (previous):

- **Already initialized:** the file begins with the current template text
  exactly. Anything after it is preserved project content. Make no write and
  no backup; go to Verify.
- **Previous template:** the file begins with the v0.1 template text exactly.
  Propose replacing only that leading text with the current template, keeping
  everything after it unchanged.
- **Other existing content:** list each project-specific rule (commands,
  constraints, conventions, prohibitions) that replacement would remove.

Note existing workflow Skills (`.claude/skills/workflow-*/SKILL.md`), and note
existing project/service Knowledge Skills (for example under `.claude/skills/`,
`.agents/skills/`, or `skills/`; this workflow and its dependencies are not
Knowledge Skills). Do not generate either.

### 3. Confirm

Show the dependency results and the proposed file content.

- **No `AGENTS.md`:** the explicit `init` request is the confirmation; write
  the template as-is.
- **Existing `AGENTS.md`:** also show a diff. For other content, the default
  proposal is the template followed by `## Project-specific rules` holding the
  existing rules verbatim. Name any rule you propose to drop. Then **ask and
  wait for explicit confirmation** of the replacement. If you cannot ask in
  this turn, stop here and report that confirmation is pending.

If the user declines, do not modify `AGENTS.md`; report BLOCKED (declined).

### 4. Write

Only after a passing preflight (and confirmation, when replacing):

1. If `AGENTS.md` exists, copy it to `AGENTS.md.bak-<YYYYmmddHHMMSS>` first.
2. Write the confirmed content to the root `AGENTS.md`.

### 5. Verify

Read back `AGENTS.md`. Confirm it begins with the current template text and
names `development-workflow`, and that the backup (if any) matches the
original. Report **READY** only if every dependency and file check passed;
otherwise **BLOCKED** with actionable reasons. Include the backup path when one
was made.

## Development tasks

Workflow-first: never start by editing code. Scale every step to the task; a
small task gets a short requirement check and a short route, not ceremony.

### 1. Understand the requirement

Invoke `superpowers:brainstorming` with the Skill tool before asking anything
or proposing routes; do not do its job inline. It owns the interview, its task
paths, and its approval gates. While it runs:

- **Facts are yours to find, decisions are the user's.** Answer anything the
  repository, its Knowledge Skills, or a command can tell you by looking it up
  (in Claude Code, a read-only Explore subagent for broad searches). Never ask
  the user for a fact. Only questions that depend on a pending lookup wait.
- **Every question carries a recommended answer.** In Claude Code, ask with
  `AskUserQuestion`, recommended option first. Follow brainstorming's own
  pacing (one question at a time).
- **Done** when the user has confirmed the understanding and no decision is
  silently assumed. Only then select a route.

### 2. Select a workflow, or compose a single-point route

List the workflow Skills in `.claude/skills/workflow-*/SKILL.md` and compare
each one's "Use when" and "Not for" with the understood requirement.

- **One fits:** select it. Check that every Skill it names is still
  available; a missing one makes it a changed workflow.
- **Several fit:** select the most specific. If it is still ambiguous, ask the
  user which one, with a recommended answer.
- **None fits:** compose a **single-point route** for this task only:

1. **Inventory widely, read narrowly.** The Skills listed in this session are
   the inventory: built-in, plugin, user, and project Skills. Read the
   descriptions; open the full `SKILL.md` only of candidates you may use.
   Capabilities that are not installed may be suggested to the user, never
   installed without authorization.
2. **Map each stage to a capability**, for example: understand
   (`brainstorming`), plan (`writing-plans`), implement and test
   (`test-driven-development`, `systematic-debugging`), review (`earn-it`, and
   other review Skills when they fit such as `security-review`), verify
   (`verification-before-completion`, the project's test commands), capture
   knowledge (`writing-skills`).
3. **Connect stages with native capabilities** (Claude Code):

   | Need | Native capability |
   | --- | --- |
   | Load a capability on demand | Skill tool |
   | A user decision or approval | `AskUserQuestion` (recommended option first) or plan approval |
   | Broad read-only fact finding | Explore subagent |
   | Independent work items | Subagents, launched in parallel when independent; each one that delivers changes is delegated work |
   | Isolated parallel changes | Git worktrees |
   | Large fan-out across many agents | Workflow tool, only with the user's explicit opt-in under that tool's rules |
   | Step progress | The session's task list |
   | Evidence | Shell commands whose output is kept |

4. **Write each step** in the format of `references/workflow-template.md`:
   `<stage> — <Skill or tool> — via <native capability> — done when <evidence>`.
   Name capabilities; never copy their procedures into the route.

### 3. Approve the route as part of the plan

- **Approved workflow, unchanged:** say which workflow you are running; no
  route approval is needed. The selected Skills' own gates (for example
  brainstorming's design approval) still apply.
- **Single-point route or changed workflow:** present it in the same approval
  the plan already has — with the in-chat design for bounded work, or at
  `writing-plans`' execution handoff for planned work — not as an extra round.
  Offer 2–3 route options with `AskUserQuestion`, recommended first. Every
  option keeps the required stages (understanding the requirement, `earn-it`
  review of code changes, verification); options differ in execution mode,
  depth, and cost, never by dropping those. Each
  description states the stages, Skills, native capabilities, rough cost, and
  whether Tallyback will track delegated work; the preview holds the full
  route. Choosing an option approves the route and its execution mode. A route
  with a Workflow step must name the Workflow and its cost in the option.
- If you cannot ask in this turn, present the options as text and stop.

After approval, write a changed workflow back to its Skill file and add an
`Updated:` line with today's date and the reason. A single-point route is not
written anywhere yet; see step 5.

### 4. Run the route

Follow the steps in order, invoking each named Skill. Then:

- **Delegated work** (a subagent or worker owns an attempt that delivers
  changes) → Tallyback, per the boundaries below. Read-only lookups and
  reviews are not delegated work.
- **Review** every completed code change → `earn-it`. Fix only findings backed
  by evidence, re-verify, then stop. Do not loop review/rewrite once evidence
  is sufficient or the remaining blocker is explicit.
- **Knowledge** → only when the task produced verified, reusable project or
  service understanding, capture it with `superpowers:writing-skills` as a
  project Skill (`.claude/skills/<name>/SKILL.md`). Most tasks produce none.

### 5. Update, solidify, and report

Ask with `AskUserQuestion`, recommended answer first, and change files only
after approval:

- **Update:** if the run deviated from the workflow, or showed a step is wrong
  or missing, propose the specific edit to that workflow Skill.
- **Solidify:** if a single-point route succeeded and this kind of problem will
  recur, propose saving it as a new workflow Skill
  (`.claude/skills/workflow-<type>/SKILL.md`, from
  `references/workflow-template.md`). If it overlaps an existing workflow,
  propose extending that workflow instead of adding a near-duplicate.

Claude Code guards writes under `.claude/` with its own permission prompt;
that prompt is part of the approval. If the write is denied or cannot be
prompted (headless), show the full file content and stop.

Report the outcome with evidence: what was run, what was not, what is
unproven.

## Handoff boundaries

- **Superpowers** owns its workflows and invocation rules. Hand off; do not
  restate its procedures.
- **Tallyback** applies only when work is actually delegated. Do not create
  fake workers or delegation to fill a ledger. Start with `tallyback handshake`
  and follow Tallyback's own Skill or `tallyback --help`. Keep Task, Attempt,
  Claim, Evidence, Verdict, and Settlement distinct: a worker's "done" is a
  Claim, not proof.
- **Earn-it** reviews complexity and strength of evidence. Its verdict does not
  replace tests, independent acceptance, Git facts, or Tallyback settlement.
  An Earn-it `PASS` never settles a Tallyback task; settlement is a separate,
  explicit `tallyback settle` decision. `Leave it alone.` is a valid outcome.
- **Workflow Skills** (`workflow-*`) hold approved routes only, one kind of
  problem each: which capabilities, in which order, connected how. They hold
  no procedures and no project knowledge.
- **Knowledge Skills** hold verified reusable understanding of this repo or an
  integrated service (interfaces, invariants, contracts, pitfalls). Never put
  this knowledge in `AGENTS.md` or a workflow Skill, and never create one per
  task.

Evidence order: repository facts and executable checks > Git facts >
Tallyback records > agent claims and memory.
