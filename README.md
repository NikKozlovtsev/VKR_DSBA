# VKR_DSBA

achelor thesis · HSE University, DSBA · April 2026
Author: N. Kozlovcev

Supplementary code and experimental notebooks for the thesis
**“Pretraining mBERT for text-analysis tasks in chemistry, biochemistry, and related disciplines.”**

What this work is about
The thesis studies efficient adaptation of transformer encoders to chemical
Named Entity Recognition (NER), and how far that adaptation transfers to
Russian, for which there is no biomedical pretraining data.

The research is organised in two stages:

Stage 1 — cheap-strategy screening. On three backbones (BioBERT, mBERT,
DeBERTa-v3) we compare four adaptation recipes: frozen encoder, partial
unfreezing of the top 6 layers (S1), tokenizer-only adaptation (S2), and
full fine-tuning (S3). The goal is to pick a model family and recipe that
are worth scaling up.

Stage 2 — scaled DAPT pipeline. On the chosen BERT-family backbones we
add Domain-Adaptive Pretraining (DAPT) via MLM on BC5CDR + BC4CHEMD, then
fine-tune for chemical NER on BC5CDR, and evaluate cross-lingual transfer
on RuDReC and on a translated BC5CDR-RU.

Everything runs end-to-end on a free-tier Google Colab T4 GPU.

Repository contents
File	Stage	Purpose
s1-s4_strategies.ipynb	Stage 1	Adaptation-strategy screening: Baseline / S1 (top-6 unfreeze) / S2 (tokenizer-only) / S3 (full FT) on BioBERT, mBERT and DeBERTa-v3. Output of this notebook drives the model and recipe choice for Stage 2.
training_for_stage_1.ipynb	Stage 1	Training utilities and supporting runs for the screening stage (loss curves, single-strategy fits, inspection of intermediate checkpoints).
Final_NER_BioBERT_DAPT.ipynb	Stage 2	The main experimental notebook of the thesis: DAPT on BC5CDR + BC4CHEMD, NER fine-tuning on BC5CDR for the four model variants (mBERT ± DAPT, BioBERT ± DAPT), and cross-lingual evaluation on RuDReC and BC5CDR-RU.
res_for_3_models.pdf	Stage 1	Tabulated screening results: F1 / P / R for Baseline, S1, S2, S3 on each of the three backbones. Source of the Stage-1 numbers in the thesis text and pre-defence slides.
README.md	—	This file.
The Stage-2 notebook is the canonical entry point — open it first if you only
want to reproduce the headline numbers.

Models compared (Stage 2)
A 2 × 2 design under one identical fine-tuning recipe:

Model	Pretrain	Vocabulary	Role
mBERT	Wikipedia, 104 languages	multilingual WordPiece	main subject of adaptation
mBERT + DAPT	+ MLM on BC5CDR + BC4CHEMD	multilingual WordPiece	proposed model
BioBERT v1.1	PubMed + PMC	English WordPiece	English upper bound
BioBERT + DAPT	+ MLM on BC5CDR + BC4CHEMD	English WordPiece	upper bound with domain
All four are BERT-base, 12 layers, ~110 M parameters.

Datasets
Dataset	Source	Used as
BC5CDR	tner/bc5cdr (parquet branch)	DAPT corpus (raw text) + EN NER train / test
BC4CHEMD	disi-unibo-nlp/bc4chemd (parquet branch)	DAPT corpus (raw text) only
RuDReC	cimm-kzn/RuDReC (raw JSONL from GitHub)	main RU NER test, few-shot
BC5CDR-RU	translated with Helsinki-NLP/opus-mt-en-ru, 500 sentences	RU sanity-check
All NER tasks are reduced to a 3-tag BIO scheme: O, B-Chemical, I-Chemical.

Frozen experimental protocol
Identical across all four Stage-2 models — any difference in F1 is
attributable to backbone and DAPT, not to hyperparameter search.

seed = 42, max_len = 256, batch = 16, fp16 (T4)

NER fine-tune: 3 epochs, LR 3e-5, warmup 0.1

DAPT (MLM): 3 epochs, LR 5e-5, mlm_probability = 0.15

Metric: token / entity-level Precision, Recall and F1 via seqeval

Single seed at this stage; multi-seed re-runs are listed under Limitations
in the thesis.

Headline results
Setup	BC5CDR F1 (EN)	RuDReC F1 (RU, zero-shot)
mBERT	0.8949	—
mBERT + DAPT	0.9030 (+0.81 vs no-DAPT)	0.290
BioBERT + DAPT	0.9245 (English upper bound)	0.000
Intrinsic check: mBERT MLM perplexity drops 11.77 → 5.31 (−54.9 %) after
DAPT — intrinsic and extrinsic signals agree.

The strongest qualitative finding is that **mBERT + DAPT transfers zero-shot
to Russian RuDReC at F1 ≈ 0.29 with no Russian biomedical pretraining data**,
while BioBERT collapses to 0 because its English WordPiece vocabulary cannot
represent Cyrillic input. Vocabulary coverage of the target language outweighs
domain specialisation when no target-language pretraining data is available.

Full numerical tables, including Stage-1 screening, are in res_for_3_models.pdf
and in the thesis text.

How to reproduce
Open the chosen notebook in Google Colab with a T4 runtime.

Run cells top-to-bottom — no manual configuration files, all hyperparameters
are literals in the code.

Datasets are pulled at runtime from the Hugging Face Hub (parquet branches
for BC5CDR and BC4CHEMD) or directly from GitHub (RuDReC raw JSONL); no
local download is required.

Single full run on T4 covers DAPT + NER fine-tune + evaluation in a few
hours per model variant.

Recommended order:
s1-s4_strategies.ipynb → training_for_stage_1.ipynb → Final_NER_BioBERT_DAPT.ipynb.

Status and limitations
Pre-defence stage. Numbers reported here are preliminary single-seed
runs. Multi-seed averaging (mean ± std over 3 seeds) and a few-shot RuDReC
baseline with the full RU train split are scheduled before the final defence.

Compute-capped. All training fits inside Colab free-tier T4 limits;
larger backbones (XLM-R-large, DeBERTa-v3-large) are not in scope.

BC5CDR-RU is a sanity-check, not a benchmark — it is machine-translated
and partly measures preserved Latin chemical names.

DAPT and NER share BC5CDR. This follows standard DAPT practice
(Gururangan et al., 2020) but is documented as a transductive setup in the
thesis; a BC4CHEMD-only DAPT control is on the next-steps list.

Citation
If you use the code or numbers from this repository, please cite the bachelor
thesis:

Kozlovcev N. *Pretraining mBERT for text-analysis tasks in chemistry,
biochemistry, and related disciplines.* Bachelor thesis, HSE University,
DSBA, 2026.
