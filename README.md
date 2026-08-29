<div align="center">

# MeDocVL: Noise-Aware Post-Training of Visual Language Models for Medical Document Parsing

**EMNLP 2026 Findings**

Visual Computing Group (VCG), Ping An Property & Casualty Insurance Company

[![Paper](https://img.shields.io/badge/Paper-EMNLP%20Anthology-b31b1b.svg)](TODO_ANTHOLOGY_URL)
[![arXiv](https://img.shields.io/badge/arXiv-TODO-b31b1b.svg)](TODO_ARXIV_URL)
[![Tech Report](https://img.shields.io/badge/Tech%20Report-MeDocVL-blue.svg)](https://github.com/Dejavuvvw/MeDocVL)
[![Dataset](https://img.shields.io/badge/Dataset-Tianchi%20131815-orange.svg)](https://tianchi.aliyun.com/dataset/131815)
[![License](https://img.shields.io/badge/License-Apache%202.0-green.svg)](LICENSE)

</div>

Official repository for **MeDocVL: Noise-Aware Post-Training of Visual Language Models for Medical Document Parsing**.
This repo hosts the **re-annotated medical-invoice benchmark**, the **controlled noise splits**, and the **evaluation protocol** used in the paper.

> Looking for the extended technical report, method figures and full derivations? See [Dejavuvvw/MeDocVL](https://github.com/Dejavuvvw/MeDocVL).

---

## 📢 News

- **2026-08** — MeDocVL is accepted to **EMNLP 2026 Findings** 🎉
- **2026-01** — Technical report and re-annotated dataset released at [Dejavuvvw/MeDocVL](https://github.com/Dejavuvvw/MeDocVL)

---

## Abstract



---

## Why this task is hard

Medical documents are *not* a transcription problem. A single wrong character in a date, an amount, or an insurance code invalidates the entire field and breaks everything downstream. At the same time, real deployments need **query-driven** access — users ask for fields in natural language rather than against a fixed schema.

|                        | Pipeline OCR + LLM               | General-purpose VLM              | **MeDocVL**                      |
| ---------------------- | -------------------------------- | -------------------------------- | -------------------------------- |
| Query-driven interface | ✗ (needs a downstream parser)    | ✓                                | ✓                                |
| Field-level exactness  | Limited by OCR error propagation | Token-level errors, hallucination | ✓ token-wise optimized           |
| Robust to noisy labels | Not addressed                    | Not addressed                    | ✓ explicitly modeled             |

---

## Dataset

We build on the publicly available medical invoice dataset released by [Alibaba Cloud Tianchi (#131815)](https://tianchi.aliyun.com/dataset/131815) — 800 real-world medical billing documents from diverse scenarios.

**What we changed.** We **re-annotated** the dataset: field definitions were refined and expanded, semantically redundant fields consolidated, and fine-grained fields commonly required in real-world medical invoice processing added. The original document content and the fixed data split are preserved. The same re-annotated labels and evaluation protocol are applied to **every** baseline and to our method, so comparisons are fair and fully reproducible.

Although all documents belong to a single category, medical invoices carry substantial intra-class variability: invoice metadata, medical terminology, itemized billing records, patient fields, printed text, tabular layouts, handwritten content, and official seals — often on one page.

### Files

Under [`dataset/`](dataset/):

| File | Lines | Description |
| ---- | ----- | ----------- |
| `a_li_yun_medical_train.jsonl` | 800 | Full re-annotated training pool |
| `a_li_yun_medical_train_stage1.jsonl` | 700 | Stage-1 split, human-annotated (clean) |
| `a_li_yun_medical_train_stage2.jsonl` | 100 | Stage-2 split, human-annotated (clean) |
| `a_li_yun_medical_train_stage1_noisy.jsonl` | 700 | Stage 1 with **30% injected annotation noise** |
| `a_li_yun_medical_train_stage1_noisy_from_judgeModelPred.jsonl` | 700 | The noisy split after **TLR refinement** (Refined Data) |
| `a_li_yun_medical_test.jsonl` | 150 | Held-out test set, human-annotated |

### Noise injection protocol

Noise is injected into **annotations, not images**. In real pipelines, inputs are pre-filtered so the visual content is relatively reliable, while labels carry non-negligible noise from human error or automatic (OCR/MLLM) labeling.

Corrupted labels are **not** random characters. Values are replaced with *semantically plausible but incorrect* alternatives — valid-looking dates, amounts, or medical terms — preserving surface realism while violating field-level correctness. Noise ratios of 20% / 30% / 50% / 70% are studied in the paper; the released noisy split uses **30%**.

### Record schema

Each line is a chat-format record compatible with common VLM SFT toolkits:

```json
{
  "messages": [
    {"role": "user", "content": "请识别下图发票的内容，返回以下字段及其对应值，即标题、税或非税、…，并严格按照json格式返回。\n<image>"},
    {"role": "assistant", "content": "```json\n{\"标题\":\"河北省医疗住院收费票据\", \"发票号\":\"0113981\", …}\n```"}
  ],
  "solution": "```json\n{…}\n```",
  "images": ["<path to the document image>"]
}
```

- The **user turn** encodes the query-driven setting: the requested field list is given in natural language, with `<image>` marking the document.
- The **assistant turn** / `solution` is the ground-truth key–value JSON. Missing fields are explicitly `"无"` (none) rather than omitted — the model must decide *absence*, not just extraction.
- `solution` duplicates the assistant turn so RL trainers can read the reward reference without parsing the dialogue.

### Setup

1. Download the source images from [Tianchi dataset 131815](https://tianchi.aliyun.com/dataset/131815) (registration required; images are **not** redistributed here).
2. Rewrite the `images` field to point at your local image directory. The released files contain absolute paths from our training machine:

```bash
python - <<'PY'
import json, pathlib
IMG_ROOT = "/your/local/path/a_li_yun_medical_dataset"   # <- edit
for f in pathlib.Path("dataset").glob("*.jsonl"):
    rows = [json.loads(l) for l in f.open()]
    for r in rows:
        r["images"] = [f"{IMG_ROOT}/{p.split('/')[-2]}/{p.split('/')[-1]}" for p in r["images"]]
    with f.open("w") as out:
        for r in rows:
            out.write(json.dumps(r, ensure_ascii=False) + "\n")
PY
```

---


## Citation

```bibtex
@inproceedings{medocvl2026,
  title     = {MeDocVL: Noise-Aware Post-Training of Visual Language Models for Medical Document Parsing},
  author    = {TODO_AUTHOR_LIST},
  booktitle = {Findings of the Association for Computational Linguistics: EMNLP 2026},
  year      = {2026},
  publisher = {Association for Computational Linguistics}
}
```

If you use the benchmark, please also cite the original source dataset:

```bibtex
@misc{tianchi_medical_invoice,
  title  = {Medical Invoice OCR Dataset},
  author = {{Alibaba Tianchi}},
  year   = {2023},
  url    = {https://tianchi.aliyun.com/dataset/131815}
}
```

---

## Contributors

**Project leads:** Liang Diao
**Implementation:**  Wei Wu, Wenjie Wang, Ying Liu, Yuan Zhao, Xiaole Lv

**Contact:** diaoliang145@pingan.com.cn, zhaoyuan041@pingan.com.cn

---

## License

Code and annotations in this repository are released under the [Apache License 2.0](LICENSE).
The **source document images** are governed by the terms of the [Alibaba Cloud Tianchi dataset](https://tianchi.aliyun.com/dataset/131815) and are not redistributed here.

The dataset contains real-world medical billing documents. It is provided **for research use only**; do not use it to attempt re-identification of individuals or institutions.

---

## Acknowledgements

We thank Alibaba Cloud Tianchi for releasing the source medical invoice dataset, and the Qwen team for the Qwen2.5-VL backbone.
