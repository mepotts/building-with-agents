# Verification tiers

Generation is cheap; verification is the bottleneck. Every unit of work runs plan, execute, verify.

- **Plan.** A scoped spec with a definition of done: acceptance criteria and the end-to-end check that proves them, written before code.
- **Execute.** The implementer builds inside scope, isolated (own worktree, branch and data).
- **Verify.** An independent adversarial review plus objective gates, with evidence rather than assertion. A green typecheck is not a shippable artifact; verify the real deploy or run target.

## The tiers

Match depth to risk. When a change spans tiers, the highest applies.

| Tier | Change | Required before merge | Human sign-off |
|---|---|---|---|
| 0 | Docs, copy, non-behavioral | Self-check; build and typecheck | Spot-check |
| 1 | Feature code; no schema, no production data | One independent adversarial review; the lint gate; tests that ran (not skipped) | Integrator merges |
| 2 | Schema, persistent, shared or client-shipped state | Tier 1, plus a forward and rollback round trip on production-shaped data, a rehearsal against a copy of production, a committed rollback | Explicit yes before production |
| 3 | Production data writes, destructive actions, publishing, spending | Tier 2, plus the exact command or diff, a logged read-only count of what it touches, a known recovery path | Per-change approval by the human, never delegated |
| 4 | Security surface: auth, uploads, callbacks, parsing, permissions, privacy | The change's own tier, plus a review briefed with a threat model; feeds a periodic full audit and a whole-branch pass after merges | Security review |

Add the top-risk row your domain has. A library's public-API break is Tier 3. For ML, a change that affects evaluation gates on a metric threshold on held-out data.

## What "independent" means

A separate agent with a fresh context, or a human, briefed to refute. It reads the diff, the assertions and the direct artifacts, and re-runs the checks itself. It is not independent if it reads the implementer's summary, casts a second vote on that summary, or is a different model looking at the same summary. Give reviewers enough capability for the risk.

## Reviewer brief (template)

- The outcome under review; checkout path, branch and revision; the files it may touch.
- "Try to refute this change. Report what you could not verify."
- Re-run the acceptance checks. Read the assertions (a matching test name is not proof). Run the red-green proof. Look beside the fix for the same defect shape.
- Output: a verdict (pass, pass with follow-ups, fail), reproduction steps for each finding, and what was not covered.
- If you are pinned to one branch, say so. A claim that depends on another branch is reported as unsupported, not guessed.

## Three proofs to demand

1. **Red-green.** Revert the fix; the new test must fail. Restore it; the test must pass. Say which cases actually discriminate: an at-most assertion cannot prove a bound got tighter.
2. **Hollow-gate.** Break the thing the gate exists to catch and confirm it goes red. Tests generated from the filesystem or a manifest can vanish instead of failing, so drive cases from live discovery plus a committed snapshot, and never regenerate the snapshot to silence a failure.
3. **Fault injection for graders.** An eval or model judge earns trust only if it fails when the behavior it grades is deliberately broken.

Treat "skipped" and "0 tests found" as failures until shown otherwise.

## Whole-branch verification

Per-change reviewers attack their own diff, so regressions live between changes. After a batch merges, verify the merged branch as a whole. For security-sensitive work, re-run the original analysis on the merged result and add a closure check on every high-severity finding.

## Scaling review

- Partition findings, files or screenshots across reviewers with no gaps and no overlaps, and have each recompute hashes.
- Give a high-severity finding two skeptics with different lenses (for example reachability from the entry point, and source correctness against existing mitigations). Drop it only if both refute it. Arbitrate against the source, and settle contradictions by reading code or running the check, not by vote.
- Agent count is not a success metric. Do not audit a copy change like a migration, and do not ship a production-data change on one pass.
