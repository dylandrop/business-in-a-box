# Orchestrator: Discovery, Assessment, Splitting

Detailed phase instructions. Read ONLY when state.phase is "discovery", "assessment", or "splitting".

---

## Discovery

**Skip if** `.claude/audit/workflows.md` exists and `state.workflowCount > 0`. Set `state.phase = "assessment"` and proceed to §Assessment.

Launch one Agent (`subagent_type: "Explore"`, very thorough):

> Map every user-facing workflow in this codebase. Read your detailed instructions at `.claude/commands/audit-templates/discover-workflows.md`. Write output to `.claude/audit/workflows.md`.

On completion: read the file, count workflows, update `state.workflowCount` and `state.phase = "assessment"`.
On failure: record in `state.errors`, retry once. If retry fails, STOP and ask the user.

---

## Assessment

Read `state.iteration` → `I`. Run: `mkdir -p .claude/audit/findings/iter-{I}`

Only launch agents for lenses where `state.assessment.{lens}` is `"pending"` (enables resume).

### Sharding note
If `state.workflowCount > 15`, prepend to each agent prompt: "There are {N} workflows. Process in batches of 10, appending findings after each batch."

### Launch agents

Launch one Agent per pending lens (`subagent_type: "general-purpose"`), all in parallel. Use this parameterized prompt for each, substituting `{LENS}`:

> You are a {LENS} auditor. Iteration: {I}. Read your instructions at `.claude/commands/audit-templates/assess-{LENS}.md`. Read workflows from `.claude/audit/workflows.md`. Write findings to `.claude/audit/findings/iter-{I}/findings-{LENS}.md`. {Only if I > 1: "Prior iteration findings at `.claude/audit/findings/iter-{I-1}/findings-{LENS}.md` — read those to check resolution status."}

Where `{LENS}` is each of: `ux`, `security`, `performance`.

### After each agent completes
Set `state.assessment.{lens} = "complete"` immediately (incremental checkpoint).

### On failure
Set `state.assessment.{lens} = "failed"`, record in `state.errors`, retry once. If retry fails, proceed without that lens and note the gap.

### When all done
Set `state.phase = "splitting"`.

---

## Splitting

**Delegate to an agent** to keep findings content out of orchestrator context.

Launch one Agent (`subagent_type: "general-purpose"`):

> You are a work-splitting coordinator. Read ALL findings from:
> - `.claude/audit/findings/iter-{I}/findings-ux.md`
> - `.claude/audit/findings/iter-{I}/findings-security.md`
> - `.claude/audit/findings/iter-{I}/findings-performance.md`
>
> 1. Collect every finding where Severity is Critical, High, or Medium AND Status is Open. Count them.
> 2. If zero: write `{"noFindings":true}` to `.claude/audit/splits/iter-{I}/manifest.json` and return "No actionable findings."
> 3. Build a file ownership map: `file → [finding IDs that touch it]`.
> 4. Create conflict-free splits where NO two splits share any file:
>    - Sort findings by severity (Critical first).
>    - Assign each to a split owning one of its files, or create a new split.
>    - If a finding's files span two splits, merge those splits.
>    - Target 3-5 splits.
> 5. Run `mkdir -p .claude/audit/splits/iter-{I}` then write each split to `.claude/audit/splits/iter-{I}/split-{N}.md`:
>    ```
>    # Split {N}: {Theme}
>    ## Owned Files (ONLY modify these)
>    - `file.ts`
>    ## Findings to Address
>    ### {ID}: {Title}
>    - Severity / Description / Implementation Plan
>    ## Validation Checklist
>    - [ ] Files compile
>    - [ ] Tests pass
>    ```
> 6. Write manifest to `.claude/audit/splits/iter-{I}/manifest.json`:
>    `{"noFindings":false,"totalSplits":N,"splits":[{"id":N,"theme":"...","fileCount":N,"findingCount":N,"highestSeverity":"..."}],"openCounts":{"critical":N,"high":N,"medium":N}}`
> 7. VERIFY no file appears in multiple splits.

On completion:
1. Read `.claude/audit/splits/iter-{I}/manifest.json`.
2. If `noFindings` is true: set `state.phase = "report"`.
3. Otherwise: update `state.openFindings` from `openCounts`, set `state.execution.totalSplits`, set `state.phase = "execution"`.
