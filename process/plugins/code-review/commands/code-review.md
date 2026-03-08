---
allowed-tools: Bash(gh issue view:*), Bash(gh search:*), Bash(gh issue list:*), Bash(gh pr comment:*), Bash(gh pr diff:*), Bash(gh pr view:*), Bash(gh pr list:*), mcp__github_inline_comment__create_inline_comment
description: Code review a pull request
---

You are a senior code reviewer. Read CLAUDE.md for the project description, tech stack, and critical rules.

Your job is to review a pull request for correctness, security, and adherence to CLAUDE.md rules. You are thorough but ONLY flag high-signal issues. False positives erode trust and waste reviewer time.

## Step 1: Pre-flight check

Launch a **haiku** subagent. The subagent should check:
1. Is this PR closed? If so, stop.
2. Is this PR a draft? If so, stop.
3. Is this PR trivial (only docs, only whitespace, only lockfile)? If so, stop.
4. Has Claude already reviewed this PR (check for existing review comments from Claude)? If so, stop.

Note: PRs authored by Claude should still be reviewed.

If any check is true, explain why and stop.

## Step 1.5: Local quality gate

Run your project's quality gate commands (as defined in CLAUDE.md) to verify the branch
is clean before reviewing.

- If **lint or typecheck fails**: warn the user — "Lint/typecheck failures found. Fix
  these before code review so findings focus on semantic issues, not linter-catchable
  problems." Then stop.
- If **tests fail**: warn but continue — test failures may be intentional (e.g., TDD)
  or pre-existing.
- If all pass: proceed to Step 2.

## Step 2: Gather CLAUDE.md files

Launch a **haiku** subagent. Return a list of file paths of all CLAUDE.md files that are relevant to the review. This includes the root CLAUDE.md and any CLAUDE.md files in directories that contain modified files.

## Step 3: Summarize the PR

Launch a **sonnet** subagent. View the PR and return a summary of its changes.

## Step 4: Parallel review

Launch **4 agents in parallel**:

### Agent 1 (sonnet): CLAUDE.md Compliance — Pass A
Audit changes for CLAUDE.md compliance. Only consider CLAUDE.md files that share a file path with the modified file or its parents. Check all rules in CLAUDE.md's Critical Rules section, including:
- Code quality rules (types, linting, shared code patterns)
- Security rules (auth, injection, data exposure)
- **Dead code**: If the PR replaces a file/component (deletes imports, changes names, migrates logic), verify the old source was also deleted or is still used elsewhere. Flag with confidence 80+ if a PR clearly replaces something but leaves the old file with no remaining importers.
- **Platform targets**: If CLAUDE.md defines supported deployment targets and the PR introduces platform-specific APIs, verify compatibility across all targets.

### Agent 2 (sonnet): CLAUDE.md Compliance — Pass B
Same as Agent 1, independently. Two passes catch more violations.

### Agent 3 (opus): Bug Detection
Scan the diff for obvious bugs. Only flag bugs that are clearly wrong — code that will:
- Fail to compile/parse (syntax errors, type errors, missing imports)
- Definitely produce wrong results regardless of inputs
- Cause runtime crashes (null dereference, undefined access on non-optional)

Do NOT flag: potential issues that depend on inputs, performance concerns, stylistic preferences.

### Agent 4 (opus): Security & Logic
Look for problems in introduced code:
- Security vulnerabilities (injection, XSS, auth bypass, secret exposure)
- Incorrect business logic (wrong conditions, off-by-one, race conditions)
- Missing authorization checks in database queries or API endpoints

Do NOT flag: pre-existing issues, code style, general suggestions.

---

**Critical filtering rule for ALL agents:** Only flag HIGH SIGNAL issues:
- Code that will fail to compile/parse
- Code that will definitely produce wrong results
- Clear, unambiguous CLAUDE.md violations where you can quote the exact rule
- Security vulnerabilities with concrete exploit path

Do NOT flag:
- Code style preferences
- Potential issues that depend on specific inputs
- Subjective suggestions
- Linter-catchable issues
- Pre-existing issues not introduced by this PR
- General code quality (unless specifically required by CLAUDE.md)
- Premature abstractions — don't recommend helpers, wrappers, or extra branching for code that works correctly today
- Missing library calls that the library's wrapper/provider already handles internally

**Verification rule (MANDATORY before reporting any finding):**
Before flagging an issue, the agent MUST verify its assumption is correct — not just pattern-match. Check the actual API behavior, the library docs, or whether a framework wrapper already handles the concern. Static pattern matching alone is insufficient. Findings that fail verification must be suppressed.

## Step 5: Validate findings

For each issue flagged by agents 3 and 4, launch parallel subagents to validate:
- **Opus** subagents for bug/logic issues
- **Sonnet** subagents for CLAUDE.md violations

Each validator should independently confirm the issue is real, with confidence 0-100:
- 0-25: False positive (suppress)
- 26-50: Minor, limited impact (suppress)
- 51-79: Real but uncertain (suppress)
- **80-100: Report these only**

## Step 6: Filter

Remove any issues that failed validation (confidence < 80).

## Step 7: Output to terminal

Print a structured summary:
- PR title and number
- Number of files changed
- Issues found (grouped by severity: Critical, Important)
- For each issue: file, line, description, confidence score, relevant CLAUDE.md rule (if applicable)

If `--comment` was NOT passed as an argument, stop here.
If `--comment` was passed but no issues found, post a summary comment via `gh pr comment` and stop.

## Step 8: Plan comments

Create an internal list of all comments to post (for self-review before posting).

## Step 9: Post inline comments

For each issue, use `mcp__github_inline_comment__create_inline_comment` to post an inline comment. Each comment should include:
- Brief description of the issue
- For small self-contained fixes (< 6 lines): include a suggestion block
- For larger fixes: describe the issue without a suggestion block
- Only ONE comment per unique issue
- Reference the specific CLAUDE.md rule being violated (if applicable)

GitHub links should use full SHA with line ranges: `https://github.com/owner/repo/blob/[full-sha]/path/file.ext#L[start]-L[end]`
