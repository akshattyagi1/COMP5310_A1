# COMP5310 Assignment 2 - NSW Train Crowding Classification

## Purpose

This project supports an NSW train operations team to identify station-stop observations that may be crowded, so potential capacity or timetable reviews can be prioritised. `Crowded` is 1 for `FEW_SEATS_AVAILABLE` or `STANDING_ROOM_ONLY`, and 0 for `MANY_SEATS_AVAILABLE`.

Predictions are decision-support prompts for investigation, not automatic instructions to change a service.

## Files and run order

```text
COMP5310_A2/
├── notebooks/
│   ├── 01_data_pipeline.ipynb
│   └── 02_modelling_evaluation.ipynb
├── data/
│   ├── raw/train_occupancy.csv                 # supplied from Canvas when packaging
│   └── processed/train_occupancy_model_ready.csv
├── README.md
└── requirements.txt
```

1. Install the packages in `requirements.txt`.
2. Place the original `train_occupancy.csv` in `data/raw/`. In the full repository, the pipeline can also find the Assignment 1 copy at `../../data/raw/train_occupancy.csv`.
3. From `COMP5310_A2/notebooks/`, run `01_data_pipeline.ipynb` from top to bottom. It creates `../data/processed/train_occupancy_model_ready.csv` with 49,927 records and the `Crowded` target.
4. Run `02_modelling_evaluation.ipynb` from top to bottom. It loads that exact output and reproduces the baseline, Logistic Regression and Random Forest results.

## Reproducibility

- Relative paths are used throughout.
- The train/test split and both models use `random_state=42`.
- The 80/20 split is stratified by `Crowded`.
- Imputation, encoding and scaling are fitted within a scikit-learn pipeline on training data only.
- Expected runtime is approximately 1-3 minutes on a standard laptop; Random Forest is the slowest step.

## Implementation record: changes made in Assignment 2

This section records the changes made when the initial Assignment 2 notebooks were reviewed. It is included so that every change can be traced to a clear reason.

| Step | What was found | Change made | Why it matters |
|---|---|---|---|
| 1. Review existing work | The data pipeline cleaned the raw data but did not create `Crowded` or save the output promised in its objectives. | Completed the cleaning pipeline, created `Crowded`, validated the final dataset and saved `train_occupancy_model_ready.csv`. | The modelling notebook now uses a traceable output from the pipeline rather than a manually selected earlier file. |
| 2. Preserve Assignment 1 cleaning evidence | The earlier work established the correct 49,927-row cleaned dataset but the Assignment 2 pipeline recorded only part of the evidence. | Added a cleaning log covering whitespace, inconsistent labels, missing targets, exact duplicates and timestamp handling. | This documents the evidence, action, justification and before/after impact required for a reproducible pipeline. |
| 3. Connect the notebooks | The modelling notebook loaded the Assignment 1 cleaned CSV directly. | Changed it to require the output of `01_data_pipeline.ipynb`. | Running the notebooks in order now reproduces the data used for all reported models. |
| 4. Validate the target | `Crowded` was previously created inside the modelling notebook. | Moved target creation to the pipeline and added an assertion that it agrees with `Occupancy Status`. | Target definition is now created once and checked before modelling. |
| 5. Control obvious leakage | Occupancy fields and actual timestamps could reveal the outcome or be unavailable at prediction time. | Excluded `Occupancy Status`, `Occupancy Range`, actual arrival/departure times, and the actual departure-time band from features. | This reduces target leakage and avoids relying on information that may not exist when a decision is made. |
| 6. Test feature availability risk | `Trip.Name` is a repeated service identifier. | Added a train/test overlap diagnostic. It found 2,919 trip IDs in both random splits. | The benchmark may reflect prediction for known services. It must not be presented as proven performance for entirely unseen services. |
| 7. Make experiments traceable | Model settings and metrics were spread across separate cells. | Added an experiment log and a shared results table for the baseline, Logistic Regression and Random Forest. | Each reported comparison can be traced to its data, features, split, settings and metric. |
| 8. Examine decision trade-offs | The initial comparison used the default 0.50 Logistic Regression threshold only. | Added threshold results at 0.25, 0.50 and 0.75. | This shows the stakeholder how higher recall causes more false alarms, while higher precision creates more missed crowding. |
| 9. Analyse model errors | Aggregate metrics alone do not show where errors occur. | Added confusion-matrix figures and tables of false negatives and false positives by service line and station. | These outputs support the required error analysis and responsible stakeholder recommendation. |

## Verified pipeline outcome

Running the completed pipeline on the raw dataset gives:

- 50,250 raw records;
- 73 records removed because `Occupancy Status` is missing;
- 250 redundant exact duplicate rows removed;
- 49,927 final records;
- 1,994 crowded records (3.99%);
- zero remaining exact duplicates.

## Learning guide: confusion matrices and error analysis

### Why we use a confusion matrix

A confusion matrix is a four-number table that compares the known result in the test data with the model's prediction. It does **not** train or change a model. It is a way to inspect the errors made after the model has predicted the held-out test set.

For this project, the actual outcome is whether a station-stop observation was crowded. The rows are the actual status and the columns are the prediction:

