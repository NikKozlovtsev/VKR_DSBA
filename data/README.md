# Datasets

Data is fetched at runtime by the notebooks; nothing is committed to git.

| Dataset  | Source                                                                 | Used as                                  |
|----------|------------------------------------------------------------------------|------------------------------------------|
| BC5CDR   | `tner/bc5cdr` (Hugging Face Hub, parquet branch)                       | DAPT corpus + English NER train / test   |
| BC4CHEMD | `disi-unibo-nlp/bc4chemd` (Hugging Face Hub, parquet branch)           | DAPT corpus only                         |
| RuDReC   | `cimm-kzn/RuDReC` (raw JSONL on GitHub)                                | Russian NER train / test (Stage 3)       |

Direct RuDReC URL used by Stage 3:

```
https://raw.githubusercontent.com/cimm-kzn/RuDReC/master/data/rudrec_annotated.json
```

Note the **absence** of a `rudrec/` sub-folder in the URL — the file lives
directly under `data/` in the upstream repo.

### Label mapping (Stage 3)

To stay compatible with the English experiments, RuDReC drug-related
labels are collapsed into a single `Chemical` class:

| RuDReC label | Mapped to    |
|--------------|--------------|
| `Drugname`   | `Chemical`   |
| `Drugclass`  | `Chemical`   |
| `Drugform`   | `Chemical`   |
| `DI`         | dropped      |
| `ADR`        | dropped      |

The final BIO scheme is 3-tag: `O`, `B-Chemical`, `I-Chemical`.
