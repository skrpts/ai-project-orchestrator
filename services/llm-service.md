---
type: service
id: llm-service
title: LLM Service
description: "Language model service for analysis, synthesis, and document generation"
tags: [Production, Tested]
connections: []
metadata:
  serviceType: llm
  auth_type: api_key
---

## LLM Service

This skrpt uses a language model for analytical and generative tasks. The LLM handles structured analysis, content synthesis, document generation, and quality validation across each stage of the workflow.

### Usage Pattern

The LLM is invoked at each stage of the pipeline. Earlier stages produce structured analysis (frameworks, assessments, breakdowns), while later stages synthesise outputs into coherent documents. The final stage is typically the most token-intensive, requiring cross-referencing across all previous outputs.

### Configuration

- **Temperature:** 0.3 for structured analysis tasks, 0.5 for narrative and synthesis tasks
- **Max tokens:** 4,000 per invocation, 8,000–10,000 for final assembly stages
- **Context window:** Each stage receives the outputs of all previous stages. The assembly stage requires the full context window.

### Requirements

- A configured LLM provider in skrptiq settings
- Sufficient token quota for the full pipeline
- No external network access required beyond your AI provider's endpoint
