# SubAgent Orchestration

Main Agent reads this before delegating; each worker receives and reads it before work.
Use delegation for independent tasks with clear ownership and a parallelism benefit.
Single tasks, sequential work, and overlapping core changes stay with Main Agent + `tdd`.
The final cleanup task is dependent work, not a reason to invent parallelism.

## Hard Limit: 5 SubAgent Launches

For one user request, never launch more than **5 SubAgents in total**, including
replacement workers and restarts. This is a cumulative launch limit, not just a
concurrency limit. Finishing or stopping a worker does not refund a launch.
Track the count before every launch, including launches through different tools;
reserve capacity before a batch so concurrent calls cannot exceed the limit.
Carry the count through phases, replanning, continuations, context compaction,
and repeated skill invocations for the same request. Do not split a request to reset it.

Only an explicit user statement permitting **unlimited SubAgent use** removes
this cap. General approval for parallel work, urgency, or "use more agents" does
not. Host limits and other applicable restrictions still apply. At the cap,
reuse available workers without another launch or finish through Main Agent + `tdd`;
do not stop useful work merely to request a higher limit. Workers still cannot
spawn further workers, even when the user has lifted the numerical cap.

## Assignment and Isolation

Main Agent updates project knowledge and resolves shared design decisions before
assigning implementation. Every assignment includes:
- Required behavior, acceptance criteria, file/module ownership, and dependencies.
- The three project documents, current-versus-planned changes, and relevant contracts.
- The canonical implementation to reuse/extend and supported caller compatibility.
- Accessible `tdd` SKILL.md plus required references; the worker must actually load them.
- [Code quality contract](code-quality.md), including user-visible errors and no patch chains.
- Required test evidence, integration expectations, and proposed document changes to report.

Do not overlap file or module ownership or dispatch conflicting architecture/business
logic changes. Assign shared implementation to one owner and sequence dependents.
Other workers consume its public interface; they must not duplicate it locally.
The parent having loaded a skill or reference does not prove the worker has access.

## Worker Responsibilities

Read assigned project knowledge and references, perform lightweight grill, and use
`tdd` to implement the assigned behavior. Ask Main Agent about product or cross-module
decisions; Main Agent coordinates human input and updates shared documents.

Workers may use grill and `tdd`. They may not run project-level workflow/supervision
loops, spawn further workers, create project knowledge documents, or modify
CONTEXT.md, ARCHITECTURE.md, or LOGIC.md. Keep work within assigned ownership and
report conflicts before making dependent changes.

Return changes, relevant RED/GREEN evidence and final checks, quality/recovery findings,
error propagation and compatibility results, blockers, and proposed document updates.
Do not claim a skill was used or a test passed without actual execution.

## Main Agent Supervision and Integration

While workers are active, use host status/wait facilities to monitor progress,
results, blockers, scope deviation, and architecture or business-rule conflicts.
Default review cadence: about three minutes, or sooner when a worker reports a
blocker or finishes; avoid busy polling and unsupported background-loop claims.
Do not interrupt healthy workers. Correct deviations; stop or restart a worker
when needed within the launch limit, without discarding unrelated work. Ask users
only for real decisions.

Review each returned diff and evidence before integration. Reject unjustified recovery,
duplicate capability, lost errors, or incompatible contracts even if happy-path tests
pass. If execution metadata is available, verify selected agentName, route, Provider,
and upstream model agree with selector output. Every assigned worker must return a
result or have its interrupted work accounted for and completed through reassignment.

Main Agent owns shared-document updates and final acceptance. After all work is
integrated and no worker is still editing affected code, perform the mandatory
cleanup task, verify the combined behavior, and reconcile all project documents.
