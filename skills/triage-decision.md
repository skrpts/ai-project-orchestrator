---
type: skill
id: triage-decision
title: Triage Decision
description: "Converts review outcomes into GitHub Issues, watchpoints, or dismissals"
tags: [Production, Tested, Orchestration, Issue-Management]
connections:
  - target: llm-service
    type: runs_on
metadata:
  complexity: medium
  avg_tokens: 1500
  role: orchestrator
---

## Capability

Takes the final review output (Architect findings + CTO responses) and converts each accepted finding into a concrete action: a new GitHub Issue, a watchpoint for future monitoring, or a documented dismissal. This is where review findings become trackable work.

## Triage Categories

### GitHub Issue

Created when a finding requires code changes. The issue includes:

- **Title** — imperative mood, concise (e.g., "Fix unchecked null access in auth middleware")
- **Body** — finding details, code evidence, recommended action, and the review record reference
- **Labels** — appropriate combination of: `bug`, `enhancement`, `security`, `architecture`, `hub`, `app`, `internal`, `cross-repo`
- **Severity mapping** — Critical/High findings get an additional `priority:high` label

### Watchpoint

Created when a finding is valid but does not require immediate action. Watchpoints are:

- Logged in the orchestrator's tracking file
- Re-evaluated at the next push review for the affected repo
- Automatically escalated to an Issue if the same pattern appears in two consecutive pushes

### Dismissal

Created when a finding was rejected by the CTO. Dismissals are:

- Documented with the CTO's rationale
- Available for future reference if the same pattern is flagged again
- Not re-raised unless the Architect provides new evidence

## Cross-Repo Triage

When a finding affects multiple repos:

- Create the Issue in the shared issues tracker (not in individual repos)
- Apply all relevant repo labels (`hub`, `app`, etc.)
- Add the `cross-repo` label
- Include a "Propagation plan" section listing which repos need changes and in what order

## Output Format

Returns a triage manifest with:

1. **Issues to create** — full issue specifications ready for submission
2. **Watchpoints to add** — patterns to monitor in future reviews
3. **Dismissals to record** — rejected findings with rationale
4. **Issue updates** — existing issues to comment on, close, or re-label based on the push

## Limitations

Triage decisions are based on the review record. If the Architect missed a finding or the CTO misjudged severity, the triage will inherit that error. The orchestrator operator should spot-check triage output periodically.
