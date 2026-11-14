# Model Evaluation & Performance Improvement

Internship deliverable evaluating 8 classification models on the Breast Cancer Wisconsin Diagnostic Dataset using Accuracy, Precision, Recall, F1-Score, ROC-AUC, 5-Fold Stratified Cross-Validation, and Learning Curves to diagnose and resolve overfitting and underfitting.

## Repository Structure
```text
├── data/
│   └── breast_cancer_model_ready.csv                     # Preprocessed & engineered dataset
├── notebooks/
│   └── model_evaluation_improvement.ipynb                # Executable Jupyter / Colab notebook
├── reports/
│   ├── model_evaluation_and_improvement_report.md        # Full metric rationale, overfitting analysis & strengths/weaknesses
│   ├── model_performance_comparison_table.csv            # Quantitative model comparison table
│   ├── overfitting_underfitting_diagnosis.png            # Train vs. CV vs. Test accuracy comparison chart
│   ├── roc_curves_comparison.png                         # Multi-model ROC-AUC curve overlay
│   └── learning_curve_best_model.png                     # Training vs. validation learning curve
├── .gitignore
├── README.md
└── requirements.txt
```

## Quick Start
```bash
pip install -r requirements.txt
jupyter notebook notebooks/model_evaluation_improvement.ipynb
```
