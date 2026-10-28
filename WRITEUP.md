Reference: MATACS-TH-2026-02

The prompts in PROMPTS.md have not been run against a model. I calculated the sample ranking by hand using the method below. It matches ranking.json.

## 1. AI vs. rules

AI is used for one step: reading a proposal and extracting evidence for each criterion component in the configuration (a verbatim quote, a page, a value and a status). Proposal wording varies too much for simple rules to do this reliably.

Everything else is plain code. That covers PDF text extraction, pattern-based screening for embedded instructions, quote verification, mandatory checks, normalisation, weighting, ranking and the written reasons (templates, not AI text). I drew the line at anything that decides an outcome. Those steps must be repeatable and defensible, and a model cannot guarantee either. The model never sees weights, thresholds or other bidders, and it does not score, rank or judge eligibility.

Criteria live in versioned admin configuration. Publishing is blocked unless:
- weights sum to exactly 100;
- every weighted criterion has a rubric and at least one extractable component;
- every mandatory criterion has a pass/fail definition;
- no criterion is both mandatory and weighted.

Mandatory checks run separately from, and before, weighted scoring.

## 2. Consistency and explainability

The brief gives no scoring formula, so I made these assumptions. The resulting ranking is one defensible result, not the only possible one.

- **Mandatory criteria:**
  - **Pass:** the bidder itself states the fact and the quote verifies.
  - **Fail:** the proposal affirmatively shows non-compliance, including conditional statements such as a licence held by a subcontractor "to be confirmed". The proposal becomes `ineligible`, with null rank and score.
  - **Unverified:** the fact is missing or ambiguous, and the proposal is `flagged`.
- **Normalisation:** among proposals that are not ineligible, lower-is-better criteria score `100 × lowest ÷ value` and higher-is-better criteria score `100 × value ÷ highest`. A price over the ceiling, or a time over 16 weeks, scores 0.
- **Sub-scores:** warranty splits equally between panel and workmanship. Team and approach has four equal items: named lead, crew, safety plan and monitoring plan. Experience is scored on the stated count of installs. Where the five-year window is unclear (A, C and E in the sample), the reviewer gets a note rather than a silent adjustment.
- **Missing weighted evidence:** the criterion scores 0 and the proposal is flagged. Flagged proposals get a provisional score, shown for reference but not placed in the numbered ranking until a human clears them.
- **Ties:** scores are rounded to one decimal. Equal scores display as a tie for the reviewer to resolve, with no arbitrary tiebreak.

Sample result: A (98.0) and E (84.8) are ranked 1 and 2. C (77.1) and D (69.8) are flagged with provisional scores. B is ineligible. Scores are relative to the batch, so adding a proposal can change the others. The reference set is logged with each run to make this visible.

**Repeatability.** The calculation is a pure function of the verified extraction and the pinned configuration, so identical inputs always give an identical ranking. A new model call could extract differently even at temperature 0. The validated extraction is therefore stored, and re-scoring reuses it. A fresh extraction is a new run, and any fields that differ are listed for review.

**Explainability.** Each position links to its criterion scores, the verified quotes with page numbers, and any unverified fields. The reason text is generated from those numbers, so it cannot contradict them.

## 3. Human in the loop

- **Upload:** the reviewer confirms the category after seeing its criteria summary.
- **Flag queue:** a reviewer must resolve every flagged proposal: embedded instructions, unreadable files, failed extraction, and missing or ambiguous mandatory evidence. Finalising is blocked while flags remain.
- **Corrections:** a reviewer can correct a value or override eligibility or rank. A written reason is required, and the system re-runs the deterministic steps.
- **Escalation:** if a reviewer cannot resolve an item or disputes a result, it goes to the facilities lead. Any clarification obtained from a bidder is entered as a separate, labelled evidence record.
- **Award:** a person makes the decision. The tool records who decided, when, and on which run. It never awards.

The append-only audit log stores:
- configuration version;
- file and extracted-text hashes, and screened spans;
- prompt identifier, rendered prompt hash, model identifier and call settings;
- raw and validated model output, and validation results;
- the reference set, score breakdown and ranking;
- code version;
- every human action, with user, time, before and after values, and reason.

## 4. Changing criteria

Published configurations are immutable. Any edit creates a new version, and each run is pinned to the version it used. Earlier results stay unchanged, labelled with their version and reproducible. Rankings from different versions are never merged.

- **Weights, thresholds or rubric changed:** the application re-scores the stored verified evidence in code. No AI call is needed. The result is a new run with a new reference set.
- **Criterion definitions or components changed:** the evidence the model must find has changed, so the proposals are re-extracted under the new version, as a new run.

Reviewers see a notice when a category has a newer version than a run. If an award decision already exists on an older run, it stays attached to that run. The new run is available for comparison, and a human decides whether it matters.

## 5. Risks

| Risk | Safeguard |
|---|---|
| **Wrong or missed extraction** | Each value needs a quote, checked verbatim against the source and parsed from it. Failures become `unverified`, are treated as not stated, and flag the proposal. Reviewers can correct fields. Limit: a verified quote shows the proposal says something, not that it is true. |
| **Fabricated facts** | A quote that is not in the source is never used. Output must match a strict schema, with one retry. If the retry fails, the proposal is flagged and not scored. |
| **Document manipulation** | Proposal C contains an instruction to automated reviewers. Three layers apply: screening removes and flags the text; the prompt treats the proposal as data inside a random boundary; the model has no scoring or ranking authority. C is scored on verified content only and sent to a human. Screening cannot catch every phrasing, so the other layers matter. |
| **Bias** | The prompt ignores company name, size, writing quality and reputation. Scoring uses only configured criteria, applied identically. I have not tested for bias. Reviewers should periodically inspect overrides and extraction corrections. |
| **Inconsistent AI output** | Temperature 0, pinned model and prompt, stored validated output, and a diff when a re-extraction differs. |
| **Cost** | One extraction call per proposal per configuration version, with a file size limit and a single retry. Re-scoring after weight changes makes no AI calls. |
| **Wrong configuration** | Publish-time validation, plus category confirmation at upload. |