# Model Evaluation & Performance Improvement Report

## 1. Metric Selection Rationale
In clinical breast cancer screening, evaluating models solely on **Accuracy** can be misleading because classes are imbalanced (~63% Benign, ~37% Malignant) and misclassifications carry unequal real-world consequences:

- **Recall (Sensitivity):** Measures the proportion of actual positive/benign or malignant cases correctly caught. In medical diagnostics, minimizing False Negatives is critical so malignant tumors are never dismissed without biopsy.
- **Precision:** Measures how reliable positive predictions are, preventing unnecessary invasive procedures caused by False Positives.
- **F1-Score:** The harmonic mean of Precision and Recall ($2 \cdot \frac{\text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}}$), serving as our primary single-number model ranking metric.
- **ROC-AUC:** Evaluates threshold-independent discrimination across all classification probability cutoffs (`0.0` to `1.0`).
- **Train vs. 5-Fold CV vs. Test Gap:** Quantifies **overfitting** (high training score, sharp drop on test data) versus **underfitting** (low scores on both training and test splits).

---

## 2. Performance Comparison Table

| Model | Train Acc | 5-Fold CV Acc | Test Acc | Train-Test Gap | Precision | Recall | F1-Score | ROC-AUC | Fit Diagnosis |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| **Support Vector Machine (RBF)** | 0.9890 | 0.9692 | 0.9825 | +0.0066 | 0.9861 | 0.9861 | 0.9861 | 0.9957 | Well-Generalized (Balanced) |
| **K-Nearest Neighbors (k=7)** | 0.9670 | 0.9538 | 0.9649 | +0.0021 | 0.9474 | 1.0000 | 0.9730 | 0.9798 | Well-Generalized (Balanced) |
| **Logistic Regression (Tuned L2)** | 0.9868 | 0.9780 | 0.9649 | +0.0219 | 0.9722 | 0.9722 | 0.9722 | 0.9940 | Well-Generalized (Balanced) |
| **Random Forest (n=150)** | 1.0000 | 0.9648 | 0.9561 | +0.0439 | 0.9589 | 0.9722 | 0.9655 | 0.9949 | Overfitting (High Variance) |
| **Gradient Boosting** | 1.0000 | 0.9582 | 0.9561 | +0.0439 | 0.9589 | 0.9722 | 0.9655 | 0.9940 | Overfitting (High Variance) |
| **Decision Tree (Depth=1, Underfit)** | 0.9231 | 0.9033 | 0.9211 | +0.0020 | 0.9200 | 0.9583 | 0.9388 | 0.9077 | Underfitting (High Bias) |
| **Decision Tree (Unconstrained, Overfit)** | 1.0000 | 0.9231 | 0.9123 | +0.0877 | 0.9697 | 0.8889 | 0.9275 | 0.9206 | Overfitting (High Variance) |
| **Dummy Baseline (Underfit)** | 0.6264 | 0.6264 | 0.6316 | -0.0052 | 0.6316 | 1.0000 | 0.7742 | 0.5000 | Underfitting (High Bias) |

---

## 3. Overfitting vs. Underfitting Diagnosis & How Performance Was Improved

1. **Underfitting (`Dummy Baseline` at 63.16% & `Decision Tree Depth=1` at 92.11%):**
   - A single-split decision stump lacks representational capacity to capture multi-feature interactions across 38 tumor geometry attributes, resulting in high bias on both training (`92.31%`) and test (`92.11%`) sets.
2. **Overfitting (`Unconstrained Decision Tree` — Train 100.00% vs. Test 91.23%):**
   - Allowing unlimited tree depth (`max_depth=None`) memorized noise in the training set (`100%` training accuracy) while dropping to `91.23%` on unseen test samples (**+8.77% overfitting gap**).
3. **Resolving Overfitting & Improving Generalization:**
   - **Ensemble Averaging (`Random Forest` & `Gradient Boosting`):** Reduced variance and lifted Test F1-Score from `0.9275` (single tree) to **0.9655** (`ROC-AUC = 0.9949`).
   - **Regularization & Standardization (`SVM RBF` & `Tuned Logistic Regression`):** Scaling features inside `Pipeline` objects and tuning regularization (`C=2.0` for SVM, `C=0.5` L2 penalty for Logistic Regression) shrank the train-test gap to **0.66%** and achieved **98.25% Test Accuracy**, **0.9861 F1-Score**, and **0.9957 ROC-AUC**.

---

## 4. Model Strengths & Weaknesses

| Model Family | Key Strengths | Key Weaknesses | Best Use Case |
| :--- | :--- | :--- | :--- |
| **Support Vector Machine (RBF)** | Highest Test F1 (`0.9861`) and ROC-AUC (`0.9957`); excels on smooth non-linear boundaries in standardized continuous spaces. | Less interpretable ("black box"); slower training on very large datasets ($N > 50,000$). | High-accuracy diagnostic classification on medium-sized tabular datasets. |
| **Logistic Regression (Tuned L2)** | Highest 5-Fold CV accuracy (`97.80%`); fast inference; transparent feature coefficients and well-calibrated probabilities. | Assumes a linear decision boundary in log-odds space; requires feature scaling. | Regulated healthcare/finance pipelines requiring explainability. |
| **Random Forest / Gradient Boosting** | Captures complex non-linear interactions without requiring feature scaling; resistant to outliers. | Can overfit small datasets (`100%` train accuracy) if tree depth/leaf size is not constrained. | Mixed tabular datasets with non-linear feature interactions. |
| **K-Nearest Neighbors (`k=7`)** | Simple, non-parametric; achieved `100%` Recall on test set. | Sensitive to irrelevant features and scaling; slow prediction time at scale. | Low-dimensional similarity search baselines. |
| **Single Decision Tree** | Fully human-readable rule paths; zero preprocessing required. | High variance (overfits when deep) or high bias (underfits when shallow). | Exploratory rule extraction rather than standalone deployment. |
