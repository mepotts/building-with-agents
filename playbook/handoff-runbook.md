# Handoff and resume runbook

**Goal.** Any agent, from any vendor, can resume the work from repository state alone. Handoffs will happen: usage limits end sessions mid-task, machines restart, and you will switch tools.

## Before you stop: persist

- The objective, acceptance criteria, exact checkout, branch and source revision.
- Completed commits; uncommitted files and who owns them.
- Commands run and verified results, with evidence paths, hashes and platform limits.
- Failed approaches with diagnosed causes. Label hypotheses as hypotheses.
- The next concrete action, blockers, pending approvals and shared-resource ownership.
- Any obligation that outlives local completion (observing the release, recovery), who carries it and what triggers it. A local pass does not close live health.
- If a usage limit caused the stop, the time it resets.

## The checkpoint file

A small private file outside version control that points at the canonical work record. It is neither a permanent lock nor a second task queue.

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
2. **Confirm the other session actually stopped.** Check processes, containers, lock files, run directories and the last record of the other tool's transcript. A stale timestamp does not prove the owner is gone; a recorded run-ended event does. When unsure, treat the lease as live.
3. **Reconstruct from the most durable source first:** worktrees (ahead, behind, dirty), the checkpoint's next action, the work record and evidence, the newest run directories, and only then transcripts. Transcripts fill gaps; they do not outrank durable records. Read the head of a transcript for identity and the tail for the last records.
4. **Do not rerun a completed expensive gate** against an unchanged frozen candidate. Confirm the candidate and its comparison base, and reuse proof for that exact candidate.
5. **Keep one integration coordinator at a time.** Approval boundaries (no push, no merge to main, no production change, no publishing) are unchanged whichever vendor coordinates.
6. **Before handing back,** persist state, then release shared runtime or lease it explicitly with owner and ports recorded. Never hold it silently.

## Parallel lanes

- **Isolation.** One worktree per lane, with its own data container and port range. A device, emulator or acceptance database belongs to one lane at a time.
- **Writing.** Write only in your worktree and your lane file. Stage with explicit pathspecs, never "add all." No bare stash. Never rebase a shared branch.
- **One writer per file** across lanes. Shared files belong to the coordinator; message it before touching the runner or shared configuration.
- **Merge protocol.** A lane reports branch, head, files changed, commands and results, reviewer and verdict, the criteria it moves and its limitations. The coordinator merges with a merge commit, re-runs the focused checks and announces the new head.
- **Freeze.** The coordinator announces a freeze at a named revision. Afterwards only gate repairs it routes may land. The frozen candidate is what is gated, built and evaluated.
- **Cross-review.** A different lane reviews security-sensitive packages. Findings go to the author, not into shared files.

## Knowledge layer

Keep a short entry file, a project map and typed records (intent, state, decision, runbook). Each record lists its source dependencies with content hashes, line endings normalized. A checker fails on broken links and on a changed dependency whose record did not change, comparing against the real merge base. A stale record is a request to inspect the source, not permission to refresh the hash; a fresh hash cannot prove the prose is right. Refresh hashes only in a clean tree you own.

## Fresh-agent exercise

After a material change to workflow or knowledge layout, start a strong agent with only the checkout path and ask twelve questions, requiring citations. Cover what is being built and who decides, what agents may do without approval, what was last verified and whether that proves HEAD, how to recover a failed run, what remains unproven, and how to reclaim work from an agent that vanished.

Score three checks per answer: a correct conclusion, a current citation and the relevant limitation. Every safety and readiness question must pass. Record the score, commit, model and evidence access, and vary the questions each time. A pass shows retrieval works; it does not show months of autonomous maintenance.

## Restarts, limits and duplicates

- A restart can rename every session. Address peers by stable session id, and on return say "I was X, I am now Y, this is my lane and head."
- A killed gate leaves its lock behind. The lock belongs to the coordinator's lease; never clear another lane's lock.
- Run long workflows from a resumable journal so a usage-limit death does not re-spend finished work. On resume, verify only what is missing.
- Never message a running subagent to steer it; that can spawn a duplicate acting in the same worktree. Steer through the spawn prompt, and state the working directory and first action in it.
