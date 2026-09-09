---
name: grill-with-workflow
description: Coordinate project iterations with grill, documentation before planning, the tdd skill, SubAgent delegation, and mandatory cleanup. Use to initialize or take over projects and implement requirements with visible errors, reuse, backward compatibility, and no speculative fallbacks.
---

# Grill With Workflow

Main Agent owns project governance, decisions, shared knowledge, and acceptance.
SubAgents are temporary execution workers. Grill, the `tdd` skill, and useful
SubAgent delegation remain core capabilities; documentation and cleanup support them.

## Required Reading

- On every requirement, read root `CONTEXT.md`, `ARCHITECTURE.md`, and `LOGIC.md`.
  Create missing files using the process below; do not wait for a separate request.
- Before the quality scan, task planning, or coding, read
  [code-quality.md](references/code-quality.md). Every coding worker must read it too.
- Before delegating, read [subagents.md](references/subagents.md) and pass it to workers.
- Before behavior-changing implementation, the executor must locate and read the
  installed `tdd` skill and its applicable references. Install it alongside this skill.
  If inaccessible, report the missing dependency; do not silently code without it.

## 1. Understand and Grill

Read the project knowledge and inspect relevant code, configuration, and tests.
Resolve questions the repository can answer yourself. Walk unresolved design
branches and their dependencies: behavior, public interfaces, business rules,
module ownership, failure semantics, compatibility, and acceptance criteria.

For decisions needing the user, ask one focused question at a time, provide a
recommended answer and its tradeoff, and follow its consequences until shared
understanding is reached. Do not reopen settled decisions or invent consequential
requirements. Record decisions in the appropriate document as they crystallize.
Grill is substantive design clarification, not a generic requirements summary.

## 2. Update Knowledge Before Listing Tasks

Create missing documents from verified code, tests, configuration, and available
project explanations, using these references:
- [CONTEXT.md format](CONTEXT-FORMAT.md): domain terms, entities, naming rules.
- [ARCHITECTURE.md format](ARCHITECTURE-FORMAT.md): boundaries, ownership, dependencies.
- [LOGIC.md format](LOGIC-FORMAT.md): workflows, states, rules, errors, and edge cases.

Include useful code paths. Mark unknowns and inferences explicitly. For an empty
project, record known scope and unresolved choices; do not invent implementation
or historical decisions. Use [ADR guidance](ADR-FORMAT.md) only when warranted.

Search for existing capabilities and scan affected code plus direct callers,
dependencies, and tests under the quality reference. Check repository status and
the current diff to distinguish this iteration from pre-existing user work.
Use findings and grill decisions to update affected documents **before listing
implementation tasks, starting TDD, or delegating**. Separate verified current
behavior from agreed changes marked **planned, not yet implemented**.

Review accurate, unaffected documents without cosmetic edits. If scope or design
changes during execution, Main Agent updates the documents before replanning or
assigning dependent work. Workers report proposed updates to Main Agent.
Do not create unnecessary intermediate documents. For documentation-only requests,
finish with document/reference checks; do not manufacture implementation tasks.

## 3. Plan and Select Execution Routes

List behavior-oriented tasks with owners, file/module scope, dependencies, and
acceptance criteria. Include quality findings with locations, reasons, and checks.
Identify the existing implementation to reuse or extend for each capability.

Select a route for every implementation task:
- **Main Agent + tdd skill:** single tasks, sequential work, or overlapping core changes.
- **SubAgent + tdd skill:** use when the host supports delegation and multiple tasks
  have independent ownership, clear boundaries, and a benefit from parallel work.

Evaluate delegation every iteration; do not do everything alone when these conditions
hold. If delegation is unavailable, state the limitation and use Main Agent + `tdd`.
Do not simulate workers. Delegation changes the executor, never the TDD requirement.
Follow the delegation, isolation, and supervision rules in the required reference.

For N implementation tasks, append task N+1: **Cleanup, documentation synchronization,
and verification**. Name concrete cleanup targets; if none were found, still review
the integrated result. This task depends on all implementation work and integration;
it does not create an independent task merely to justify spawning another worker.

## 4. Implement Under the Quality Contract

Every executor loads and follows `tdd`; saying "use TDD" is insufficient. Do not
assume a parent's loaded skills transfer to workers. Provide accessible skill paths
or instructions and required references with each assignment.

Use incremental Red → Green → Refactor through public behavior. Confirm RED fails
for the missing behavior, not broken setup. Do not implement everything before tests.
For behavior-preserving refactors, establish or supplement coverage, then keep it green.
Documentation-only work needs no artificial code tests.

The detailed quality reference is mandatory; these rules always apply:
- **Errors reach users:** propagate every error to the top-level interaction boundary;
  provide a user-visible failure or recovery status. No swallowed errors, log-only
  failures, or fake success. Lower modules propagate structured errors, not UI effects.
- **Reuse before adding:** search existing implementations and callers first. Reuse or
  extend one owning module; never rewrite the same capability in another module.
- **Backward compatibility:** preserve supported caller contracts and observable behavior
  while extending existing capabilities; verify old and new callers. A deliberate
  breaking change requires an explicit user decision and a migration plan.
- **Keep modules decoupled:** use public interfaces and clear ownership, not circular
  dependencies or access to another module's internals. Avoid redundant wrappers,
  unnecessary deep loops/conditionals, and repeated work inside loops.
- **No speculative fallbacks:** require a concrete failure or explicit contract,
  a responsible layer, recovery semantics, and verification before adding recovery.
  Retries must be finite; do not multiply retries or defaults across layers.
- **Fix causes, not patch chains:** remove superseded workarounds. After two unsuccessful
  fixes of the same failure, stop patching and return to a minimal reproduction and
  a revised diagnosis. This does not count normal TDD RED/GREEN cycles as failed fixes.

Keep documentation current when decisions change. Workers perform lightweight grill
within their assignments and return cross-module or product decisions to Main Agent.

## 5. Integrate, Clean Up, and Accept

Main Agent supervises active workers and reviews their results before integration.
After all implementation is integrated and every worker has returned, execute N+1:
1. Rescan the integrated diff and affected paths. Resolve verified dead code, duplicated
   capability, redundant fallbacks, temporary artifacts, nesting, and coupling problems.
2. Verify error propagation and user-visible outcomes, reuse of the owning implementation,
   backward compatibility, and required recovery behavior under the quality reference.
3. Refactor with green tests, then rerun affected tests and applicable project lint,
   type, and build checks after final edits. Do not weaken tests to hide failures.
4. Reconcile all three documents with actual code. Remove superseded statements and
   stale planned status; keep genuine unresolved items explicit and links accurate.
5. Review repository status and report actual changes, cleanup, verification, and blockers.

Preserve necessary error handling and compatibility; do not delete code merely because
it is nested or defensive. Clean this iteration's additions and verified problems in
affected paths; record unrelated legacy debt without expanding into a repository rewrite.
Uncommitted or untracked user work is not garbage. Do not discard it during cleanup.

Accept only after agreed behavior, execution routes, TDD evidence, documentation, and
the final cleanup task are verified. Happy-path tests alone do not establish error
visibility or compatibility. Do not claim unrun checks passed or declare completion
while required in-scope cleanup or a known regression remains unresolved.
