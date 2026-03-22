---
description: Evolve an auto-improve rubric by analyzing results and finding quality gaps
---

Analyze the latest auto-improve results to find quality issues not covered by the current evaluation rubric, then update the rubric accordingly.

The auto-improve skill to analyze: $ARGUMENTS

---

## Overview

After running an auto-improve cycle, the evaluation rubric may have blind spots — quality issues that exist in the output but aren't captured by any rubric question. This skill:
1. Reads the latest extraction results from the most recent auto-improve run
2. Performs a deep, rubric-independent quality analysis
3. Identifies gaps where real issues exist but no rubric question catches them
4. Proposes new rubric questions and gets user approval
5. Updates the evaluation rubric

---

## Phase 1: Load Context

1. **Identify the feature**: Parse `$ARGUMENTS` to determine the auto-improve skill name. Expected format: a feature name like `comparisons`, `products`, `features`, etc.

2. **Load the current rubric**: Read `.claude/commands/evaluate-{feature}.md` (or the appropriate evaluate file — check `.claude/commands/` for the matching evaluate skill).

3. **Load the latest results**: Find the debug/output directory by reading the auto-improve skill file (`.claude/commands/auto-improve-{feature}.md`) and locating its configured paths (typically `.debug/auto-improve-{feature}/` but may vary). Read the most recent extraction files from the extractions subdirectory. Sort by modification time and take the latest iteration's files.

4. **Load the results TSV**: Read the `results.tsv` file from the auto-improve output directory to understand the score trajectory and which failures were already addressed.

5. **Load the latest evaluations**: Read the evaluations subdirectory for the most recent iteration to see which rubric questions passed/failed.

If any of these files don't exist, report what's missing and stop.

---

## Phase 2: Deep Quality Analysis

For each extraction file from the latest iteration, perform a **rubric-independent quality review**. This means: forget the rubric exists and evaluate the output fresh, looking for ANY quality issue regardless of whether a rubric question covers it.

### Analysis Dimensions

For each extracted result, systematically check:

#### 2a. Structural Issues
- Missing or empty fields that should be populated
- Unexpected null values or empty arrays
- Schema violations or malformed data
- Inconsistent field presence across items

#### 2b. Content Quality Issues
- Factual errors or inconsistencies within the data
- Repetitive or templated content that lacks specificity
- Content that doesn't match the context (wrong category, wrong domain)
- Vague or generic text where specifics are needed
- Grammar, spelling, or formatting problems

#### 2c. Logical Issues
- Internal contradictions (e.g., "budget pick" with highest price)
- Missing logical connections between related fields
- Rankings or scores that don't match the evidence
- Comparisons that don't make sense

#### 2d. Domain & App-Type-Specific Issues

Check dimensions appropriate to the feature's app type:

**For APIs**: Response codes, headers, error formats, pagination correctness, auth handling
**For UIs**: Accessibility (ARIA, keyboard), responsive behavior, loading/error states, visual consistency
**For data pipelines**: Schema drift, null rates, distribution shifts, dedup correctness, freshness
**For ML/AI**: Prediction distribution, confidence calibration, prompt leakage, hallucination patterns
**For CLIs**: Exit codes, help text accuracy, error message clarity, output format consistency
**For libraries**: API contract correctness, type safety, backward compatibility

Also check general domain-specific concerns:
- Industry-standard expectations not met
- Common sense violations for the feature's domain
- Missing information that a domain expert would expect
- Incorrect use of domain terminology

#### 2e. Edge Case Handling
- How does the output handle unusual inputs?
- Are there degenerate cases (all same score, single item, missing data)?
- Does the output gracefully handle partial information?

#### 2f. User Experience Issues
- Would a reader/user be confused by anything?
- Is there misleading information?
- Are there broken references or dead links?
- Does the output serve its intended purpose?

For each issue found, record:
- **Issue description**: What's wrong, with specific evidence (quotes, values)
- **Frequency**: How many test items exhibit this issue (X/N)
- **Severity**: Critical (misleads user) / Major (quality gap) / Minor (polish)
- **Current rubric coverage**: Does any existing question catch this? Which one?

---

## Phase 3: Gap Analysis

Compare your findings against the current rubric:

1. **Fully covered issues**: Issues caught by existing rubric questions → skip these
2. **Partially covered issues**: Existing questions touch on this but don't fully catch it → note for refinement
3. **Uncovered issues**: No rubric question addresses this → candidate for new questions

