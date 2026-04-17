# Project Overview

## What This Repository Is

This repository captures a notebook-driven implementation of a DeepSynergy-style workflow for predicting drug-combination synergy scores. The committed code focuses on four stages:

1. Inspect the raw labeled dataset.
2. Normalize the feature matrix and derive fold-based splits.
3. Train and evaluate a dense neural network in TensorFlow/Keras.
4. Visualize prediction behavior with KDE plots and heatmaps.

The notebooks are the application. There is no reusable package, CLI entrypoint, environment file, or module hierarchy in the repository.

## High-Level Intent

The workflow models a continuous synergy score for a triple:

- `drug_a`
- `drug_b`
- `cell_line`

It then derives a binary classification view of the same task by thresholding synergy at `30`, which is the cutoff used in `model_training.ipynb` for accuracy and balanced-accuracy calculations.

## Repository Contents

| File | Purpose | Notes |
| --- | --- | --- |
| `checkout_data.ipynb` | Examines the raw `labels.csv` distribution and generates exploratory KDE plots and heatmaps. | Useful for understanding label density per cell line before modeling. |
| `normalize.ipynb` | Loads the feature matrix and labels, creates fold splits, normalizes features, and prepares a serialized dataset. | The pickle save is present but commented out. |
| `model_training.ipynb` | Loads normalized data, reads hyperparameters, builds a multilayer perceptron, trains it, evaluates it, and writes prediction artifacts. | Uses TensorFlow/Keras and scikit-learn metrics. |
| `plot_results.ipynb` | Joins training outputs with label metadata and plots prediction-oriented KDE charts and heatmaps. | Assumes prediction CSVs already exist on disk. |
| `model_training.zip` | Archived notebook bundle for training. | Contains `model_training.ipynb`. |
| `plot_results.zip` | Archived notebook bundle for plotting. | Contains `plot_results.ipynb`. |

## External Inputs and Outputs

The repository references several files that are not committed here.

| Path or Name | Expected By | Type | Role |
| --- | --- | --- | --- |
| `/home/nidhi/Documents/freelancing/DeepSynergy/data/labels.csv` | `checkout_data.ipynb`, `normalize.ipynb`, `plot_results.ipynb` | CSV | Raw label metadata with synergy scores and fold assignments. |
| `/home/nidhi/Documents/freelancing/DeepSynergy/data/X.p.gz` | `normalize.ipynb` | gzipped pickle | Input feature matrix for drug-pair and cell-line examples. |
| `hyperparameters` | `model_training.ipynb` | text file executed as Python | Defines model-layer and optimizer parameters. |
| `/home/nidhi/Documents/freelancing/DeepSynergy/data/data_test_fold0_tanh.p.gz` | `model_training.ipynb` | gzipped pickle | Normalized split dataset generated from preprocessing. |
| `/home/nidhi/Documents/freelancing/DeepSynergy/data/Results/predictions.csv` | `plot_results.ipynb` | CSV | Predicted synergy values aligned to the test split. |
| `/home/nidhi/Documents/freelancing/DeepSynergy/data/Results/predictions_per_epoch_acc_50_epochs.csv` | `model_training.ipynb` | CSV | Per-epoch prediction dump from the second training loop. |
| `/home/nidhi/Documents/freelancing/DeepSynergy/data/Results/metrics_per_epoch_50epochs.csv` | `model_training.ipynb` | CSV | Per-epoch regression and classification metrics. |

## Inferred Data Contract

From the notebook code, the label file is expected to provide at least these columns:

- `drug_a_name`
- `drug_b_name`
- `cell_line`
- `synergy`
- `fold`

The feature matrix `X.p.gz` is expected to align row-for-row with the label dataframe used during normalization. The notebooks mention that the feature matrix stores both orderings of each drug pair:

- `drug A -> drug B -> cell line`
- `drug B -> drug A -> cell line`

That duplication is important because some notebooks explicitly duplicate the label dataframe to match the doubled feature representation.

## Runtime Dependencies

The imports across the notebooks imply these dependencies:

- `numpy`
- `pandas`
- `matplotlib`
- `seaborn`
- `pickle`
- `gzip`
- `tensorflow` / `keras`
- `scikit-learn`

No lockfile or environment specification is included, so dependency versions must be inferred or recreated separately.

## Main Technical Characteristics

- Notebook-centric workflow rather than modular Python code.
- Five-fold data partitioning using a `fold` column.
- Custom feature normalization implemented manually in NumPy.
- Dense feed-forward neural network with configurable hidden layers.
- Regression as the primary task, with thresholded classification metrics used as secondary evaluation.
- Result interpretation focused on:
  - per-cell-line synergy distributions
  - specific drug-pair prediction lookup
  - heatmap visualization of predicted synergy landscapes

## Constraints and Maintenance Risks

- Hard-coded absolute paths make the notebooks non-portable without edits.
- Missing source data and hyperparameter files prevent direct re-execution from this repository alone.
- Important artifacts are written outside the repository tree.
- Some outputs are only saved in commented lines, which makes the workflow partly manual.
- Logic is duplicated across notebooks instead of being centralized in reusable code.
