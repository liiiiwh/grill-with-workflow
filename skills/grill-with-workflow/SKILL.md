---
name: grill-with-workflow
description: Coordinate project iterations with grill, documentation before planning, TDD, optional SubAgents, and mandatory cleanup. Use to initialize project knowledge, take over a project, or implement requirements while keeping terminology, architecture, and business logic current.
---

# Grill With Workflow

## Core Principle

Main Agent is the project governance layer.
SubAgents are temporary execution workers.

Persistent project knowledge:
- CONTEXT.md
- ARCHITECTURE.md
- LOGIC.md

Do not create unnecessary intermediate documents.

## Main Agent

Available:
- grill
- workflow
- subagent
- tdd
- loop

Responsibilities:
1. Read project knowledge and inspect the current code.
2. Clarify requirements and scan relevant code quality problems.
3. Update project knowledge before listing implementation tasks.
4. Plan work, including a final cleanup task, and decide delegation.
5. Implement with TDD or supervise workers using TDD.
6. Integrate, clean up, synchronize documents, and accept results.

## Documentation Before Every Iteration

For every new requirement or iteration, Main Agent must read the three
documents at the project root before planning or implementation. Do not wait
for the user to name them or request initialization.

- [CONTEXT.md format](CONTEXT-FORMAT.md): domain terminology, entity definitions,
  and naming rules.
- [ARCHITECTURE.md format](ARCHITECTURE-FORMAT.md): module boundaries, ownership,
  dependencies, and system structure.
- [LOGIC.md format](LOGIC-FORMAT.md): workflows, states, business rules, and edge cases.

Create any missing document from the existing code, configuration, tests, and
available project explanations. Include useful code paths. Mark unknowns and
inferences explicitly; do not invent business rules or historical decisions.
For an empty project, record the known scope and unresolved choices instead
of inventing an implementation.

Before listing tasks, verify relevant documented facts against the code,
perform the quality scan below, clarify material design questions, and update
the affected documents. Keep current behavior separate from agreed changes
that are **planned, not yet implemented**. If a document is already accurate
and unaffected, review it without making cosmetic edits.

Only after this documentation pass may Main Agent list implementation tasks,
start TDD, or delegate work. If scope or design changes during execution,
update the affected documents before replanning or assigning dependent work.
Use [ADR guidance](ADR-FORMAT.md) only for decisions that warrant an ADR.

Keep documents current during the work, not just at handoff. Main Agent owns
all three documents; workers report findings and proposed changes to it.
At completion, reconcile planned changes with the actual implementation,
remove superseded statements, and retain explicit unresolved items.

For a documentation-only initialization request, perform this process and
review the resulting documents; do not manufacture implementation tasks.

## Code Quality Scan

On each iteration, inspect the code involved in the requirement and its
direct callers, dependencies, and tests. Use the current diff and repository
status to distinguish pre-existing work from this iteration's changes.

Look for concrete cleanup candidates:
- Dead or unreachable code, unused imports, duplicate implementations,
  abandoned branches, and temporary debugging or generated artifacts.
- Redundant fallback chains, defaults that hide invalid states, swallowed
  errors, and compatibility paths whose consumers no longer exist.
- Deeply nested loops or conditionals, repeated scans or I/O inside loops,
  and tangled control flow that obscures business rules.
- Circular dependencies, module boundary violations, duplicated business
  rules, and wrappers or abstractions that add no useful responsibility.

Record actionable findings in the task plan with locations, reasons, and
verification needs. Scan first, but wait until the documentation pass is
complete before publishing the task list. Avoid a separate audit document.
Scan again after integration, since combined changes may introduce new issues.

Assess behavior and callers before removing code. A fallback or nested loop
is not automatically redundant: preserve required error handling,
compatibility, ordering, and business semantics. Prefer simpler control flow,
appropriate data structures, and clear module boundaries; do not merely move
complexity into new helpers or introduce speculative abstraction layers.

Clean problems introduced by this iteration and verified issues within its
affected paths. Record unrelated legacy problems for follow-up rather than
turning every request into a repository-wide rewrite. Uncommitted or untracked
user work is not garbage; never discard it as part of cleanup.

## Task Planning and Final Cleanup

After updating project knowledge, list behavior-oriented implementation tasks
with ownership and acceptance criteria. For N implementation tasks, always
append task N+1: **Cleanup, documentation synchronization, and verification**.
Name the concrete cleanup targets found in the scan; if none were found, the
final task still checks the integrated result and documents that outcome.

This final task depends on all implementation tasks and their integration.
Do not run it concurrently with workers still changing the same code. Its
presence does not make an otherwise sequential request eligible for delegation.

Main Agent performs the final task once implementation is integrated and all
workers, if any, have returned:
1. Review the integrated diff and affected paths against the initial findings.
2. Remove verified dead code, duplication, redundant fallbacks, and temporary
   artifacts; simplify unnecessary nesting and resolve affected coupling issues.
3. Refactor while relevant tests are green. Add behavior coverage first where
   needed to protect a cleanup, then rerun affected tests and applicable project
   lint, type, and build checks after the final edits. Documentation-only work
   needs document and reference checks, not invented code tests.
4. Synchronize terminology, architecture, and business logic with the final
   code. Ensure file paths, examples, and planned-versus-implemented status agree.
5. Review repository status, account for changed files, and report cleanup
   results, actual verification, and any unresolved issues with reasons.

Useful local cleanup may happen during each implementation task; the final
task is still required to catch duplication and coupling across their results.
Do not declare completion while required in-scope cleanup remains unresolved,
or claim a check passed when it was not run.

## Delegation Policy

Do not spawn SubAgents by default.

If only one meaningful task exists:
- Main Agent handles it directly.
- Do not create a SubAgent.

Spawn SubAgents only when:
- multiple independent tasks exist.
- boundaries are clear.
- parallel execution improves efficiency.

Avoid spawning for:
- small fixes
- sequential work
- shared core file changes

## Task Isolation

Before spawning:
- documents reflect verified current behavior and the agreed planned changes
- each task names its file/module scope, dependencies, and acceptance criteria
- no file ownership overlap
- no module ownership overlap
- no architecture conflict
- no business logic conflict

## SubAgent

Allowed:
- grill
- tdd

Forbidden:
- workflow
- subagent
- loop

SubAgents:
- read project knowledge
- perform lightweight grill
- use TDD
- implement task
- keep assigned code clean and report quality findings and document changes

Do not:
- create project documents
- modify CONTEXT.md
- modify ARCHITECTURE.md
- modify LOGIC.md

Conflicts are returned to Main Agent.

## Supervision Loop

When SubAgents are active, Main Agent internally monitors them.

Default interval:
3 minutes.

Check:
- progress
- context/results
- blockers
- scope deviation
- architecture conflicts

Healthy agents are not interrupted.

Problems:
- send correction
- terminate and restart if necessary

Ask human only for real design decisions.

## Acceptance

Main Agent validates:
- requirements
- architecture
- logic
- tests
- the final cleanup task completed after integration
- verified in-scope quality findings were resolved and behavior preserved
- all three project documents agree with the final implementation
- each Worker returned a result
- selected `agentName`, route, Provider, and upstream model match the selector output when execution metadata is available
