# Bike Sharing Demand – Model Training and Error Analysis (DS8)

## Overview

This project applies and compares **two regression models**—**Linear Regression** and a **Decision Tree Regressor (moderate model)**—on the **Kaggle Bike Sharing Demand dataset**. The workflow is organized into three practical tasks that progressively move from baseline modeling to diagnostic error analysis.

The primary objective is not only to compare overall performance, but also to **understand when and why models fail** using time-based error slicing.

---

## Dataset

* **Source**: Kaggle – Bike Sharing Demand
* **File used**: `train.csv` and `test.csv`
* **Target variable**: `count` (total bike rentals per hour)

---

## Practical 1: Linear Regression (Baseline Model)

### Objective

To establish a **baseline model** and highlight the limitations of linear assumptions when applied to non-linear, time-dependent demand data.

### Practical Steps

1. Load the dataset and parse the `datetime` column
2. Extract time-based features:

   * `hour`, `day`, `month`, `year`
3. Drop non-predictive or leakage-prone columns:

   * `datetime`, `casual`, `registered`
4. Split the dataset into **train / validation / test** sets
5. Apply **feature scaling** using `StandardScaler`
6. Train a **Linear Regression** model
7. Evaluate performance using:

   * Mean Absolute Error (MAE)
   * Root Mean Squared Error (RMSE)
8. Visualize results:

   * Target distribution
   * Actual vs Predicted plot

### Validation and Test Metrics

* **Validation MSE**: 19,742.83
* **Validation R²**: 0.383
* **Test MSE**: 20,166.45
* **Test R²**: 0.390

### Validation and Test Metrics

**Baseline Decision Tree (Untuned)**

* **Validation MSE**: 3,549.81
* **Validation R²**: 0.889

**Tuned Decision Tree Regressor**

* **Validation MSE**: 4,255.57
* **Validation R²**: 0.867

### Key Observations

* Both Decision Tree models significantly outperform Linear Regression in terms of error reduction and explained variance
* The **baseline (untuned) tree** achieves the lowest validation MSE but risks overfitting due to unrestricted growth
* The **tuned Decision Tree**, while slightly worse on validation metrics, provides better generalization and stability for downstream error analysis
* The tuned model is therefore selected for post-hoc error slicing and interpretability

---

## Practical 2: Decision Tree Regressor (Moderate Model)

### Objective

To improve predictive performance using a **moderate-complexity model** capable of capturing non-linear relationships while controlling overfitting.

### Model Configuration

* `max_depth = 10`
* `min_samples_leaf = 20`
* `random_state = 42`

### Practical Steps

1. Reuse the same feature engineering pipeline as Task 1
2. Train an **untuned Decision Tree** as a baseline
3. Train a **tuned Decision Tree Regressor** with controlled depth
4. Compare validation performance against the baseline
5. Evaluate the final model on the test set
6. Save reusable artifacts:

   * Trained model (`joblib`)
   * Hyperparameters (`json`)
   * Feature column order (`json`)

### Key Observations

* Decision Tree significantly reduces error compared to Linear Regression
* Better captures peak-hour demand patterns
* Still exhibits weaknesses during periods of heterogeneous demand

---

## Practical 3: Error Slicing by Hour Bins (Post-hoc Analysis)

### Objective

To diagnose **time-dependent failure modes** by slicing prediction errors into hour bins:

* `0–5`
* `6–11`
* `12–17`
* `18–23`

This analysis is applied to **both models**, with emphasis on the Decision Tree Regressor.

### Practical Steps

1. Load the saved Decision Tree model and configuration files
2. Recreate features exactly as during training
3. Generate predictions on the full dataset
4. Compute:

   * Prediction error (Actual − Predicted)
   * Absolute error
5. Assign hour bins using `pd.cut`
6. Aggregate error metrics per bin:

   * Mean Absolute Error (MAE)
   * Root Mean Squared Error (RMSE)
   * Sample count
7. Visualize results:

   * MAE bar plot by hour bin
   * FacetGrid of absolute error distributions

### Key Findings

* **Linear Regression**:

  * Fails most during **18–23**
  * Strong underprediction during peak demand

* **Decision Tree Regressor**:

  * Highest **average error in 12–17** due to mixed demand behavior
  * More extreme outliers in **18–23**
  * Best performance in **0–5**, where demand is stable

---

## Comparative Summary

| Model                   | Primary Failure Bin | Reason                                         |
| ----------------------- | ------------------- | ---------------------------------------------- |
| Linear Regression       | 18–23               | High bias, inability to model non-linear peaks |
| Decision Tree Regressor | 12–17               | Heterogeneous demand patterns                  |

---

## How to Run the Project

1. Mount Google Drive (if using Colab)
2. Given data csv in the `bike_data/` directory
3. Run notebooks in sequence:

   1. `easy.ipynb`
   2. `moderate.ipynb`
   3. `hard.ipynb`
4. Inspect saved model artifacts in `easy` for the easy(Linear Regression Model Configs for Practical 3) and in `moderate` for the easy(Decision Regression Tree Model Configs for Practical 3)

---

## Conclusion

This three-task workflow demonstrates that **aggregate metrics alone are insufficient** for evaluating time-dependent demand models. Error slicing reveals **when models fail**, not just **how much they fail**. While increasing model complexity improves performance, systematic errors remain during heterogeneous demand periods, highlighting the importance of diagnostic analysis in real-world forecasting systems.
