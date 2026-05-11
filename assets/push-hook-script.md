---
type: asset
id: push-hook-script
title: Post-Push Hook Script
description: "Template for the post-push hook that reads .orchestrator-msg and logs to the orchestrator's inbox"
tags: [Production, Orchestration, Automation]
connections:
  - target: session-protocol
    type: references
  - target: push-review
    type: references
metadata:
  format: template
  file_type: shell-script
---

## Post-Push Hook

This script runs after `git push` completes. It reads the agent's `.orchestrator-msg` file, collects push metadata, and delivers everything to the orchestrator's inbox.

### Installation

Place this script at `.git/hooks/post-push` (or configure it via your Git hooks manager) in each repo managed by the orchestrator.

```bash
#!/usr/bin/env bash
set -euo pipefail

# --- Configuration ---
ORCHESTRATOR_REPO="{{orchestrator_repo_path}}"
INBOX_FILE="${ORCHESTRATOR_REPO}/inbox.md"
REPO_NAME="$(basename "$(git rev-parse --show-toplevel)")"
MSG_FILE=".orchestrator-msg"
TIMESTAMP="$(date -u '+%Y-%m-%d %H:%M:%S UTC')"

# --- Collect push metadata ---
BRANCH="$(git rev-parse --abbrev-ref HEAD)"
LATEST_HASH="$(git rev-parse --short HEAD)"
COMMIT_COUNT="$(git log --oneline @{push}..HEAD 2>/dev/null | wc -l | tr -d ' ')"
DIFF_STAT="$(git diff --stat @{push}..HEAD 2>/dev/null || echo 'Stats unavailable')"

# --- Read agent message ---
if [[ -f "${MSG_FILE}" ]]; then
    AGENT_MSG="$(cat "${MSG_FILE}")"
else
    AGENT_MSG="*No .orchestrator-msg found — agent did not write a push summary.*"
fi

# --- Write to orchestrator inbox ---
{
    echo ""
    echo "---"
    echo ""
    echo "## 📥 Push — ${REPO_NAME} (${TIMESTAMP})"
    echo ""
    echo "| Field | Value |"
    echo "|-------|-------|"
    echo "| **Repo** | ${REPO_NAME} |"
    echo "| **Branch** | ${BRANCH} |"
    echo "| **Latest commit** | \`${LATEST_HASH}\` |"
    echo "| **Commits pushed** | ${COMMIT_COUNT} |"
    echo "| **Timestamp** | ${TIMESTAMP} |"
    echo ""
    echo "### Diff Statistics"
    echo ""
    echo '```'
    echo "${DIFF_STAT}"
    echo '```'
    echo ""
    echo "### Agent Summary"
    echo ""
    echo "${AGENT_MSG}"
    echo ""
} >> "${INBOX_FILE}"

# --- Clean up ---
if [[ -f "${MSG_FILE}" ]]; then
    rm "${MSG_FILE}"
    echo "[hook] Delivered .orchestrator-msg to orchestrator inbox"
else
    echo "[hook] Warning: no .orchestrator-msg found"
fi

echo "[hook] Push from ${REPO_NAME} logged to orchestrator inbox"
```

### How It Works

1. **Collect metadata** — branch name, latest commit hash, number of commits pushed, and diff statistics
2. **Read agent message** — the `.orchestrator-msg` file written by the agent during its session
3. **Append to inbox** — writes a structured entry to the orchestrator repo's `inbox.md`
4. **Clean up** — removes `.orchestrator-msg` after delivery to prevent stale messages in the next push

### Configuration Notes

- **`ORCHESTRATOR_REPO`** — must be set to the absolute path of the orchestrator repo on this machine. Use `{{orchestrator_repo_path}}` as a placeholder during setup.
- **Multiple repos** — install this hook in every repo the orchestrator manages. Each repo's pushes will appear as separate entries in the shared inbox.
- **Branch filtering** — this hook runs on all branch pushes. To limit to specific branches, add a branch check after the configuration section.
- **Error resilience** — if the orchestrator repo is not accessible or inbox.md cannot be written, the push still succeeds. The hook failure is logged to stderr but does not block the push.

### Permissions

The hook needs:
- Read access to `.orchestrator-msg` in the current repo
- Write access to `inbox.md` in the orchestrator repo
- Standard git commands available in PATH