For each uncovered or partially covered issue, draft a rubric question:

```
### Gap: {issue description}
- **Frequency**: {X/N test items}
- **Severity**: {Critical/Major/Minor}
- **Evidence**: {specific examples from extractions}
- **Proposed question**: "{Yes/No question with clear criteria}"
- **Proposed category**: {which rubric category A-J it belongs in}
- **Why not currently caught**: {explanation of the gap}
```

### Prioritization Criteria
Rank proposed new questions by:
1. **Severity × Frequency**: Critical issues seen in all items rank highest
2. **Objectivity**: Can the question be answered definitively from the data? (reject subjective questions)
3. **Actionability**: Would failing this question point to a fixable code/prompt change?
4. **Non-redundancy**: Does it genuinely test something no existing question covers?

Filter out:
- Issues that are subjective and can't be evaluated consistently
- Issues that are one-off data problems, not systematic
- Issues that would require external knowledge to evaluate
- Questions that substantially overlap with existing ones

---

## Phase 4: Present Findings

Present the analysis to the user in this format:

```
## Rubric Evolution Analysis: {feature}

### Latest Run Summary
- Iteration: {N}
- Mean score: {X}%
- Test items analyzed: {N}

### Issues Found Not Covered by Current Rubric

#### Critical Gaps (recommend adding)
1. **{Issue name}** ({frequency})
   Evidence: {quote from extraction}
   Proposed question: "{question text}"
   Category: {letter}

#### Major Gaps (recommend adding)
2. ...

#### Minor Gaps (consider adding)
3. ...

### Existing Questions to Refine
1. Q{N}: Current: "{current text}" → Proposed: "{refined text}"
   Reason: {why the refinement catches more issues}

### Questions to Consider Removing
1. Q{N}: "{text}" — Reason: {always passes, tests something no longer relevant, etc.}

### Recommended Changes Summary
- Add {N} new questions
- Refine {N} existing questions
- Remove {N} obsolete questions
- Net change: {current count} → {new count} questions
```

Wait for user feedback. The user may:
- Approve all changes
- Approve some and reject others
- Modify proposed questions
- Ask for more evidence on specific gaps

---

## Phase 5: Apply Changes

After user approval:

1. **Read the current rubric file** (`.claude/commands/evaluate-{feature}.md`)

2. **Apply approved changes**:
   - Insert new questions into the appropriate category sections
   - Renumber questions sequentially (all question numbers must be continuous)
   - Update category question counts in headers
   - Update the total question count in the skill description
   - Refine any approved question text changes
   - Remove any approved deletions
   - Update the scoring section if N/A conditions changed

3. **Update the auto-improve skill** (`.claude/commands/auto-improve-{feature}.md`):
   - Update the question count reference in the description
   - Add any new known fix targets if the gap analysis suggests them
   - Update the results.tsv header if rubric categories changed

4. **Verify consistency**:
   - All question numbers are sequential with no gaps
   - Category question counts match actual questions
   - Total in description matches actual total
   - Scoring section accounts for all question types
   - Output format template matches categories

---

## Phase 6: Validation

Run the updated rubric against one of the latest extractions to verify:
1. All new questions are answerable from the extraction data
2. No existing questions broke from renumbering
3. The rubric is internally consistent

Report:
```
## Rubric Evolution Complete: {feature}

### Changes Applied
- Added: Q{numbers} — {descriptions}
- Refined: Q{numbers} — {descriptions}
- Removed: Q{numbers} — {descriptions}
- Total questions: {old} → {new}

### Validation
- Tested against: {extraction file}
- All questions answerable: {Yes/No}
- Issues: {any problems found}

### Impact on Scores
Note: Adding questions may lower scores on the next auto-improve run since
there are now more ways to fail. This is expected and healthy — it means
the rubric is catching issues it previously missed.
```

---

## Important Notes

- **Never auto-approve rubric changes** — always present to the user first. The rubric is the most important artifact; changes affect all future evaluations.
- **Maintain objectivity** — every rubric question must be answerable from the extraction data alone, with no subjective judgment.
- **Preserve numbering stability when possible** — add new questions at the end of their category to minimize disruption to existing results.tsv data. Only renumber when removing questions.
- **Don't over-expand** — a rubric with 200 questions is worse than one with 80 well-chosen questions. Each question should earn its place.
- **Evidence is mandatory** — every proposed new question must cite specific examples from the extractions that demonstrate the gap.
