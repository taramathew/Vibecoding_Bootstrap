<!-- Last Updated: YYYY-MM-DD | Updated by: [phase/feature name] -->

# Plan & Prompt Workflow Rules

Auto-loaded contextual knowledge for `/create-plan` and `/create-prompts` commands.
Contains plan format, prompt format, quality checklists, anti-patterns, and the
Knowledge Classification system that governs where learnings are stored.

---

## 1. Knowledge Classification

All project knowledge falls into five categories. During Phase Closure, every learning
from the implementation log is routed to the correct destination using the decision
criteria below.

### Classification Matrix

| Knowledge Type | Where It Lives | When to Write | Who Consumes It | Lifespan |
|---|---|---|---|---|
| Universal project constraints (always-on rules — security, code quality, testing, platform) | CLAUDE.md | When a constraint applies to ALL code in ALL contexts | Every Claude session (auto-loaded) | Permanent until explicitly removed |
| Plan/prompt process rules (how to plan, format, anti-patterns, quality gates) | .claude/rules/plan-workflow.md | When a workflow/process learning applies to future plan/prompt creation | /create-plan, /create-prompts (auto-loaded) | Updated via Phase Closure feedback loop |
| Architectural design decisions (one-time choices between alternatives) | DECISIONS.md (ADR format) | When choosing between 2+ viable approaches for a specific problem | Developers working on that subsystem; /create-plan reads all | Permanent historical record |
| Implementation journal (what happened, chronological) | IMPLEMENTATION_LOG.md | After every code change (per CLAUDE.md rules) | Recent context for next session; Phase Closure review | Archived per major phase boundary |
| Architecture specifications (how subsystems work) | .docs/architecture/*.md | When a subsystem's design or behavior changes | Plans/prompts touching that subsystem (via "Which Module to Read" table) | Updated when the system changes |

### Promotion Path

```
IMPLEMENTATION_LOG.md (everything starts here)
    ├─→ DECISIONS.md          — if it records a choice between alternatives
    ├─→ rules/plan-workflow.md — if it's a process/workflow pattern for future plans
    ├─→ CLAUDE.md              — if it's a universal constraint for all code
    └─→ architecture/*.md      — if it changes how a subsystem works
```

### Decision Criteria ("Smell Test")

During Phase Closure, apply these 5 routing questions to each learning:

1. "Would ignoring this break code in any context?" → CLAUDE.md
2. "Would ignoring this produce a bad plan or prompt?" → rules file (.claude/rules/plan-workflow.md)
3. "Was this a choice between alternatives worth recording?" → DECISIONS.md
4. "Does this change how a subsystem works?" → architecture/*.md
5. "None of the above?" → stays in implementation log (no promotion needed)

A learning can promote to multiple destinations, but content should not be duplicated —
the rules file states the principle and references the ADR by number.

### DECISIONS.md ↔ IMPLEMENTATION_LOG.md Relationship

These are NOT duplicates — they serve fundamentally different roles with a clear
extraction relationship:

- **IMPLEMENTATION_LOG.md** is the chronological journal of ALL work — bugs, features,
  fixes, decisions, verification. Written every session, mandatory. This is the raw record.
- **DECISIONS.md** is a curated index of architectural decisions only, **extracted from**
  the log (header: "Extracted from the implementation log"). Written only when a choice
  between alternatives was made.
- The log entry **stays in the log** after extraction. The ADR is a separate artifact
  providing a structured, searchable record of "we chose X over Y because Z."
- **`/create-plan` uses them differently**: DECISIONS.md for architectural constraints
  that might conflict with the new feature (via ADR RELEVANCE CHECK);
  IMPLEMENTATION_LOG.md for recent context about what's been happening.
- **Boundary**: "what technology/pattern should we use for X going forward?" →
  DECISIONS.md (ADR). "here's what we built this session and how we fixed issues" →
  stays in log only.

---

## 2. Standard Plan Format

Every plan MUST follow this format. Consistency enables `/create-prompts` to reliably
consume plans.

```markdown
# Plan: [Feature Title]

| Field | Value |
|-------|-------|
| **Status** | Draft |
| **Created** | YYYY-MM-DD |
| **Feature File** | [link or "N/A — backlog row #NNN"] |
| **Dependencies** | [list or "None"] |
| **Architecture Modules** | [which modules this plan relates to] |
| **Prompt File** | [link to prompt file once created, or "Not yet created"] |
| **Implementation Log** | [link to log entry when done, or "Not yet implemented"] |

---

## Problem
What problem does this solve? Why now? What's the user-facing pain or technical limitation?

## Current State
How does the system work today in the area this plan affects?
Reference specific files, components, or endpoints.
(This section prevents proposing something that already exists or conflicts.)

## Solution

### Approach
High-level description of the solution. Include diagrams where helpful:
- ASCII diagrams for data flow or architecture
- Component trees for UI changes
- Sequence diagrams for multi-step interactions

### Alternatives Considered
| Approach | Pros | Cons | Why Not |
|----------|------|------|---------|
| [Alternative 1] | ... | ... | ... |
| [Alternative 2] | ... | ... | ... |
| **[Chosen approach]** | ... | ... | **Selected** |

## Implementation Steps
Ordered list of discrete steps. Each step should be:
- Small enough for one Claude Code session
- Independent enough to be committed separately
- Testable on its own

1. [Step 1 — what it does, which files, estimated complexity]
2. [Step 2 — ...]
3. [Step 3 — ...]
N. **Phase Closure** — dead code check + final quality pass + commit + push + PR creation +
   `/code-review` + fix high-confidence findings + plan status → Done
   *(Required for plans with 3+ implementation steps. For 1–2 step plans, embed
   these steps in the last implementation step instead.)*

## Files Affected
| File/Directory | Action | Description |
|----------------|--------|-------------|
| `src/...` | Modify | ... |
| `src/...` | Create | ... |

## Forward Exports (for multi-phase plans)
Exports created in this phase that will be consumed by the next phase. These are
intentionally unused — your dead code checker will flag them during Phase Closure.
List them here so the closure prompt adds `/** @public */` JSDoc tags instead of
deleting them.

| Export | File | Consuming Phase | Purpose |
|--------|------|-----------------|---------|
| `exampleFn()` | `src/services/example.ts` | Phase B | Used by upcoming UI |

If no forward exports: "None — all exports have consumers in this phase."

> **Cleanup contract:** The consuming phase's plan MUST include a step to remove
> the `@public` forward-export tags once the exports gain consumers.

## Types & Interfaces
New types or interface changes needed (these are contracts — define them in the plan):
\`\`\`typescript
// New types to add to src/types/...
interface NewFeatureType {
  // ...
}
\`\`\`

## Architecture Alignment
- [ ] Doesn't violate CLAUDE.md rules (list which rules were checked)
- [ ] Consistent with DECISIONS.md (list relevant ADRs checked)
- [ ] Uses existing patterns from the codebase (list which patterns)
- [ ] Security implications reviewed
- [ ] Platform/deployment target compatibility verified (see architecture docs for supported targets)

## Relevant ADRs
[ADR numbers and one-line summaries for past decisions that affect this feature.
Populated by the ADR RELEVANCE CHECK during /create-plan.]

- ADR-NNN: [one-line summary of relevance]

If no relevant ADRs: "None identified."

## Lessons Learned (for multi-phase plans)
[If this plan continues from a prior phase, document findings here. These become
mandatory guardrails for prompt creation.]

### From Prior Phase Code Review
[List issues found during the prior phase's code review that were NOT fixed before merge.
Each item should include: file, description, fix instruction, and which implementation
step should address it.]

### Patterns to Avoid
[List recurring anti-patterns discovered during the prior phase that will recur in this
phase. Each item should include: what went wrong, why, and the correct approach.]

If this is a first phase or standalone plan: "N/A — no prior phase."

## Open Questions
[Unresolved decisions — MUST be answered before status moves to "Approved"]

If no open questions: "None — ready for approval."

## Verification
How do we know this is done? Be specific:
- [ ] Unit tests: [what to test]
- [ ] Integration tests: [what to test]
- [ ] Manual verification: [what to check]
- [ ] Build passes: all quality gate commands pass
```

