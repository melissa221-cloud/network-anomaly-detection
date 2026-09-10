# Network Anomaly Detection with Machine Learning

A supervised machine learning project for classifying network traffic as **normal** or **attack** using the **NSL-KDD** intrusion detection dataset.

The project covers the complete workflow from exploratory data analysis and preprocessing to model training, evaluation, feature importance, and error analysis.

## Project Overview

The goal is to build and compare machine learning models that can distinguish legitimate network connections from malicious traffic.

### Models

- Logistic Regression — baseline model
- Random Forest — tree-based ensemble model
- XGBoost — gradient boosting model

### Main tasks

- Explore and understand the NSL-KDD dataset
- Convert the original multi-class target into a binary target
- Encode categorical features using one-hot encoding
- Prepare train/test datasets with stratification
- Standardize features for Logistic Regression
- Train and compare three machine learning models
- Evaluate predictions using Accuracy, Precision, Recall, and F1-score
- Analyze false positives and false negatives
- Identify important features using XGBoost

## Dataset

- **Dataset:** NSL-KDD
- **File used:** `KDDTrain+.txt`
- **Size:** 125,973 rows × 43 columns
- **Features:** 41 traffic features + `label` + `difficulty_level`
- **Target:** `attack_flag`
  - `0` = normal
  - `1` = attack

The original `label` contains `normal` and multiple attack types. For this project, it is converted into a binary classification target.

Class distribution in the training file:

- **67,343 normal connections**
- **58,630 attack connections**

The dataset is therefore reasonably balanced, so no heavy class resampling was required.

> The raw dataset and generated CSV files are intentionally excluded from the repository through `.gitignore`. Download the dataset from the official source to reproduce the project.

## Project Structure

```
network-anomaly-detection/
├── 01_exploration.ipynb   # EDA, preprocessing, modeling and evaluation
├── README.md
└── .gitignore
```

## Methodology

### 1. Data preparation

The notebook keeps two DataFrames with different purposes:

| | `df` | `df_clean` |
|---|---|---|
| Purpose | Exploration and visualization | Machine learning |
| Original categorical columns | Kept | One-hot encoded |
| `label` | Available for exploration | Removed from modeling data |
| Modeling | No | Yes |

The categorical features:

- `protocol_type`
- `service`
- `flag`

are encoded with **one-hot encoding** using `pd.get_dummies()`.

One-hot encoding is preferred here because the categories do not have a natural numerical order. For example, assigning `tcp = 0`, `udp = 1`, and `icmp = 2` could incorrectly suggest an ordinal relationship.

The encoded dataset contains **123 columns**.

No missing values or duplicate rows were found during the initial exploration.

### 2. Exploratory Data Analysis

The notebook includes:

- Dataset dimensions and structure
- Missing-value analysis
- Duplicate analysis
- Class distribution
- Correlation analysis
- Feature distribution analysis

The most correlated numeric feature with `attack_flag` was:

`dst_host_srv_serror_rate`

The boxplot showed that normal traffic is generally concentrated near zero, while attack traffic has a wider distribution.

> Correlation is used here as an exploratory tool. It does not mean that this feature is automatically the most important feature for the final machine learning model.

### 3. Train/Test Split

The data is divided using an **80/20 stratified split**:

- 80% training
- 20% testing

`stratify=y` is used to preserve the class distribution in both sets.

A fixed `random_state=42` is used for reproducibility.

### 4. Feature Scaling

`StandardScaler` is applied to the training data and then used to transform the test data.

The scaled data is used for **Logistic Regression**.

Tree-based models such as Random Forest and XGBoost are trained on the original numeric feature values and do not require standardization.

### 5. Model Training and Evaluation

Three models are compared using the same test set.

| Model | Accuracy | F1-score | Precision | Recall |
|---|---:|---:|---:|---:|
| Logistic Regression | 0.99 | 0.99 | 0.99 | 0.99 |
| Random Forest | TBD | TBD | TBD | TBD |
| XGBoost | TBD | TBD | TBD | TBD |

The evaluation uses:

- **Accuracy** — overall proportion of correct predictions
- **Precision** — proportion of predicted attacks that are actually attacks
- **Recall** — proportion of real attacks that are detected
- **F1-score** — balance between precision and recall

For an intrusion/anomaly detection system, **Recall for the attack class is particularly important**, because false negatives represent attacks that were not detected.

### 6. Confusion Matrix and Error Analysis

The project also examines misclassified test samples.

The confusion matrix contains:

- **True Negative (TN):** normal traffic correctly classified
- **False Positive (FP):** normal traffic classified as attack
- **False Negative (FN):** attack classified as normal
- **True Positive (TP):** attack correctly classified

False negatives receive particular attention because they correspond to missed attacks.

The notebook extracts misclassified samples so that the model's weaknesses can be investigated instead of relying only on a single performance score.

### 7. Feature Importance

XGBoost's built-in feature importance is used to identify features that contribute strongly to the model.

The current analysis identifies features such as:

- `service_http`
- `src_bytes`
- `service_ecr_i`

among the most important features.

An important observation is that `dst_host_srv_serror_rate`, despite being the most correlated numeric feature in the exploratory analysis, does not necessarily rank among the most important XGBoost features.

This illustrates the difference between **correlation** and **model-based feature importance**:

- Correlation evaluates the relationship between one feature and the target.
- Feature importance evaluates how useful a feature is within the trained model and alongside other features.

A feature can therefore have a strong individual correlation while adding less information once other features are available.

## Important Limitation: `difficulty_level`

`difficulty_level` is metadata associated with NSL-KDD rather than a genuine network traffic measurement.

Because this information would not normally be available for newly observed network traffic, including it in a production-style model can introduce **data leakage**.

It also appears among the important features in the current XGBoost analysis.

### Planned experiment

The next step is to retrain the models **without `difficulty_level`** and compare the results.

This will provide a more realistic estimate of how the models perform using actual traffic-related features.

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/melissa221-cloud/network-anomaly-detection.git
cd network-anomaly-detection
```

### 2. Create and activate a virtual environment

**Windows:**

```bash
python -m venv venv
venv\\Scripts\\activate
```

**macOS/Linux:**

```bash
python -m venv venv
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install pandas numpy scikit-learn matplotlib xgboost jupyter
```

### 4. Launch Jupyter Notebook

```bash
jupyter notebook
```

Open `01_exploration.ipynb` and run the notebook cells in order.

## Reproducibility

The project uses:

- `random_state=42` for reproducible train/test splitting
- An 80/20 stratified split
- A documented preprocessing pipeline
- Fixed model parameters for the current baseline experiments

The raw dataset is not committed to GitHub because it is excluded by `.gitignore`.

## Future Work

- Evaluate the models on the official `KDDTest+.txt` dataset
- Retrain without `difficulty_level`
- Tune XGBoost hyperparameters using cross-validation
- Compare additional classification models
- Analyze attack types individually instead of only using binary classification
- Improve error analysis, especially for false negatives
- Add SHAP-based explanations for individual predictions
- Separate exploration, preprocessing, modeling, and evaluation into dedicated notebooks as the project grows

## Author

**Melissa Hallal**

Network anomaly detection project combining networking and machine learning.
