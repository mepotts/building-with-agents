# Exact-candidate release gate (template)

**Principle.** Readiness belongs to one exact candidate: a source revision and the artifacts built from it. "Implemented," "tested," "deployed," "enabled" and "healthy" are separate claims. Any source change after a run voids it. Missing evidence on a new machine means unavailable, not passed. The gate publishes nothing; it produces evidence a person or release process can rely on.

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

Pending means pending: a mock or an offline pass never closes a runtime criterion. New behavior needs assertions for success, cancel, persistence, navigation, loading, error and permission states. A matching test name is not proof; the reviewer reads the assertions.

## 2. Steps

Run in order on the frozen candidate. Each must pass exactly once.

| Step | Passes when |
|---|---|
| Lint self-test | Every rule fires on its failing fixture and stays silent on its passing one |
| Lint rules | Zero violations |
| Typecheck, per platform | Zero errors |
| Production build and smoke | Built from the frozen source, smoke-tested against a disposable fixture database; artifact hash recorded |
| Browser journeys | Every declared check passed once; nothing skipped, retried, focused or flaky |
| Device or emulator groups | Each group passed on an artifact whose hash equals the installed one |
| Acceptance evidence | Every criterion's named checks appear in the results as passed |
| Source stability | Commit, dirty flag and content hash identical before and after |

The runner can end a run as `checks-passed` at most, never as ready.

**What blocks:** any skip, retry, focused test, expected failure or flaky pass (declared skips must be allow-listed by exact file, title and criterion id); capture-only or single-platform runs; stale artifacts; a source change mid-run; an image baseline updated to clear a failure. Baseline changes are deliberate, reviewed and committed with the reason.

## 3. Evidence

Each run gets a private directory: a report (source, steps, results, limitations), logs, every screenshot with its SHA-256, and a review template bound to the report's hash. Blocked runs are retained. Never overwrite them or relabel them as passes.

## 4. Independent review

Use at least three reviewer agents. Split the screenshots into contiguous partitions with no gaps and no overlaps. Each reviewer recomputes every hash, then grades clipped or overlapping controls, readable text, safe areas, keyboard, and the loading, error, cancel and persistence states.

Also report evidence quality: byte-identical captures under different names (the state change may not have been captured), captures taken mid-animation, mislabeled screenshots. The review file records the reviewer, a verdict, notes with limitations and the image list, all bound to the report hash.

## 5. Finalize

A separate step re-derives everything from files: source unchanged, raw evidence and screenshot hashes matching, review bound to this report and covering every image. Only then does it print ready, and only for the platforms covered.

| State | Meaning |
|---|---|
| `blocked` | A step failed or the run was invalidated |
| `checks-passed` | Every step passed; no independent review yet |
| `visual-review-pending` | Review missing, unbound or incomplete |
| `partial` | Not all required platforms ran |
| `ready` | The finalizer verified everything for the covered platforms |

## 6. Shared runtime lease

Shared resources (ports, a fixture database, an emulator) sit behind an exclusive-create lock that names them. Release verifies ownership. Recovery is allowed only when the owner is provably dead on this host; a stale-looking timestamp is not proof. Refuse to start rather than share.

## 7. Repair loop

- Preserve the failing run and classify the failure: product assertion, auth or fixture setup, missing evidence, visual defect, infrastructure.
- Reproduce a product defect with an assertion before fixing it. Never weaken an assertion to pass.
- A wait can only delay a failure, never convert one. After several distinct timing races, question the time budget. A page that never appears needs a reload, not a longer wait.
- A failure that appears only when a phase runs alone is a real defect until disproven, not "an ordering artifact."
- Stop after three attempts on the same failure and report the evidence and the blocker.
- New source is a new candidate and a new run. Do not combine partial or historical runs.

## 8. State the limits

Every report lists what it does not certify: other platforms, store signing, the production network, deployment, live health. A local pass is not permission to publish; publishing stays a per-change human decision.
