---
type: skill
id: schema-drift-check
title: Schema Drift Check
description: "Compares shared schemas and contracts across repos, flags divergence that could cause integration failures"
tags: [Production, Tested, Code, Quality, Orchestration]
connections:
  - target: llm-service
    type: runs_on
metadata:
  complexity: high
  avg_tokens: 2000
  contract_types: [zod-schemas, typescript-interfaces, yaml-specs, api-contracts, database-schemas]
---

## Capability

Detects when shared contracts — schemas, types, API specs, configuration formats — have diverged between repos in a multi-repo project. This is the most common source of silent integration failures in projects with independent agent sessions per repo.

## Detection Strategy

### Contract Identification

Identify shared contracts by scanning for:

- **Shared packages** — files in `packages/shared/` or similar locations that multiple repos import
- **Spec documents** — YAML/JSON specs, OpenAPI definitions, or markdown specifications that define cross-repo contracts
- **Mirrored types** — TypeScript interfaces, Zod schemas, or data classes that exist in multiple repos with the same logical purpose
- **Configuration contracts** — environment variables, feature flags, or configuration keys that must be consistent across services

### Drift Categories

- **Type drift** — a field was added, removed, or changed type in one repo but not another
- **Validation drift** — validation rules (required/optional, min/max, enum values) differ between repos
- **Default drift** — default values for shared fields differ, causing inconsistent behaviour
- **Version drift** — one repo references a newer version of a shared spec while another still uses the old one
- **Naming drift** — the same concept uses different field names in different repos (e.g., `userId` vs `user_id`)

### Severity Classification

- **Critical** — type changes that will cause runtime errors (field removed, type changed from string to number)
- **High** — validation differences that will cause silent data loss (stricter validation in consumer than producer)
- **Medium** — default value differences that cause inconsistent but not broken behaviour
- **Low** — naming inconsistencies or documentation drift

## Output Format

Returns a structured drift report with:

1. **Contract inventory** — all shared contracts identified, with their locations across repos
2. **Drift findings** — each divergence with severity, affected repos, specific field differences, and code evidence
3. **Propagation risk** — which changes need to be propagated to which repos, in what order
4. **Recommended actions** — specific file changes needed to resolve each drift finding

## Limitations

This skill compares contracts based on structural analysis, not runtime behaviour. It cannot detect semantic drift where the same schema is used for different purposes in different repos. It also cannot verify that a schema change has been deployed — only that the source code has diverged.
