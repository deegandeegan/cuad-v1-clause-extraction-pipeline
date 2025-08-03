# CUAD v1 Clause Extraction Notebook

Extract key clauses from legal contracts using CUAD v1. This repository provides an end to end, step by step Jupyter notebook: preprocessing, dataset creation, token classification model training, validation, inference on new PDFs, and optional fuzzy matching utilities.

## Features
- Clean preprocessing and JSONL creation for CUAD v1
- Token classification with Hugging Face Transformers
- Seqeval metrics for entity level evaluation
- Inference on long PDF contracts using sliding windows
- Post processing to merge overlaps and build per file summaries
- Optional fuzzy matching helpers to complement model spans

## Repository structure
```
.
├── notebooks/
│   └── CUAD_V1 - Full Code Step by Step.ipynb
├── data/
│   ├── raw/            # original CUAD v1
│   ├── interim/        # cleaned JSONL from Step 7
│   └── new_pdfs/       # unseen PDFs for Step 14
├── outputs/
│   └── model/          # trained model and tokenizer, predictions.csv
└── README.md
```

If you prefer paths without spaces, you can rename the notebook to `CUAD_V1_Full_Code_Step_by_Step.ipynb`. Either works on GitHub.

## Quick start
1. Create and activate a Python environment.
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Windows: .venv\Scripts\activate
   pip install -U pip
   ```

2. Install dependencies.
   ```bash
   pip install -r requirements.txt
   ```

3. Open the notebook.
   ```bash
   jupyter lab  # or jupyter notebook
   ```

4. Follow the Steps inside the notebook. Steps 1 to 7 produce a cleaned, model ready JSONL. Steps 8 to 12 train. Step 13 evaluates. Step 14 runs inference on new PDFs. Step 14B cleans and summarizes predictions.

## Data
- CUAD v1 contracts and annotations
- Place raw files under `data/raw/`
- The pipeline writes a cleaned JSONL to `data/interim/` during Step 7

## Configuration
Set the key paths near the top of the notebook:
- `MASTER_PATH` base folder for data and outputs
- `DRIVE_DIR` persistent storage location
- `LOCAL_DIRS` local fallbacks that the resolver scans
- `OUTPUT_DIR` model and artifacts destination
- `NEW_PDFS_PATH` folder with unseen PDFs for inference

## Training, Steps 8 to 12
- Uses `AutoTokenizer` and `AutoModelForTokenClassification`
- `TrainingArguments` control batch size, learning rate, evaluation strategy, and checkpointing
- After training, the notebook saves only essentials to `OUTPUT_DIR` and removes large artifacts

## Evaluation, Step 13
**What it does**
- Runs `trainer.predict(val_dataset)`
- Converts predictions to BIO tags and aligns with non padding tokens, skips `-100`
- Computes entity level Precision, Recall, and F1 using `seqeval`
- Prints a compact token preview for manual inspection

**Why it matters**
- Measures generalization beyond training loss
- The token preview helps find BIO drift and label confusions

**Output**
- Printed `seqeval` classification report
- Token preview for one validation example

## Inference on new PDFs, Step 14
**What it does**
- Reads only unseen PDFs from `NEW_PDFS_PATH`. If empty, raises an error and stops
- Loads model and tokenizer from `OUTPUT_DIR`
- Extracts and lightly cleans text per PDF
- Applies sliding window inference with stride, then decodes BIO to character spans
- Collects rows with `filename, source_dir, clause, start, end, pred_text`
- Writes `outputs/model/predictions.csv` by default

**Why it matters**
- Evaluates the model on real contracts
- Sliding windows prevent missing entities that cross chunk boundaries

**Output**
- `predictions.csv` with one row per predicted span

## Cleaning and summary, Step 14B
**What it does**
- Loads `predictions.csv`
- Drops error rows and tiny spans
- Merges overlaps per file and clause, keeps the longest text
- Creates a per file wide summary of top spans

**Why it matters**
- Produces readable results for QA or downstream use
- Removes duplicates and noise

**Output**
- In memory DataFrames for cleaned spans and a wide summary
- Optional CSVs if you set `SAVE=True`
