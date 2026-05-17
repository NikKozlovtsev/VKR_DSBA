# VKR_DSBA — mBERT + DAPT for chemical NER, with cross-lingual extension to Russian

Bachelor thesis · HSE University, DSBA · 2026
**Author:** Nikita Kozlovtsev

Supplementary code and notebooks for the bachelor thesis
**"Pretraining mBERT for text-analysis tasks in chemistry, biochemistry, and
related disciplines."**

---

## What this work is about

The thesis studies **domain adaptation** of pre-trained Transformer encoders
for chemical Named Entity Recognition (NER), and whether the same recipe
transfers to a Cyrillic-script target language (Russian) for which no
biomedical pretraining data is available.

The research is organised in **three stages**:

1. **Stage 1 — strategy screening.** On three backbones (BioBERT, mBERT,
   DeBERTa-v3) we compare four adaptation recipes (frozen / top-6 unfreezing
   / tokenizer-only / full fine-tuning) on BC5CDR. The goal is to pick a
   model family and recipe that are worth scaling up.
2. **Stage 2 — DAPT pipeline.** On the chosen BERT-family backbones we add
   Domain-Adaptive Pre-Training (DAPT) via MLM on BC5CDR ∪ BC4CHEMD, then
   fine-tune for chemical NER on BC5CDR. We also measure intrinsic gain
   (MLM perplexity) and cross-lingual zero-shot transfer to RuDReC.
3. **Stage 3 — supervised RuDReC + ablation.** We fine-tune mBERT directly
   on RuDReC train and run a four-lever ablation (L0–L3) to isolate the
   contribution of (a) target-language supervision, (b) cross-lingual
   English DAPT, and (c) raw Russian MLM data.

Everything runs end-to-end on a free-tier Google Colab T4 GPU.

---

## Repository layout

```
stage1/                    strategy-screening notebooks (4 recipes × 3 backbones)
  s1-s4_strategies.ipynb
  training_for_stage_1.ipynb
  res_for_3_models.pdf

stage2/                    DAPT + fine-tuning pipeline on BC5CDR ∪ BC4CHEMD
  Final_NER_BioBERT_DAPT.ipynb

stage3/                    cross-lingual supervised RuDReC + L0–L3 ablation
  Stage3_RuDReC_supervised_clean.ipynb
  stage3_results.png       (Figure 5 of the thesis)

data/                      dataset pointers (no data committed)
  README.md

results/                   CSV outputs behind every table in the thesis
  stage3_levers.csv

thesis/                    LaTeX source and final PDF (added on submission)
  main.tex
  main.pdf

requirements.txt           pinned package versions
LICENSE                    MIT
README.md                  this file
```

The Stage 3 notebook `stage3/Stage3_RuDReC_supervised_clean.ipynb` is the
canonical entry point for the final headline numbers.

---

## Headline results

### Stage 2 — English (BC5CDR)

| Setup           | BC5CDR F1   |
|-----------------|-------------|
| mBERT           | 0.8949      |
| mBERT + DAPT    | **0.9030**  |
| BioBERT + DAPT  | 0.9245      |

Intrinsic check — mBERT MLM perplexity on the chemical corpus drops from
**1.10 × 10⁵ → 3.78** after DAPT.

### Stage 3 — Russian (RuDReC, 4-lever ablation)

| Lever | Backbone | DAPT             | Supervised on RuDReC | RuDReC test F1 |
|-------|----------|------------------|----------------------|----------------|
| L0    | BioBERT  | —                | — (zero-shot)        | 0.085          |
| L1    | mBERT    | —                | ✓                    | **0.927**      |
| L2    | mBERT    | EN (BC4CHEMD)    | ✓                    | 0.922          |
| L3    | mBERT    | EN + RU raw      | ✓                    | 0.920          |

Figure 5 of the thesis is `stage3/stage3_results.png`.

---

## Main finding

- On the **source language without supervision**, DAPT is the main lever
  (mBERT +0.0081 F1 on BC5CDR; zero-shot to RuDReC moves from 0 to 0.290).
- On the **target language with supervision**, DAPT **saturates**: L1 vs L2
  vs L3 differ by less than 0.01 F1, and supervised mBERT pulls the metric
  from 0.085 (BioBERT zero-shot) up to 0.927.

The thesis interprets this as a *labels-vs-pretraining* trade-off: domain
vocabulary and continued MLM dominate when no target-language labels
exist; once labels are present, the contribution of further DAPT washes
out.

---

## Frozen experimental protocol

Identical across all stages — any difference in F1 is attributable to the
listed lever, not to hyperparameter search.

- `seed = 42`, `max_len = 256`, `batch = 16`, `fp16` (T4)
- NER fine-tune: 5 epochs, LR 3 × 10⁻⁵, warm-up 0.10
- DAPT (MLM): 3 epochs, LR 5 × 10⁻⁵, `mlm_probability = 0.15`
- Metric: entity-level Precision / Recall / F1 via strict `seqeval`
- 3-tag BIO scheme: `O`, `B-Chemical`, `I-Chemical`
- RuDReC drug labels collapsed: `Drugname` / `Drugclass` / `Drugform` →
  `Chemical`

---

## How to reproduce

1. Open the chosen notebook in **Google Colab** with a T4 runtime.
2. Run cells top-to-bottom — no config files, all hyperparameters are
   literals.
3. Datasets are pulled at runtime; nothing is committed to this repo.
4. A full Stage 3 run (DAPT + fine-tune + eval) finishes in a few hours
   per lever on a free-tier T4.

Recommended order to reproduce thesis numbers:

```
stage1/s1-s4_strategies.ipynb
stage2/Final_NER_BioBERT_DAPT.ipynb
stage3/Stage3_RuDReC_supervised_clean.ipynb
```

Pinned package versions are in `requirements.txt`.

---

## Datasets

| Dataset  | Source                                | Used as                                  |
|----------|---------------------------------------|------------------------------------------|
| BC5CDR   | `tner/bc5cdr` (HF Hub, parquet)       | DAPT corpus + English NER train / test   |
| BC4CHEMD | `disi-unibo-nlp/bc4chemd` (HF Hub)    | DAPT corpus only                         |
| RuDReC   | `cimm-kzn/RuDReC` raw JSONL on GitHub | Russian NER train / test (Stage 3)       |

Direct RuDReC URL:
`https://raw.githubusercontent.com/cimm-kzn/RuDReC/master/data/rudrec_annotated.json`

---

## Status and limitations

- Single-seed runs at this stage; multi-seed averaging is listed as future
  work in Section 7 of the thesis.
- All training fits inside Colab free-tier T4 limits; larger backbones
  (XLM-R-large, DeBERTa-v3-large) are out of scope.
- The Stage 3 ablation uses a single RuDReC train / test split
  (70 / 15 / 15); k-fold cross-validation is on the next-steps list.

---

## Citation

> Kozlovtsev N. *Pretraining mBERT for text-analysis tasks in chemistry,
> biochemistry, and related disciplines.* Bachelor thesis, HSE University,
> DSBA, 2026.

The thesis PDF is in `thesis/main.pdf` once the final version is
submitted.
