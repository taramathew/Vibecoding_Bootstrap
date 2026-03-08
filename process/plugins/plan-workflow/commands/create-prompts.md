---
description: Convert an approved plan into executable prompts
argument-hint: "[plan-name] — filename (without .md) from .docs/plans/"
---

You are converting an approved implementation plan into executable prompts.
Each prompt will be run in a fresh Claude Code session. Follow this workflow step by step.

## Step 1: Read context files

Read these files to understand the project rules and current state:
1. `CLAUDE.md` — project rules and constraints
2. `.docs/logs/DECISIONS.md` — past architectural decisions
3. `.docs/logs/IMPLEMENTATION_LOG.md` — recent implementation context

## Step 2: Read the plan

Read the plan: `.docs/plans/$ARGUMENTS.md`

If the file is not found: list available plan files in `.docs/plans/` and ask the
user to clarify which plan to convert.

## Step 3: Read workflow rules

Read `.claude/rules/plan-workflow.md` — this contains the prompt format, quality
checklist, anti-patterns, cross-prompt context rules, and error handling philosophy.

## Step 4: Verify plan status

Check the plan's Status field:
- **Draft**: Warn — "This plan is still in Draft. Plans should be Approved before
  generating prompts. Proceed anyway?"
- **Approved**: Good — proceed normally
- **In Progress**: Warn — "This plan is already In Progress. You may be re-generating
  prompts. Existing prompt file may be overwritten."
- **Done**: Warn — "This plan is marked Done. Re-generating prompts is unusual."

## Step 5: Check for prior phase context

Read the plan's "Lessons Learned" section (if it exists). These are mandatory
guardrails that MUST be wired into relevant prompts:
- **Code review findings** from prior phase — each must be assigned to a prompt
- **Patterns to avoid** — each must be referenced in the prompt where it applies
- **Deferred items** — must be included in the appropriate prompt's scope

## Step 6: Check existing prompt files

Read existing prompt files in `.docs/prompts/` for style and pattern consistency.
Match the established format and conventions.

## Step 7: Extract sequence number

Extract the sequence number from the plan filename:
- e.g., `023-admin-console` → `023`
- e.g., `001-first-feature` → `001`

If the plan has no number prefix, ask the user for the sequence number.

## Step 8: Create prompts

Convert each implementation step from the plan into a self-contained prompt.
The prompt file structure:

### Prompt 0 — Branch Setup

Every series starts with Prompt 0:

```
Read CLAUDE.md.
Read the plan: .docs/plans/[plan-name].md

This is Prompt 0 — Branch Setup for the [feature-name] series.

Create the feature branch:
  git checkout main
  git pull origin main
  git checkout -b feature/[branch-name]

Update plan status:
  .docs/plans/[plan-name].md: Status → "In Progress"
  Update backlog/tracker with the current status (if applicable).

Commit:
  git add .docs/plans/[plan-name].md [backlog file if updated]
  git commit -m "docs: set [plan-name] plan status to In Progress"
```

### Prompts 1-N — Implementation

One prompt per implementation step (or group small related steps). Each prompt MUST
be fully self-contained and include ALL of the following:

**Header — Read instructions:**
```
Read CLAUDE.md.
Read .docs/architecture/[RELEVANT_MODULE].md
Read the plan: .docs/plans/[plan-name].md (section "[relevant section]")
```

**Scope description:**
Brief description of what this prompt does, which files it touches, and risk level
(LOW/MEDIUM/HIGH).

**Prior steps context block (for Prompts 2+):**
```
**Prior steps context (read IMPLEMENTATION_LOG.md for details):**
- Step N-1 modified [file] — verify [what to check]
- Step N-2 created [resource] — verify [actual state vs planned]
- If any ADRs were created in prior steps, read them — they may affect this step
```

Include this block when:
- This prompt modifies files that a prior prompt also modified
- This prompt depends on infrastructure created by a prior prompt
- A prior prompt had conditional/fallback paths
Skip for Prompt 1 and fully independent prompts.

**Implementation instructions:**
Detailed, with specific file paths, component names, and expected behavior.

**Error handling philosophy block:**
```
If ANY errors occur: STOP. Do NOT do trial-and-error quick fixes.
  - Read the full error message, trace it back to the source, understand WHY it happened.
  - Cross-reference the fix against the relevant architecture module.
    If the architecture specifies a particular approach, understand WHY that decision
    was made before deviating.
  - Prefer the architecturally correct fix over a quick workaround.
  - If a fix requires changing the architecture, document the deviation in
    IMPLEMENTATION_LOG.md AND add an entry to DECISIONS.md.
  - If unsure about the root cause, investigate further (read source code, check docs)
    before attempting a fix.
```

**Documentation update template:**
```
APPEND a new entry at the END of .docs/logs/IMPLEMENTATION_LOG.md:
### [DATE] — [Short Title]
**Phase**: [Current phase]
**Files Changed**: [list]
**What**: [description]
**Why**: [rationale]
**Decisions**: [key decisions and reasoning]
**Issues**: [problems hit and resolutions]
**Verification**: quality gate results
```

