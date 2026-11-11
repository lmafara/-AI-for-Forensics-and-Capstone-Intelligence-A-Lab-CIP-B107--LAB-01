import pypandoc, os, textwrap
md = r"""# ICDFA CIP-B107-LAB-01 — Project README

**Student:** NASIRU LAWAL  
**Student ID:** C11/26/DFIT/17305  
**Case ID:** ICDFA-DFIR-AI-026  
**Course:** CIP-B107 — AI for Forensics and Capstone Intelligence A  
**Lab reference:** CIP-B107-LAB-01

## 1. Project purpose

This project demonstrates an offline workflow for identifying and reviewing possible IPv4-address candidates in supplied synthetic forensic teaching data. The workflow preserves source context, validates candidate syntax, trains a local text classifier, compares it with simple baselines, and records evaluation limitations.

**Interpretation warning:** `Positive` and `Negative` are source-dataset annotations. They do not prove that an address is malicious, benign, publicly routable, attributable to a person, or connected to an incident. A model prediction is a triage aid only.

## 2. Folder structure

The project is organised as follows:

```text
CIP-B107/
├── originals/                 # Supplied evidence files; preserve unchanged
├── work/                      # Reproducible Python scripts
├── outputs/                   # Generated CSVs, models, metrics and charts
│   └── charts/                # Performance and profiling charts
├── README.md                  # This file
└── ...
```

Do not include `.venv/`, API keys, passwords, access tokens, browser profiles, or other secrets in the submission ZIP.

## 3. Environment

The recorded analysis environment was:

- Operating system: Kali Linux
- Python interpreter: `/usr/bin/python`
- Python package versions recorded during analysis:
  - pandas 2.3.3
  - scikit-learn 1.9.0
  - matplotlib 3.10.9
- Dependencies listed in `outputs/requirements.txt`

The recorded runs used the system Python interpreter; no virtual environment was active. Package versions should be checked on the submission machine before reproducing results. The exact Python version and `joblib` version were not recorded in the captured project notes, so they are not asserted here.

## 4. Setup

From the project directory:

```bash
cd ~/CIP-B107

python --version
python -m pip --version

python -m pip install -r outputs/requirements.txt
```

If pip reports that system packages are externally managed, create a local virtual environment for reproduction (do not include it in the submission ZIP):

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r outputs/requirements.txt
```

## 5. Reproducing the analysis

Run the scripts from the project root (`~/CIP-B107`) so the relative input and output paths resolve correctly.

### Phase B — candidate extraction and forensic test cases

```bash
python work/ip_candidate_analysis.py
python work/forensic_test_cases.py
```

The candidate review output is:

```text
outputs/candidate_review.csv
outputs/forensic_test_cases.csv
```

The extraction retained candidate strings and their original context. Strict IPv4 validation was performed separately from source labelling.

### Phase C — offline model

```bash
python work/phase_c_model.py
python work/train_model.py
python work/test_saved_model.py
```

The project contains more than one model artefact because it includes a secondary random-split experiment and a fixed JSONL holdout evaluation. Do not treat their results as interchangeable.

Expected model artefacts include:

```text
outputs/ip_classifier.pkl
outputs/fixed_holdout_classifier.pkl
outputs/phase_c_prepared.csv
```

### Fixed-holdout evaluation and leakage sensitivity

```bash
python work/fixed_holdout_evaluation.py
python work/leakage_sensitivity.py
python work/phase_e_evaluation.py
```

Important outputs:

```text
outputs/predictions.csv
outputs/fixed_holdout_metrics.txt
outputs/leakage_reduced_predictions.csv
outputs/leakage_sensitivity.txt
outputs/phase_e_predictions.csv
outputs/phase_e_metrics.csv
outputs/phase_e_evaluation_summary.txt
```

### Historical result interpretation and charts

```bash
python work/analyze_historical.py
python work/create_charts.py
```

The historical metrics are derived from the supplied `originals/result.csv`. They are not results from the student's offline classifier. The supplied historical scripts that upload data or create remote fine-tuning jobs were not executed.

### Eight-record analyst review

```bash
python work/create_analyst_review.py
```

Output:

```text
outputs/eight_record_analyst_review.csv
```

Complete the `analyst_assessment` and `analyst_rationale` fields through independent human review. Do not infer maliciousness from source labels or IPv4 syntax alone.

## 6. Recorded results

### Fixed 46-record holdout

| Evaluation | Accuracy | Precision | Recall | F1-score | TN | FP | FN | TP |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Offline model | 97.83% | 97.14% | 100% | 98.55% | 11 | 1 | 0 | 34 |
| Rule-based baseline | 95.65% | 94.44% | 100% | 97.14% | 10 | 2 | 0 | 34 |
| Majority-class baseline | 73.91% | 73.91% | 100% | 85.00% | 0 | 12 | 0 | 34 |

### Leakage sensitivity

Thirty exact prompts overlapped between the supplied training and test files. Removing overlapping prompts from training reduced the training set from 183 to 157 records. On the same 46 test records, accuracy decreased to **82.61%** (confusion matrix: TN=4, FP=8, FN=0, TP=34).

This is a sensitivity analysis, not fully independent external validation. The result is a warning that the original 97.83% fixed-holdout accuracy may be optimistic; near-duplicate text may remain.

### Historical supplied experiment

The supplied `result.csv` contains 733 rows, with final non-null metrics at step 733:

- Accuracy: 97.83%
- Precision: 97.14%
- Recall: 100%
- F1-score: 98.55%
- AUROC: 99.75%
- AUPRC: 99.91%

Classification metrics are present in only 4 of 733 rows. `fine_tuning_steps.txt` records 734 steps, so the two historical files have a one-step count discrepancy. These are historical supplied results, not a newly performed remote fine-tuning run.

## 7. Key outputs for the submission

At minimum, include:

- `outputs/evidence_manifest.csv`
- `outputs/candidate_review.csv`
- `outputs/predictions.csv`
- `outputs/phase_e_predictions.csv`
- `outputs/phase_e_metrics.csv`
- `outputs/eight_record_analyst_review.csv`
- `outputs/historical_metrics_summary.txt`
- `outputs/leakage_sensitivity.txt`
- `outputs/charts/*.png`
- `work/*.py`
- Signed forensic report PDF
- Executive briefing
- Genuine, timestamped screenshots of the student's own terminal/notebook outputs

Keep the original evidence files in `originals/` unchanged. Include them in the submission only if the lab submission instructions require them.

## 8. Evidence integrity and final checks

The evidence manifest records SHA-256 hashes for five supplied source files. Before submission, verify each original file against the manifest:

```bash
cd ~/CIP-B107
sha256sum originals/dataset_small.csv
sha256sum originals/dataset_train.jsonl
sha256sum originals/dataset_test.jsonl
sha256sum originals/result.csv
sha256sum originals/fine_tuning_steps.txt
```

Compare each displayed hash character-for-character with `outputs/evidence_manifest.csv`. If any hash differs, stop and investigate before submitting.

Check that the expected outputs exist:

```bash
find work outputs -maxdepth 2 -type f -print | sort
```

Capture screenshots directly from your own Kali terminal showing the commands and successful outputs. Do not fabricate, edit, or present generated screenshots as live terminal evidence.

## 9. Limitations and responsible interpretation

- The data are synthetic teaching artefacts.
- The dataset is small and class-imbalanced.
- Exact prompt overlap affected the supplied fixed train/test split.
- Source labels may disagree with strict IPv4 syntax; those disagreements were retained for review.
- A syntactically valid IPv4 value is not necessarily a usable host address or relevant forensic indicator.
- The offline model was not validated on an independent real-world forensic dataset.
- The historical remote fine-tuning scripts were treated as reference material and not executed.
- No claim of compromise, maliciousness, ownership or attribution is made.

**Recommended use:** analyst assistance only. Validate syntax, inspect original context, preserve source records, and obtain independent corroboration before drawing forensic conclusions.
"""
out="/mnt/data/README_CIP-B107.md"
pypandoc.convert_text(md, "md", format="md", outputfile=out, extra_args=["--standalone"])
print("Created:", out, "Bytes:", os.path.getsize(out))
