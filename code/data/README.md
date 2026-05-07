# MAHED Task 1 Gemini Prompting Experiment

This repository contains Colab notebooks for running and evaluating a Gemini-based Arabic text classification experiment on **MAHED Task 1**.

The experiment classifies each Arabic text into one of three labels:

- `hate`
- `hope`
- `not_applicable`

The main goal is to compare different prompt formulations while keeping the experimental setup fixed.

---

## 1. Files

| File | Purpose |
|---|---|
| `Evaluation.ipynb` | Main experiment notebook. Loads data, creates or loads the validation subset, runs Gemini inference, saves predictions, resumes incomplete runs, and computes metrics. |
| `prompt_templates.ipynb` | Utility notebook for printing and inspecting prompt templates and few-shot examples before running the full experiment. |
| `visualization.ipynb` | Analysis notebook for merging result files, recomputing metrics, generating comparison tables, confusion matrices, and plots. |

---

## 2. What should stay fixed

When running the experiment on another Colab account, keep the experiment settings unchanged unless you are intentionally creating a different experiment.

Do **not** change:

- model name
- subset size
- batch size
- sleep time
- labels
- metric code
- resume logic
- output file structure
- few-shot example sampling settings
- validation subset creation settings
- random seed

Only change:

1. file paths
2. prompt setting
3. Gemini API key setup

---

## 3. Colab setup

### 3.1 Install required packages

Run this in Colab before executing the notebooks:

```python
!pip install -q -U google-genai scikit-learn pandas matplotlib seaborn
```

### 3.2 Mount Google Drive

```python
from google.colab import drive
drive.mount('/content/drive')
```

---

## 4. Dataset requirements

The experiment expects two CSV files:

- `train.csv`
- `validation.csv`

Both files must contain these columns:

```text
text
label
```

The labels must remain exactly:

```text
hate
hope
not_applicable
```

The code automatically normalizes label text by stripping spaces and converting labels to lowercase, but the label set itself should not be changed.

---

## 5. Change the dataset paths

In `Evaluation.ipynb`, find:

```python
TRAIN_PATH = "/content/drive/MyDrive/496/MAHED Sub Task 1/MAHED Sub Task 1/train.csv"
VAL_PATH = "/content/drive/MyDrive/496/MAHED Sub Task 1/MAHED Sub Task 1/validation.csv"
```

Replace them with the actual paths in your own Google Drive.

Example:

```python
TRAIN_PATH = "/content/drive/MyDrive/MyProject/MAHED/train.csv"
VAL_PATH = "/content/drive/MyDrive/MyProject/MAHED/validation.csv"
```

To get the correct path in Colab:

1. Mount Google Drive.
2. Open the Files panel.
3. Find `train.csv` or `validation.csv`.
4. Right-click the file.
5. Select **Copy path**.
6. Paste the copied path into `TRAIN_PATH` or `VAL_PATH`.

Use the same path changes in `prompt_templates.ipynb` if you want to inspect prompts before running the experiment.

---

## 6. Change the output path

In `Evaluation.ipynb`, find:

```python
OUTPUT_DIR = "/content/drive/MyDrive/496/MAHED_Gemini_Task3"
```

Replace it with a folder path in your own Google Drive.

Example:

```python
OUTPUT_DIR = "/content/drive/MyDrive/MyProject/MAHED_Gemini_Task3"
```

This folder is only the saving location. Changing it does not change the experiment design.

The notebook creates and saves files such as:

```text
mahed_val_subset_500.csv
few_shot_examples.csv
metrics_summary.csv
metrics_per_class.csv
confusion_matrices.json
results_<prompt_type>_gemini_2_5_flash.csv
```

Note: the result filename suffix is part of the current code. Do not rename it unless you also update the visualization notebook accordingly.

---

## 7. Gemini API key setup

The code expects a Colab secret named exactly:

```text
gemini
```

In the new Colab account:

1. Open the Colab **Secrets** panel.
2. Add a new secret named exactly `gemini`.
3. Paste the Gemini API key as the value.
4. Enable notebook access for the secret.

The relevant code is:

```python
api_key = userdata.get("gemini")
```

Do not rename the secret unless you also update this line in the code.

The code also supports using an environment variable named `GEMINI_API_KEY`, but the intended Colab setup is the `gemini` secret.

---

## 8. Select the prompt setting

In `Evaluation.ipynb`, find:

```python
PROMPT_TYPES = ["three_shot_cultural"]
```

Replace it with the prompt type or prompt types you want to run.

Available prompt types in the experiment notebook are:

```python
PROMPT_TYPES = ["zero_shot_definition"]
```

```python
PROMPT_TYPES = ["zero_shot_nli"]
```

```python
PROMPT_TYPES = ["zero_shot_distinction"]
```

```python
PROMPT_TYPES = ["few_shot_definition"]
```

```python
PROMPT_TYPES = ["three_shot_cultural"]
```

To run multiple prompts in one experiment:

