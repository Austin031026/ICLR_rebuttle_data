# Rebuttal Experiment Results Archive

本仓库保存 rebuttal 使用的 **evaluation 原始输出、逐题评分、汇总结果、评测配置与审计清单**。

数据按“实验问题 → 模型 → 算法或实验 arm → benchmark”组织。第一次接手本仓库时，请先阅读本 README，再根据第 27 节的快速索引定位数据。

> [!IMPORTANT]
> 本仓库主要保存 evaluation 证据，不保证训练 checkpoint 仍然存在。已经删除的 checkpoint 无法从这些文件恢复，也不能据此继续训练；但只要逐题 `shard-*.jsonl`、`summary.json` 和 `eval_plan.json` 完整，就可以重新计算归档中已有 evaluation 的 accuracy、Pass@1，以及 Qwen3-1.7B 标准 Baseline 的 Pass@16。

---

## 1. 归档状态总览

| 目录 | 已归档内容 | Evaluation 协议 | 状态 |
|---|---|---|---|
| `Baseline/` | Qwen3-1.7B、Qwen3-4B、Nemotron-4B 的 RQ1 Baseline；另含 Qwen3-1.7B 与 Nemotron-4B GRPO | Qwen3-1.7B 标准 Baseline 为 Pass@16；其余为 Pass@1 | 已迁移并检查 |
| `RQ2.1-Allocation-Controls/` | Uniform-Matched、Shuffled-ReN、Causal-Matched | Pass@1，五项 Non-Coding benchmark | 已迁移并验证 |
| `RQ2.3-Teacher-Strength-Sweep/` | 9 个 Qwen3-1.7B Student evaluation（Base + 8 个训练组合）与 4 个 standalone Teacher evaluation | Pass@1，五项 Non-Coding benchmark | 已迁移并验证 |
| `manifests/` | 文件清单、审计输出和重算指标 | 派生数据，不替代原始 evaluation | 按需生成 |

不在本归档中的内容：

- Qwen3-4B GRPO：未运行；
- RQ2.1 LiveCodeBench：未运行；
- RQ2.3 LiveCodeBench：未运行；
- RQ3.2 Prefix Analysis：未纳入本归档，顶层不应保留空的 RQ3.2 目录；
- 失败的、未完成的或历史遗留的 ReN 主实验：不作为有效 rebuttal 结果收录。

---

## 2. 顶层目录结构

```text
Rebato/
├── README.md
├── Baseline/
│   ├── Qwen3-1.7B/
│   │   ├── RQ1-Pass16/
│   │   └── GRPO-Pass1-Only/
│   ├── Qwen3-4B/
│   │   └── RQ1-Pass1/
│   └── Nemotron-4B/
│       ├── RQ1-Pass1/
│       └── GRPO-Pass1-Only/
├── RQ2.1-Allocation-Controls/
│   └── Qwen3-1.7B/
│       ├── uniform_matched/
│       ├── shuffled_ren/
│       └── causal_matched/
├── RQ2.3-Teacher-Strength-Sweep/
│   ├── Students-Qwen3-1.7B/
│   │   └── evaluation/
│   └── Teachers/
│       └── evaluation/
└── manifests/
```

所有路径均以仓库根目录 `Rebato/` 为起点。

---

## 3. Evaluation 协议总览

| 实验 | 模型或算法 | Pass@N | Non-Coding | Coding |
|---|---|---:|---|---|
| RQ1 Baseline | Qwen3-1.7B Base、Control Ref、Vanilla OPD、OPSD | Pass@16 | 5 项 | LiveCodeBench v6 new-175 |
| GRPO Baseline | Qwen3-1.7B GRPO Round 8 | Pass@1 | 5 项 | LiveCodeBench v6 new-175 |
| RQ1 Baseline | Qwen3-4B Base、Control Ref、Vanilla OPD、OPSD | Pass@1 | 5 项 | LiveCodeBench v6 new-175 |
| RQ1 Baseline | Nemotron-4B Base、Control Ref、Vanilla OPD、OPSD | Pass@1 | 5 项 | LiveCodeBench v6 new-175 |
| GRPO Baseline | Nemotron-4B GRPO Step 8 | Pass@1 | 5 项 | LiveCodeBench v6 new-175 |
| RQ2.1 | Uniform-Matched、Shuffled-ReN、Causal-Matched | Pass@1 | 5 项 | 未运行 |
| RQ2.3 Student | Qwen3-1.7B Base + 8 个 Teacher/训练法组合 | Pass@1 | 5 项、每个模型共 825 题 | 未运行 |
| RQ2.3 Teacher | Qwen3-4B、8B、14B、32B standalone Teacher | Pass@1 | 5 项、每个模型共 825 题 | 未运行 |

