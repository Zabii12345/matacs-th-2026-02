# MATACS Associate Software Developer Assessment

**Candidate:** Muhammad Zohaib Ahmed 

**Reference:** MATACS-TH-2026-02

**Time spent:** 37 minutes

## Overview

A design for an AI-assisted proposal triage tool. AI extracts quoted evidence from proposals. Eligibility checks, scoring and ranking are deterministic code. People review flagged items and make the final award decision. No code is included, and the prompts and ranking were not run against a model.

## Files

- [Process flow diagram](diagram/flow.png)
- [Write-up](WRITEUP.md): answers to the five questions
- [Prompts](PROMPTS.md): extraction prompt, retry prompt and output schema
- [Expected ranking](ranking.json): proposals A–E

## AI tool use

I used Claude to draft the architecture, prompts, write-up and diagram, and I reviewed each output. The diagram was rendered with Graphviz. I checked the ranking scores by recalculating them by hand. 
**Initially, the AI suggested treating all proposals with missing or uncertain information in the same way. I disagreed with this approach because missing evidence does not always mean a proposal fails a mandatory requirement. For example, Proposal D does not specify a completion timeline, so I would flag it for clarification rather than automatically declare it ineligible. This makes the evaluation fairer and ensures that mandatory failures are distinguished from missing information that requires human review.**
