# Agent Development Instructions

Development in this repository is workflow-first, not change-first.
Do not start by editing code. Load `development-workflow` for every
development task and follow it:

1. Understand the requirement first. Look facts up yourself; ask the user
   only for decisions, each with a recommended answer.
2. Workflow first, single-point second: run the matching `workflow-*`
   Skill for this kind of problem. Only if none fits, compose a single-point
   route for this task from the available Skills and native capabilities.
3. A new or changed route is approved by the user as part of the plan.
   Approved workflows are reused without re-approval.
4. Run the route. Propose updating a workflow when the run shows it should
   change, or solidifying a single-point route that will recur as a new
   workflow.

Follow each selected Skill's own instructions. Treat repository facts and
executable checks as stronger evidence than agent claims.
