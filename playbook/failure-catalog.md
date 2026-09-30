# Failure catalog

Common ways agent-built work goes wrong, each with the guardrail that stopped it recurring. Every entry happened at least once. Start your own catalog the first time an agent surprises you, and prefer guardrails a machine enforces.

## Reporting and verification

| Failure | What it looks like | Guardrail |
|---|---|---|
| False completion | Every parallel stream reports "complete"; some parts are orphaned, unreachable or compiled out of the shipped build. | Verify the shipped artifact and the running system, never the summary. Diff the build against the last one. Check that client calls resolve to real routes. |
| Hollow gate | A suite generated from a directory or manifest goes silent when the thing it enumerates disappears; deleting the target makes the test vanish. | "Break it, confirm red" on every new gate. Drive cases from live discovery plus a committed snapshot. |
| Skip-to-green | Tests that need a database skip when it is absent and the run reports green. | Count run against skipped. Fail on unexpected skips. Require the datastore. |
| Theater assertions | Half a table of cases is unchanged by the fix; an at-most check cannot prove a bound got tighter; a hardcoded file list. | Reviewers read assertions. Compare strictly at the sizes that discriminate. Read the directory instead of listing it. |
| Guard dead at small size | A rule mixes an absolute floor with a fraction, so the floor exceeds the population and the guard never fires. The same shape survives next door. | Evaluate guards at the smallest real population. After fixing one, search for its twin. |
| Wrong measurement grain | An ad hoc query measures at a different grain than the code (the newest snapshot instead of one per kind) and reports a catastrophe that is not real. | Find the system's own partition and identity level first. Measure before escalating, and verify the measurement. |
| Confident wrong diagnosis | An agent dismisses a real failure as an ordering artifact, or reports "already fixed" from one branch. | Treat it as a defect until disproven. Check ancestry between the fix and the audited revision before believing "stale" or "still open." |
| Remembered counts | A recap repeats an old number or calls new debt "pre-existing." | Re-run the check. Attribute by diff, not memory. |
| Verifying the wrong path | A change is checked through a command the real pipeline never runs, so a break in the real path reaches the gate. | Verify through the exact command the gate runs. Reproduce the defect with an assertion on that path first. |

## Environment and data

| Failure | What it looks like | Guardrail |
|---|---|---|
| Fix on momentum | A production data change is chosen over a code fix because it is one command away. | Per-change approval. Prefer code. Scripts that can reach production refuse by default. Log a read-only count before any write. |
| Worktree inherits production | An environment file copied into an agent's workspace points at production. | Never copy environment files. Set the data target explicitly. Guard throwaway databases by host, port and name. Verify the target by row count. |
| Stale checkout | The main checkout sits on an old branch, so agents answer confidently from stale files. | When an agent is confidently wrong about repository state, check the branch first. Keep the primary checkout on main. Back registries and counters with a lint that fails loudly. |

## Coordination

| Failure | What it looks like | Guardrail |
|---|---|---|
| Wrong-checkout writes | Parallel agents' commits or file writes land in the main checkout because the shell's directory drifted. | Every git command names its worktree. Write paths are worktree-prefixed. Check both checkouts after each write. |
| Commits over a peer's work | A coordinator commits in a shared checkout while a peer has uncommitted files; a helper hashes the half-written files. | One integration coordinator. Commit in a shared checkout only if it is clean or every dirty file is yours. Helpers refuse dirty trees. |
| Duplicate subagents | Messaging a running subagent resumes a second copy that acts in the same worktree. | Steer only through the spawn prompt, stating the working directory and first action. |
| Orphans after restart or limit | Sessions renamed, agents dead, locks left behind. | Address peers by stable id. Use a lease with ownership checks. Run long jobs from a resumable journal. |
| Scoping gaps | The coordinator's briefs cover the top findings but not every item, or a plan specifies the wrong convention and the implementer follows it faithfully. | Copy conventions from a real existing file. Give the plan an independent completeness check. Run a delta pass after merging. |
| Regressions between changes | Each reviewer finds its own diff clean, yet the merged branch is broken. | Whole-branch verification after merges. |
| Cross-branch claims | A reviewer pinned to one branch cannot confirm a claim about two branches and correctly marks it unsupported. | Tell pinned reviewers to report the dependency. Re-check from a checkout that sees both before dropping the finding. |
| Never-run harness | A test adapter passes authoring review but fails on first execution: a wrong byte comparison, import-time side effects. | Execute its load path against real source, network blocked, before trusting the review. |
| Skipped audit stage | Under time pressure a parallel run drops the adversarial review, and gaps follow. | Never skip the independent audit because the implementers are confident. |

## Cost and process

| Failure | What it looks like | Guardrail |
|---|---|---|
| Default model nobody chose | Every subagent launches on the top tier at high effort. | Audit launches. Route by role and record what actually ran. |
| Timing-race whack-a-mole | Each failed run gets one more wait; the suite slows and stays flaky. | After several distinct races, question the time budget. A wait only delays a failure. Reload once for a half-loaded page. Restart a machine whose state has accumulated. |
