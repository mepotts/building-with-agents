# FlowState

**Status:** Private repository. Nothing here has been published or submitted. Claude Code and Codex agents wrote the code. I set the direction, hand-corrected the 1,061 gold onset labels, designed the experiments and gates, and decided what counted as a result.

**Result:** Median syllable-onset error fell from 42 to 18 ms beyond a fitted-offset baseline on 12 a cappella tracks. It measured 21.6 ms on 9 held-out tracks (a single-use, preregistered holdout). These are research numbers on silver-standard labels. They are not production numbers.

**Frontier label refiners:** Rejected. The median improved from 34.9 to 12.9 ms, but a frozen safety screen caught 143 body labels (144 of all 1,061 labels) made worse by more than 25 ms.

## Problem

FlowState measures how rappers place syllables against a beat. Every flow metric depends on per-syllable onset times, so onset accuracy is the main target. The test set is 12 a cappella excerpts (about 2,000 scored syllables) from Mitchell Ohriner's public flowBook corpus. Its phone-level timings were hand-corrected from a forced aligner's output.

## Results

**First honest measurement:** 44 ms median error. Two changes took it to 18 ms. One targeted the vowel onset instead of the consonant attack, and the other fixed an outlier guard that vetoed the right shift on s, sh, and ch words.

**Re-derivation:** I tried to break the number. A 16-agent fleet re-derived it. A fresh GPU recompute regenerated all 12 cache files byte-identically, and two independent scorers agreed. However, the claim shrank.

**Offset baseline:** About half of the drop from 44 to 18 ms was landmark-convention alignment, which is not a detection gain. The defensible claim is against a leave-one-out fitted constant offset (41.9 ms median). The full stack scores 18.0 and wins on 12 of 12 tracks. The result is also in-sample, because the 12 tracks were both development and test.

**Silver labels:** 47.6% of scored onsets were never hand-corrected. Given label noise of about 9 ms, a perfect pipeline would read about 6 ms. I adopted a stop rule: accept a change only at a 2 to 3 ms pooled gain with at least 8 of 12 tracks improved. Seven claims in my own status document had to be corrected or retired.

**Holdout:** 30 more tracks by one artist, untouched. The preregistered sync gate stopped my first attempt, rejecting 29 of 30 tracks and 5 of the 12 known-good ones. The instrument was broken, so the holdout stayed unspent. I recalibrated the gate on the 12 alone, froze the bands and the nine admitted tracks in git, and spent the holdout once. The pooled median was 21.6 ms against a 32 ms bar. The signed median was +2.3 ms (not offset-confounded), and 82.7% of onsets were within 100 ms.

**Scope:** Nine tracks from one artist, admitted by a gate, now development data for good. On the four full-mix songs I hand-corrected, the pipeline's own median error is about 35 ms, and about a quarter of onsets are off by more than 100 ms.

## Governance

The 1,061 hand-corrected onsets are the scarcest asset, so the rules are mechanical.

**Worktrees:** 49, with one writing agent each.

**Gold:** Read-only to agents. Experiments use immutable SHA-256-addressed snapshots, and each run manifest pins the hash. Writes to gold need an owner token checked against a server-side secret, and the guard fails closed if the secret is unset.

**Sealing:** One experiment hash-sealed 552 agent responses before the private gold was joined. Another scored 163 sealed entries by code, and all 510 arm, group, and stratum combinations were recomputed independently.

**Preregistration:** The corrected-rerun harness refuses to run until six owner decisions are committed and its implementation blockers clear.

**Access:** Shared agent settings pre-approve only file read, write, and search. CODEOWNERS routes gold-touching paths to me, and offline CI scans for secrets and checks that the safeguards still exist.

## Frontier agents as label refiners

A frontier OpenAI model, driven through Codex subagents, proposed corrected onsets from waveform and spectrogram images plus numeric energy frames (never audio). The proposals were scored against all 1,061 hand-corrected labels, across four songs and three artists. The screen was declared before scoring. It required at least a 5 ms median gain, no worse mean, 95th-percentile, more-than-100 ms, or more-than-250 ms error rates, and no song median regressing by more than 5 ms.

The median looked like a win, from 34.9 to 12.9 ms across all labels. However, the screen rejected it. The agents acted on 982 of 1,004 body labels and made 143 worse by more than 25 ms, against 79 to 102 for six conservative linear selectors. The severe tail (over 250 ms) grew from 27 to 29. On one song the median improved while the mean, 95th percentile, and tail all worsened (49 harmful moves in 149 labels). Image-only input was worse, with 332 harmful moves.

Two frozen follow-ups tried to keep the good part. A gate that accepts a move only when a separate judge agent lands within 20 or 40 ms cut harms to 34 and 50 and passed the screen. It recovered one of 24 severe stack errors where the ungated agent recovered 23, and a learned gate did not transfer across artists. A later rule releases raw proposals only for coherent groups of stacked syllables. It cut stack median error from 170 to 40 ms and left every body prediction as the judge gate had it. That is development evidence on already-inspected songs. Nothing is deployed, and the rule stays frozen for fresh labels.

## Nulls and errors

**Stalled refiner:** Candidate snapping needs about 80% top-1 accuracy to pay, because a right pick saves about 12 ms and a wrong one costs about 33. More labels lifted ranking accuracy from 0.39 to 0.75, but placement never moved. Bounded regression was significantly worse (30.2 to 36.5 ms, p = 6.5e-6). A first cut showed 29.0 to 17.5 ms, but it had tested only the easy slots and reversed on all slots.

**Clock mismatch:** Three days later a code audit found that the harness assumed a 2.000 ms frame clock on 44.1 kHz audio. The true hop is 1.995 ms, about 136 ms of drift per minute. The audit also found a learning curve scoring its own training slots. I put an integrity notice on the result, suspended its causal conclusions, and preregistered a corrected rerun. The rerun reached the same stall for cleaner reasons and withdrew two conclusions the original could not support.

**Wrong claim:** An agent said the production pipeline drifts 108 ms early because of dereverb. The claim was code-cited and wrong. Onset edges were identical with and without dereverb (median 0 ms, 1,294 peaks), so the 108 ms was almost certainly a matching artifact. I retracted it the same session.

**Nulls:** Five methods (phone aligners, a reranker, topology, a neural sequence model) moved the median by 0.0 to 0.6 ms. A pooled +10.3 ms gain failed one registered per-song clause by 0.68 ms and stayed failed. A promising controller's shuffled-label null also passed the gate, and so did 11 of 24 nulls in that comparison.

## What transfers

The median can improve while the work gets worse, so I judge agents by the harm they do. I make the protocol mechanical, with snapshots, seals, guards, and gates in code, and I treat an agent's confidence as untrusted.

## What is not claimed

The 18 ms is a benchmark number from a cappella audio with perfect lyrics and silver labels. Next I want fresh human labels from new artists.
