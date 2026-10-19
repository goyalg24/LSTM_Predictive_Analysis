# LSTM_End-to-End_Predictive_Analytics

An end-to-end predictive analytics workflow for reactor process data, made up of a
series of Jupyter notebooks. They preprocess and explore the batch-process dataset
(`data/Data.xlsx`, sheet `Samples`, 4,380 rows), engineer and select features
(PCA and Random Forest feature importance), and train a two-layer LSTM in
TensorFlow/Keras that predicts **Reactor Temperature** from sequences of the 10
preceding time steps of Reactor Temperature, Agitation, Solution Concentration and
Volume.

## Project structure

```
.
├── data/
│   ├── Data.xlsx                              # raw process data (Date-Time, Agitation, Batch ID, Operation,
│   │                                          #   Phase, Product, Reactor Temperature, Solution Concentration, Volume)
│   ├── First_Two_Principal_Components.xlsx    # PC1/PC2 written by notebook 03
│   └── PCA_Results.xlsx                       # PCA components (PC1–PC4) + explained variance (not produced by the current notebooks)
├── notebooks/
│   ├── 01_Data_Preprocessing.ipynb            # missing values, summary statistics, IQR outlier detection + box plots
│   ├── 02_Exploratory_Data_Analysis.ipynb     # trends over time, distributions, correlation heatmap, per-product / per-batch analysis
│   ├── 03_Feature_Engineering.ipynb           # standardisation, PCA, PC–feature correlations, Random Forest feature importance + CV
│   └── 05_LSTM_Model.ipynb                    # LSTM training, loss curves, actual vs. predicted, per-feature importance, metrics
├── requirements.txt
└── README.md
```

There is no notebook `04` in the repository. The numbering jumps from 03 to 05.

## Setup

```bash
pip install -r requirements.txt
```

`openpyxl` is listed in `requirements.txt` because `pandas.read_excel` / `to_excel`
need it for `.xlsx` files. The LSTM notebook was last run in a
`py3.7-tf2.0` conda environment (TensorFlow 2.0).

## Usage

Launch Jupyter and run the notebooks in order from the `notebooks/` folder:

```bash
jupyter notebook notebooks/
```

Each notebook loads its data with the relative path `../data/Data.xlsx`, so Jupyter's
working directory must be `notebooks/`. That is the default when you open a notebook
from there. Notebook 03 writes `../data/First_Two_Principal_Components.xlsx`, and
notebook 05 saves the trained model as `lstm_temperature_prediction_model.h5` in
`notebooks/`. That file is git-ignored.

## Results (from the saved notebook outputs)

- **Feature engineering (notebook 03).** Random Forest predicting Reactor Temperature,
  5-fold cross-validation:
  - all features: mean R² = 0.873
  - reduced to Solution Concentration + Agitation: mean R² = 0.869
- **LSTM (notebook 05).** 20 epochs, 80/20 chronological split: test MSE loss of
  0.0066 on min-max-scaled data.
- **Per-feature LSTM importance (notebook 05).** Each feature was paired with the
  target, and the test loss was recorded (lower = more important): Reactor Temperature
  0.0084, Volume 0.0092, Agitation 0.0100, Solution Concentration 0.0119.
- **Evaluation metrics cell (notebook 05).** The saved output shows MSE = 93,739.53,
  MAE = 218.14 and R² = 0.8985. This cell runs after the per-feature loop, which
  overwrites `model`, `X_test` and `scaler`. The numbers therefore describe the last
  model from that loop (Volume + Reactor Temperature), not the main 4-feature LSTM.