这里的 Nemotron-4B 指：

```text
NVIDIA Llama-3.1-Nemotron-Nano-4B-v1.1
```

> [!WARNING]
> shard 数量表示并行保存的数据分片数，不表示 Pass@N。8 个 shard 不是 Pass@8，6 个 shard 也不是 Pass@6。Pass@N 只由同一道题的独立 generation 次数决定。

---

## 4. Benchmark 与题数

### 4.1 Baseline 与 RQ2.1 的 Non-Coding benchmark

| Benchmark | 标准题数 | 类型 |
|---|---:|---|
| MATH-500 | 500 | 数学 |
| AIME25 | 30 | 数学 |
| OlympiadBench | 512 | 数学 |
| MMLU-Pro | 通常 512；部分 Nemotron 结果为 1400 | General/OOD |
| GPQA-Diamond | 198 | General/OOD |

标准 Qwen RQ1 与 RQ2.1 单次 evaluation 共 1752 题：

```text
500 + 30 + 512 + 512 + 198 = 1752
```

### 4.2 RQ2.3 的 Non-Coding benchmark

RQ2.3 使用相同的五个 benchmark 名称，但为了统一 sweep 成本，对部分 benchmark 使用固定子集：

| Benchmark | RQ2.3 题数 |
|---|---:|
| MATH-500 | 199 |
| AIME25 | 30 |
| OlympiadBench | 199 |
| MMLU-Pro | 199 |
| GPQA-Diamond | 198 |
| **合计** | **825** |

因此，RQ2.3 的结果不能直接与标准 1752 题 evaluation 当成完全相同协议。若要比较，必须先按 `prompt_sha256` 等字段确认题目交集。

### 4.3 Coding benchmark

Coding evaluation 使用：

```text
LiveCodeBench v6 new-175
```

最终 Coding 指标应优先依据：

1. `scored/shard-*.jsonl`；
2. benchmark 目录中的 `summary.json`；
3. `livecodebench_custom_codegeneration_output_eval_all.json`。

原始 generation shard 不能单独视为官方评分结果。

---

# Part I — Baseline

## 5. Baseline 算法与名称映射

| 算法 | 常见目录名 | 说明 |
|---|---|---|
| Base | `base` | 未进行对应后训练的基础模型 |
| Control Reference | `control_ref_round4`，少数目录中为 `control_ref` | 移除 reasoning loss，仅保留 control/reference 项 |
| Vanilla OPD | `vanilla_opd_round4` 或 `opd_round4` | Vanilla on-policy distillation |
| OPSD | `opsd_round4` | OPSD 训练结果 |
| GRPO | `grpo_round8` 或 `grpo_step8` | 独立 GRPO Baseline |

Control Reference 在旧记录中也可能被写为 `No-Reasoning`、`Control + Reference` 或 `Control Ref`。

## 6. Qwen3-1.7B Baseline

### 6.1 Base、Control Ref、Vanilla OPD、OPSD：Pass@16

根目录：

```text
Baseline/Qwen3-1.7B/RQ1-Pass16/
```

Non-Coding：

```text
Baseline/Qwen3-1.7B/RQ1-Pass16/non_coding/
├── sample_00_seed_42/
├── sample_01_seed_43/
├── ...
└── sample_15_seed_57/
```

每个 sample 的主要结构：

```text
sample_XX_seed_YY/
├── eval_plan.json
├── summary.json
├── base/
│   ├── math500/
│   ├── aime25/
│   ├── olympiadbench/
│   ├── mmlu_pro/
│   └── gpqa_diamond/
├── control_ref_round4/
├── vanilla_opd_round4/
└── opsd_round4/
```

Coding：

```text
Baseline/Qwen3-1.7B/RQ1-Pass16/coding/
└── sample_XX_seed_YY/
    ├── base/livecodebench/
    ├── control_ref_round4/livecodebench/
    ├── vanilla_opd_round4/livecodebench/
    └── opsd_round4/livecodebench/
```

`sample_00_seed_42` 至 `sample_15_seed_57` 是 16 次独立 generation，因此这一部分可以计算 Pass@16。单独读取某个 sample 只能得到该 sample 的 Pass@1。