---

## 3. Plan Quality Checklist

Before marking a plan as "Approved", verify ALL of these:

### Research Completeness
- [ ] Codebase was searched for existing related code
- [ ] Existing patterns were identified and plan reuses them
- [ ] All affected files are listed in "Files Affected"
- [ ] Dependencies are identified (other features, schema migrations, etc.)

### Architecture Alignment
- [ ] Plan checked against CLAUDE.md Critical Rules
- [ ] Relevant architecture modules were read and referenced
- [ ] DECISIONS.md was checked for conflicting past decisions
- [ ] No existing plans conflict with this plan
- [ ] Platform/deployment target compatibility checked (if the feature uses platform-specific APIs, verify against architecture docs)

### Design Quality
- [ ] Problem is clearly stated (not just "add feature X")
- [ ] Current State section explains how things work today
- [ ] At least one alternative was considered (even if obvious)
- [ ] Implementation steps are ordered by dependency
- [ ] Each step is small enough for one Claude Code session
- [ ] Phase Closure is the final step for plans with 3+ implementation steps
- [ ] Types/interfaces are defined (not left to implementation)
- [ ] Forward exports (if any) are declared in the "Forward Exports" table
- [ ] Verification criteria are specific and testable

### Prior Phase Context (for multi-phase plans)
- [ ] Code review findings from prior phase PR are documented in "Lessons Learned" section
- [ ] Each unfixed finding has a specific fix instruction and is assigned to an implementation step
- [ ] Recurring anti-patterns from prior phase are documented with "what went wrong" and "correct approach"
- [ ] Deferred items from prior phase are explicitly listed in the plan scope (not left implicit)
- [ ] Prior phase's `@public` forward-export tags are listed; a cleanup step is included to remove them once consumers exist

