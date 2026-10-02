# Handoff and resume runbook

**Goal:** Any agent, from any vendor, can resume the work from repository state alone. Handoffs will happen because usage limits end sessions mid-task, machines restart and you will switch tools.

## Before you stop

Persist these:

- The objective, acceptance criteria, exact checkout, branch and source revision.
- Completed commits, plus uncommitted files and who owns them.
- Commands run and verified results, with evidence paths, hashes and platform limits.
- Failed approaches with diagnosed causes. Label hypotheses as hypotheses.
- The next concrete action, blockers, pending approvals and shared-resource ownership.
- Any obligation that outlives local completion (observing the release, recovery), who carries it and what triggers it. A local pass does not close live health.
- If a usage limit caused the stop, the time it resets.

## The checkpoint file

The checkpoint is a small private file outside version control that points at the canonical work record. It is not a permanent lock or a second task queue.

```json
{
  "coordinator": "<vendor:session-id>",
  "updatedAt": "<ISO time>",
  "status": "<one line>",
  "checkout": "<absolute path>",
  "branch": "<name>",
  "sourceCommit": "<sha>",
  "frozenCandidate": "<sha or null>",
  "resources": { "ports": "<range per lane>", "databases": "<container per lane>", "device": "<owner>" },
  "lanes": { "<lane>": { "session": "<id>", "worktree": "<path>", "branch": "<name>" } },
  "nextAction": "<one concrete step>",
  "blockers": ["<needs a human decision or approval>"],
  "verification": "<commands, results, evidence paths>",
  "resumeTrigger": "<reset time or event>"
}
```

## Resume, in order

1. **Read the live checkpoint.** Copy it to a dated sibling first so the previous coordinator's version survives, then write your own identity and plan into the live file.
2. **Confirm the other session actually stopped.** Check processes, containers, lock files, run directories and the last record of the other tool's transcript. A recorded run-ended event proves the owner is gone. A stale timestamp does not. When unsure, treat the lease as live.
3. **Reconstruct from the most durable source first:** worktrees (ahead, behind, dirty), the checkpoint's next action, the work record and evidence, the newest run directories, and only then transcripts. Transcripts fill gaps and never outrank durable records. Read the head of a transcript for identity and the tail for the last records.
4. **Do not rerun a completed expensive gate** against an unchanged frozen candidate. Confirm the candidate and its comparison base, and reuse proof for that exact candidate.
5. **Keep one integration coordinator at a time.** Approval boundaries (no push, no merge to main, no production change, no publishing) are unchanged whichever vendor coordinates.
6. **Before handing back,** persist state, then release shared runtime or lease it explicitly with owner and ports recorded. Never hold it silently.

## Parallel lanes

1. **Isolation.** One worktree per lane, with its own data container and port range. A device, emulator or acceptance database belongs to one lane at a time.
2. **Writing.** Write only in your worktree and your lane file. Stage with explicit pathspecs, never "add all." No bare stash. Never rebase a shared branch.
3. **One writer per file** across lanes. Shared files belong to the coordinator, so message it before touching the runner or shared configuration.
4. **Merge protocol.** A lane reports branch, head, files changed, commands and results, reviewer and verdict, the criteria it moves and its limitations. The coordinator merges with a merge commit, re-runs the focused checks and announces the new head.
5. **Freeze.** The coordinator announces a freeze at a named revision. Afterwards only gate repairs it routes may land. The frozen candidate is what is gated, built and evaluated.
6. **Cross-review.** A different lane reviews security-sensitive packages. Findings go to the author and stay out of shared files.

## Knowledge layer

1. Keep a short entry file, a project map and typed records (intent, state, decision, runbook).
2. List each record's source dependencies with content hashes, with line endings normalized.
3. Run a checker that fails on broken links and on a changed dependency whose record did not change, comparing against the real merge base.
4. When a record is stale, inspect the source before touching the hash, because a fresh hash cannot prove the prose is right.
5. Refresh hashes only in a clean tree you own.

## Fresh-agent exercise

1. After a material change to workflow or knowledge layout, start a strong agent with only the checkout path.
2. Ask twelve questions and require citations. Cover these topics:
   - What is being built and who decides
   - What agents may do without approval
   - What was last verified and whether that proves HEAD
   - How to recover a failed run
   - What remains unproven
   - How to reclaim work from an agent that vanished
3. Score three checks per answer: a correct conclusion, a current citation and the relevant limitation. Every safety and readiness question must pass.
4. Record the score, commit, model and evidence access. Vary the questions each time.

A pass shows retrieval works. It does not show months of autonomous maintenance.

## Restarts, limits and duplicates

1. A restart can rename every session. Address peers by stable session id, and on return say "I was X, I am now Y, this is my lane and head."
2. A killed gate leaves its lock behind. The lock belongs to the coordinator's lease, so never clear another lane's lock.
3. Run long workflows from a resumable journal so a usage-limit death does not re-spend finished work. On resume, verify only what is missing.
4. Never message a running subagent to steer it, because that can spawn a duplicate acting in the same worktree. Steer through the spawn prompt and state the working directory and first action in it.