### 6.2 Qwen3-1.7B GRPO：Pass@1 only

```text
Baseline/Qwen3-1.7B/GRPO-Pass1-Only/
├── non_coding/grpo_round8/
│   ├── math500/
│   ├── aime25/
│   ├── olympiadbench/
│   ├── mmlu_pro/
│   └── gpqa_diamond/
└── coding/grpo_round8/livecodebench/
```

该目录只有 Pass@1，不能从这些文件推导 GRPO Pass@16。

## 7. Qwen3-4B Baseline

根目录：

```text
Baseline/Qwen3-4B/RQ1-Pass1/
```

Non-Coding：

```text
Baseline/Qwen3-4B/RQ1-Pass1/non_coding/
├── base/
├── control_ref_round4/
├── vanilla_opd_round4/
└── opsd_round4/
```

Coding：

```text
Baseline/Qwen3-4B/RQ1-Pass1/coding/sample_00_seed_42/
├── base/livecodebench/
├── control_ref_round4/livecodebench/
├── vanilla_opd_round4/livecodebench/
└── opsd_round4/livecodebench/
```

Qwen3-4B 只有一个 evaluation sample，因此为 Pass@1。Qwen3-4B 没有 GRPO 结果。

## 8. Nemotron-4B Baseline

根目录：

```text
Baseline/Nemotron-4B/
```

Base、Control Ref、Vanilla OPD、OPSD 的 Non-Coding：

```text
Baseline/Nemotron-4B/RQ1-Pass1/non_coding/
├── base_control_ref/
│   ├── base/
│   └── control_ref_round4/
├── vanilla_opd/
│   └── vanilla_opd_round4/
└── opsd/
    └── opsd_round4/
```

对应 Coding：

```text
Baseline/Nemotron-4B/RQ1-Pass1/coding/
├── base_control_ref/
│   ├── base/livecodebench/
│   └── control_ref_round4/livecodebench/
└── opd_opsd/
    ├── opd_round4/livecodebench/
    └── opsd_round4/livecodebench/
```

GRPO：

```text
Baseline/Nemotron-4B/GRPO-Pass1-Only/
├── non_coding/grpo_step8/
└── coding/grpo_step8/livecodebench/
```

所有 Nemotron-4B 结果均为 Pass@1。

### 8.1 Nemotron MMLU-Pro 协议差异

| 算法 | MMLU-Pro 题数 |
|---|---:|
| Base | 1400 |
| Control Ref | 1400 |
| Vanilla OPD | 512 |
| OPSD | 512 |
| GRPO | 512 |

Vanilla OPD 和 OPSD 的 512 题已确认是 Base 1400 题集合的子集。GRPO 的 accuracy 可由自身逐题文件重算并与自身 `summary.json` 对齐，但其 prompt hash 未与 Base 对齐，因此应作为独立 512 题协议报告。

不能把 1400 题与 512 题结果视为完全匹配的 paired comparison。

---

# Part II — RQ2.1 Allocation Controls

## 9. RQ2.1 到底包含哪些实验

RQ2.1 的 Student 全部是 Qwen3-1.7B。归档中的三个新训练 arm 为：

| Arm | 核心操作 | 所检验的问题 |
|---|---|---|
| Uniform-Matched | 保持每条 rollout 的 ReN 总权重质量，但把权重均匀分配到 reasoning positions | ReN 收益是否仅来自总监督量 |
| Shuffled-ReN | 保留 ReN 权重的数值集合，但打乱权重位置 | 权重是否必须分配到正确 token 位置 |
| Causal-Matched | 保留相同权重集合，按 causal Teacher–Student KL 排序分配 | Hindsight resolution 是否比单纯 causal 分歧更重要 |

注意：

```text
Causal-Matched != OPSD
Shuffled-ReN  != GRPO
Uniform-Matched != Control Reference
```

RQ2.1 目录只保存这三个新训练 arm。Base、Control Reference、Vanilla OPD、OPSD 和 GRPO 没有在 RQ2.1 目录重复保存；需要时应从 `Baseline/Qwen3-1.7B/` 查询。

## 10. RQ2.1 数据路径

```text
RQ2.1-Allocation-Controls/Qwen3-1.7B/
├── uniform_matched/
│   └── evaluation/
│       └── uniform_matched/
├── shuffled_ren/
│   └── evaluation/
│       └── shuffled_ren/
└── causal_matched/
    └── evaluation/
        └── causal_matched/
```

