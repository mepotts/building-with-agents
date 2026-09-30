# Working with AI coding agents: an operating playbook

A method for one person, or a small team, to direct AI coding agents and ship software that holds up. The agents write the code. The human sets direction, specifies the work, designs the verification that decides what ships, and owns the irreversible decisions.

It is distilled from a single product built over about 5.5 months and 100+ sprints with two agent vendors, then generalized. It has not been validated on large teams. Numbers in these files illustrate scale; they are not thresholds to copy. Every rule in it came from a failure that actually happened, so your list will differ from mine.

## The premise

Generation is cheap; correctness is scarce. The method therefore spends its effort in five places:

1. **Specify before code.** A scoped spec with a definition of done comes first.
2. **Isolate each agent.** Its own worktree, its own data, its own ports.
3. **Verify independently.** A separate reviewer told to refute, plus objective gates.
4. **Gate exact candidates.** Readiness belongs to one source revision and its built artifacts.
5. **Turn failures into mechanical rules.** A lint rule, a check, a lock. Not "be more careful."

## What is in this folder

| File | Use it for |
|---|---|
| [operating-rules.md](operating-rules.md) | The non-negotiables and why they exist; roles; model routing; context hygiene |
| [verification-tiers.md](verification-tiers.md) | Risk tiers, required checks and reviewers, how to brief an adversarial reviewer |
| [release-gate.md](release-gate.md) | A template for an exact-candidate gate with independent evidence review |
| [handoff-runbook.md](handoff-runbook.md) | Making work resumable across sessions, agents and vendors; parallel lanes |
| [failure-catalog.md](failure-catalog.md) | Common agent failure modes and the guardrail for each |

## When to use it

Use it for anything that touches real data, real people, money or secrets, and for any setup where several agents edit one codebase. Skip most of it for a throwaway spike, but keep one rule: never wire a spike to production data or credentials. If a spike graduates, it enters the full harness.

Adapt the specifics to your project type:

- **Research or data work.** "Irreversible" includes overwriting source data or experiment logs and large compute spend. The gate is a metric on held-out data, not a build; "the pipeline ran" is not "the result is valid."
- **Library or SDK.** The public API is the contract. Publishing is irreversible, so treat it as the top tier.
- **Mobile or client-shipped state.** There is no rollback after ship. Make changes forward-only and version-tolerant, and keep a server-side kill switch.
- **Team with agents.** Agents open pull requests and respect branch protection and code owners. Some review classes are not delegable.

## Suggested adoption order

1. Put the [operating rules](operating-rules.md) and the tier table in your agents' instruction file. Keep that file short.
2. Start a failure log. Each time an agent goes wrong, add one line to the [catalog](failure-catalog.md) and, where you can, one mechanical guardrail.
3. Add a gate ([template](release-gate.md)) as soon as you have something you would ship.
4. Add the [handoff runbook](handoff-runbook.md) the day a second agent or vendor touches the repository.

## Conventions

`<angle brackets>` are placeholders. A **gate** is a deterministic check that passes or fails without a person. A **candidate** is an exact source revision plus the artifacts built from it. A **lane** is one agent's isolated slice of parallel work.
