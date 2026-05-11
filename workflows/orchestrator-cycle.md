---
type: workflow
id: orchestrator-cycle
title: Orchestrator Cycle
description: "Main orchestration workflow: push review → drift check → 3-way review loop → triage → issue management → briefing generation"
tags: [Production, Tested, Orchestration, Multi-Agent]
connections:
  - target: push-review
    type: uses
  - target: schema-drift-check
    type: uses
  - target: architect-review
    type: uses
  - target: cto-response
    type: uses
  - target: triage-decision
    type: uses
  - target: issue-management
    type: uses
  - target: briefing-generation
    type: uses
  - target: llm-service
    type: runs_on
metadata:
  estimated_duration: "60-180 seconds"
  avg_tokens: 15000
  trigger: post-push
output_step: "briefing-generation"
composite_steps:
  - "push-review"
  - "schema-drift-check"
  - "architect-review"
  - "cto-response"
  - "triage-decision"
  - "issue-management"
  - "briefing-generation"
loops:
  - id: "review-loop"
    mode: "until_pass"
    steps: ["architect-review", "cto-response"]
    verifier: "cto-response"
    maxIterations: 2
    freshContextPerIteration: true
execution:
  - skill: "push-review"
    prompt: "review-push"
    step_type: "synthesis"
  - skill: "schema-drift-check"
    prompt: "check-schema-drift"
    step_type: "review"
    context:
      push_review_output: "{{steps.push-review.output}}"
  - skill: "architect-review"
    prompt: "architect-findings"
    step_type: "review"
    context:
      push_review_output: "{{steps.push-review.output}}"
      drift_check_output: "{{steps.schema-drift-check.output}}"
      previous_cto_response: "{{steps.cto-response.output}}"
  - skill: "cto-response"
    prompt: "cto-pressure-test"
    step_type: "validation"
    context:
      architect_findings_output: "{{steps.architect-review.output}}"
  - skill: "triage-decision"
    prompt: "triage-outcomes"
    step_type: "synthesis"
    context:
      architect_findings_output: "{{steps.architect-review.output}}"
      cto_response_output: "{{steps.cto-response.output}}"
      push_review_output: "{{steps.push-review.output}}"
  - skill: "issue-management"
    prompt: "manage-issues"
    step_type: "generation"
    context:
      triage_output: "{{steps.triage-decision.output}}"
      push_review_output: "{{steps.push-review.output}}"
  - skill: "briefing-generation"
    prompt: "generate-briefing"
    step_type: "generation"
    context:
      push_review_output: "{{steps.push-review.output}}"
      drift_check_output: "{{steps.schema-drift-check.output}}"
      triage_output: "{{steps.triage-decision.output}}"
      issue_management_output: "{{steps.issue-management.output}}"
---

## Overview

The Orchestrator Cycle is the core workflow for multi-agent project coordination. It runs after every push to a managed repo and produces three outputs: a review record, updated GitHub Issues, and a fresh briefing for the next agent session.

## Pipeline

### Step 1: Push Review (synthesis)

**Skill:** push-review | **Prompt:** review-push

Analyses the incoming push: diffs, commit messages, agent context from `.orchestrator-msg`, and correlation with open issues. Produces a structured summary that all downstream steps consume.

**Input:** The user provides push details, repo name, and optionally the current open issues.

**Output:** Structured push review with summary, commit quality assessment, change categories, impact matrix, issue correlation, and recommendations.

### Step 2: Schema Drift Check (review)

**Skill:** schema-drift-check | **Prompt:** check-schema-drift

Takes the push review output and checks whether any shared contracts (schemas, types, API specs) have diverged between repos. This catches the most common source of silent integration failures in multi-repo projects.

**Input:** Push review output from Step 1.

**Output:** Contract inventory, drift findings with severity, propagation plan, or a clean confirmation.

### Step 3–4: Review Loop (until_pass, max 2 iterations)

The Architect and CTO engage in a structured review loop. The loop runs until the CTO passes the review or the maximum of 2 iterations is reached.

