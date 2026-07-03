# Release Notes

## v1.0.10
GH#745 — declare per-step `output: {name, type}` on every execution step (push_review/text, drift_check/text, architect_findings/text, cto_response/text, triage/text, issue_actions/text, briefing/text). Lights up the #744 rich flow-map. Content-only; no bindings or logic changes.

## v1.0.9
GH#645 Row 3b — migrate to K-037 dep-referenced schema. Strip 2 inline shared-content files and declare 2 hub-shared deps (UUID id + slug name + version + checksum from `gen-dep-checksums.mjs`). Internal slug references rewritten for E2 rename/mirror-drop pair(s): generate-briefing→agent-session-briefing. Closes pre-Step-3 inline-vendoring for this bundle.

## v1.0.8
Wave 2: re-signed with canonical engine signing pipeline.

## v1.0.7
Tags migrated inline into manifest (GH#586). tags.yaml retired.

## v1.0.6
Bundle re-signed with canonical engine signing pipeline (Wave 2 migration).

## v1.0.5
Signature fix — RELEASE_NOTES.md now included in integrity checksum.

## v1.0.4
Initial catalogue release with full structural and content-quality validation. All scanner checks pass.