### Completeness
- [ ] No "TBD" or "TODO" placeholders remain
- [ ] Open questions are either resolved or explicitly listed
- [ ] Security implications are addressed
- [ ] Files Affected table is complete
- [ ] Lessons Learned section is populated (or marked "N/A" for first-phase plans)

---

## 4. Plan Lifecycle

```
Draft  →  Approved  →  In Progress  →  Done
  │                                      │
  └──→ Abandoned          Superseded ←───┘
```

| Status | Meaning | Who Changes It |
|--------|---------|----------------|
| **Draft** | Plan written, may have open questions | Auto (on creation) |
| **Approved** | All questions resolved, approach confirmed | You (after review) |
| **In Progress** | Prompts created and being executed | Auto (when first prompt runs) |
| **Done** | All prompts executed, feature verified | You (after verification) |
| **Superseded** | Different approach was taken | You (link to replacement) |
| **Abandoned** | Feature no longer needed | You (add reason) |

### Status Transitions
- **Draft → Approved**: You review the plan, resolve open questions, confirm approach
- **Approved → In Progress**: `/create-prompts` is used to create prompts. First prompt executes.
- **In Progress → Done**: All implementation steps are complete and verified
- **Any → Superseded**: Add a note at the top linking to the replacement plan/approach

### Seeded Principles

- Architecture reconciliation: After completing a phase, update architecture docs with any deviations discovered during implementation.
- Architecture hardening: When a plan touches a new subsystem, the first prompt should review and update the relevant architecture docs before writing code.

---

## 5. Multi-Phase Plans

Some features are too large for a single phase. When a plan spans multiple phases,
follow these guidelines:

### Splitting a Plan into Phases
- Each phase should have a clear, self-contained scope
- Define explicit boundaries: what's IN scope for Phase A and what's DEFERRED to Phase B
- List deferred items in a dedicated section with full inventory
- Each phase gets its own prompt file, but shares the same plan document

### Phase Transition Process
When Phase A completes and Phase B begins:

1. **Close Phase A** — Execute the Phase Closure Prompt: dead code check + quality gates + commit + push + create PR + `/code-review` + fix findings + merge to main
2. **Document findings** — Add unfixed code review findings and lessons learned to the plan's "Lessons Learned" section. Each finding needs: file, description, fix instruction, and which Phase B step should address it
3. **Update plan status** — Mark Phase A as "Done" in the plan header. Add Phase B status.
4. **Create Phase B prompts** — Use `/create-prompts` to create prompts that carry forward the full context (closure summary, lessons, deferred items, unfixed findings)
5. **New branch** — Phase B starts on a fresh branch from the updated `main`

### Forward Exports Across Phases

When Phase A creates exports (API wrappers, types, utility functions) that Phase B
will consume, they must be managed explicitly — otherwise dead code checks delete them
and Phase B recreates them (wasted effort, potential drift).

**Phase A responsibilities:**
1. Declare forward exports in the plan's "Forward Exports" table
2. During Phase Closure, add `/** @public Forward export: Phase [X] ([purpose]) */`
   JSDoc tags to each forward-exported symbol at its declaration site
3. For barrel re-exports: separate the forward export into its own `export {}` statement
   with the `@public` tag above it, so other exports in the barrel remain visible to dead code checkers
4. Document the tags in the implementation log