```python
PROMPT_TYPES = [
    "zero_shot_definition",
    "few_shot_definition",
    "zero_shot_nli"
]
```

The notebook will create a separate result file for each prompt type.

---

## 9. Using an NLI-style prompt

The notebook already includes a `zero_shot_nli` prompt type.

To run it directly, use:

```python
PROMPT_TYPES = ["zero_shot_nli"]
```

If you want to replace the current zero-shot definition prompt with a different NLI-style formulation, edit only the prompt text inside:

```python
build_batch_prompt(...)
```

Keep the same labels and output format.

Example replacement for the `zero_shot_definition` block:

```python
if prompt_type == "zero_shot_definition":
    return f"""For each Arabic text, decide which label is best supported by the text.

Labels:
hate = the text expresses hate, hostility, abuse, or harmful meaning toward a person or group
hope = the text expresses support, encouragement, empathy, or constructive meaning
not_applicable = the text does not clearly express hate or hope

Return exactly one line per text using this format:
1: hate
2: hope
3: not_applicable

Use only these labels: hate, hope, not_applicable.
Do not explain.

Texts:
{numbered_texts}""".strip()
```

This changes only the prompt formulation while keeping the experiment fixed.

---

## 10. Running the main experiment

After changing only the paths, prompt setting, and API key setup, run `Evaluation.ipynb` from top to bottom.

The notebook will automatically:

1. load `train.csv` and `validation.csv`
2. clean and validate labels
3. create or load the fixed 500-sample validation subset
4. create or load few-shot examples
5. run Gemini inference in batches
6. save row-level predictions
7. resume from the last unfinished row if Colab stops
8. compute summary metrics
9. compute per-class metrics
10. save confusion matrices

---

## 11. Resume behavior

The experiment saves progress after each batch.

If Colab disconnects or the run stops, run the notebook again. The code will load the existing result file and continue only the unprocessed rows.

Do not delete the result CSV unless you want to restart that prompt from the beginning.

---

## 12. Output files

The main output folder contains:

| Output file | Description |
|---|---|
| `mahed_val_subset_500.csv` | Fixed validation subset used for evaluation. |
| `few_shot_examples.csv` | Fixed few-shot examples sampled from the training set. |
| `results_<prompt_type>_gemini_2_5_flash.csv` | Row-level predictions for each prompt type. |
| `metrics_summary.csv` | Overall metrics for the selected prompt types. |
| `metrics_per_class.csv` | Per-class precision, recall, F1, and support. |
| `confusion_matrices.json` | Confusion matrices for each prompt type. |

Each row-level result file includes columns such as:

```text
sample_id
text
label
prompt_type
model
pred
raw_output
processed
correct
batch_id
timestamp
```

---

## 13. Running prompt inspection

Use `prompt_templates.ipynb` before the full run if you want to inspect the exact prompts.

Update the same paths:

```python
TRAIN_PATH = ".../train.csv"
VAL_PATH = ".../validation.csv"
OUTPUT_DIR = ".../MAHED_Gemini_Task3"
```

Then run the notebook. It prints examples of the prompt templates and few-shot examples without running the full Gemini evaluation.

---

## 14. Running visualization and comparison

Use `visualization.ipynb` after one or more prompt runs have finished.

In the first cell, set:

```python
RESULTS_DIR = "/content/"
```

Change it to the folder that contains the result files, metrics files, or copied outputs.

Example:

```python
RESULTS_DIR = "/content/drive/MyDrive/MyProject/MAHED_Gemini_Task3"
```

The visualization notebook creates a folder named:

```text
comparison_outputs
```

It can generate:

```text
merged_summary_metrics.csv
table_overall_prompt_comparison.csv
table_per_class_prompt_comparison.csv
fig_overall_prompt_comparison.png
fig_classwise_hate.png
fig_classwise_hope.png
fig_classwise_not_applicable.png
fig_cm_<prompt>_count.png
fig_cm_<prompt>_normalized.png
prediction_distribution_by_prompt.csv
prompt_mcnemar_fliprate_stats.csv
ANNOTATE_group_A_shared_failures.csv
ANNOTATE_group_B_prompt_specific.csv
```

If the notebook asks for `SAMPLE_PATH`, set it to the same fixed subset file:

```python
SAMPLE_PATH = "/content/drive/MyDrive/MyProject/MAHED_Gemini_Task3/mahed_val_subset_500.csv"
```

---

## 15. Important reproducibility note

Do not change any other configuration unless you are intentionally creating a different experiment.

Changing any of the following makes the results less comparable:

- `MODEL_NAME`
- `SUBSET_SIZE`
- `BATCH_SIZE`
- `SLEEP_SEC`
- `SEED`
- `VALID_LABELS`
- few-shot sampling settings
- validation subset creation settings
- metric computation code
- output file naming

For a comparable run on another Colab account, only update:

1. `TRAIN_PATH`
2. `VAL_PATH`
3. `OUTPUT_DIR`
4. `PROMPT_TYPES`
5. Colab secret named `gemini`
