# Baseline Evaluation Archive

This directory archives the evaluation outputs used for the baseline experiments.

## Scope

The archive contains evaluation outputs, scored records, summaries, and evaluation
metadata. It does not imply that the corresponding model checkpoint weights or full
training-state directories are stored here.

## Preserved experiment matrix

| Model | Base | Control Reference | Vanilla OPD | OPSD | GRPO |
|---|---|---|---|---|---|
| Qwen3-1.7B | Pass@16 | Pass@16 | Pass@16 | Pass@16 | Pass@1 only |
| Qwen3-4B | Pass@1 | Pass@1 | Pass@1 | Pass@1 | Not run |
| Nemotron-4B | Pass@1 | Pass@1 | Pass@1 | Pass@1 | Pass@1 only |

Nemotron-4B refers to the NVIDIA Llama-3.1-Nemotron-Nano-4B-v1.1 model family.

The standard baseline algorithms are:

- Base model
- Control Reference
- Vanilla OPD
- OPSD

GRPO is stored separately because its Pass@N protocol is not always the same as the
standard baseline protocol.

## Benchmarks

Each completed model/algorithm combination contains evaluation results for:

### Non-coding

- MATH-500
- AIME25
- OlympiadBench
- MMLU-Pro
- GPQA-Diamond

### Coding

- LiveCodeBench v6 new-175

## Metric reconstruction

The archived JSONL shard files can be used to recompute evaluation accuracy.

For a single evaluation sample:

    accuracy = sum(reward) / number_of_examples

The recomputed accuracy has been checked against the corresponding `summary.json`
files. For the inspected baseline results, the values match.

For Qwen3-1.7B standard baselines, 16 independent evaluation samples are retained.
They can be used to calculate:

- Per-seed Pass@1 accuracy
- Mean Pass@1 accuracy over 16 samples
- Per-problem Pass@16 success, where a problem is counted as solved if at least one
  of the 16 samples is correct
- Aggregate Pass@16 accuracy

Qwen3-1.7B GRPO, Qwen3-4B, and Nemotron-4B contain Pass@1 evaluation results only.
Pass@16 must not be inferred from those Pass@1 directories.

For LiveCodeBench, use the scored JSONL files and the official combined evaluation
output. The raw generation shards alone should not be treated as final correctness
scores.

## Nemotron MMLU-Pro protocol note

The Nemotron-4B MMLU-Pro evaluations do not all use the same number of examples:

- Base: 1400 examples
- Control Reference: 1400 examples
- Vanilla OPD: 512 examples
- OPSD: 512 examples
- GRPO: 512 examples

The Vanilla OPD and OPSD 512-example sets were verified to be subsets of the
1400-example Base evaluation.

The GRPO result is internally valid and its recomputed accuracy matches its summary,
but its prompt hashes do not align with the Base prompt hashes. This may be caused by
a different prompt or chat template. Therefore, the GRPO MMLU-Pro result should be
reported as a separate 512-example protocol unless dataset identity is confirmed
through additional metadata.

Do not directly compare a 1400-example MMLU-Pro score with a 512-example score
without explicitly stating the evaluation subset.

## Interpretation

The archive is sufficient for:

- Recomputing checkpoint evaluation accuracy
- Verifying reported summary metrics
- Computing Qwen3-1.7B standard-baseline Pass@1 and Pass@16
- Comparing algorithms that use the same benchmark subset
- Inspecting individual predictions, rewards, and scored coding outputs

The archive alone is not sufficient for:

- Restoring deleted model checkpoints
- Resuming training
- Rerunning generation without the original model weights
- Deriving Pass@16 from a Pass@1-only evaluation
