# Notebook Reference

## `checkout_data.ipynb`

### Purpose

Exploratory inspection of the raw label dataset before or alongside modeling work.

### Inputs

- `labels.csv`

### Main Operations

- loads label metadata
- duplicates labels to mirror both drug-order feature arrangements
- groups rows by `cell_line`
- plots KDE distributions for cell-line-specific synergy scores
- aggregates duplicate drug-pair rows with a mean synergy score
- pivots aggregated data into a heatmap per cell line

### Visual Outputs

- cell-line KDE plots
- cell-line drug-pair heatmaps

### Key Details

- example cell line hard-coded in the notebook: `A2058`
- heatmaps use `RdBu` with range `[-150, 150]`
- duplicate `(drug_a_name, drug_b_name, cell_line)` rows are averaged before pivoting

### Why It Matters

This notebook is the quickest place to understand how synergy labels are distributed and how much structure exists per cell line before looking at the model.

## `normalize.ipynb`

### Purpose

Prepares the feature matrix and labels for downstream training.

### Inputs

- `X.p.gz`
- `labels.csv`

### Main Operations

- defines `test_fold` and `val_fold`
- creates train/validation/test index sets from the `fold` column
- slices feature and label arrays into:
  - `X_tr`, `X_val`, `X_train`, `X_test`
  - `y_tr`, `y_val`, `y_train`, `y_test`
- applies custom normalization
- filters zero-variance features
- leaves behind a commented-out pickle serialization step

### Normalization Function

`normalize(...)` performs:

1. feature standard deviation calculation
2. zero-variance feature filtering
3. z-score scaling
4. optional `tanh`
5. optional second normalization pass after `tanh`

### Output Contract

The intended serialized output is a pickle tuple containing:

```python
(X_tr, X_val, X_train, X_test, y_tr, y_val, y_train, y_test)
```

The save command is commented out in the committed notebook, so reproducing the artifact requires manual re-enabling or equivalent code.

### Caveats

- paths are absolute
- the notebook markdown discusses `tanh_norm`, while the code sets `norm = 'tanh'`
- label duplication is commented out here, unlike some of the analysis notebooks

## `model_training.ipynb`

### Purpose

Trains and evaluates the DeepSynergy neural network for one cross-validation run.

### Inputs

- `hyperparameters`
- `data_test_fold0_tanh.p.gz`

### Main Operations

- loads model hyperparameters by executing a local text file
- loads preprocessed arrays from a gzipped pickle
- configures TensorFlow GPU memory growth
- constructs a dense feed-forward Keras model
- trains on `X_tr` / `y_tr` with validation on `X_val` / `y_val`
- evaluates on `X_test` / `y_test`
- predicts continuous synergy scores
- derives classification labels via threshold `30`
- computes accuracy and balanced accuracy
- runs a second epoch-by-epoch evaluation loop for metrics tracking
- saves `.h5` model artifacts
- writes CSV outputs for downstream plotting

### Model Behavior

The architecture is parameterized by the external `hyperparameters` file and follows a standard multilayer perceptron shape:

- hidden dense layers
- dropout after input and intermediate hidden layers
- linear output layer
- SGD optimizer with momentum

### Persisted Outputs

- `model_15_epochs.h5`
- `model.h5`
- `predictions.csv` (commented in one cell)
- `predictions_per_epoch_acc_50_epochs.csv`
- `metrics_per_epoch_50epochs.csv`

### Evaluation Logic

Regression metrics:

- MSE
- RMSE
- MAE

Derived classification metrics:

- accuracy
- balanced accuracy

Threshold rule:

- predicted positive if `prediction > 30`
- ground-truth positive if `y_test > 30`

### Caveats

- uses `exec()` for configuration loading
- depends on files not committed to the repository
- mixes model training, evaluation, persistence, and plotting preparation in one notebook
- early stopping is analyzed manually rather than managed via callbacks

## `plot_results.ipynb`

### Purpose

Creates post-training visual diagnostics from saved prediction files.

### Inputs

- `labels.csv`
- `predictions.csv`

### Main Operations

- recreates fold-based index slices from the labels dataframe
- duplicates labels to align with the prediction structure
- joins predictions onto `plot_y_test`
- selects a specific `(drug_a, drug_b, cell_line)` triple
- overlays its prediction onto a KDE of training synergy values
- aggregates predicted synergy by drug pair and cell line
- pivots aggregated predictions into a heatmap

### Default Example Parameters

- `drug_a = "5-FU"`
- `drug_b = "DINACICLIB"`
- `cell_line = "OCUBM"`

### Visual Outputs

- KDE of training-set synergy values for the chosen cell line
- vertical line showing the model prediction for the chosen drug pair
- predicted-synergy heatmap for the selected cell line

### Caveats

- assumes prediction CSVs already exist and align correctly with the test split
- duplicates labels before joining predictions
- uses hard-coded plot example values rather than parameterized inputs

## ZIP Archives

### `model_training.zip`

Contains a notebook copy of the training workflow.

### `plot_results.zip`

Contains a notebook copy of the plotting workflow.

These archives do not introduce additional code paths in the checked-out repository; they serve as packaged notebook copies.