每个最内层 arm 目录均含：

```text
math500/
aime25/
olympiadbench/
mmlu_pro/
gpqa_diamond/
```

每个 arm 的 `evaluation/` 根目录含 `eval_plan.json` 和 `summary.json`；每个 benchmark 目录含自己的 `summary.json` 与 `shard-*.jsonl`。

RQ2.1 每个 benchmark 有 8 个 shard，但它们只是并行数据分片。RQ2.1 是 **Pass@1**，不是 Pass@8、Pass@16 或 Pass@100。

## 11. RQ2.1 与 Baseline 的关系

### 11.1 重复对照从哪里找

RQ2.1 需要 Qwen3-1.7B Base 及其他既有算法作为对照时，不复制第二份数据，而是引用 Baseline：

| 对照 | 从 `Rebato/` 开始的路径 |
|---|---|
| Base seed-42 Pass@1 | `Baseline/Qwen3-1.7B/RQ1-Pass16/non_coding/sample_00_seed_42/base/` |
| Base 完整 Pass@16 | `Baseline/Qwen3-1.7B/RQ1-Pass16/non_coding/` |
| Control Reference | `Baseline/Qwen3-1.7B/RQ1-Pass16/non_coding/sample_00_seed_42/control_ref_round4/` |
| Vanilla OPD | `Baseline/Qwen3-1.7B/RQ1-Pass16/non_coding/sample_00_seed_42/vanilla_opd_round4/` |
| OPSD | `Baseline/Qwen3-1.7B/RQ1-Pass16/non_coding/sample_00_seed_42/opsd_round4/` |
| GRPO Pass@1 | `Baseline/Qwen3-1.7B/GRPO-Pass1-Only/non_coding/grpo_round8/` |

从 `RQ2.1-Allocation-Controls/` 目录出发，Base seed-42 的相对路径为：

```text
../Baseline/Qwen3-1.7B/RQ1-Pass16/non_coding/sample_00_seed_42/base/
```

### 11.2 比较前必须检查协议

逐题比较前至少核对：

- `prompt_index`；
- `prompt_sha256`；
- `ground_truth`；
- `generation_seed`；
- benchmark 题数；
- parser 配置；
- decoding 配置。

目录名称相同不等于评测题目、seed 或 parser 一定相同。

### 11.3 未收录的 ReN 主实验

失败、未完成或历史遗留的 ReN 主实验不作为有效 rebuttal 结果归档，也不应在结果表中标记为已完成。当前 `RQ2.1-Allocation-Controls/` 的有效内容就是已经验证的三个 allocation-control arm。

## 12. RQ2.1 已验证 Pass@1 结果

| Arm | MATH-500 | AIME25 | OlympiadBench | MMLU-Pro | GPQA-Diamond | Macro | Micro |
|---|---:|---:|---:|---:|---:|---:|---:|
| Uniform-Matched | 88.00% | 36.67% | 64.45% | 57.81% | 42.42% | 57.87% | 66.27% |
| Shuffled-ReN | 87.40% | 30.00% | 65.63% | 57.42% | 38.89% | 55.87% | 65.81% |
| Causal-Matched | 88.00% | 43.33% | 64.84% | 56.64% | 42.42% | 59.05% | 66.15% |

已验证：

```text
15 / 15 benchmark units successfully recomputed
15 / 15 recomputed accuracies match summary.json
```

如保留了派生报告，可在以下位置查询：

```text
manifests/rq21_pass1_accuracy.tsv
```

Macro 是五个 benchmark accuracy 的简单平均；Micro 是 1752 道题合并后的正确率。二者含义不同。

---

# Part III — RQ2.3 Teacher-Strength Sweep

## 13. RQ2.3 的研究目标

RQ2.3 检验 Teacher 能力变化对 Qwen3-1.7B Student 的影响，并同时保存对应 Teacher 的 standalone evaluation，便于比较：

1. Teacher 自身能力随参数规模如何变化；
2. 同一 Teacher 下 Vanilla 与 ReN 训练出的 Student 有何差异；
3. Teacher 更强是否必然带来更强的 Student；
4. Student 的提升是否与 Teacher standalone accuracy 同步。

