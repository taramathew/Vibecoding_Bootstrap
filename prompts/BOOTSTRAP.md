# Bootstrap Prompt

Paste this into a fresh Claude Code session to set up a new project with the Vibecoding Bootstrap workflow.

## Before You Start

1. Clone the bootstrap repo (if not already):
   ```bash
   git clone https://github.com/your-org/Vibecoding_Bootstrap.git
   ```
2. Navigate to your target project directory (empty or existing)
3. Have your project's design document ready (path to it).
   If you don't have one yet, use `templates/DESIGN_DOC.template.md` from the bootstrap repo as a starting point — fill it in before running this prompt.
4. Open Claude Code: `claude`

---

## The Prompt

```
I'm setting up a new project and need you to bootstrap it with a battle-tested development workflow.

## 1. Read the Bootstrap Package

The Vibecoding Bootstrap repo is cloned at: [PASTE PATH TO CLONED REPO]
(e.g., ~/repos/Vibecoding_Bootstrap)

Read these files from the bootstrap repo:
- README.md — understand what this package provides
- templates/CLAUDE.md.template — the CLAUDE.md skeleton
- templates/logs/IMPLEMENTATION_LOG.md — log entry format
- templates/logs/DECISIONS.md — ADR format
- templates/FEATURE_BACKLOG.md — backlog format
- templates/DESIGN_DOC.template.md — design doc format (for reference)

## 2. Read the Design Document

My project's design document is at: [PASTE PATH TO YOUR DESIGN DOC]

Read it to understand:
- What we're building (scope, goals, architecture)
- Tech stack choices
- Data model / schema design
- Any specific constraints or principles

If the path doesn't work, ask me for the correct one.

## 3. Initialize the Project

- git init (if not already a repo)
- Initialize with the appropriate package manager (ask me which one if unclear from the design doc)
- Create .gitignore (node_modules, dist, .env, *.local, .DS_Store)
- Create .env.example with placeholder environment variables from the design doc

## 4. Install Core Dependencies

Based on the design doc's tech stack:
- TypeScript (strict mode)
- Only what's needed for the first milestone — do NOT over-install

If you think any dependency choice in the design doc should be different, say so BEFORE installing. I want your honest opinion. If anything seems over-engineered for v1, flag it.

## 5. Configure TypeScript

- strict mode, no any
- ES module output
- Path aliases if helpful but keep it simple

## 6. Create Folder Structure

Create the folder structure based on the design doc. At minimum, include:

```
[project-name]/
  .docs/
    solutions/       # Design docs (copy design doc here)
    architecture/    # Architecture module docs (create as subsystems are built)
    logs/            # Implementation log + DECISIONS.md
    plans/           # Implementation plans (created by /create-plan)
    prompts/         # Executable prompts (created by /create-prompts)
    guides/          # How-to guides (created as needed)
  src/               # Source code (structure per design doc)
  tests/             # Test files
```

Adapt the `src/` structure to match the design doc's architecture. Ask me if unclear.

## 7. Set Up CLAUDE.md

Copy the CLAUDE.md template from the bootstrap repo (templates/CLAUDE.md.template) to the project root.

Customize it for THIS project:
- Fill in the project name
- Set the foundational principles (adjust priority order if needed)
- Point architecture references to the actual design doc and architecture paths
- Fill in the tech stack from the design doc
- Add project-specific critical rules from the design doc (security, data integrity, etc.)
- Fill in the quality gate commands (based on the tools installed in step 4)
- Set the merge strategy
- List deployment targets if applicable

The template has comments showing what to fill in. Remove the comments after filling in.

## 8. Copy the Development Workflow (Claude Code plugins + rules)

Copy the `process/` directory from the bootstrap repo into `.claude/`:

```bash
cp -r [BOOTSTRAP_REPO_PATH]/process/rules/ .claude/rules/
cp -r [BOOTSTRAP_REPO_PATH]/process/plugins/ .claude/plugins/
```

Then customize these project-specific parts:
1. **.claude/plugins/.claude-plugin/marketplace.json** — Update `name` and `owner.name`
2. **.claude/rules/plan-workflow.md §6 (Which Module to Read)** — Update rows to match this project's architecture docs
3. **.claude/rules/plan-workflow.md §13 (Quality Gates)** — Add this project's quality gate commands (must match CLAUDE.md)

Do NOT regenerate these files from scratch — they encode hard-won process lessons.

## 9. Seed Documentation Files

Copy the log templates from the bootstrap repo into the project:
- templates/logs/IMPLEMENTATION_LOG.md → .docs/logs/IMPLEMENTATION_LOG.md
- templates/logs/DECISIONS.md → .docs/logs/DECISIONS.md
- templates/FEATURE_BACKLOG.md → .docs/FEATURE_BACKLOG.md

Copy the design document into the project:
- [DESIGN_DOC_PATH] → .docs/solutions/[design-doc-name].md

Create empty placeholder files for architecture docs (populated as subsystems are built):
- .docs/architecture/OVERVIEW.md — with a header "# Architecture Overview" and a note to fill in during first implementation phase

## 10. Create the First Milestone (from design doc)

Read the design doc for the recommended starting point / first milestone.
Create the initial code artifacts (schema files, folder structure, configuration).

If the design doc doesn't specify a first milestone, ask me what to build first.

Seed the feature backlog (.docs/FEATURE_BACKLOG.md) with at least the first milestone as row #001.

## 11. Add Package Scripts

Add to package.json (adapt to your package manager):
- dev, build, typecheck, lint, test
- Any other scripts from the design doc (e.g., db:generate, db:migrate)

## 12. Create First Implementation Log Entry

Append to `.docs/logs/IMPLEMENTATION_LOG.md` documenting:
- What was set up
- Dependencies chosen (and why)
- Any decisions made during bootstrap
- Verification results

## 13. Initial Commit

Stage everything and commit:
  git add -A
  git commit -m "chore: project bootstrap — structure, config, first milestone"

## 14. Cleanup

The bootstrap repo ([BOOTSTRAP_REPO_PATH]) is no longer needed for day-to-day work.
You can keep it around to run prompts/REVIEW.md later (to update the process files
with new learnings), or delete it if you prefer:

```bash
rm -rf [BOOTSTRAP_REPO_PATH]
```

The project now has its own copy of all process files in `.claude/` — the bootstrap
repo is not a runtime dependency.

---

IMPORTANT NOTES:
- If you think any choice in the design doc should be different, say so BEFORE implementing. I want your honest opinion.
- If anything seems over-engineered for v1, flag it. We can simplify.
- Ask me questions if anything is unclear. Don't assume.
- After this bootstrap, you'll have /create-plan, /create-prompts, /code-review, and /optimise-docs available as slash commands.
```