**DECISIONS.md reminder (conditional):**
```
If any architectural decisions were made (new patterns adopted, tech choices,
deviations from existing architecture), also APPEND to .docs/logs/DECISIONS.md
using the ADR format:
### ADR-NNN: [Title] (YYYY-MM-DD)
**Context:** [situation]
**Decision:** [what was decided]
**Rationale:** [why]
**Alternatives Considered:** [what else was evaluated, if applicable]
**Status:** Implemented
```

**Verification gate:**
```
After implementation, run ALL quality checks (typecheck, lint, test, build).
Fix ALL failures before committing.
```

**Git commit instruction:**
```
Commit all changes:
  git add [list specific files]
  git commit -m "[type]: [short description of changes]"
```

### Phase Closure (for series with 3+ implementation prompts)

The final prompt in the series. Contains NO implementation work — only branch closure.

```
Read CLAUDE.md.
Read .docs/logs/IMPLEMENTATION_LOG.md (ALL entries from this phase — review the full series).
Read .docs/logs/DECISIONS.md (all ADRs from this phase — ensure none are missing or incomplete).
Read the plan: .docs/plans/[plan-name].md

This is the Phase Closure Prompt — the final prompt in the [feature-name] series.
Its only job is to get the branch into a mergeable state.

STEP 1: Dead code gate
Run your project's dead code detection tool.
If it reports findings, triage each one:
  - DEAD: delete the file / remove the export / remove the dependency
  - FORWARD EXPORT (declared in plan): add @public JSDoc tag
  - UNKNOWN: investigate before deciding (check plan, check git log)
Iterate until it exits cleanly.

STEP 2: Final quality pass
Run the full quality gate (typecheck + lint + test + build).
Fix any failures before proceeding.

STEP 3: Commit and push
APPEND a final entry at the END of .docs/logs/IMPLEMENTATION_LOG.md summarizing the series.
Then commit and push:
  git add [all modified/created files]
  git commit -m "[type]: [feature description] — phase closure"
  git push -u origin [branch-name]

STEP 4: Create PR and run code review
  gh pr create --title "[title]" --body "[description]"
  /code-review --comment
For each finding (confidence >= 80%): fix it, then commit the fix.

STEP 4.5: Knowledge promotion (rules feedback loop)
Review ALL IMPLEMENTATION_LOG entries from this series. For each learning, apply
the Knowledge Classification decision criteria (.claude/rules/plan-workflow.md §1):
  1. "Would ignoring this break code in any context?" → propose CLAUDE.md update
  2. "Would ignoring this produce a bad plan or prompt?" → update .claude/rules/plan-workflow.md
  3. "Was this a choice between alternatives?" → write ADR in DECISIONS.md
  4. "Does this change how a subsystem works?" → update relevant architecture doc
  5. "None of the above?" → leave in IMPLEMENTATION_LOG (no promotion needed)

Print a classification summary table before committing.
Include all knowledge promotions in the Phase Closure commit.

STEP 5: Update plan status
Update .docs/plans/[plan-name].md: Status → Done

STEP 6: Merge (requires user confirmation)
Present the PR URL and ask the user to confirm before merging.
When confirmed:
  gh pr merge --merge --delete-branch
```

## Step 9: Wire cross-prompt context

For each prompt that depends on prior prompts:
- Add `Read .docs/logs/IMPLEMENTATION_LOG.md` instruction (scoped to current phase)
- Add `Read .docs/logs/DECISIONS.md` instruction (scoped to recent ADRs)
- Add a "Prior steps context" block listing specific things to verify from prior steps
- Each entry must be specific: name the file modified, the decision point, or the
  fallback path — not generic "check prior work"

## Step 10: Run quality checklist

Verify each prompt against this checklist. All items must pass.

Key checks:
- [ ] Prompt 0 exists as the first prompt
- [ ] Phase Closure is the last prompt (for 3+ step series)
- [ ] Every prompt starts with "Read CLAUDE.md" + relevant architecture modules
- [ ] Every prompt references the plan
- [ ] Every prompt has the error handling philosophy block inside its code fence
- [ ] Every prompt has IMPLEMENTATION_LOG.md update template
- [ ] Every prompt has DECISIONS.md conditional reminder
- [ ] Every prompt has a verification gate
- [ ] Every prompt has a git commit instruction
- [ ] Dependent prompts have "Prior steps context" blocks
- [ ] Cross-prompt deferrals verified: if a prompt says "this happens in Step N",
      Step N's prompt actually includes the deferred work
- [ ] Lessons learned from prior phase are wired into relevant prompts

## Step 11: Save the prompt file

Save to: `.docs/prompts/[NNN]_[PLAN_NAME]_PROMPTS.md`

Where:
- `NNN` = sequence number from Step 7
- `PLAN_NAME` = plan name in UPPER_SNAKE_CASE

## Step 12: Update the plan

Update the plan's metadata table:
- Set `Prompt File` to the link to the new prompt file

## Step 13: Present for review

Present a summary to the user:
- Number of prompts created (Prompt 0 + N implementation + Phase Closure if applicable)
- Brief description of each prompt's scope
- Any lessons learned that were wired into prompts
- Any concerns about cross-prompt dependencies

Ask the user to review the prompts before execution.
