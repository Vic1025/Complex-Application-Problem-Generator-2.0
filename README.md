# Mathematical Word Problem Generation Pipeline V2

A scalable, programmatic pipeline for synthesizing hierarchically structured mathematical word problems with controllable complexity. Built on top of Physics of Language Models (Part 2.1) with significant extensions to support multi-level entity hierarchies, long reasoning chains, and diverse logical relationships.

---

## Overview

This project addresses key limitations of prior math word problem generation approaches (OP10/OP20) by introducing a **hierarchical entity library** and a **logic tree synthesis engine** capable of producing problems with 5–30+ interrelated entities.

The generated data has been shown to significantly improve model reasoning on complex multi-step math problems, with up to **+276% accuracy improvement** on business-domain problems and strong generalization across standard benchmarks (MATH-500, GSM8K, ASDiv, SVAMP).

---

## Key Features

- **Hierarchical entity library**: 50 entity sets across diverse domains, each supporting up to 2⁵⁰ unique entity combinations
- **Controllable complexity**: Parameterized by entity count, tree depth, answer path length, and layer width
- **Mixed arithmetic support**: Addition/subtraction via containment relations; multiplication/division via unit extension
- **~70 linguistic paraphrase templates** for condition expression diversity
- **LLM-based COT polishing**: GPT-4o post-processing achieves ~90% usable rate
- **Full RL/SFT/DPO training pipeline** with benchmark evaluations

---

## Repository Structure

```
├── application_problem_complex_tree/   # Main generation pipeline
│   ├── entity_library/                 # 50 hierarchical entity sets
│   ├── tree_generator/                 # Logic tree synthesis
│   ├── problem_composer/               # Problem & COT generation
│   └── polish/                         # LLM polishing scripts
├── training/
│   ├── sft/                            # SFT training configs
│   ├── rl/                             # RL (GRPO via verl) configs
│   └── dpo/                            # DPO training configs
├── eval/                               # Benchmark evaluation scripts
└── data/                               # Sample data and entity libraries
```

---

## Quick Start

### 1. Generate Problems

```bash
cd application_problem_complex_tree
python generate.py \
  --entity_set farm_crops \
  --num_entities 15 \
  --max_depth 4 \
  --answer_path_len 10 \
  --layer_width 3 \
  --use_multiplication true \
  --polish true \
  --output_path ./output/problems.json
```

### 2. Train with RL (GRPO)

```bash
python3 -m verl.trainer.main_ppo \
  algorithm.adv_estimator=grpo \
  data.train_files=<path_to_train.parquet> \
  data.val_files=<path_to_val.parquet> \
  data.train_batch_size=128 \
  actor_rollout_ref.model.path=<model_path> \
  actor_rollout_ref.rollout.n=8 \
  trainer.n_gpus_per_node=8
```

### 3. Evaluate

```bash
python eval/run_eval.py \
  --model_path <checkpoint_path> \
  --test_file eval/data/sft_test_0-5.json \
  --few_shot 5
```

---

## Data Format

Each generated sample follows this structure:

```json
{
  "instruction": "",
  "input": "<background story>\n<known entity conditions>\n<logical/numerical relations>\n<question>",
  "output": "Given Data:\n...\nStatements of Connections:\n...\nCalculation process:\n...\nSo, the final answer is {answer}."
}
```

The ground truth answer is extracted as `{answer}` from the output field.

---

## Benchmark Results (Zero-Shot)

| Model Setting | MATH-500 | GSM8K | ASDiv | SVAMP | Avg | op0-10 | op10-15 | op15-20 |
|---|---|---|---|---|---|---|---|---|
| Setting 1: Base + BizData RL | 57.0 | 83.7 | 91.3 | 91.2 | 80.8 | **95.33** | 81.33 | 43.0 |
| Setting 2: SFT (full) + RL | 29.8 | 29.8 | 67.7 | 51.5 | 44.7 | 69.0 | 36.33 | 16.0 |
| Setting 3: SFT (500) + RL | 55.2 | 77.6 | 87.9 | 85.7 | 76.6 | 90.67 | 65.0 | **41.33** |
| Setting 4: Base + Math RL | **64.0** | 86.2 | 92.7 | 92.9 | 83.9 | 25.33 | 2.67 | 0 |
| Setting 5: Base + Mixed RL | 65.8 | **87.9** | **93.4** | **92.9** | **85.0** | 46.33 | 5.67 | 1.0 |

*op = number of unknown entities in the problem*

---

## Training Settings

| Setting | Base Model | Training Data | Strategy |
|---|---|---|---|
| 1 | Qwen2.5-3B-Instruct | Synthesized business data | Direct RL |
| 2 | Qwen2.5-3B-Instruct | Synthesized (SFT full → RL) | SFT + RL |
| 3 | Qwen2.5-3B-Instruct | Synthesized (SFT 500 → RL) | Cold-start SFT + RL |
| 4 | Qwen2.5-3B-Instruct | MATH hard samples | Direct RL |
| 5 | Qwen2.5-3B-Instruct | Mixed (business + MATH) | Direct RL |

---

## Key Findings

- **Synthetic data markedly boosts domain reasoning**: RL on synthetic data alone raises op0-10 accuracy from 25% to 95% (+276%).
- **"Less SFT is more"**: Full-dataset SFT before RL causes overfitting and collapse on harder problems; cold-start with only 500 SFT samples preserves the model's capacity to extend reasoning chains.
- **Mixed data shows threshold behavior**: Adding ~10% synthetic data to math RL data improves easier problems (op0-10: +21%) but has limited impact on harder ones (op15-30).
- **SFT with mixed difficulty is most robust**: Mixed op0-20 SFT achieves the best generalization, maintaining near-flat accuracy decay as problem difficulty increases.

---

## Citation

If you use this pipeline, please cite:

```bibtex
@misc{mathwp_v2_2025,
  title  = {Mathematical Word Problem Generation Pipeline V2},
  year   = {2025},
  note   = {Internal technical report}
}
```

**Reference papers:**
- [Physics of Language Models: Part 2.1, Grade-School Math and the Hidden Reasoning Process](https://arxiv.org/pdf/2407.20311)
- [ControlMath: Controllable Data Generation Promotes Math Generalist Models](https://arxiv.org/abs/2409.15376)

---

## License

Internal research use only.