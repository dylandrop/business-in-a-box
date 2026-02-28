# Split Execution — Agent Instructions

You are a software engineer implementing production-readiness fixes assigned to your split.

## Inputs

Your prompt specifies:
- **Iteration number** and **split number**
- **Split file path**: Contains your assignment — owned files, findings, implementation plans

Read your split file FIRST before doing anything else.

## Rules

1. **ONLY modify files in the "Owned Files" section.** This is a hard constraint — other agents are modifying other files in parallel. Touching files outside your list WILL cause merge conflicts.

2. If you discover you need to modify a file NOT in your list, **do NOT modify it**. Instead, document the need in your result file as a "deferred item."

3. **Do NOT run package manager install commands** (`npm install`, `yarn add`, `pip install`, `cargo add`, etc.). These modify shared manifest and lock files (`package.json`, `package-lock.json`, `yarn.lock`, `Cargo.lock`, etc.) which are NOT in your owned file list and will cause merge conflicts. If your fix requires a new dependency, document it as a deferred item with the exact package name and version.

4. Implement EVERY finding in your split. Follow the implementation plan for each.

5. Do NOT introduce new features or refactor code beyond what the findings require.

## Process

### Step 1: Read your assignment
Read the split file at the path specified in your prompt. Understand all findings and their implementation plans.

### Step 2: Implement changes
For each finding in your split:
1. Read the target file(s).
2. Implement the fix as described in the implementation plan.
3. Verify the change makes sense in context (don't blindly apply — adapt if the code has changed).

### Step 3: Validate

Discover how to build and test this project:
- Check `package.json` for `build`, `test`, `lint` scripts
- Check for `Makefile`, `Cargo.toml`, `pyproject.toml`, `go.mod`
- Check for test configs: `jest.config.*`, `vitest.config.*`, `playwright.config.*`, `cypress.config.*`
- Check for `e2e/`, `tests/`, `__tests__/`, `spec/` directories

Then:
1. **Build**: Run the project's build step. Fix any compilation errors.
2. **Unit tests**: Run tests scoped to your changed files if possible (e.g., `npx jest path/to/file`). If scoping isn't possible, run the full test suite.
3. **E2E tests**: If the project has E2E tests, run them. Fix failures caused by your changes.
4. **Lint**: If a linter is configured, run it on changed files.

**Port and resource isolation**: If tests require a database, server port, or temp directory, use your split number to avoid collisions with other parallel agents:
- Set `PORT={5000 + split_number}` (e.g., split 2 → PORT=5002)
- Set `TEST_DATABASE_URL` to use a unique DB name: `test_audit_split_{N}`
- Use `/tmp/audit-split-{N}/` for any temp file output

If a test fails due to a pre-existing issue (not your changes), note it in your result file but don't block on it.

### Step 4: Commit

Stage ONLY your owned files: `git add {file1} {file2} ...`

Commit with message: `audit: iteration {I} split {N} - {brief description}`

Do NOT push. The orchestrator will handle merging.

### Step 5: Write result file

**CRITICAL**: Write a result file to `.claude/audit/splits/iter-{I}/split-{N}-result.md`. The orchestrator reads this to recover branch names after context flushes.

Use this EXACT format:
```markdown
# Split {N} Result — Iteration {I}

## Branch
{your current git branch name — run `git branch --show-current` to get it}

## Status
{complete | partial}

## Changes Made
{list each finding ID and what was done}

## Test Results
- Build: pass|fail
- Unit tests: pass|fail ({N} passed, {N} failed)
- E2E tests: pass|fail|skipped
- Lint: pass|fail|skipped

## Deferred Items
{any files you needed to modify but couldn't because they're outside your owned set}

## Issues Encountered
{any problems, blockers, or unexpected situations}
```

## Return to Orchestrator

Your final response message must be ONE LINE only:
`Split {N}: {complete|partial}. {X}/{Y} findings implemented. Branch: {branch_name}.`
Do NOT describe changes in your response — they are in the result file on disk.

## Error Handling

- If a file in your owned list doesn't exist, skip it and note in your result file.
- If the implementation plan doesn't match the current code (code was already changed), adapt the fix to the current state.
- If you can't figure out how to fix a finding, implement what you can and document what's left in your result file.
- Do NOT leave the codebase in a broken state. If your changes break the build, either fix them or revert them.
