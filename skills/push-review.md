---
type: skill
id: push-review
title: Push Review
description: "Reviews a git push: diffs, commit messages, agent context — produces a structured summary and impact assessment"
tags: [Production, Tested, Code, Review, Orchestration]
connections:
  - target: llm-service
    type: runs_on
metadata:
  complexity: high
  avg_tokens: 2500
  trigger: post-push
---

## Capability

Analyses the full context of a git push event: the diff, commit messages, any agent-generated context files (`.orchestrator-msg`), and the current state of open issues. Produces a structured summary that downstream steps use for drift detection, review, and briefing generation.

## Analysis Dimensions

### Commit Quality

- **Message clarity** — do commit messages explain *why*, not just *what*? Messages like "fix bug" or "update" are flagged.
- **Atomicity** — does each commit represent a single logical change? Commits that mix features, fixes, and refactoring are flagged for future discipline.
- **Scope creep** — are changes contained within the expected scope (as defined by open issues or the briefing)?

### Diff Analysis

- **Files changed** — categorize by type: source code, configuration, documentation, tests, generated files.
- **Change magnitude** — lines added, removed, and modified. Flag large diffs (500+ lines) that may need splitting.
- **Cross-cutting changes** — identify changes that touch shared interfaces, schemas, or contracts. These are candidates for drift detection.
- **New dependencies** — flag added imports, packages, or service connections that expand the project's dependency surface.

### Impact Assessment

- **Breaking changes** — modifications to public APIs, database schemas, shared types, or configuration formats that other repos or consumers depend on.
- **Risk indicators** — changes to authentication, authorisation, data handling, or deployment configuration.
- **Test coverage** — were tests added or modified alongside functional changes? Flag untested changes to critical paths.

## Output Format

Returns a structured push review with:

1. **Summary** — one-paragraph overview of what this push does and why
2. **Commit log** — each commit with a quality assessment (good / needs improvement / poor)
3. **Change categories** — files grouped by type with change statistics
4. **Impact matrix** — breaking changes, risk indicators, and cross-repo implications
5. **Open issue correlation** — which open issues this push addresses, partially addresses, or potentially conflicts with
6. **Recommendations** — actions for the orchestrator: issues to close, labels to update, drift checks to run

## Limitations

This skill analyses the push in isolation. It does not have access to the full repository history or runtime behavior. Cross-repo impact is inferred from file patterns and naming conventions, not verified against the actual sibling repos. The schema-drift-check skill handles that verification.
