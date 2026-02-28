# Orchestrator: Execution, Merging, Loop Check, Report

Detailed phase instructions. Read ONLY when state.phase is "execution", "merging", "loop-check", or "report".

---

## Execution

Read `state.iteration` → `I` and `state.execution`.

**Resume logic**: Only launch agents for splits NOT in `execution.completed`.

For each pending split N, launch Agent (`subagent_type: "general-purpose"`, `isolation: "worktree"`) in parallel:

> Software engineer. Iteration: {I}, Split: {N}. Read instructions at `.claude/commands/audit-templates/execute-split.md`. Assignment at `.claude/audit/splits/iter-{I}/split-{N}.md`. Write result to `.claude/audit/splits/iter-{I}/split-{N}-result.md`.

As each completes:
1. Read `split-{N}-result.md` for branch name.
2. Update state: add N to `execution.completed`, record `execution.branches[N] = {branch, worktreePath}`.

On failure: record in `state.errors`, retry once. Skip on second failure.
After all done: set `state.phase = "merging"`.

---

## Merging

Read `state.execution.branches` and `state.execution.merged`.

### Recovery (if branch info lost after context flush)
If `branches` is empty but `completed` is not: scan `split-*-result.md` files for branch names. Fallback: `git worktree list` and `git branch --list 'audit-*'`.

### Sequential merge
For each unmerged branch:
1. Check already merged: `git merge-base --is-ancestor {branch} HEAD` → skip if yes.
2. Merge: `git merge {branch} --no-edit`
3. On conflict: record in `state.errors`, ask user.
4. Cleanup: `git worktree remove {path} 2>/dev/null; git branch -d {branch} 2>/dev/null`
5. Checkpoint: add to `state.execution.merged` immediately.

### Post-merge verification
Run full build + full test suite to catch cross-split interface breakage. Fix and commit if needed: `audit: iteration {I} - post-merge integration fixes`

### Deferred dependencies
Check `split-*-result.md` files for deferred packages. If any: install all at once, rebuild, retest, commit: `audit: iteration {I} - add deferred dependencies`

### Commit state
`git add .claude/audit/ && git commit -m "audit: complete iteration {I}"`
Set `state.phase = "loop-check"`.

---

## Loop Check

Read `state.iteration` → `I`.

1. If `I >= state.maxIterations`: set `state.phase = "report"`. Go to §Report.
2. Else: set `state.iteration = I + 1`, reset `state.assessment` to all `"pending"`, clear `state.execution` and `state.openFindings`, set `state.phase = "assessment"`.
3. Go back to the assessment phase (read `orchestrator-discover-and-assess.md` §Assessment).

---

## Report

Read all findings files across all iterations. Write `.claude/audit/REPORT.md`:

```markdown
# Production Readiness Audit Report
_Completed: {date} | Iterations: {N}_

## Executive Summary
{2-3 sentences}

## Findings Summary
| Category    | Critical | High | Medium | Low | Total |
|-------------|----------|------|--------|-----|-------|
| UX          |          |      |        |     |       |
| Security    |          |      |        |     |       |
| Performance |          |      |        |     |       |

## Resolved Findings
{ID, title, iteration fixed}

## Remaining Open Findings
{ID, title, why not addressed}

## Errors Encountered
{from state.errors}

## Files Modified
{from git log}
```

Set `state.phase = "complete"`. Present the report to the user.
