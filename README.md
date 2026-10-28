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

I used Claude [and ChatGPT, only if true] to draft the architecture, prompts, write-up and diagram, and I reviewed each output. The diagram was rendered with Graphviz. I checked the ranking scores by recalculating them by hand. **[Your real disagreement or correction goes here: what the AI proposed, what you changed or rejected, and why.]**
