# FlowState: measuring rap syllable timing, and shrinking the claim until it was true

*Status: private repository; nothing here has been published or submitted. Claude Code and Codex agents wrote the code. I set the direction, hand-labeled a gold set of 1,061 onsets, designed the experiments and gates, and decided what counted as a result.*

**At a glance**
- Median syllable-onset error fell from 42 to 18 ms beyond a fitted-offset baseline on 12 a cappella tracks, and measured 21.6 ms on 9 held-out tracks (single-use, preregistered holdout). Research numbers on silver-standard labels, not production numbers.
- Governed with 49 worktrees, SHA-256-frozen gold labels, an owner-token write guard, and hash-sealed predictions.
- Frontier agents as automatic label refiners: rejected. The median improved from 34.9 to 12.9 ms, but a frozen safety screen caught 143 body labels made worse by more than 25 ms.

## The problem

FlowState analyzes how rappers place syllables against a beat. Every flow metric depends on per-syllable onset times, so onset accuracy is the north star. The measuring stick is Mitchell Ohriner's public flowBook corpus, whose phone-level timings were hand-corrected from a forced aligner's output; I used 12 a cappella excerpts, about 2,000 scored syllables. The pipeline separates vocals, force-aligns them, and anchors each syllable to its vowel onset.

## The result, and how it got smaller

The first honest measurement was a 44 ms median error. Two changes took it to 18 ms: target the vowel onset instead of the consonant attack, and fix an outlier guard that vetoed the right shift on s, sh, and ch words.

Then I tried to break the number. A 16-agent fleet re-derived it. The 18 ms reproduced: a fresh GPU recompute regenerated all 12 cache files byte-identically, and two independent scorers agreed. The claim shrank anyway.

- About half of the 44-to-18 journey was landmark-convention alignment, not detection. The defensible claim is against a leave-one-out fitted constant offset (41.9 ms median); the full stack scores 18.0 and wins on 12 of 12 tracks.
- It was in-sample: the 12 tracks were both development and test.
- The ground truth is silver. 47.6% of scored onsets were never hand-corrected and label noise is about 9 ms, so a perfect pipeline would read about 6 ms. I adopted a stop rule: accept a change only at a 2-3 ms pooled gain with at least 8 of 12 tracks improved.
- Seven claims in my own status document were corrected or retired.

The fix for in-sample was an untouched holdout: 30 more tracks by one artist. My first attempt was stopped by the preregistered sync gate, which rejected 29 of 30 tracks and also 5 of the 12 known-good ones. The instrument was broken, so the holdout stayed unspent. I recalibrated the gate on the 12 alone, froze the bands and the nine admitted tracks in git, and spent the holdout once: pooled median 21.6 ms against a 32 ms bar, signed median +2.3 ms (not offset-confounded), 82.7% of onsets within 100 ms. The scope is nine tracks from one artist, admitted by a gate, now development data forever. On the four full-mix songs I labeled by hand, the pipeline's own median error is about 35 ms and about a quarter of onsets are off by more than 100 ms.

## Governing the fleet

The scarcest asset is the 1,061 onsets I placed by hand, so the rules are mechanical.

- **One writing agent per worktree** (49 in all): own branch, assigned paths, explicit staging, no self-merging.
- **Gold is read-only to agents.** Experiments consume immutable SHA-256-addressed snapshots, and each run manifest pins the hash. Marks are keyed by syllable index, so a re-run can silently re-pair marks with the wrong syllables.
- **Writes to gold need an owner token** checked against a server-side secret. The guard fails closed if the secret is unset and recognizes gold by content, so a migrated copy stays protected.
- **Sealed, then scored.** One experiment hash-sealed 552 agent responses before private gold was joined; another scored 163 sealed entries by code and had all 510 arm, group, and stratum combinations recomputed independently.
- **Preregistration in code.** The corrected-rerun harness refuses to run until six owner decisions are committed and its implementation blockers clear.
- **Small blast radius.** Shared agent settings pre-approve only file read, write, and search; CODEOWNERS routes gold-touching paths to me; offline CI scans for secrets and checks the safeguards still exist.

## Frontier agents as label refiners

A frontier OpenAI model, driven through Codex subagents, proposed corrected onsets from waveform and spectrogram images plus numeric energy frames (never audio). I scored the proposals against all 1,061 of my hand-placed labels, across four songs and three artists. The acceptance screen was declared before scoring: at least 5 ms median gain, no worse mean, 95th-percentile, >100 ms, or >250 ms error rates, and no song median regressing more than 5 ms.

The median looked like a win: 34.9 to 12.9 ms across all labels. The screen said no. The agents acted on 982 of 1,004 body labels and made 143 worse by more than 25 ms, against 79-102 for six conservative linear selectors. The severe tail (>250 ms) grew from 27 to 29. On one song the median improved while the mean, 95th percentile, and tail all worsened (49 harmful moves in 149 labels). Image-only input was worse, with 332 harmful moves.

Two frozen follow-ups tried to keep the good part. A gate that accepts a move only when a separate judge agent lands within 20 or 40 ms cut harms to 34 and 50 and passed the screen, but recovered one of 24 severe stack errors where the ungated agent recovered 23; a learned gate did not transfer across artists. A later rule that releases raw proposals only for coherent groups of stacked syllables cut stack median error from 170 to 40 ms and left every body prediction as the judge gate had it. That is development evidence on already-inspected songs. Nothing is deployed, and the rule stays frozen for fresh labels.

## Honest nulls and self-caught errors

- **A stalled refiner, with a mechanism.** Candidate snapping needs about 80% top-1 accuracy to pay: a right pick saves about 12 ms, a wrong one costs about 33. More labels lifted ranking accuracy from 0.39 to 0.75 while placement never moved, and bounded regression was significantly worse (30.2 to 36.5 ms, p = 6.5e-6). A first cut showed 29.0 to 17.5 ms, but it had tested only the easy slots and reversed on all slots.
- **A clock mismatch.** Three days later a code audit found the harness assumed a 2.000 ms frame clock on 44.1 kHz audio (true hop 1.995 ms, about 136 ms of drift per minute) and a learning curve scoring its own training slots. I put an integrity notice on the result, suspended its causal conclusions, and preregistered a corrected rerun. It reached the same stall for cleaner reasons and withdrew two conclusions the original could not support.
- **A confident agent, wrong.** The claim that the production pipeline drifts 108 ms early because of dereverb was detailed, code-cited, and wrong: onset edges were identical with and without dereverb (median 0 ms, 1,294 peaks), and the 108 ms was almost certainly a matching artifact. I retracted it the same session.
- **Nulls on the record.** Five methods (phone aligners, a reranker, topology, a neural sequence model) moved the median 0.0-0.6 ms. A pooled +10.3 ms gain failed one registered per-song clause by 0.68 ms and stayed failed. A promising controller's shuffled-label null also passed the gate (as did 11 of 24 nulls in that comparison).

## What transfers

Judge agents on harm, not the median. Make protocol mechanical: snapshots, seals, guards, gates in code. Treat agent confidence as untrusted.

## What is not claimed

The 18 ms is a benchmark number: a cappella audio, perfect lyrics, silver labels. Next: fresh human labels from new artists.
