---
type: source
id: session-protocol
title: Session Protocol
description: "Agent session lifecycle: read briefing, work priority queue, commit, write .orchestrator-msg, push"
tags: [Production, Orchestration, Protocol]
connections:
  - target: push-review
    type: references
  - target: briefing-generation
    type: references
metadata:
  last_updated: "2026-04-01"
  applies_to: [agent-sessions]
---

## Purpose

Defines the standard lifecycle of an agent session in a multi-repo project. Every agent, regardless of which repo it operates on, follows this protocol.

## Session Lifecycle

### Phase 1: Orientation (~30 seconds)

The agent reads exactly two files:

1. **CLAUDE.md** — repo-level instructions (tech stack, conventions, testing requirements, communication protocol)
2. **BRIEFING.md** — session-specific work orders (priority queue, recent context, cross-repo facts)

The agent does **not** read architecture docs, decision logs, history files, or any other documentation unless the briefing explicitly directs it to a specific file for a specific task.

### Phase 2: Work the Priority Queue

The agent works items in the priority queue in order:

1. Read the item's context pointer (the specific file(s) referenced)
2. Understand the acceptance criteria
3. Implement the change
4. Run tests (as specified in CLAUDE.md's testing section)
5. Commit with an atomic, descriptive commit message

Between items, check for `[compact]` markers. If the next item is marked, clear context and re-read CLAUDE.md and BRIEFING.md before starting.

### Phase 3: Commit Discipline

Commits happen after each logical unit of work — not at the end of the session. Good commit granularity:

- One feature per commit
- One fix per commit
- One content batch per commit

Commit messages explain **why**, not just **what**:

- Bad: "Update auth.ts"
- Good: "Fix token refresh to handle expired refresh tokens (GH#234)"

### Phase 4: Write .orchestrator-msg

Before the session ends, the agent writes a summary to `.orchestrator-msg` in the repo root. This file is read by the post-push hook and delivered to the orchestrator.

The `.orchestrator-msg` format:

```
## [repo-name] Push Summary

### Changes
- [commit hash]: [one-line summary]
- [commit hash]: [one-line summary]

### Issues Addressed
- GH#NNN: [status — resolved / partially addressed / blocked]

### Cross-Repo Implications
- [Any changes that affect sibling repos — schema changes, API changes, etc.]

### Notes for Orchestrator
- [Anything the orchestrator needs to know — blockers found, questions, concerns]
```

### Phase 5: Push (Human-Initiated)

The agent does **not** push. It commits locally and tells the human operator to push. The push triggers the orchestrator's post-push hook, which:

1. Reads `.orchestrator-msg`
2. Logs the summary to `inbox.md` in the orchestrator repo
3. Includes the commit summary and diff statistics

## Session Rules

### Do Not

- Read files not referenced in the briefing or required by the current task
- Modify files in sibling repos (each repo has its own agent session)
- Push to the remote (the human pushes)
- Skip tests (even if the change "looks safe")
- Combine unrelated changes in a single commit
- Approve your own plans (the orchestrator approves plans)

### Do

- Commit early and often — atomic commits after each logical unit
- Follow the repo's code conventions (defined in CLAUDE.md)
- Challenge bad ideas, even if they come from the human operator
- Report blockers in `.orchestrator-msg` rather than silently working around them
- Use British English in all user-facing strings, comments, and documentation

## Error Handling

### Blocked Tasks

If a priority item is blocked (missing dependency, broken upstream, unclear requirements):

1. Add a comment to `.orchestrator-msg` explaining the blocker
2. Skip to the next priority item
3. Do not attempt to work around the blocker unless the briefing explicitly authorises it

### Test Failures

If tests fail after a change:

1. Fix the test or the code — do not skip the test
2. If the failure is in unrelated code, note it in `.orchestrator-msg` as a pre-existing issue
3. Do not commit code that fails tests

### Context Exhaustion

If the agent's context window is running low before completing all priority items:

1. Commit all completed work
2. Write `.orchestrator-msg` with progress summary
3. Note which priority items remain
4. The next session will pick up where this one left off (the briefing will be regenerated)
