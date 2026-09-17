# P-CULTA: Pakistani Urdu Cultural Pragmatics Benchmark

P-CULTA is a benchmark for evaluating whether LLMs can generate **sociopragmatically appropriate** Urdu — responses that respect deference, indirectness, reciprocity, and face-management norms specific to Pakistani culture, not just fluent or semantically correct text.

The benchmark contains **306 Pakistani interactional scenarios** across **17 sociopragmatic topics** (e.g. refusing offers, asking permission, giving orders, expressing gratitude). Each scenario specifies:

- **User Role → Model Role** (interlocutor identities, e.g. student → teacher)
- **Power Distance** (High / Medium / Low)
- **Register** (Formal / Informal)
- **Sensitivity** (Positive / Neutral / Negative)
- **Pragmatic Genre** (the interactional strategy, e.g. "deferential refusal")
- **Context** and **User Utterance**
- A **curated gold response**, selected from 3 candidate responses per scenario via two-reviewer scoring and majority agreement

We evaluate four instruction-tuned LLMs (GPT-4o-mini, Llama-3.1-8B, Qwen2.5-7B, and Qalb-1.0, an Urdu-specialized model) across **zero-shot, few-shot, few-shot fine-tuning, and QLoRA supervised fine-tuning** regimes, under four progressively enriched information conditions: Utterance (U), +Context (U+C), +Roles (U+C+R), +Power Distance (U+C+R+PD).

Each response is scored by two independent judges on six dimensions (Relational Alignment, Indirectness, Face-saving, Cultural Appropriateness, Directness Threat, Hierarchy Violation), aggregated into two metrics:

- **CAS (Cultural Appropriateness Score)** — alignment with the gold response profile (higher is better)
- **FTV (Face-Threat/Violation)** — intensity of face-threatening or hierarchy-violating behavior (lower is better)

**Key finding:** sociopragmatic competence does not come from model scale or Urdu-specific pretraining alone. Explicit social information (roles, power distance) and adaptation data both drive large gains, with the best fine-tuned models reaching **0.95 CAS** and **below 0.01 FTV** on held-out data.

---

## Repository Structure

```
P-CULTA/
├── Dataset_files/                     # The benchmark itself
│   ├── P_CULTA_V2_306.csv             # Full 306-instance benchmark (all metadata + gold responses)
│   ├── P_CULTA_V2.csv                 # 255-item evaluation split (306 minus the 51-item few-shot/fine-tuning bank)
│   ├── fewshot_examples.csv           # 51-item few-shot/fine-tuning demonstration bank (disjoint from evaluation split)
│   ├── fewshot_examples_17_set1/2/3.csv  # Three disjoint 17-item demonstration subsets drawn from the 51-item bank
│   └── Gold_response_curation/        # Provenance of gold-response selection (candidate scores, curation notebook, inter-annotator agreement)
│
├── codes/                             # Notebooks for generation, fine-tuning, and evaluation
│   ├── Baseline/                      # Zero-shot prompting — GPT-4o-mini, Llama, Qwen, Qalb
│   ├── fewshot_GPT/, fewshot_Llama/, fewshot_Qwen/, fewshot_Qalb/   # Few-shot prompting per model (Sets 1–3, all 4 info conditions)
│   ├── fewshot_fineteuning_llama/qalb/qwen/  # QLoRA few-shot fine-tuning (17/51-item scales)
│   ├── fewshot_finetuning_evaluation/ # Scoring/aggregation for few-shot fine-tuned outputs
│   ├── sft_llama/qalb/qwen/           # QLoRA supervised fine-tuning on the 70/30 held-out split
│   └── sft_evaluation/                # Scoring/aggregation for the 70/30 SFT held-out evaluation
│
├── Generations/                       # Raw model outputs per regime × model × condition
├── annotations/                       # Two-judge (judge1/judge2) sociopragmatic ratings + mapping keys, per regime × condition
└── Results/                           # Aggregated CAS/FTV/Bias, inter-annotator κ, and plots (Tables 1–4 in the paper)
```

## Reading the Dataset

- **`Dataset_files/P_CULTA_V2_306.csv`** is the complete benchmark — one row per scenario, with Topic, User Role, Model Role, Power Distance, Register, Pragmatic Genre, Sensitivity, Context, User Utterance, and the curated Gold Response.
- **`Dataset_files/P_CULTA_V2.csv`** is the 255-item subset used for evaluation once the 51-item few-shot/fine-tuning bank is held out.
- **`fewshot_examples*.csv`** are the disjoint demonstration items used only for in-context examples and fine-tuning — never part of the evaluation set.
- **`Dataset_files/Gold_response_curation/`** documents how each gold response was chosen: 3 authored candidates per scenario, scored by two reviewers on the six sociopragmatic dimensions, with the highest-CAS/lowest-FTV candidate retained on reviewer agreement.

## Reproducing Results

1. **Generation** — run the notebooks in `codes/Baseline/` (zero-shot) or `codes/fewshot_*` (few-shot) to produce outputs into `Generations/`.
2. **Fine-tuning** — run `codes/fewshot_fineteuning_*` (17/51-item QLoRA) or `codes/sft_*` (70/30 held-out QLoRA).
3. **Evaluation** — generations are rated by two judges (`annotations/`) and aggregated into CAS/FTV/Bias with Cohen's weighted κ via `codes/sft_evaluation/` and `codes/fewshot_finetuning_evaluation/`, producing the results in `Results/`.
