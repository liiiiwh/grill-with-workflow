---
name: grill-with-workflow
description: Coordinate project iterations with grill-driven design clarification, documentation before planning, the tdd skill, SubAgent delegation, and mandatory cleanup. Use to initialize project knowledge, take over a project, or implement requirements while preventing speculative fallbacks and keeping project knowledge current.
---

# Grill With Workflow

## Core Principle

Main Agent is the project governance layer.
SubAgents are temporary execution workers.

Grill, execution through the `tdd` skill, and SubAgent orchestration are core
capabilities. Documentation and cleanup support this workflow; they do not
replace design clarification, test-first implementation, or useful delegation.

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
5. Invoke the `tdd` skill directly or delegate to workers that invoke it.
6. Integrate, clean up, synchronize documents, and accept results.

## Grill Requirements and Design

Before finalizing the documents and task plan, examine the requirement against
the code and project knowledge. Walk through unresolved design branches and
their dependencies: intended behavior, public interfaces, business constraints,
module ownership, failure semantics, and acceptance criteria.

Investigate questions that the code can answer yourself. For decisions that
need the user, ask one focused question at a time, include a recommended answer
and its tradeoff, and follow the consequences until you reach shared
understanding. Use established answers; do not repeatedly reopen settled
decisions or ask questions just to perform a ritual.

Record decisions in the appropriate project document as they crystallize.
Do not replace grill with a generic summary, silently invent consequential
requirements, or declare an unresolved design ready for implementation.
Workers perform a lightweight grill of their assigned task and return
cross-module or product decisions to Main Agent, which coordinates user input.

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

## Prevent Fallback Accumulation During Implementation

These rules apply whenever this skill is invoked, to Main Agent and every
worker, throughout implementation and repair. Do not defer them to cleanup.

- Start from the documented contract and one canonical implementation path.
  Fix invalid data or broken invariants at their source, or report an explicit
  error at the appropriate boundary. Do not silently invent successful results.
- Before adding a fallback, identify the observed failure or explicit contract
  requiring it, its owner, trigger, recovery behavior, and verification. If
  these cannot be established, investigate instead of adding defensive code.
  Record material recovery semantics in LOGIC.md before implementing them.
- Recovery belongs at the layer that can make the decision. Do not repeat
  validation, defaults, catches, or retries across layers for the same failure.
  Multiple recovery paths require distinct documented cases; never add another
  fallback merely because the previous workaround failed.
- Do not add speculative compatibility branches, catch-all exception handlers
  returning empty data, optional chaining to conceal required values, or chains
  of defaults without a defined meaning. For example, replacing a required
  configuration error with `config.value ?? cachedValue ?? ""` needs an explicit
  contract; making a test stop throwing is not sufficient justification.
- A retry must address a recoverable failure, have finite attempt/time bounds,
  and a defined exhaustion result. Check whether the operation can safely be
  repeated, including whether it has side effects. Account for retries in
  callers and dependencies so independent layers do not multiply attempts.
  Never respond to retry exhaustion by silently adding an outer retry loop.
- When a change fails, reproduce the failing behavior, inspect the evidence,
  and revise the root-cause hypothesis. Remove or revise the failed workaround
  instead of leaving it in place and layering another patch on top. After two
  consecutive unsuccessful fixes for the same failure, stop patching and
  re-diagnose from a minimal reproduction before attempting another change.
  If essential evidence is unavailable, report the blocker rather than invent
  more branches. This limit concerns patch accumulation, not normal TDD cycles.
- Use TDD to cover the intended behavior. For required recovery logic, verify
  its trigger, successful recovery, and exhaustion/error behavior as applicable.
  Keep errors observable; do not weaken tests or swallow failures to obtain green.

Pass these constraints explicitly with delegated tasks. Review each returned
diff for unjustified catches, defaults, retries, and compatibility paths before
integration. Reject accumulation of workarounds as unfinished work even when
happy-path tests pass. Explain retained non-obvious recovery behavior in the
final review, and remove superseded workarounds during the cleanup task.

## Execution Through the TDD Skill

Every behavior-changing implementation task must use the installed `tdd`
skill. Locate and read its SKILL.md and applicable references before coding;
do not treat the words "use TDD" in this document as a substitute for loading
and following that skill. The companion `tdd` skill is distributed in this
repository and should be installed alongside `grill-with-workflow`.

Choose and record the execution route for each task:
- **Main Agent + tdd skill:** Main Agent loads and follows `tdd` directly for
  a single task, sequential work, or work unsuitable for delegation.
- **SubAgent + tdd skill:** Main Agent delegates independent tasks under the
  policy below. Each worker loads and follows `tdd` in its own context; the
  parent's loaded skill must not be assumed to transfer automatically.

Delegation changes who implements the behavior; it does not waive TDD. Pass
the resolved skill location or an accessible copy of its instructions and
required references with each task, together with contracts and acceptance
criteria. If the skill cannot be accessed, report the missing dependency;
do not silently substitute implementation-first work or claim the skill ran.

Follow the skill's incremental Red → Green → Refactor workflow: one failing
behavior test, the minimum implementation to pass, then the next behavior.
Confirm that a RED failure is caused by the missing behavior, not a broken
test environment. Do not write all implementation first and add tests later.
Workers return relevant RED/GREEN evidence, final checks, remaining issues,
and proposed document updates for Main Agent's acceptance.

Documentation-only tasks do not require artificial code tests. For refactoring
that preserves behavior, establish or supplement behavior coverage before
changing the code, then keep it green under the `tdd` skill's refactoring rules.

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

Use SubAgents when the host supports them and all these conditions hold:
- multiple independent tasks exist.
- boundaries are clear.
- parallel execution improves efficiency.

Assess this route for every task plan. Do not default to doing all work alone
when independent tasks meet these conditions. If the host lacks delegation,
state that limitation and use Main Agent + `tdd`; do not simulate workers or
claim parallel execution occurred.

Avoid spawning for:
- small fixes
- sequential work
- shared core file changes

## Task Isolation

Before spawning:
- documents reflect verified current behavior and the agreed planned changes
- each task names its file/module scope, dependencies, and acceptance criteria
- each worker can load the `tdd` skill and its applicable references
- each worker receives the fallback-prevention rules and relevant contracts
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
- load and follow the `tdd` skill, including applicable references
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
- consequential design questions were resolved through grill and documented
- every implementation task followed its selected execution route and `tdd`
- delegated work includes behavior-test evidence and was supervised
- requirements
- architecture
- logic
- tests
- the final cleanup task completed after integration
- verified in-scope quality findings were resolved and behavior preserved
- new recovery paths have documented reasons, bounded retries where applicable,
  and verified failure behavior; superseded workarounds were removed
- all three project documents agree with the final implementation
- each Worker returned a result
- selected `agentName`, route, Provider, and upstream model match the selector output when execution metadata is available
