---
type: skill
id: briefing-generation
title: Briefing Generation
description: "Reads git state, open issues, and cross-repo context to produce a slim BRIEFING.md per repo"
tags: [Production, Tested, Orchestration, Documentation]
connections:
  - target: llm-service
    type: runs_on
  - target: briefing-protocol
    type: references
  - target: briefing-md-template
    type: references
metadata:
  complexity: medium
  avg_tokens: 2000
  output_format: markdown
---

## Capability

Generates a concise, actionable briefing document for an agent session. The briefing is the primary input an agent reads at session start — it must contain everything the agent needs to begin work, and nothing it does not.

## Design Principles

### Context Budget Awareness

Agents read approximately 140 lines at session start: the repo's `CLAUDE.md` (~80 lines) plus `BRIEFING.md` (~60 lines). Every line in the briefing competes for attention. The briefing must be ruthlessly prioritized.

### Priority Queue Structure

The briefing's core is a numbered priority queue. Each item includes:

- **Priority number** — determines work order
- **Issue reference** — GitHub Issue number or internal tracking ID
- **One-line summary** — what needs doing, in imperative mood
- **Context pointer** — specific files or docs the agent should read (not "see docs", but "read `src/lib/auth.ts` lines 42-68")
- **Acceptance criteria** — how the agent knows the task is done
- **Compact marker** — `[compact]` if the agent should run `/compact` before starting this item

### Cross-Repo Context

When work in one repo depends on changes in another, the briefing includes a "Cross-Repo Context" section with:

- What changed in the sibling repo and why
- Which files or interfaces were affected
- What the current repo needs to do in response
- Whether the sibling change is merged or still in flight

### Constraints Section

Time-sensitive information that affects how the agent should work:

- Deployment freezes or release windows
- Blocked dependencies (waiting on another repo's push)
- Known broken states that the agent should avoid touching

## Output Format

Produces a markdown file following the briefing template structure:

1. **Header** — repo name, date, orchestrator session reference
2. **Priority queue** — numbered list of work items
3. **Recently completed** — last 3-5 completed items (so the agent knows what just happened)
4. **Cross-repo context** — facts propagated from sibling repo sessions
5. **Constraints** — time-sensitive blockers or conditions

## Limitations

The briefing reflects the orchestrator's understanding at generation time. If an agent completes work that changes priorities, the briefing becomes stale. Briefings should be regenerated before each agent session, not cached.
