---
type: skill
id: issue-management
title: Issue Management
description: "Updates GitHub Issues: comments with commit hashes, closes single-repo issues, removes labels for cross-repo"
tags: [Production, Tested, Orchestration, Issue-Management]
connections:
  - target: llm-service
    type: runs_on
metadata:
  complexity: medium
  avg_tokens: 1200
  integrations: [github-issues]
---

## Capability

Executes the issue lifecycle actions determined by the triage step. Handles the nuanced rules for closing issues in a multi-repo project where a single issue may span multiple repos.

## Issue Actions

### Comment with Commit Hash

When a push addresses an open issue (fully or partially):

- Post a comment with the commit hash, a one-line summary of what was done, and which files were changed
- Format: `Fixed in [commit_hash]. [summary]. Files: [file_list]`
- If the fix spans multiple commits, list all relevant hashes

### Close Single-Repo Issue

When a push fully resolves an issue that belongs to a single repo:

- Post the commit hash comment (as above)
- Close the issue
- Only close if all acceptance criteria from the issue are met — partial fixes get a comment but stay open

### Cross-Repo Issue Handling

When a push addresses one repo's portion of a cross-repo issue:

- Post the commit hash comment
- Remove the current repo's label (e.g., remove `hub` label after Hub work is done)
- Do **not** close the issue — it remains open until all repo labels are removed
- Add a comment noting which repo's work is complete and which repos still have outstanding work

### Create New Issues

When the triage step specifies new issues to create:

- Create with the specified title, body, and labels
- Cross-reference the review record that generated the issue
- Assign to the appropriate repo label(s)

### Update Existing Issues

When the push reveals new information about an existing issue:

- Add a comment with the new context
- Update labels if the scope or classification has changed
- Do not close or reopen without explicit triage instruction

## Rules

1. Never close a cross-repo issue — only remove your repo's label
2. Never create duplicate issues — check for existing issues with similar titles before creating
3. Always include commit hashes — issues without traceability are useless
4. Comment before closing — the close action should never be the first interaction with an issue

## Output Format

Returns an action log with:

1. **Comments posted** — issue number, comment content, timestamp
2. **Issues closed** — issue number, closing commit hash
3. **Labels updated** — issue number, labels added/removed
4. **Issues created** — new issue number, title, labels
5. **Errors** — any actions that failed (e.g., permission denied, issue not found)

## Limitations

This skill generates the actions but relies on the executing environment having appropriate GitHub API access. If the agent lacks write access to the issues tracker, actions will be logged but not executed.
