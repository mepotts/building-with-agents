# Building with agents

I'm Matthew Potts, a data scientist (11+ years) who builds software and research pipelines by directing AI coding agents: mostly Claude Code and Codex, working in parallel. **The agents write the code.** My job is deciding what to build, specifying it precisely, and designing the verification that decides what ships, because agents are fast, confident, and sometimes wrong.

This repo holds write-ups of that work and the playbook I use. The product code itself is private; I'm happy to walk through any of it on a call.

![MovieCellar: the product, the lanes, the release gate, and a reviewer rejecting agent work](demo/moviecellar-tour.gif)

*A 20-second tour. The full 2½-minute walkthrough is [demo/moviecellar-walkthrough.mp4](demo/moviecellar-walkthrough.mp4).*

## Case studies

| | What it shows |
|---|---|
| [**MovieCellar**](case-studies/moviecellar.md) | A web, iOS, and desktop product plus a vision-model service, built in 5.5 months by a two-vendor agent fleet (now in private beta). It covers the orchestration rules, a release gate built to reject agent work, a two-round multi-agent security audit (process only), and the failures behind each rule. |
| [**Agents auditing agents**](case-studies/agents-auditing-agents.md) | A five-agent adversarial audit of agent-written backtesting code that caught look-ahead bias (Sharpe 3.29, really 1.00) and made the evaluation harness a mandatory gate. |
| [**FlowState**](case-studies/flowstate.md) | An audio-ML research program run under sealed evaluation, with hash-frozen gold labels and predictions scored only after freeze. It includes an experiment where frontier agents improved the median and still failed the safety screen. |

## The playbook

[`playbook/`](playbook/) is the operating method distilled from these projects, written so another team could adopt it:

- [operating rules](playbook/operating-rules.md)
- [verification tiers](playbook/verification-tiers.md)
- [an exact-candidate release gate](playbook/release-gate.md)
- [a handoff runbook](playbook/handoff-runbook.md) for resuming work across agents and vendors
- [a catalog of agent failure modes](playbook/failure-catalog.md), with the guardrail for each

## Public code

- [astronomy](https://github.com/mepotts/astronomy): about 20 open-data research fronts run by agents under standing verification gates.
- [exosat-rv](https://github.com/mepotts/exosat-rv): a radial-velocity reanalysis whose own audit withdrew its headline claims.
- [llm-introspection](https://github.com/mepotts/llm-introspection): a concept-injection introspection experiment on Gemma-2-2B.

## Contact

matthew.e.potts@gmail.com · [linkedin.com/in/matthewpotts](https://linkedin.com/in/matthewpotts)
