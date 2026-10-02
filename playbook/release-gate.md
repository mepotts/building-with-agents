# Exact-candidate release gate (template)

**Principle:** Readiness belongs to one exact candidate, which is a source revision and the artifacts built from it. "Implemented," "tested," "deployed," "enabled" and "healthy" are separate claims. Any source change after a run voids it. Missing evidence on a new machine means unavailable and never passed. The gate publishes nothing. It produces evidence a person or release process can rely on.

## 1. Acceptance declaration (before coding)

One small file per change: every changed product path (including deletions), the visible outcomes, and for each outcome the named executable checks per platform. Every excluded platform needs a written reason, and a change may not exclude all of them. The runner refuses a change whose files the declaration does not cover.

```json
{
  "id": "<feature-slug>",
  "files": ["<every changed product path>"],
  "criteria": [
    {
      "id": "c1",
      "intent": "<observable outcome, including cancel and persistence>",
      "web": { "tests": [{ "file": "<spec file>", "title": "<exact test title>" }] },
      "device": { "notApplicable": "<why this platform does not apply>" }
    }
  ],
  "pendingCriteria": [{ "id": "p1", "reason": "<why runtime proof is not possible yet>" }]
}
```

A mock or an offline pass never closes a runtime criterion, so it stays pending. New behavior needs assertions for success, cancel, persistence, screen changes, loading, error and permission states. A matching test name is not proof, so the reviewer reads the assertions.

## 2. Steps

Run in order on the frozen candidate. Each must pass exactly once.

| Step | Passes when |
|---|---|
| Lint self-test | Every rule fires on its failing fixture and stays silent on its passing one |
| Lint rules | Zero violations |
| Typecheck, per platform | Zero errors |
| Production build and smoke | Built from the frozen source and smoke-tested against a disposable fixture database, with the artifact hash recorded |
| Browser tests | Every declared check passed once, with nothing skipped, retried, focused or flaky |
| Device or emulator groups | Each group passed on an artifact whose hash equals the installed one |
| Acceptance evidence | Every criterion's named checks appear in the results as passed |
| Source stability | Commit, dirty flag and content hash identical before and after |

The runner can end a run as `checks-passed` at most, never as ready.

**What blocks:**

- Any skip, retry, focused test, expected failure or flaky pass (declared skips must be allow-listed by exact file, title and criterion id)
- Capture-only or single-platform runs
- Stale artifacts
- A source change mid-run
- An image baseline updated to clear a failure

Baseline changes are deliberate, reviewed and committed with the reason.

## 3. Evidence

Each run gets a private directory: a report (source, steps, results, limitations), logs, every screenshot with its SHA-256, and a review template bound to the report's hash. Blocked runs are retained. Never overwrite them or relabel them as passes.

## 4. Independent review

1. Use at least three reviewer agents.
2. Split the screenshots into contiguous partitions with no gaps and no overlaps.
3. Have each reviewer recompute every hash, then grade clipped or overlapping controls, readable text, safe areas, keyboard, and the loading, error, cancel and persistence states.
4. Also report evidence quality: byte-identical captures under different names (the state change may not have been captured), captures taken mid-animation, mislabeled screenshots.
5. Record the reviewer, a verdict, notes with limitations and the image list in the review file, all bound to the report hash.

## 5. Finalize

A separate step re-derives everything from files: source unchanged, raw evidence and screenshot hashes matching, review bound to this report and covering every image. Only then does it print ready, and only for the platforms covered.

| State | Meaning |
|---|---|
| `blocked` | A step failed or the run was invalidated |
| `checks-passed` | Every step passed, but no independent review yet |
| `visual-review-pending` | Review missing, unbound or incomplete |
| `partial` | Not all required platforms ran |
| `ready` | The finalizer verified everything for the covered platforms |

## 6. Shared runtime lease

Shared resources (ports, a fixture database, an emulator) sit behind an exclusive-create lock that names them. Release verifies ownership. Recovery is allowed only when the owner is provably dead on this host. A stale-looking timestamp is not proof. If the lock is held, refuse to start.

## 7. Repair loop

1. Preserve the failing run and classify the failure: product assertion, auth or fixture setup, missing evidence, visual defect, infrastructure.
2. Reproduce a product defect with an assertion before fixing it. Never weaken an assertion to pass.
3. A wait can only delay a failure, never convert one. After several distinct timing races, question the time budget. Reload a page that never appears and do not wait longer.
4. A failure that appears only when a phase runs alone is a real defect until disproven. Do not write it off as "an ordering artifact."
5. Stop after three attempts on the same failure and report the evidence and the blocker.
6. Treat new source as a new candidate and a new run. Do not combine partial or historical runs.

## 8. State the limits

Every report lists what it does not certify: other platforms, store signing, the production network, deployment, live health. A local pass is not permission to publish. Publishing stays a per-change human decision.
