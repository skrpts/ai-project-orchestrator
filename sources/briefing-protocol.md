---
type: source
id: briefing-protocol
title: Briefing Protocol
description: "Rules for context budget, priority ordering, cross-repo fact propagation, and the 140-line insight"
tags: [Production, Orchestration, Protocol]
connections:
  - target: briefing-generation
    type: references
  - target: briefing-md-template
    type: references
metadata:
  last_updated: "2026-04-01"
  applies_to: [orchestrator, agent-sessions]
---

## Purpose

Defines how briefings are constructed, prioritised, and propagated across repos. This protocol ensures that agents receive exactly the context they need — no more, no less.

## The 140-Line Insight

An agent session begins by reading two files:

1. **CLAUDE.md** (~80 lines) — repo-level instructions that rarely change
2. **BRIEFING.md** (~60 lines) — session-specific work orders that change every cycle

Together, these ~140 lines are the agent's entire orientation. Everything the agent needs to begin productive work must fit in this budget. Compare this to the alternative: reading 1,500+ lines of architecture docs, decision logs, and history files. The briefing protocol exists because 140 focused lines outperform 1,500 scattered ones.

## Priority Ordering Rules

### Severity-First Ordering

1. **Critical bugs** — anything causing data loss, security exposure, or service outage
2. **Blocking issues** — work that other repos or agents are waiting on
3. **High-priority fixes** — bugs affecting user experience
4. **Planned features** — items from the current sprint or milestone
5. **Enhancements** — improvements to existing functionality
6. **Documentation** — docs, comments, and cleanup tasks

### Tie-Breaking

When two items have the same severity:

1. Items blocking other repos come first (unblock the team)
2. Items with smaller scope come first (quick wins build momentum)
3. Items with clearer acceptance criteria come first (less ambiguity = faster completion)

### Dynamic Reprioritisation

After each push review, the priority queue may change:

- New issues from triage are inserted at the appropriate severity level
- Completed items are moved to "Recently Completed"
- Blocked items are annotated with their blocker
- Items affected by schema drift are promoted (they may need immediate attention)

## Context Propagation Rules

### What Propagates

Facts from one repo's session that affect another repo's work:

- Schema or type changes that consumers need to adopt
- API contract changes (new endpoints, changed parameters, deprecated routes)
- Deployment state changes (a service is down, a migration is pending)
- Completed work that unblocks a sibling repo's task

### What Does Not Propagate

- Internal implementation details (how a function works, which pattern was chosen)
- Session metadata (how long the session took, how many commits were made)
- Speculative plans (what the agent intends to do next, unless it affects siblings)
- Tool or framework choices that do not affect the interface

### Propagation Format

Each cross-repo fact is a single line in the briefing:

```
- [Repo]: [Fact] — [Why it matters to this repo]
```

Example:
```
- App: Manifest upload now sends `version: 2` header — Hub must accept both v1 and v2 until app rollout completes
```

### Staleness Rules

- Facts older than 3 push cycles are removed unless still relevant
- Facts about completed work are removed once the dependent repo has actioned them
- Deployment state facts are removed once the state resolves

## Compact Markers

The `[compact]` marker on a priority item tells the agent to run `/compact` (clear context) before starting that item. This is used when:

- The next item is unrelated to the previous one (different area of the codebase)
- The previous item loaded a large amount of context that would pollute the next task
- The agent has been working for several items and context is likely fragmented

Do not mark every item as `[compact]` — context from the previous task is often useful for the next one when they are related.

## Anti-Patterns

1. **Kitchen sink briefings** — cramming every open issue into the priority queue. Limit to 5-7 items maximum.
2. **Vague context pointers** — "see the auth module" instead of `src/lib/auth.ts` lines 42-68
3. **Stale cross-repo facts** — propagating facts from 5 cycles ago that are no longer relevant
4. **Missing acceptance criteria** — items without a "done when" condition lead to ambiguous scope
5. **Motivational padding** — "Great work on the last session!" wastes lines. Agents do not need encouragement.