|  | Predicted not crowded | Predicted crowded |
|---|---:|---:|
| Actually not crowded | True negative (TN) | False positive (FP) |
| Actually crowded | False negative (FN) | True positive (TP) |

- **True negative:** correctly leaves a non-crowded observation unflagged.
- **True positive:** correctly flags a crowded observation for review.
- **False positive (false alarm):** flags an observation that was not crowded. This can waste staff time or prompt an unnecessary review.
- **False negative (missed crowding):** fails to flag an observation that was crowded. This can leave passengers facing avoidable discomfort, poor service experience and possible safety concerns during busy periods.

Accuracy combines all correct results, `(TP + TN) / all records`. It can be misleading here because 96.01% of records are not crowded. A model could predict "not crowded" every time and still look accurate, despite never identifying a single crowding case. The confusion matrix makes this problem visible.

### Exact test-set results

All models used the same stratified 80/20 test set: 9,986 records, including 399 actually crowded observations and 9,587 not-crowded observations. Rows below are actual classes; columns are predicted classes.

#### Majority-class baseline

| Actual status | Predicted not crowded | Predicted crowded |
|---|---:|---:|
| Not crowded | 9,587 | 0 |
| Crowded | 399 | 0 |

The baseline achieves 96.00% accuracy simply by always predicting not crowded. Its crowded-class recall is 0%, because it finds none of the 399 genuine crowding observations. This is why it is a reference point, not a useful operational solution.

#### Logistic Regression

| Actual status | Predicted not crowded | Predicted crowded |
|---|---:|---:|
| Not crowded | 9,000 | 587 |
| Crowded | 44 | 355 |

Interpretation:

- It correctly identifies **355 of 399** crowded observations.
- It misses **44** crowded observations.
- It produces **587** false alarms.
- Crowded-class recall is `355 / (355 + 44) = 88.97%`.
- Crowded-class precision is `355 / (355 + 587) = 37.69%`.
- Crowded-class F1 is 0.5295 and accuracy is 93.68%.

This model is useful when missing a crowded service is the more serious risk. However, approximately 62% of its crowding flags are false alarms, so staff should treat a flag as a reason to investigate, not an automatic instruction to add capacity.

#### Random Forest

| Actual status | Predicted not crowded | Predicted crowded |
|---|---:|---:|
| Not crowded | 9,520 | 67 |
| Crowded | 217 | 182 |

Interpretation:

- It correctly identifies **182 of 399** crowded observations.
- It misses **217** crowded observations.
- It produces only **67** false alarms.
- Crowded-class recall is `182 / (182 + 217) = 45.61%`.
- Crowded-class precision is `182 / (182 + 67) = 73.09%`.
- Crowded-class F1 is 0.5617 and accuracy is 97.16%.

Random Forest is more selective: most of its flags are correct, but it misses more than half of the actual crowding observations. Its higher accuracy and F1 do not override the pre-selected primary metric, crowded-class recall.

### How the confusion-matrix figure is made

The modelling notebook uses `ConfusionMatrixDisplay.from_predictions(...)` to draw one matrix for each candidate model. The code uses the already-created `y_test`, `y_pred` and `rf_pred` values. It does not refit either model or change any metrics. The figure is saved as `figures/01_confusion_matrices.png` so it can be used in the report after visual checking.

### Why inspect errors by line and station

The confusion matrix gives totals only. The following diagnostic tables group Logistic Regression errors by service line and stop station, so the group can ask whether recurring errors need further investigation.

| Error type | Highest-count service-line pattern | Records |
|---|---|---:|
| False negative | City Circle | 11 |
| False negative | North | 6 |
| False negative | North Shore | 6 |
| False positive | City Circle | 116 |
| False positive | North Shore | 93 |
| False positive | West | 75 |

| Error type | Highest-count station pattern | Records |
|---|---|---:|
| False negative | Central | 12 |
| False negative | Strathfield | 5 |
| False negative | Town Hall | 3 |
| False positive | Central | 53 |
| False positive | Redfern | 42 |
| False positive | Town Hall | 41 |

These are **counts, not error rates**. Busy lines and stations naturally generate more records, so a large count is not evidence that a line or station causes the model to fail. Before making a service-specific claim, compare the error count with the number of test observations in that group and test the pattern on later data.

### Decision lesson from these results

The stakeholder must choose the acceptable trade-off:

- If the priority is to miss as little crowding as possible, Logistic Regression is the better initial decision-support model because it misses 44 cases rather than 217.
- If staff can investigate only a small number of flags and false alarms are very expensive, Random Forest may be more practical, but it carries a much greater risk of missed crowding.
- Neither model should automatically alter a timetable or train capacity. A prediction should trigger a review using operational information not included in this dataset.

## Leakage controls and limits

`Occupancy Status` is the source of the target and `Occupancy Range` is closely related to it, so both are excluded from predictors. Actual arrival/departure timestamps and the derived departure-time band are also excluded until their availability at the operational decision time is confirmed.

`Trip.Name` appears in both the random training and test sets. This benchmark can therefore reflect prediction for later observations of known services and should not be interpreted as performance for entirely unseen services. Future validation should test a group-by-trip or later-time split.
