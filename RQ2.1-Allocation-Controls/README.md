# RQ2.1 Allocation Controls

This directory contains the RQ2.1 supervision-allocation control experiments.

## Student and training protocol

All trained Students are Qwen3-1.7B models.

The RQ2.1 arms use the same general training pool, Student initialization,
training budget, prompt schedule, and four-round structure as the matched
Qwen3-1.7B experiment. The experimental variable is how supervision weight is
allocated across reasoning positions.

## Newly trained RQ2.1 arms

| Arm | Meaning | Archive path |
|---|---|---|
| Uniform-Matched | Spreads the same per-rollout ReN weight mass uniformly over reasoning positions | `Qwen3-1.7B/uniform_matched/` |
| Shuffled-ReN | Preserves the ReN weight multiset but permutes its position assignment | `Qwen3-1.7B/shuffled_ren/` |
| Causal-Matched | Assigns the matched weight multiset according to causal Teacher-Student KL rank | `Qwen3-1.7B/causal_matched/` |

These three arms are new RQ2.1 training experiments. They are not duplicates of
Control Reference, Vanilla OPD, or OPSD.

In particular, Causal-Matched must not be identified as OPSD. They use different
supervision constructions.

## Reused comparison models

RQ2.1 logically compares the three new arms against a fresh Base model and the
source matched ReN model.

### Qwen3-1.7B Base

The Base model overlaps with the RQ1 Baseline archive. For the seed-42 Pass@1
view, use the following path relative to this directory:

    ../Baseline/Qwen3-1.7B/RQ1-Pass16/non_coding/sample_00_seed_42/base/

The complete sixteen-sample Base evaluation is located at:

    ../Baseline/Qwen3-1.7B/RQ1-Pass16/non_coding/

Do not copy the Base directory into every RQ2.1 arm merely to duplicate it.
Analysis code may read the Base results from the relative path above.

Before performing paired per-question statistics, verify that the RQ2.1 and
Baseline views have matching prompt indices, prompt hashes, ground truths,
generation seeds, parser settings, and benchmark rows.

### Source matched ReN

The source ReN model is the proposed method rather than a Baseline algorithm.
Its results should eventually be archived under:

    ../RQ1-ReN/Qwen3-1.7B/Matched-R4/

RQ2.1 analysis may reference that directory after the matched ReN archive has
been created.

## Evaluation protocol

The completed RQ2.1 evaluations contain the following non-coding benchmarks:

- MATH-500
- AIME25
- OlympiadBench
- MMLU-Pro
- GPQA-Diamond

RQ2.1 does not currently include LiveCodeBench.

Each new arm has its own evaluation directory, root summary, evaluation plan,
per-benchmark summary, and JSONL shards.

Pass@1 accuracy can be recomputed from the per-example records as:

    accuracy = sum(reward) / number_of_examples

RQ2.1 is a single-sample evaluation. It must not be reported as Pass@16.

## Planned archive structure

    RQ2.1-Allocation-Controls/
    ├── README.md
    └── Qwen3-1.7B/
        ├── uniform_matched/
        │   └── evaluation/
        ├── shuffled_ren/
        │   └── evaluation/
        └── causal_matched/
            └── evaluation/

The experiment source directories remain under `Lulu_outputs/experiments`.
This archive should contain evaluation evidence, not deleted checkpoint weights.
