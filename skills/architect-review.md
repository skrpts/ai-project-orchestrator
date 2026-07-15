---
type: skill
id: architect-review
title: Architect Review
description: "Produces structured findings with code evidence, severity, classification, and confidence ratings"
tags: [Production, Tested, Code, Review, Orchestration]
connections:
  - target: llm-service
    type: runs_on
  - target: review-record-template
    type: references
metadata:
  complexity: high
  avg_tokens: 3000
  role: architect
  review_type: structured
---

## Capability

Acts as the Architect in the 3-way review loop. Examines push review output, drift check results, and any flagged issues to produce structured findings. Each finding is backed by specific code evidence and classified for the CTO to pressure-test.

## Review Methodology

### Evidence-Based Findings

Every finding must include:

- **Finding ID** — sequential identifier (e.g., `ARCH-001`) for reference in the CTO response
- **Code evidence** — specific file paths, line numbers, and code snippets that demonstrate the issue
- **Severity** — Critical, High, Medium, or Low
- **Classification** — one of: Bug, Security, Performance, Architecture, Maintainability, Convention, Documentation
- **Confidence** — High (certain this is an issue), Medium (likely an issue but context may change the assessment), Low (possible issue, needs investigation)
- **Impact statement** — what happens if this is not addressed
- **Recommended action** — specific steps to resolve

### Review Scope

The Architect reviews three inputs:

1. **Push review output** — the summary and impact assessment from the push-review skill
2. **Drift check output** — any schema or contract divergence detected
3. **Open issues context** — whether the changes align with assigned work or introduce scope creep

### Prioritization

Findings are ordered by a composite score:

- Severity weight: Critical = 4, High = 3, Medium = 2, Low = 1
- Confidence multiplier: High = 1.0, Medium = 0.7, Low = 0.4
- Score = severity weight × confidence multiplier

Findings with the same score are ordered by classification: Security > Bug > Architecture > Performance > Maintainability > Convention > Documentation.

## Output Format

Returns a structured review record with:

1. **Review header** — repo name, push reference, timestamp, total finding count
2. **Findings list** — each finding in the structured format above, ordered by priority score
3. **Summary assessment** — overall health of the push: Clean, Minor Issues, Significant Concerns, or Requires Rework
4. **Cross-repo implications** — findings that affect sibling repos, with specific propagation recommendations

## Interaction with CTO Response

The CTO receives this output and pressure-tests each finding. The Architect should:

- Not hedge — state findings clearly even if confidence is medium
- Provide enough evidence that the CTO can evaluate without re-reading the full diff
- Flag disagreements with previous review rounds explicitly (if this is iteration 2+)
- Note any findings that were previously rejected by the CTO and explain why they are being re-raised (with new evidence)

## Limitations

The Architect reviews structure and patterns, not runtime behavior. Performance findings are based on static analysis heuristics. Security findings are based on known patterns, not penetration testing.
