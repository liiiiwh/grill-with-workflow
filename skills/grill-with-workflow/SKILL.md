---
name: grill-with-workflow
description: Govern Claude Code project work with grill, workflow, SubAgents, and TDD. Use for implementation work that needs persistent project knowledge, safe delegation, supervision, and acceptance.
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
1. Understand requirements.
2. Maintain project knowledge.
3. Human grill when needed.
4. Decide delegation.
5. Supervise agents.
6. Accept results.

## Knowledge Bootstrap

Check root:
- CONTEXT.md
- ARCHITECTURE.md
- LOGIC.md

Create missing documents when needed.

CONTEXT.md:
- domain terminology
- entity definitions
- naming rules

ARCHITECTURE.md:
- module boundaries
- ownership
- dependencies
- system structure

LOGIC.md:
- workflows
- states
- business rules
- edge cases

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
- each Worker returned a result
- selected `agentName`, route, Provider, and upstream model match the selector output when execution metadata is available