#### Step 3: Architect Review (review)

**Skill:** architect-review | **Prompt:** architect-findings

Examines the push review and drift check outputs. Produces structured findings, each backed by code evidence with severity, classification, and confidence ratings. On iteration 2, the Architect addresses the CTO's feedback from the previous round.

**Input:** Push review output, drift check output, and (on iteration 2) the CTO's previous response.

**Output:** Structured findings with review header, prioritised finding list, summary assessment, and cross-repo implications.

#### Step 4: CTO Response (validation, gate)

**Skill:** cto-response | **Prompt:** cto-pressure-test

Pressure-tests every Architect finding. For each finding, the CTO issues a verdict: Accept, Reject, Refine, Downgrade, or Escalate. Then issues an overall Pass or Fail.

- **Pass:** The loop ends and the workflow proceeds to triage.
- **Fail:** The loop returns to the Architect for a second pass with fresh context.

**Input:** Architect findings from Step 3.

**Output:** Per-finding responses with verdicts, gate decision (Pass/Fail), summary, and escalations.

### Step 5: Triage Decision (synthesis)

**Skill:** triage-decision | **Prompt:** triage-outcomes

Converts the final review record into concrete actions: GitHub Issues for accepted findings, watchpoints for things to monitor, and documented dismissals for rejected findings.

**Input:** Architect findings, CTO response, and push review context.

**Output:** Triage manifest: issues to create, watchpoints to add, dismissals to record, existing issues to update.

### Step 6: Issue Management (generation)

**Skill:** issue-management | **Prompt:** manage-issues

Executes the triage decisions: posts comments with commit hashes, closes resolved single-repo issues, removes labels for cross-repo issues, and creates new issues.

**Input:** Triage manifest and push review context.

**Output:** Action log of all issue operations performed.

### Step 7: Briefing Generation (generation)

**Skill:** briefing-generation | **Prompt:** generate-briefing

Produces a fresh BRIEFING.md for the affected repo, incorporating new priorities from the triage, completed work from the push, and cross-repo context from the drift check.

**Input:** Push review, drift check, triage outcomes, and issue management actions.

**Output:** A slim (~60 line) BRIEFING.md ready for the next agent session.

## The Review Loop

The review loop (Steps 3–4) is the quality gate of the workflow. Key properties:

- **`until_pass` mode** — the loop repeats until the CTO issues a Pass verdict
- **Maximum 2 iterations** — prevents infinite loops. If the CTO still fails after iteration 2, the findings are escalated to the human operator
- **Fresh context per iteration** — on iteration 2, the Architect starts with clean context plus the CTO's feedback. This prevents context pollution from the first pass
- **The CTO is the verifier** — their Pass/Fail decision controls the loop

## Error Handling

- If the push review fails (malformed input), the workflow aborts with a clear error
- If the drift check fails, the workflow continues but flags "drift check unavailable" in the Architect review
- If the review loop hits max iterations without a Pass, all outstanding findings are escalated to the human operator via the triage step
- If issue management fails (API errors), actions are logged but not retried — the next cycle will pick up any missed actions

## Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `maxIterations` | 2 | Maximum review loop iterations before escalation |
| `freshContextPerIteration` | true | Clear context between loop iterations |
| `trigger` | post-push | When the workflow runs |

## Example Run

A typical cycle after a Hub push:

1. **Push review** analyses 3 commits touching auth and API endpoints
2. **Drift check** finds the Hub now validates a field that the App does not send — Medium drift
3. **Architect** raises ARCH-001 (missing validation, High) and ARCH-002 (drift, Medium)
4. **CTO** accepts ARCH-001, downgrades ARCH-002 to Low (App update is already planned) → **Pass**
5. **Triage** creates GH#250 for ARCH-001, adds a watchpoint for the drift
6. **Issue management** creates the issue, comments on GH#234 (partially addressed), removes `hub` label from GH#238
7. **Briefing** generates a new BRIEFING.md with GH#250 at priority 1 and the drift watchpoint in Cross-Repo Context
