# Verification tiers

Every unit of work runs plan, execute, verify.

1. **Plan.** Write a scoped spec before any code. Its definition of done is the acceptance criteria and the end-to-end check that proves them.
2. **Execute.** The implementer builds inside scope and stays isolated (own worktree, branch and data).
3. **Verify.** Run an independent adversarial review plus objective gates, with evidence for each claim. A green typecheck is not a shippable artifact, so verify the real deploy or run target.

## The tiers

Match depth to risk. When a change spans tiers, the highest applies.

| Tier | Change | Required before merge | Human sign-off |
|---|---|---|---|
| 0 | Docs, copy, non-behavioral | Self-check, build and typecheck | Spot-check |
| 1 | Feature code with no schema and no production data | One independent adversarial review, the lint gate, and tests that ran (not skipped) | Integrator merges |
| 2 | Schema, persistent, shared or client-shipped state | Tier 1, plus a forward and rollback round trip on production-shaped data, a rehearsal against a copy of production, and a committed rollback | Explicit yes before production |
| 3 | Production data writes, destructive actions, publishing, spending | Tier 2, plus the exact command or diff, a logged read-only count of what it touches, and a known recovery path | Per-change approval by the human, never delegated |
| 4 | Security surface: auth, uploads, callbacks, parsing, permissions, privacy | The change's own tier, plus a review briefed with a threat model. Feeds a periodic full audit and a whole-branch pass after merges | Security review |

Add the top-risk row for your domain. A library's public-API break is Tier 3. For ML, a change that affects evaluation is gated on a metric threshold on held-out data.

## What "independent" means

An independent reviewer is a separate agent with a fresh context, or a human, briefed to refute. It reads the diff, the assertions and the direct artifacts, and re-runs the checks itself. It is not independent if it reads the implementer's summary, casts a second vote on that summary, or is a different model looking at the same summary. Give reviewers enough capability for the risk.

## Reviewer brief (template)

- The outcome under review, the checkout path, branch and revision, and the files it may touch.
- "Try to refute this change. Report what you could not verify."
- Re-run the acceptance checks. Read the assertions (a matching test name is not proof). Run the red-green proof. Look beside the fix for the same defect shape.
- Output: a verdict (pass, pass with follow-ups, fail), reproduction steps for each finding, and what was not covered.
- If you are pinned to one branch, say so. A claim that depends on another branch is reported as unsupported and not guessed.

## Three proofs to demand

1. **Red-green.** Revert the fix and the new test must fail. Restore it and the test must pass. Say which cases actually discriminate. An at-most assertion cannot prove a bound got tighter.
2. **Hollow-gate.** Break the thing the gate exists to catch and confirm it goes red. Tests generated from the filesystem or a manifest can vanish instead of failing. Drive cases from live discovery plus a committed snapshot, and never regenerate the snapshot to silence a failure.
3. **Fault injection for graders.** Trust an eval or model judge only if it fails when the behavior it grades is deliberately broken.

Treat "skipped" and "0 tests found" as failures until shown otherwise.

## Whole-branch verification

Per-change reviewers attack their own diff, so regressions live between changes. After a batch merges, verify the merged branch as a whole. For security-sensitive work, re-run the original analysis on the merged result and add a closure check on every high-severity finding.

## Scaling review

1. Partition findings, files or screenshots across reviewers with no gaps and no overlaps, and have each recompute hashes.
2. Give a high-severity finding two skeptics with different lenses (for example reachability from the entry point, and source correctness against existing mitigations). Drop it only if both refute it. Arbitrate against the source. Settle contradictions by reading code or running the check, and never by vote.
3. Agent count is not a success metric. Do not audit a copy change like a migration, and do not ship a production-data change on one pass.