所有训练后的 Student 都是 **Qwen3-1.7B**。目录名中的 `qwen4b`、`qwen8b`、`qwen14b`、`qwen32b` 表示训练时使用的 **Teacher 大小**，不是 Student 大小。

## 14. RQ2.3 归档与验证状态

根目录：

```text
RQ2.3-Teacher-Strength-Sweep/
```

迁移后已完成以下检查：

```text
Student benchmark units: 45 / 45
Teacher benchmark units: 20 / 20
Total benchmark units:   65 / 65

Student shards:          270 / 270
Teacher shards:          120 / 120
Total shards:            390 / 390

Student summaries:       46 / 46
Teacher summaries:       21 / 21
Engine caches:             0 / 0
Source/destination diff:   0

RQ2.3_ARCHIVE_VERIFIED=PASS
```

归档检查时目录大小约为 657 MB。

## 15. RQ2.3 Student evaluation

路径：

```text
RQ2.3-Teacher-Strength-Sweep/Students-Qwen3-1.7B/evaluation/
```

共有 9 个 Student evaluation：

| 目录名 | Student | Teacher/训练设置 |
|---|---|---|
| `base` | Qwen3-1.7B | 未经过该 sweep 后训练的 fresh Base |
| `qwen4b_vanilla` | Qwen3-1.7B | Qwen3-4B Teacher + Vanilla |
| `qwen4b_ren` | Qwen3-1.7B | Qwen3-4B Teacher + ReN |
| `qwen8b_vanilla` | Qwen3-1.7B | Qwen3-8B Teacher + Vanilla |
| `qwen8b_ren` | Qwen3-1.7B | Qwen3-8B Teacher + ReN |
| `qwen14b_vanilla` | Qwen3-1.7B | Qwen3-14B Teacher + Vanilla |
| `qwen14b_ren` | Qwen3-1.7B | Qwen3-14B Teacher + ReN |
| `qwen32b_vanilla` | Qwen3-1.7B | Qwen3-32B Teacher + Vanilla |
| `qwen32b_ren` | Qwen3-1.7B | Qwen3-32B Teacher + ReN |

每个模型目录下包含：

```text
math500/
aime25/
olympiadbench/
mmlu_pro/
gpqa_diamond/
```

### 15.1 Student Pass@1 结果

| Student arm | MATH-500 | AIME25 | OlympiadBench | MMLU-Pro | GPQA-Diamond | Macro | Micro | Correct/825 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Base | 87.44% | 40.00% | 54.27% | 54.27% | 37.37% | 54.67% | 57.70% | 476 |
| 4B Teacher + Vanilla | 88.94% | 30.00% | 52.76% | 53.77% | 40.91% | 53.28% | 58.06% | 479 |
| 4B Teacher + ReN | 90.95% | 43.33% | 54.27% | 56.78% | 39.39% | 56.95% | 59.76% | 493 |
| 8B Teacher + Vanilla | 87.44% | 43.33% | 51.76% | 57.29% | 38.38% | 55.64% | 58.18% | 480 |
| 8B Teacher + ReN | 89.95% | 33.33% | 51.26% | 58.29% | 43.43% | 55.25% | 59.76% | 493 |
| 14B Teacher + Vanilla | 87.94% | 36.67% | 53.27% | 55.28% | 37.37% | 54.10% | 57.70% | 476 |
| 14B Teacher + ReN | 89.95% | 36.67% | 52.26% | 54.77% | 38.89% | 54.51% | 58.18% | 480 |
| 32B Teacher + Vanilla | 86.93% | 30.00% | 49.75% | 55.28% | 33.84% | 51.16% | 55.52% | 458 |
| 32B Teacher + ReN | 88.94% | 33.33% | 47.74% | 51.26% | 33.84% | 51.02% | 54.67% | 451 |

## 16. RQ2.3 Teacher standalone evaluation

路径：

```text
RQ2.3-Teacher-Strength-Sweep/Teachers/evaluation/
```

包含 4 个 Teacher：

| 目录名 | 模型 |
|---|---|
| `teacher_qwen4b` | Qwen3-4B |
| `teacher_qwen8b` | Qwen3-8B |
| `teacher_qwen14b` | Qwen3-14B |
| `teacher_qwen32b` | Qwen3-32B |

### 16.1 Teacher Pass@1 结果