**Phase B responsibilities:**
1. Include a cleanup step that removes the `@public` forward-export tags from Phase A
2. Verify the forward exports are actually consumed (dead code checker will catch any that aren't)

**Why `@public` JSDoc tags instead of config-file-level ignores:**
- **Per-export granularity** — dead code checker still catches new dead code added to the same file
- **Self-documenting** — the tag explains why it's unused and which phase consumes it
- **Co-located** — the annotation lives next to the code, not in a separate config file

**JSDoc convention:**
```typescript
// For declarations (functions, classes, constants):
/** @public Forward export: Phase B (column editing UI) — POST /api/tables/{id}/columns */
export function createColumn(...) { ... }

// For barrel re-exports (separate from other exports to preserve dead code checker coverage):
/** @public - Forward export: Phase B (view config validation for view editing UI) */
export { viewConfigSchema } from "./validation.js";
```

### Why This Matters
- Claude Code sessions are stateless — a new session has zero memory of the prior phase
- Code review findings that aren't documented in the plan are effectively lost
- Lessons learned that aren't wired into specific prompts will produce the same bugs again
- The plan document is the single source of truth that bridges phases

---

## 6. Which Module to Read

Customize this table to match your project's architecture documentation structure.

| Task Type | Required Reading |
|-----------|-----------------|
| **Any task** (always) | `CLAUDE.md`, `IMPLEMENTATION_LOG.md` |
| Database / schema changes | `.docs/architecture/DATA_MODEL.md` |
| API endpoints / backend | `.docs/architecture/API.md` |
| Auth, permissions, roles | `.docs/architecture/ACCESS_CONTROL.md` |
| AI agents, tools, hooks | `.docs/architecture/AI_SERVICE.md` |
| Infrastructure, CI/CD | `.docs/architecture/INFRASTRUCTURE.md` |
| UI design, wireframes | `.docs/architecture/UI_SPEC.md` |
| Past architectural decisions | `.docs/logs/DECISIONS.md` |
| Design docs / solution specs | `.docs/solutions/*.md` or as defined in CLAUDE.md |

> **Note:** Add rows to this table as your project grows. Each architecture doc
> should cover one coherent subsystem. Include design/solution docs if they serve
> as the source of truth for specific subsystems.

---

## 7. How Plans Become Prompts

Each "Implementation Step" in an approved plan becomes one self-contained prompt.

1. **Read the plan** — Understand the feature, steps, and affected files
2. **Check for prior phase context** — If this plan continues from a prior phase, read "Lessons Learned" and "Code Review Findings." These are mandatory guardrails.
3. **Create Prompt 0** — Every multi-prompt series starts with branch setup
4. **Plan a Phase Closure Prompt** — For series with 3+ implementation prompts, plan a dedicated final prompt. For 1–2 prompt series, embed closure steps in the last prompt.
5. **Convert steps to prompts** — One prompt per step, ordered by dependency
6. **Wire cross-prompt context** — Each prompt runs in a fresh session with no memory. Dependent prompts MUST read `IMPLEMENTATION_LOG.md` and `DECISIONS.md` from prior steps.
7. **Wire lessons into prompts** — Each prompt references specific lessons that apply to its scope
8. **Reference the plan** — `Read the plan: .docs/plans/[plan-name].md` in each prompt

---

## 8. Prompt 0 — Branch Setup

Every multi-prompt series MUST start with a Prompt 0 that:
- Creates a feature branch from `main` (e.g., `feature/[feature-name]`)
- Updates the plan status from "Draft" or "Approved" to "In Progress"
- Updates the backlog/tracker with the current status
- Commits the status change as the first commit on the branch

This establishes a clean starting point and ensures the plan status tracks reality.

---

## 9. Phase Closure Prompt

Every multi-prompt series with **3+ implementation prompts** MUST end with a dedicated
Phase Closure Prompt. Its only job is to get the branch into a mergeable state.

**Why standalone?** The final implementation prompt is already doing real work. Adding
dead code checks + push + PR + code review on top forces a context switch mid-session.

**Tiny plans exception:** For series with 1–2 implementation prompts, embed the closure
steps in the last implementation prompt.

### Phase Closure Steps

**STEP 1: Dead code gate**
Run your project's dead code detection tool (e.g., `knip`, `ts-prune`, custom script).
If it reports findings, triage each one:
- **Dead code with no future consumer** → delete it
- **Forward exports declared in the plan's "Forward Exports" table** → add
  `/** @public Forward export: Phase [X] ([purpose]) */` JSDoc tag at the declaration site
- **Everything else** → investigate before deciding (check plan, check git log)

Iterate until the dead code checker exits cleanly.

**STEP 2: Final quality pass**
Run your project's full quality gate (typecheck + lint + test + build).
Fix any failures before proceeding.

**STEP 3: Commit and push**
Append a final IMPLEMENTATION_LOG.md entry summarizing the series.
Commit and push: `git push -u origin [branch-name]`

**STEP 4: Create PR and run code review**
```
gh pr create --title "[title]" --body "[description]"
/code-review --comment
```
Fix ALL findings with confidence >= 80%, regardless of severity label (Critical,
Important, etc.). Severity labels are for reporting priority — the fix threshold
is the confidence score alone. Do not cherry-pick by severity; fix every qualifying
finding before committing.

**STEP 4.5: Knowledge promotion (rules feedback loop)**
Review ALL IMPLEMENTATION_LOG entries from this series. For each entry's
**Decisions** and **Issues** sections, apply the Knowledge Classification
decision criteria (§1) to determine its destination:
1. "Would ignoring this break code in any context?" → propose CLAUDE.md update
2. "Would ignoring this produce a bad plan or prompt?" → update .claude/rules/plan-workflow.md
3. "Was this a choice between alternatives?" → write ADR in DECISIONS.md
4. "Does this change how a subsystem works?" → update relevant architecture doc
5. "None of the above?" → leave in IMPLEMENTATION_LOG (no promotion needed)

**Required artifact — classification summary table.** Print this table to the
terminal before committing. Every entry with a **Decisions** section gets a row:

| # | Learning | Route | Destination | Action |
|---|----------|-------|-------------|--------|
| 1 | [Example learning] | Q3 | DECISIONS.md | ADR-NNN |
| 2 | [Example learning] | Q5 | stays in log | — |

This table is the proof of work for Step 4.5. It must exist even if all entries
route to Q5 ("stays in log"). Do NOT skip this step — the table itself is the
minimum deliverable.

For rules file updates: state the principle in 1-2 sentences + reference the ADR
number or phase for full rationale. Do not duplicate ADR content.
Include all knowledge promotions in the Phase Closure commit.

**STEP 5: Update plan status**
Update plan: Status → Done

**STEP 6: Merge (requires user confirmation)**
Present the PR URL and ask the user to confirm before merging.
When confirmed, merge using your project's preferred strategy (e.g., `gh pr merge --merge --delete-branch`).
Define the merge strategy in CLAUDE.md so it's consistent across all contributors.

### Seeded Principles

- Knowledge promotion requires a classification artifact: The table is the proof of
  work — without it, "nothing to promote" is indistinguishable from "didn't do it."
- Each distinct step in any prompt must be its own TodoWrite item. Bundling multiple
  steps into one todo hides incomplete work.

---

## 10. Cross-Prompt Context

**CRITICAL — Each prompt in a multi-prompt series runs in a fresh Claude Code session
with zero memory of prior sessions.** If Prompt 4 depends on decisions made in Prompt 3
(e.g., a fallback path was taken, an interface shape was different than expected, an ADR
was written), Prompt 4 has no way to know unless explicitly told to read the logs.

**The problem:** A plan may say "Step 3 creates the endpoint; Step 4 consumes it." But
if Step 3 deviated from the plan (different URL, wrapper file created, schema changed),
Step 4 follows stale assumptions and produces broken code or redundant work.

**The fix:** Every prompt that depends on prior steps must include:

1. **Read instructions** for `IMPLEMENTATION_LOG.md` and `DECISIONS.md`:
   ```
   Read .docs/logs/IMPLEMENTATION_LOG.md (Phase N entries — understand what changed).
   Read .docs/logs/DECISIONS.md (recent ADRs — check for decisions from prior steps).
   ```

2. **A "Prior steps context" block** that lists specific things to verify:
   ```
   **Prior steps context (read IMPLEMENTATION_LOG.md for details):**
   - Step 2 modified `[file]` — verify current shape before adding new code
   - Step 3 created `[endpoint/resource]` — verify actual URL/interface and whether a
     wrapper was created (the prompt had a fallback plan)
   - If any ADRs were created in Steps 1-3, read them — they may affect this step
   ```

**When to include cross-prompt context:**
- **Always include** when a prompt modifies files that a prior prompt also modified
- **Always include** when a prompt depends on infrastructure created by a prior prompt
- **Always include** when a prior prompt had conditional/fallback paths
- **Skip** for Prompt 1 (nothing before it) and for fully independent prompts

The Phase Closure Prompt should always read ALL implementation log entries and ADRs
from the series to review the full body of work.

### Seeded Principle

- Verification spike results carry forward: When Prompt 0 produces findings (e.g., SDK behavior differs from docs), subsequent prompts must reference those findings explicitly — sessions are stateless.

---

## 11. Error Handling Philosophy

**CRITICAL — This block MUST be included inside every prompt's code fence. Prompts are
copy-pasted individually into new Claude Code sessions — anything outside the code block
is lost.**

```
**Error handling philosophy:**
- When you encounter ANY error (build failure, runtime crash, deploy failure, test failure),
  STOP and THINK deeply about the root cause before writing a fix. Do NOT do trial-and-error
  quick fixes.
- Read the full error message, trace it back to the source, and understand WHY it happened.
- Cross-reference the fix against the relevant architecture module to ensure it aligns with
  the architectural rationale. If the architecture specifies a particular approach, understand
  WHY that decision was made before deviating.
- If a fix requires changing the architecture, document the deviation and rationale in
  IMPLEMENTATION_LOG.md AND add an entry to DECISIONS.md.
- Prefer the architecturally correct fix over a quick workaround.
- If you're unsure about the root cause, investigate further (read source code, check docs)
  before attempting a fix.
```

### Seeded Principles

- Identify the architecturally correct fix, not the fastest one.
- Plans touching deployment must include end-to-end verification before declaring done.
- Plans touching networking/URLs must verify protocol compatibility and URL paths end-to-end.

---

## 12. Mandatory Documentation Updates

Every prompt session that modifies code MUST update these:

### 1. Implementation Log (ALWAYS)

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

### 2. Decisions Log (WHEN architectural decisions are made)

**Trigger:** Did you make a choice that future developers need to know about? If yes, write an ADR.

Examples of what NEEDS an ADR:
- Choosing a technology or library
- Adopting a new pattern project-wide
- Deviating from existing architecture
- Infrastructure/deployment choices

Examples of what does NOT need an ADR:
- Bug fixes that don't change architecture
- Routine feature implementation following existing patterns
- Test fixes, lint fixes, refactors within existing patterns

```
If any architectural decisions were made during this session, APPEND to
.docs/logs/DECISIONS.md following the ADR format:
### ADR-NNN: [Title] (YYYY-MM-DD)
**Context:** [situation]
**Decision:** [what was decided]
**Rationale:** [why]
**Alternatives Considered:** [what else was evaluated, if applicable]
**Status:** Implemented
```

### 3. Architecture Modules (WHEN architecture changes)

```
If the implementation deviates from or extends the architecture, update the SPECIFIC
architecture module file in .docs/architecture/.
```

---

## 13. Quality Gates

Define your project's quality gate commands in CLAUDE.md (e.g., `pnpm turbo typecheck lint test build`).
All references to "quality gate" or "quality checks" in this file mean those commands.

### Intermediate Prompts (Prompt 1 through N)

```
After implementation, run ALL quality checks (typecheck, lint, test, build).

Fix ALL failures before marking the task as complete.
If a failure requires an architectural change, document it in DECISIONS.md.

Commit all changes:
  git add [all modified/created files]
  git commit -m "[type]: [short description]"
```

### Phase Closure Prompt

```
[dead code check]                  # Must exit cleanly before committing
[fix any dead code findings]
git add [all modified/created files]
git commit -m "[type]: [feature description] — phase closure"
git push -u origin [branch-name]
gh pr create --title "[title]" --body "[description]"
/code-review --comment             # Reviews the FULL branch diff (all commits in the PR)
[fix any /code-review findings and commit]
```

**Multi-prompt series:** For series with 3+ implementation prompts, `/code-review` runs
once in a dedicated Phase Closure Prompt. Each intermediate prompt still runs
the full quality gate + commit.

### Seeded Principles

- Run the full pipeline (build + lint + typecheck), not individual checks. `typecheck` alone can pass while `build` (which includes typecheck + bundling) fails.
- Lint at each prompt boundary, not just at the end.
- Never pipe quality gate commands — shell pipes report the pipe's exit code, not the command's.

---

## 14. File Numbering

- 3-digit zero-padded sequential prefix (001, 002, ...)
- Number derived from backlog/tracker `#` column
- Plan filename: `[NNN]-[feature-name].md` (e.g., `001-feature-name.md`)
- Prompt filename: `[NNN]_[FEATURE_NAME]_PROMPTS.md` (e.g., `001_FEATURE_NAME_PROMPTS.md`)
- Commands auto-determine next number by scanning existing files
- Numbering starts fresh at 001 after archiving old files

---

## 15. Rules Feedback Loop

- **When**: During Phase Closure, after reviewing all IMPLEMENTATION_LOG entries (Step 4.5)
- **What**: Apply Knowledge Classification decision criteria (§1) to each learning
- **How**: Update the relevant section in this file (principle + ADR/phase reference), commit in the same PR
- **Metadata**: Update the `<!-- Last Updated: ... -->` comment at the top of this file

This creates a self-improving system: each feature execution can refine the rules that
govern future feature planning. The rules file grows organically from real implementation
experience, not from theoretical best practices.

---

## 16. Plan Anti-Patterns

| Anti-Pattern | Why It's Bad | Correct Approach |
|---|---|---|
| No "Current State" section | Claude proposes something that already exists or conflicts | Always document how things work today |
| No alternatives considered | May miss a simpler or better approach | Always evaluate at least one alternative |
| "TBD" in approved plans | Implementation prompts can't be written from incomplete plans | Resolve everything or keep as Draft |
| Steps too large ("Implement the whole feature") | Can't be done in one Claude Code session | Break into steps that each take one session |
| Steps too small ("Add import statement") | Wastes sessions on trivial changes | Group related small changes into one step |
| No types defined | Implementation invents types that may conflict | Define shared types in the plan |
| No security review | May introduce vulnerabilities | Always check CLAUDE.md security rules |
| Copy-pasting from another plan | Context differs — may carry wrong assumptions | Start fresh, reference the other plan if useful |
| No file paths | Implementation has to search for everything | List specific files to modify/create |
| Multi-phase plan with no "Lessons Learned" section | Prior phase findings are lost; same bugs recur in next phase | Always document code review findings and anti-patterns before closing a phase |
| Deferred items listed in prose but not inventoried | Implementation prompts miss components that need work | Create a full inventory table with file paths and complexity |
| Code review findings not assigned to implementation steps | Findings are documented but nobody fixes them | Each finding must map to a specific step number |
| Phase B plan assumes context from Phase A | Claude Code sessions are stateless — Phase B sessions know nothing about Phase A | Plan must be self-contained; all relevant context from Phase A belongs in the plan document |
| No Phase Closure step in multi-step plans | The PR + /code-review gets embedded in the last implementation step, mixing two distinct jobs | Add a dedicated Phase Closure step as the final implementation step |
| Assuming SDK/API behavior without verification | Unverified assumptions produce broken code when reality differs from docs | Always include a verification spike (Prompt 0) for new SDK/API integrations |
| Multi-phase plan with no "Forward Exports" section | Phase Closure deletes exports that the next phase needs, forcing re-creation (wasted effort, potential drift) | Declare forward exports in the plan. Phase Closure adds `@public` JSDoc tags. The consuming phase removes the tags. See §5 "Forward Exports Across Phases." |
| Plan lists a dependency as "already installed" without verifying | Implementation prompt discovers the dependency is missing, wasting time installing and adjusting | Always verify claimed pre-existing dependencies in the plan's "Current State" section |
| Replacing an endpoint without inventorying its validation constraints | Input validation regressions — the new code omits bounds that the old code enforced | When a plan deletes an endpoint and recreates its functionality elsewhere, explicitly list all validation constraints from the deleted code that must be preserved |
| Replacing a UI component without listing affected test files in "Files Affected" | Test mocks for the old component break — discovered only after running tests | When a plan step replaces a component, grep for test mocks referencing the old component and list affected test files |
| Incomplete Docker/CI context in deployment plans | Missing build context files cause deploy failures that are hard to debug remotely | Plans touching containerization must list ALL files needed in build context and account for dependency resolution (lockfiles, shared packages, config files) |
| No platform compatibility check for platform-specific APIs | Feature works on one target but breaks on others (e.g., mobile, PWA, desktop) | Check architecture docs for supported deployment targets. If the feature uses platform-specific APIs, verify compatibility and provide fallback paths |

> **Add your own project-specific anti-patterns here as you discover them during
> Phase Closure reviews.**

---

## 17. Prompt Anti-Patterns

| Anti-Pattern | Correct Approach |
|---|---|
| Asking for multiple unrelated changes in one prompt | One focused task per prompt |
| No verification step at the end | Always end with the full quality gate |
| No implementation log update | ALWAYS append to IMPLEMENTATION_LOG.md |
| No DECISIONS.md reminder in prompt footer | Add conditional DECISIONS.md update after IMPLEMENTATION_LOG block |
| "Fix it somehow" or "make it work" | Describe the expected behavior and root cause |
| Error handling / verification only in file header, not inside code blocks | Every prompt code block must be self-contained — include verification + error handling + log update inside the fences |
| Referencing files that may not exist | Verify file paths before referencing |
| Ignoring deferred items from previous phases | Check IMPLEMENTATION_LOG.md for deferred items |
| Coding a complex feature without a plan | Create a plan in .docs/plans/ first, then convert to prompts |
| Plan with no status or stale status | Always update plan status as it progresses (Draft → Done) |
| Prompts that don't reference their plan | Include "Read the plan: .docs/plans/[name].md" in each prompt |
| Plan with open questions marked "Approved" | Resolve all open questions before approving |
| Deferring work to "Step N" without verifying Step N includes it | Cross-check all "happens in Step N" notes against the target prompt |
| No git commit step inside the prompt code block | Every prompt must include a commit instruction with a suggested message |
| No Prompt 0 (branch setup) in multi-prompt series | Every series needs a clean branch + plan status update as its first commit |
| Ignoring lessons learned from prior phase | Wire each lesson into the specific prompt where it applies |
| Unfixed code review findings left orphaned | Findings must be explicitly assigned to prompts in the next phase |
| Continuation prompts with no phase closure context | New sessions have no memory — summarize what Phase A accomplished and deferred |
| Running individual checks instead of the full gate | Always run the full quality gate — match CI locally |
| No Phase Closure Prompt for series with 3+ implementation prompts | Plan a dedicated Phase Closure Prompt — it's the counterpart to Prompt 0 |
| Dependent prompt doesn't read IMPLEMENTATION_LOG.md or DECISIONS.md from prior steps | Add Read instructions for both logs to every prompt that depends on prior steps |
| No "Prior steps context" block in dependent prompts | Add a block calling out specific things to verify: files modified, fallback paths taken, interfaces that may differ |
| Bundling multiple prompt steps into one TodoWrite item | Each distinct step in a prompt gets its own todo item. Bundling hides incomplete work. |
| Phase B prompt doesn't clean up Phase A's `@public` forward-export tags | Stale `@public` tags accumulate — dead code checker permanently ignores exports that may become truly dead |

> **Add your own project-specific anti-patterns here as you discover them during
> Phase Closure reviews.**

---

## 18. Testing Anti-Patterns

These patterns cause subtle bugs — infinite re-renders, OOM crashes, or flaky tests.
Prompts that include test writing should reference these.

### Framework-Agnostic

| Anti-Pattern | Why It's Dangerous | Correct Approach |
|---|---|---|
| Tests that pass but leave hanging timers/promises | Can cause OOM in CI when many test files run sequentially | Always restore real timers and clean up after each test |
| Skipping end-to-end deploy verification | Deploy-breaking bugs are only caught after push, causing fix-commit cycles | Plans with deploy steps must include a post-deploy smoke test |
| Mock returns new object/function references on every call | Frameworks with dependency tracking (React, Vue, Svelte) re-run effects on reference changes → infinite loops → OOM | Always return stable references from mocks — define objects/functions once, reuse across calls |
| Tests depend on execution order | Passes locally, fails in CI (parallel runners, different file order) | Each test must be fully independent. Use setup/teardown to reset state |
| Tests mock implementation details instead of behavior | Tests break on refactors that don't change behavior — high maintenance cost | Test observable behavior (inputs → outputs, DOM changes, API calls), not internal wiring |

### React + Vitest Specific

These apply when your project uses React and Vitest. Skip if using a different framework.

| Anti-Pattern | Why It's Dangerous | Correct Approach |
|---|---|---|
| Inline object literals in hook mocks (e.g., `useAuth: () => ({ user: {}, token: "..." })`) | Creates new references every render → `useEffect`/`useCallback` dependency arrays trigger infinite loops (OOM) | Define stable references inside the mock factory: `const user = {}; return { useAuth: () => ({ user }) };` |
| `vi.fn()` inside a mock return | Same referential instability — `vi.fn()` creates a new function on each call | Define `vi.fn()` as a stable const, not inline in the return |
| Using `getByText()` when multiple elements match | "Found multiple elements" error or wrong element selected | Use `getByTestId()` or scope with `within()` |
| Missing `act()` wrapper around async state updates | React warns about state updates outside `act()`, results may be stale | Wrap timer advances and async operations in `await act(async () => { ... })` |
| Referencing `const` variables inside `vi.mock()` factory without `vi.hoisted()` | `vi.mock()` is hoisted above all imports. `const` variables hit the Temporal Dead Zone — mock factory runs before `const` is initialized | Use `vi.hoisted()` to define stable references that survive hoisting |
| Inline object literals as hook arguments in `renderHook` | Each render creates a new reference → infinite loop → OOM if the hook uses the argument in a dependency array | Extract arguments to stable `const` references outside the `renderHook` callback |

> **General principle:** Any mock that returns objects consumed by reactive dependency
> tracking (React's `useEffect`, Vue's `watch`, Svelte's `$:`) MUST return **stable references**.

> **Add your own project-specific testing anti-patterns here as you discover them.**
