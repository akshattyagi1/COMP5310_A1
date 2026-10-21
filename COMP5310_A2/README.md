# COMP5310 Assignment 2 — Group 208

This project compares Logistic Regression and Random Forest with a majority-class baseline to identify crowded train observations.

## How to run

1. Install the dependencies with `pip install -r requirements.txt`.
2. Keep the raw dataset at `data/raw/train_occupancy.csv`.
3. Open Jupyter from the `notebooks` folder.
4. Run `01_data_pipeline.ipynb` and then `02_modelling_evaluation.ipynb` from top to bottom.

## Files and outputs

- `01_data_pipeline.ipynb` cleans the raw data and creates `data/processed/train_occupancy_model_ready.csv`.
- `02_modelling_evaluation.ipynb` trains and evaluates the models. All tables and figures are displayed directly in the notebook.
- `requirements.txt` lists the required Python packages.

The cleaned dataset contains 49,927 observations. Running both notebooks may take approximately 5–15 minutes, depending on the computer.

## Reproducibility

The project uses relative paths and random seed 42. Preprocessing is fitted using the training data only. The code was tested with Python 3.11.9.

## Main limitation

Both models perform substantially worse when tested on unseen trip identifiers. Further validation is required before either model could be considered for operational use.