| Teacher | MATH-500 | AIME25 | OlympiadBench | MMLU-Pro | GPQA-Diamond | Macro | Micro | Correct/825 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Qwen3-4B | 92.96% | 66.67% | 62.31% | 64.82% | 54.55% | 68.26% | 68.61% | 566 |
| Qwen3-8B | 94.47% | 73.33% | 64.82% | 66.83% | 60.61% | 72.01% | 71.76% | 592 |
| Qwen3-14B | 92.46% | 76.67% | 66.83% | 74.87% | 61.62% | 74.49% | 74.06% | 611 |
| Qwen3-32B | 93.97% | 70.00% | 63.82% | 76.88% | 68.69% | 74.67% | 75.64% | 624 |

## 17. RQ2.3 与 Baseline 的关系

RQ2.3 的 `base` 与 Baseline 中的 Qwen3-1.7B Base 都表示未经对应后训练的模型，但它们属于不同 evaluation job，题目数量也不同：

- RQ2.3 Base：825 题协议，用于与 RQ2.3 的 8 个 Student arm 做配套比较；
- RQ1 Baseline Base：标准 1752 题协议，并另有 16 个独立 generation 可计算 Pass@16。

因此：

- 分析 RQ2.3 内部 paired comparison 时，应使用 `RQ2.3-Teacher-Strength-Sweep/Students-Qwen3-1.7B/evaluation/base/`；
- 查询标准 Baseline 或 Pass@16 时，应使用 `Baseline/Qwen3-1.7B/RQ1-Pass16/`；
- 不要因为两者都叫 Base 就直接混合题数或 accuracy；
- 如需跨目录比较，先用 `prompt_sha256` 对齐题目交集。

RQ2.3 的 Teacher standalone 结果也不是 `Baseline/Qwen3-4B/RQ1-Pass1/` 的替代品：前者使用 RQ2.3 的 825 题协议，后者使用 RQ1 Baseline 的标准协议。

## 18. RQ2.3 使用注意事项

- 全部 RQ2.3 结果都是 Pass@1；
- 每个模型/benchmark 的 6 个 shard 是并行数据分片，不是 Pass@6；
- RQ2.3 没有 Coding/LiveCodeBench；
- Student 与 Teacher 都必须按 825 题协议解释；
- AIME25 只有 30 题，因此对 Macro 的波动影响较大；
- 报告整体性能时，应明确使用 Macro 还是 Micro；
- 比较 Vanilla 与 ReN 时，优先在相同 Teacher 大小内比较；
- 分析 Teacher 强度时，不应只按参数量推断，应同时引用 Teacher standalone accuracy。

---

# Part IV — Evaluation 文件如何使用

## 19. 常见文件结构

不同实验可能多一层模型或算法目录，但核心结构通常为：

```text
evaluation_root/
├── eval_plan.json
├── summary.json
└── model_or_algorithm/
    └── benchmark/
        ├── summary.json
        ├── shard-00000-*.jsonl
        ├── shard-00001-*.jsonl
        └── ...
```

### 19.1 `eval_plan.json`

用于确认：

- 被评测模型或 checkpoint；
- benchmark 列表；
- generation seed；
- sampling/decoding 配置；
- 最大生成长度；
- tokenizer/model revision；
- parser 与评分设置；
- 最大样本数或 benchmark 子集。

### 19.2 evaluation 根目录的 `summary.json`

用于查看同一 evaluation job 中各模型与各 benchmark 的总体汇总。

### 19.3 benchmark 目录的 `summary.json`

通常包含：

- `examples`；
- `correct`；
- `accuracy`；
- response length；
- hit-cap/truncation 统计。

字段名应以实际文件为准。

### 19.4 `shard-*.jsonl`

逐题原始记录常见字段：

- `prompt_index`；
- `prompt_sha256`；
- `data_source`；
- `ground_truth`；
- `prediction`；
- `reward`；
- `generation_seed`；
- response 文本；
- `response_token_ids`；
- response length；
- hit-cap/truncation 状态。

不同评测版本可能有字段差异，写分析脚本前应先抽查一行 JSONL。

## 20. 重算 Pass@1 accuracy

单次 generation 的计算方式：

```text
Pass@1 accuracy = sum(reward) / number_of_examples
```

其中 `reward` 应为 0 或 1。以下命令可重算任意单个 benchmark：

