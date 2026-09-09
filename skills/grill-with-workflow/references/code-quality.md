# Code Quality Contract

Required before scanning, planning, or coding, and again during final review.
Applies to Main Agent and every worker. Use the existing task plan to track findings;
do not create a separate audit document.

## Scan and Reuse Before Implementation

Search the repository for matching behavior, terminology, public APIs, services,
utilities, and their callers before adding code. Record the owning implementation
and whether it will be reused or extended. An unfamiliar name is not evidence that
capability is absent; inspect behavior before deciding a new implementation is needed.

Do not implement the same capability or business rule in different modules. Reuse
through a public interface or evolve the existing owner. If existing duplicates
are in scope, consolidate their behavior behind one owner and migrate callers while
preserving supported contracts. A compatibility adapter delegates to that owner;
it must not become a second implementation. Workers report shared changes to Main
Agent instead of creating private copies to avoid coordinating.

Inspect affected paths and direct dependencies for:
- Dead/unreachable code, unused imports, duplicate logic, abandoned branches,
  temporary debugging, and obsolete generated artifacts.
- Default chains, swallowed errors, recovery that hides broken invariants, and
  compatibility branches with no supported consumers.
- Deep nested loops/conditionals, repeated scans or I/O inside loops, tangled state
  transitions, and unbounded retries or recursive work.
- Circular dependencies, internals accessed across module boundaries, duplicated
  validation/business rules, and abstractions without a useful responsibility.

Use early returns, suitable data structures, cohesive operations, and explicit
interfaces to simplify necessary work. Do not merely hide nesting in helpers or
create generic wrapper layers. Judge necessary loops by semantics and cost, not
an arbitrary nesting count; preserve ordering and business behavior.

## Propagate Every Error to the User Boundary

Identify the top-level owner of each operation and document its error route in
LOGIC.md before implementation: origin → intermediate modules → interaction boundary.
An interaction boundary is the UI, CLI, API response, or user-accessible job/task status.
For a library, return a documented error to its caller; verify the actual presentation
at the consuming application's boundary when that application is in scope.

Lower layers must propagate errors with meaningful type/code and cause, using the
project's exception or result convention. Catch only to recover under a documented
contract, add context and rethrow/return an error, or translate at an explicit boundary.
Do not catch only to log, discard the cause, return empty data, or pretend success.
Propagate failures from async tasks, callbacks, workers, and parallel branches too;
no detached promise, lost callback error, or permanently running failed job.

The boundary must visibly distinguish failure, partial completion, and recovered
success. Explain what failed and the meaningful next action when one exists.
Use safe user-facing details; raw secrets or internal stack traces need not be displayed.
Logging alone does not satisfy user visibility. A caught-and-recovered error still
reaches the boundary as recovery/warning status; do not silently erase it.
Related errors may be aggregated into one visible outcome while preserving their
causes; avoid duplicate notifications from every layer or retry attempt.

Expected domain outcomes such as "no search matches" are not errors when defined by
the contract. Do not convert genuine failures into those normal outcomes to hide them.
For CLI failures, use an actionable message and unsuccessful exit status; for APIs,
use the existing error schema and appropriate failure status. For background jobs,
record and expose failure/recovery in user-accessible status rather than logs alone.

Verify a lower-layer failure through the public entry point: it reaches the caller
and visible status, never false success. Cover async propagation, partial results,
and recovery where used. If a UI consumer is outside scope, document the observable
error contract and the unverified consumer boundary; do not claim end-to-end visibility.

## Bounded, Justified Recovery

Before adding recovery, identify its observed failure or explicit contract, owner,
trigger, result, and verification. Fix broken invariants or invalid inputs at their
source, or report an error. Defaults are allowed only when they have defined semantics.

Do not introduce speculative compatibility paths, catch-all success responses, optional
chaining to conceal required values, or default chains such as
`config.value ?? cachedValue ?? ""` merely to stop a failure. Distinct recovery paths
need distinct supported cases; each failure has one recovery decision owner.

Retries require a recoverable failure, finite attempt/time bounds, an exhaustion
result, and safe repeat semantics including side effects. Count retries in dependencies
and callers so nested policies do not multiply attempts. Never add an outer retry
because an inner policy exhausted. Keep intermediate errors available to the boundary.

When a fix fails, reproduce the behavior and examine evidence. Remove or revise the
failed workaround rather than stacking another branch. After two unsuccessful fixes
for the same failure, re-diagnose from a minimal reproduction before the next change.
If required evidence is unavailable, report the blocker. Normal TDD cycles are excluded.

Use `tdd` to verify the trigger, recovery, exhaustion, and visible error/recovery result
as applicable. Do not weaken assertions, swallow failures, or accept passing happy-path
tests as proof of recovery correctness.

## Backward-Compatible Evolution

Inspect existing consumers and preserve their supported signatures, return shapes,
error contracts, defaults, side effects, ordering, and persisted/configuration formats
where relevant. Prefer additive extensions with existing behavior when new options
are absent. Do not silently change old callers to make a new implementation easier.

Add behavior coverage for existing contracts before changing them, alongside tests for
new behavior. Keep required compatibility in a narrow adapter at the owning boundary,
sharing the canonical implementation. Do not preserve every historical workaround
speculatively; identify actual supported consumers and contract obligations.

If the requested visible-error behavior conflicts with an existing public contract,
surface that conflict during grill, record the decision before planning, and use an
agreed compatible extension or explicitly authorized migration. Do not silently choose
either swallowing errors or breaking callers. Breaking changes need an explicit user
decision, affected-consumer inventory, and migration plan.

## Final Quality Review

Rescan after integration for cross-task duplication, conflicting shared changes, lost
errors, deep/redundant control flow, and multiplied retries. Review actual callers before
removing recovery or compatibility. Verify behavior after cleanup and reconcile project
documents. Explain retained non-obvious recovery and report real unresolved issues;
never relabel required cleanup as future work merely to declare the task complete.
