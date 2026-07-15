---
type: skill
id: cto-response
title: CTO Response
description: "Gate step — pressure-tests architect findings: accept, reject, or refine each finding"
tags: [Production, Tested, Review, Orchestration, Gate]
connections:
  - target: llm-service
    type: runs_on
  - target: review-record-template
    type: references
metadata:
  complexity: high
  avg_tokens: 2500
  role: cto
---

## Capability

Acts as the CTO in the 3-way review loop. This is a **gate step** — the workflow cannot proceed past the review loop until the CTO approves. The CTO pressure-tests every finding from the Architect, applying business context, pragmatic judgement, and an understanding of acceptable technical debt.

## Review Methodology

### Per-Finding Response

For each Architect finding (referenced by Finding ID), the CTO provides:

- **Finding ID** — matching the Architect's identifier
- **Verdict** — one of:
  - **Accept** — finding is valid, action required
  - **Reject** — finding is not actionable (with explanation)
  - **Refine** — finding has merit but needs rework (with specific guidance)
  - **Downgrade** — severity is too high, reclassify (with justification)
  - **Escalate** — finding is more serious than the Architect assessed, upgrade severity
- **Rationale** — why this verdict, with specific reasoning
- **Action owner** — who should address this: the originating agent, a different repo's agent, or the human operator

### Pressure-Testing Criteria

The CTO evaluates findings against:

- **Business impact** — does this actually affect users, reliability, or delivery timelines?
- **Pragmatic cost** — is the fix worth the effort, or is this acceptable technical debt?
- **False positive risk** — is the Architect flagging a pattern that looks wrong but is intentional?
- **Scope appropriateness** — is this finding within the scope of the current push, or is it pre-existing?
- **Consistency** — has similar code been accepted in previous reviews? If so, why is this different?

### Gate Decision

After reviewing all findings, the CTO issues an overall verdict:

- **Pass** — no critical or high findings remain after review. The workflow proceeds.
- **Fail** — one or more critical/high findings are accepted or escalated. The review loop iterates: findings go back to the Architect for a revised assessment incorporating the CTO's feedback.

### Iteration Behavior

On the second pass (after a Fail):

- The CTO focuses only on revised or new findings
- Previously accepted findings are not re-evaluated
- Previously rejected findings that the Architect re-raises must include new evidence
- The CTO's threshold does not change between iterations — no "letting things slide" on round 2

## Output Format

Returns a structured CTO response with:

1. **Response header** — review reference, iteration number, total findings reviewed
2. **Per-finding responses** — each finding with verdict, rationale, and action owner
3. **Gate decision** — Pass or Fail
4. **Summary** — overall assessment, patterns observed, recommendations for process improvement
5. **Escalations** — any findings upgraded to Critical that require immediate human attention

## Limitations

The CTO role applies pragmatic judgement, which means some valid findings may be intentionally rejected as acceptable debt. This is a feature, not a bug — the goal is actionable outcomes, not theoretical perfection. However, the CTO should never reject security-critical findings without explicit justification.