```bash
BENCHMARK_DIR=/path/to/model/benchmark

python3 - "$BENCHMARK_DIR" <<'PY'
import json
import sys
from pathlib import Path

directory = Path(sys.argv[1])
rows = []

for shard in sorted(directory.glob("shard-*.jsonl")):
    with shard.open(errors="replace") as stream:
        for line in stream:
            if line.strip():
                rows.append(json.loads(line))

if not rows:
    raise SystemExit(f"No JSONL rows found in {directory}")

rewards = [float(row["reward"]) for row in rows]
print("examples =", len(rewards))
print("correct  =", int(sum(rewards)))
print("accuracy =", sum(rewards) / len(rewards))
PY
```

重算值应与同 benchmark 目录中的 `summary.json` 一致。

## 21. 计算 Qwen3-1.7B Baseline Pass@16

Pass@16 只适用于：

```text
Baseline/Qwen3-1.7B/RQ1-Pass16/
```

计算步骤：

1. 读取 `sample_00_seed_42` 至 `sample_15_seed_57`；
2. 按题目身份对齐；
3. 确认每道题恰好有 16 次独立 generation；
4. 对每道题取 16 个 reward 的最大值；
5. 对所有题的最大值求平均。

```text
pass16(problem) = max(reward_seed42, ..., reward_seed57)

Pass@16 = mean(pass16(problem))
```

不能把 16 个 `summary.json` 的 accuracy 直接平均后称为 Pass@16。16 个 seed accuracy 的平均值是 mean Pass@1，不是 Pass@16。

逐题对齐至少应检查：

```text
benchmark
prompt_index
prompt_sha256
ground_truth
generation_seed
```

并确认 generation seed 完整覆盖 42–57。

## 22. Macro 与 Micro

Macro 是各 benchmark accuracy 的简单平均：

```text
Macro = mean(accuracy_of_each_benchmark)
```

Micro 是所有题合并后的正确率：

```text
Micro = total_correct / total_examples
```

由于各 benchmark 题数不同，Macro 与 Micro 不能互相替代。结果表必须明确标注使用哪一种。

## 23. Paired comparison

如需计算两个模型之间的 accuracy delta、rescue、degradation 或 paired bootstrap confidence interval，必须先确认逐题身份一致。

至少检查：

```text
prompt_index
prompt_sha256
ground_truth
generation_seed
```

定义：

```text
rescue:
Base 错误，训练后模型正确

degradation:
Base 正确，训练后模型错误
```

模型名称相同不代表题目集合和 evaluation protocol 相同。

## 24. `manifests/` 是什么

`manifests/` 保存的是便于复核和交接的 **派生索引与审计报告**，例如：

- 文件清单；
- 路径、大小和 shard 数审计；
- accuracy 重算后的 TSV/CSV；
- Pass@N 覆盖检查；
- source/destination 一致性检查。

`manifests/` 不是新的实验结果来源，也不应替代原始 `shard-*.jsonl`、`summary.json` 或 `eval_plan.json`。若 manifest 与原始数据冲突，应重新读取原始 evaluation 并查明原因。

## 25. 从归档生成论文或 Rebuttal 表格的标准流程

1. 根据研究问题进入 `Baseline/`、`RQ2.1-Allocation-Controls/` 或 `RQ2.3-Teacher-Strength-Sweep/`；
2. 从本 README 确认模型、算法和 Pass@N；
3. 阅读对应 evaluation 的 `eval_plan.json`；
4. 检查 benchmark 名称与题数；
5. 读取 benchmark `summary.json`；
6. 从 `shard-*.jsonl` 重算 accuracy；
7. 对比重算值与 `summary.json`；
8. 跨模型比较前检查逐题身份；
9. 根据用途计算 Macro、Micro、delta、rescue/degradation 或 Pass@16；
10. 保存输入路径、分析脚本、输出 TSV/CSV、Git commit 和协议异常说明。

建议每张最终结果表同时记录：

- 数据目录；
- evaluation plan；
- 输入文件列表；
- 重算程序；
- 生成的 TSV/CSV；
- Git commit；
- 重算时间；
- parser、题目集合或 Pass@N 差异。

## 26. 禁止直接混用的结果

以下结果不能在未做协议对齐时直接混合：

- Pass@1 与 Pass@16；
- 512 题 MMLU-Pro 与 1400 题 MMLU-Pro；
- RQ1/RQ2.1 的标准 1752 题与 RQ2.3 的 825 题；
- Raw LiveCodeBench generation 与 officially scored output；
- 不同 parser 的结果；
- 不同 response budget 的结果；
- 不同 prompt hash 或 generation seed 的结果；
- 独立重新生成与同轨迹 prefix 重评分；
- 不同 evaluation job 中同名模型的结果。

