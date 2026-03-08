---
description: Create an implementation plan from a backlog feature
argument-hint: "[feature-name] — feature title or # from backlog"
---

You are creating an implementation plan for a feature. Follow this workflow step by step.

## Step 1: Read context files

Read these files to understand the project rules and current state:
1. `CLAUDE.md` — project rules and constraints
2. `.docs/logs/DECISIONS.md` — past architectural decisions (check for conflicts)
3. `.docs/logs/IMPLEMENTATION_LOG.md` (last 500 lines) — recent context

## Step 2: Find the feature in the backlog

Read your project's backlog file (e.g., `.docs/FEATURE_BACKLOG.md`) and find the row
matching `$ARGUMENTS` (match on `#` number or feature title). Extract the description
as the starting requirement.

If no match found: list available features from the backlog and ask the user to clarify
which feature to plan.

## Step 3: Read workflow rules

Read `.claude/rules/plan-workflow.md` — this contains the plan format, quality
checklist, anti-patterns, and Knowledge Classification system to follow.

## Step 4: Read relevant architecture modules

Use the "Which Module to Read" table (§6) from the rules file to determine which
architecture modules to read based on the feature description. Read each relevant module
from `.docs/architecture/`.

## Step 5: Check for overlapping plans

Read all files in `.docs/plans/` and check if any overlap with this feature. Note
any conflicts or related plans.

## Step 6: Research the codebase

Perform these research steps:

1. **CODEBASE SCAN**: Search the codebase for existing code related to this feature.
   Identify files, patterns, types, and hooks that already exist and should be reused.

2. **ARCHITECTURE CHECK**: Verify the proposed approach doesn't violate any rules in
   CLAUDE.md or conflict with decisions in DECISIONS.md.

3. **CONFLICT CHECK**: Look for existing plans in `.docs/plans/` that overlap. Note
   any conflicts.

4. **PATTERN CHECK**: Find how similar features were implemented in the past (check
   the implementation log for patterns).

5. **DEPENDENCY MAPPING**: Identify what must exist before this feature can be built.

6. **PLATFORM CHECK**: If the feature uses platform-specific APIs (e.g., popups, clipboard,
   file system, push notifications, native modules), check the architecture docs for
   supported deployment targets. Verify the approach works across all targets or plan
   fallback paths. If no platform-specific APIs are used, skip this step.

7. **PRIOR PHASE CHECK**: If this feature continues from a prior phase, check for:
   - Unfixed code review findings from the prior phase's PR
   - Lessons learned documented in the prior plan
   - Deferred items that must be addressed in this phase

8. **ADR RELEVANCE CHECK**: Scan DECISIONS.md for ADRs that mention files, subsystems,
   or patterns overlapping with the current feature scope. Flag these as mandatory
   reading. Note any conflicting or superseded decisions. Also search archived
   implementation logs (`.docs/logs/archive/`) for entries mentioning relevant
   subsystems — older learnings may surface patterns not yet promoted to ADRs.

## Step 7: Clarify requirements

Present your codebase findings to the user. Then ask 3-6 focused clarifying questions
covering these areas (skip any that are obvious from the description):

- **INTENT**: What is the core goal? What problem are we solving?
- **USE CASE**: Who uses this? What's the user journey / workflow?
- **SCOPE**: What's explicitly NOT included? (prevents scope creep)
- **EXPECTATIONS**: Any specific UX behavior or design requirements?
- **CONSTRAINTS**: Technical limits, security rules, performance targets?
- **DEPENDENCIES**: Does this depend on anything not yet built?

Base questions on codebase findings (e.g., "I found X exists already — should this
feature extend it or replace it?"). Iterate until requirements are clear.

## Step 8: Determine sequence number

Scan `.docs/plans/` for the highest `NNN-` prefix in existing plan filenames.
Increment by 1 for this plan. If no numbered plans exist, start at 001.

## Step 9: Write the plan

Write the plan following the Standard Plan Format from `.claude/rules/plan-workflow.md` (§2).

Save to: `.docs/plans/[NNN]-[feature-name].md`

Run through the Plan Quality Checklist (§3) — all items must pass.

Include a **Relevant ADRs** section listing ADR numbers and one-line summaries for
decisions that affect this feature (populated from the ADR RELEVANCE CHECK in Step 6).

No "TBD" placeholders — resolve everything or list as explicit Open Questions.

## Step 10: Update backlog

Update the backlog: set the feature row's Status → "Planning" and add a link to the
plan in the Plan column.

## Step 11: Present for review

Present a summary of the plan to the user:
- Problem being solved
- Chosen approach (and why alternatives were rejected)
- Number of implementation steps
- Key files affected
- Any open questions remaining

Ask the user to review and confirm before finalizing.
