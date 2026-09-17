# P-CULTA: Pakistani Urdu Cultural Pragmatics Benchmark

P-CULTA is a benchmark for evaluating whether LLMs can generate **sociopragmatically appropriate** Urdu — responses that respect deference, indirectness, reciprocity, and face-management norms specific to Pakistani culture, not just fluent or semantically correct text. A fluent, on-topic response can still be culturally inappropriate: e.g. flatly refusing an elder's insistent food offer, instead of first acknowledging the offer before declining.

We evaluate four instruction-tuned LLMs (GPT-4o-mini, Llama-3.1-8B, Qwen2.5-7B, and Qalb-1.0, an Urdu-specialized model) across **zero-shot, few-shot, few-shot fine-tuning, and QLoRA supervised fine-tuning** regimes, under four progressively enriched information conditions: Utterance (U), +Context (U+C), +Roles (U+C+R), +Power Distance (U+C+R+PD).

**Key finding:** sociopragmatic competence does not come from model scale or Urdu-specific pretraining alone. Explicit social information (roles, power distance) and adaptation data both drive large gains, with the best fine-tuned models reaching **0.95 CAS** and **below 0.01 FTV** on held-out data.

---

## Dataset

### Benchmark statistics

| Property | Description |
|---|---|
| Language | Pakistani Urdu |
| Finalized scenarios | 306 |
| Sociopragmatic topics | 17 |
| Instances per topic | 18 |
| Topic share | 5.9% per topic |
| User roles | 50 |
| Model roles | 42 |
| Distinct role dyads | 58 |
| Power distance | Low, Medium, High |
| Register | Formal, Informal |
| Sensitivity | Positive, Neutral, Negative |
| Pragmatic genre labels | 243 |
| Distinct contexts | 305 |
| Distinct user utterances | 304 |
| Distinct gold responses | 306 |
| Initial candidate responses | 918 |
| Reviewers for gold curation | 2 |
| Judges for model evaluation | 2 |

### Sociopragmatic topics

The benchmark contains 17 equally represented topics:

Refusing Offers · Invitation to a Meal · Declining Gifts · Receiving Gifts · Offering Help · Paying for a Meal · Respect for Elders · Family Obligations & Requests · Handling Criticism & Disagreements · Invitation to Celebrations · Asking for Permission · Accepting & Rejecting Compliments · Apologies for Mistakes · Health Enquiry · Giving Orders & Commands · Expressing Gratitude · Negotiating Favors

### Instance structure

Each finalized instance contains:

| Field | Description |
|---|---|
| ID | Unique scenario identifier |
| Topic | Sociopragmatic topic |
| User Role | Social role of the person producing the initial utterance |
| Model Role | Expected respondent's social role |
| Power Distance | Low, Medium, or High |
| Register | Formal or Informal |
| Pragmatic Genre | Strategy through which the topic is realized |
| Sensitivity | Positive, Neutral, or Negative |
| Context | Culturally situated interactional situation |
| User Utterance | Initial utterance in the interaction |
| Curated Gold Response | Culturally grounded response selected during curation |

The benchmark preserves **directed** interpersonal relationships — a Student → Teacher interaction is not treated as equivalent to a Teacher → Student interaction, since expected deference runs in one direction.

### Gold-response curation

Each of the 306 scenarios was paired with **3 candidate responses** (918 total), deliberately varied in sociopragmatic appropriateness rather than fluency. Two academic reviewers independently scored every candidate on the same six dimensions used for model evaluation (below), aggregated into an absolute Cultural Appropriateness score and a Face-Threat/Violation score. The candidate with the **highest appropriateness and lowest threat score** was retained as gold, on reviewer agreement; scenarios with disagreement or no acceptable candidate were revised or discarded.

---

## Evaluation Protocol

Each model response (and the gold response itself) is rated by **two independent judges** on a 0–4 scale across **six sociopragmatic dimensions**:

**Alignment dimensions** (does the response fit the relationship?)
- **Relational Alignment (RA)** — calibration to the User Role → Model Role, power distance, register, and sensitivity of the scenario
- **Indirectness (I)** — a degree of indirectness appropriate to the relationship and topic, not indirectness for its own sake
- **Face-saving (FS)** — whether the response protects the face of speaker and addressee, not just polite-sounding wording
- **Cultural Appropriateness (CA)** — overall fit with Pakistani sociocultural expectations, distinct from generic politeness

**Threat dimensions** (does the response cause harm?)
- **Directness Threat (DT)** — face threat from excessive bluntness or confrontational directness
- **Hierarchy Violation (HV)** — treatment inconsistent with the specified power-distance relationship (directional, like the role structure itself)

Inter-judge agreement is high across all six dimensions (Cohen's weighted κ between 0.987 and 0.998).

### CAS and FTV

The six per-dimension scores are averaged into two complementary metrics:

- **CAS (Cultural Appropriateness Score)** — the normalized deviation of the model response's four alignment dimensions (RA, I, FS, CA) from the gold response's profile on the same dimensions. `CAS ∈ [0, 1]`, higher is better; `CAS = 1` means the response matches gold appropriateness exactly.
- **FTV (Face-Threat/Violation)** — the average of the two threat dimensions (DT, HV) for the model response alone, independent of gold. `FTV ∈ [0, 1]`, lower is better.
- **Bias = 1 − CAS**, reported as an inverted restatement of alignment error, not an independent metric.

A well-calibrated model should approach **CAS → 1** and **FTV → 0** simultaneously — reporting both prevents a response from being credited as high quality on surface politeness alone while masking elevated directness or hierarchy violation. During gold-response curation, an absolute (gold-free) variant of CAS is used instead, since no gold reference exists yet at that stage.

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