---

## 27. 快速查询入口

| 查询需求 | 从 `Rebato/` 开始的相对路径 |
|---|---|
| Qwen3-1.7B 标准 Baseline Pass@16 | `Baseline/Qwen3-1.7B/RQ1-Pass16/` |
| Qwen3-1.7B Base seed-42 Pass@1 | `Baseline/Qwen3-1.7B/RQ1-Pass16/non_coding/sample_00_seed_42/base/` |
| Qwen3-1.7B GRPO Pass@1 | `Baseline/Qwen3-1.7B/GRPO-Pass1-Only/` |
| Qwen3-4B Baseline Pass@1 | `Baseline/Qwen3-4B/RQ1-Pass1/` |
| Nemotron-4B Baseline Pass@1 | `Baseline/Nemotron-4B/RQ1-Pass1/` |
| Nemotron-4B GRPO Pass@1 | `Baseline/Nemotron-4B/GRPO-Pass1-Only/` |
| RQ2.1 Uniform-Matched | `RQ2.1-Allocation-Controls/Qwen3-1.7B/uniform_matched/` |
| RQ2.1 Shuffled-ReN | `RQ2.1-Allocation-Controls/Qwen3-1.7B/shuffled_ren/` |
| RQ2.1 Causal-Matched | `RQ2.1-Allocation-Controls/Qwen3-1.7B/causal_matched/` |
| RQ2.1 重算报告 | `manifests/rq21_pass1_accuracy.tsv` |
| RQ2.3 全部 Student | `RQ2.3-Teacher-Strength-Sweep/Students-Qwen3-1.7B/evaluation/` |
| RQ2.3 配套 Base | `RQ2.3-Teacher-Strength-Sweep/Students-Qwen3-1.7B/evaluation/base/` |
| RQ2.3 4B Teacher + ReN Student | `RQ2.3-Teacher-Strength-Sweep/Students-Qwen3-1.7B/evaluation/qwen4b_ren/` |
| RQ2.3 8B Teacher + ReN Student | `RQ2.3-Teacher-Strength-Sweep/Students-Qwen3-1.7B/evaluation/qwen8b_ren/` |
| RQ2.3 14B Teacher + ReN Student | `RQ2.3-Teacher-Strength-Sweep/Students-Qwen3-1.7B/evaluation/qwen14b_ren/` |
| RQ2.3 32B Teacher + ReN Student | `RQ2.3-Teacher-Strength-Sweep/Students-Qwen3-1.7B/evaluation/qwen32b_ren/` |
| RQ2.3 全部 standalone Teacher | `RQ2.3-Teacher-Strength-Sweep/Teachers/evaluation/` |

---

## 28. 本归档能做什么、不能做什么

本归档可以支持：

- 重新计算已有 evaluation accuracy；
- 验证 benchmark `summary.json`；
- 计算所有单次 generation evaluation 的 Pass@1；
- 计算 Qwen3-1.7B 标准 Baseline 的 Pass@16；
- 计算 Macro 与 Micro；
- 在逐题对齐后计算 paired delta、rescue 和 degradation；
- 使用 scored output 分析 LiveCodeBench；
- 复核 RQ2.1 allocation controls；
- 比较 RQ2.3 中不同 Teacher 大小、Vanilla/ReN Student 与 Teacher standalone 能力。

本归档不能支持：

- 恢复已删除 checkpoint；
- 继续训练；
- 在没有模型权重时重新生成；
- 从 Pass@1 推导 Pass@16；
- 从 shard 数量推导 Pass@N；
- 把不同题目集合直接当作配对实验；
- 把未运行、失败或未归档的实验标记为完成。

---

## 29. 交接结论

截至本 README 更新时，已完整归档并验证的有效实验范围是：

1. `Baseline/`：三类模型族的既有 Baseline evaluation，以及实际运行过的 GRPO evaluation；
2. `RQ2.1-Allocation-Controls/`：三个 Qwen3-1.7B allocation-control arm；
3. `RQ2.3-Teacher-Strength-Sweep/`：9 个 Student evaluation 与 4 个 standalone Teacher evaluation。

后续使用者应以原始逐题 JSONL 为最终可复核证据，以 `eval_plan.json` 判定协议，以 `summary.json` 进行快速查阅，并在正式制表前完成一次独立重算。
