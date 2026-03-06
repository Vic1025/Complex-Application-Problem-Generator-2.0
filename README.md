# Mathematical Word Problem Generation Pipeline V2

A scalable, programmatic pipeline for synthesizing hierarchically structured mathematical word problems with controllable complexity. Built on and extending the Physics of Language Models (Part 2.1) framework, with significant advances to support multi-level entity hierarchies, long reasoning chains (5–30+ steps), and diverse logical relationships.

---
Patent No: CN20688502A

## Motivation

Training language models to reason through complex, multi-step math problems requires data that simply doesn't exist at scale in the wild. Standard benchmarks like GSM8K top out at a few reasoning steps; real-world problems requiring 10–30 interdependent computations have no adequate training source.

This project addresses that gap by building a **fully programmatic generation engine** that can produce arbitrarily complex, verifiable math word problems — with ground-truth answers, step-by-step chain-of-thought solutions, and fine-grained difficulty control. Problems are grounded in realistic domain scenarios (factories, farms, logistics, finance, and more) and are structurally guaranteed to be solvable and consistent.

---

## How It Works

The pipeline has four stages:

**1. Hierarchical Entity Library Construction**  
A library of 50 domain-specific entity sets is used as the foundation. Each set organizes entities into 3–5 classification levels (e.g., Factory → Production Line → Production Unit), with 4–10 elements per level. Entities are assigned systematic composite codes (e.g., `CBB` = Caesar Factory's Food Processing Line's Unit B), giving every entity a unique, resolvable identity. Any parent-child containment relationship — and any arithmetic relation — is structurally embedded in this naming scheme.

**2. Logic Tree Synthesis**  
A directed tree is grown programmatically from a randomly sampled starting entity. Nodes represent entities; edges represent arithmetic or containment dependencies. The queried entity (the question target) sits at the root; known values sit at the leaves. Tree depth, answer path length, layer width, and entity count are all configurable parameters, allowing continuous difficulty scaling from simple (op5) to extremely hard (op30+).

**3. Problem Rendering**  
The logic tree is mapped onto natural language using ~70 paraphrase templates for condition expressions. Problems can be rendered in four variants that test different reasoning skills: containment-based conditions (harder, more implicit) vs. explicit arithmetic, and ordered vs. shuffled condition presentation. Redundant distractor conditions can be injected to further increase difficulty.

**4. Chain-of-Thought Generation and Polishing**  
Step-by-step solutions are first generated programmatically, then refined by GPT-4o for fluency and logical consistency. This two-stage approach achieves approximately 90% usable output rate across mixed-difficulty datasets.

---

## Key Properties

- **Unlimited scale**: The 50 entity libraries collectively support ~1 billion unique entity combinations. Combined with parameterized tree generation, the problem space is effectively inexhaustible.
- **Verified correctness**: All problems and answers are generated deterministically from the logic tree — there is no hallucination risk in the ground truth.
- **Continuous difficulty control**: Problem complexity is parameterized precisely. A single configuration change scales from textbook-level to competition-level difficulty.
- **Four linguistic variants per problem**: Allows testing of both arithmetic reasoning and natural language comprehension skills independently.
- **Diverse domains**: 50 entity sets spanning manufacturing, agriculture, logistics, retail, finance, and more.

---

## Experimental Results

Models trained with this pipeline show strong improvements on complex multi-step reasoning. Key findings from experiments on Qwen2.5-3B:

| Setting | MATH-500 | GSM8K | op0–10 | op10–15 | op15–20 |
|---|---|---|---|---|---|
| Base (no training) | — | — | 25% | 3% | 0% |
| Base + Synthetic RL | 57% | 84% | **95%** | 81% | 43% |
| Cold-start SFT (500) + RL | 55% | 78% | 91% | 65% | **41%** |
| Base + Math RL | 64% | 86% | 25% | 3% | 0% |
| Base + Mixed RL | **66%** | **88%** | 46% | 6% | 1% |

*op = number of unknown entities (problem complexity)*

Three findings stand out:

- **Synthetic domain data dramatically boosts hard-problem reasoning.** RL trained purely on this pipeline's data raises op0-10 accuracy from 25% to 95% (+276%), and unlocks non-zero performance on op25-30 problems the base model cannot solve at all.
- **"Less SFT is more."** Full-scale SFT before RL causes the model to over-fit output format, suppressing its ability to extend reasoning chains dynamically. A cold-start of only 500 SFT samples preserves reasoning flexibility while still establishing the required output structure.
- **Mixed-difficulty SFT generalizes best.** Training on problems spanning op0–op20 produces the most robust model, with near-flat accuracy degradation as difficulty increases.

---

## Related Work

This project extends and builds on:

- Ye, T., Xu, Z., Li, Y., and Allen-Zhu, Z. (2025). *Physics of Language Models: Part 2.1, Grade-School Math and the Hidden Reasoning Process.* ICLR 2025. [[arXiv]](https://arxiv.org/abs/2407.20311)
- Chen, N., Wu, N., Chang, J., Shou, L., and Li, J. (2024). *ControlMath: Controllable Data Generation Promotes Math Generalist Models.* EMNLP 2024. [[arXiv]](https://arxiv.org/abs/2409.15376)
- Shao, Z. et al. (2024). *DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open Language Models.* [[arXiv]](https://arxiv.org/abs/2402.03300)

---

## License

This project is proprietary. All rights reserved.
