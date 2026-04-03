# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DataCamp Data Scientist Professional Certification Practical Exam — binary classification task predicting high-traffic recipes for "Tasty Bytes" (a recipe subscription site). The business goal is to correctly predict high-traffic recipes at least 80% of the time (precision-focused).

## Environment & Commands

This project uses **uv** for package management with Python 3.13.

```bash
# Install dependencies
uv sync

# Launch Jupyter Lab
uv run jupyter lab

# Run a specific notebook non-interactively
uv run jupyter nbconvert --to notebook --execute pacticalExam_submission.ipynb
```

## Key Files

| File | Purpose |
|---|---|
| `pacticalExam_submission.ipynb` | **Primary deliverable** — analysis notebook (note: filename has a typo) |
| `data/recipe_site_traffic_2212.csv` | Dataset (947 recipes, 8 columns) |
| `docs/practical_exam_report.md` | Written report deliverable |
| `docs/practical_exam_instructions.md` | Full exam brief |
| `docs/grading_rubric.md` | Grading criteria — check all 7 competencies before submitting |

## GPU Acceleration

The notebook's first code cell handles GPU setup. Key constraints:

- `%load_ext cudf.pandas` enables GPU-accelerated DataFrames (works fine)
- **`cuml.accel` is disabled** — RAPIDS cu12 JIT compilation fails against CUDA 13.1 system headers. Use direct imports instead: `from cuml.ensemble import RandomForestClassifier`
- TensorFlow uses cu12 libs; PyTorch uses cu13 libs — both are preloaded via ctypes in the setup cell
- The setup cell must run first before any ML imports

## Problem Structure

**Classification problem** — `high_traffic` is the binary target ("High" vs NaN/empty).

Dataset columns: `recipe` (ID), `calories`, `carbohydrate`, `sugar`, `protein` (numerics), `category` (10 categories), `servings`, `high_traffic` (target).

## Visualization

Use **Plotly** for all visualizations unless explicitly specified otherwise.

## Implementation Plan

### Data Cleaning
1. **`high_traffic`** — Map "High" → 1, all else (NaN/None) → 0. Binary target.
2. **`servings`** — Object dtype with 6 unique values. Inspect, extract numeric, convert to int/float.
3. **`calories`, `carbohydrate`, `sugar`, `protein`** — 52 nulls each. Impute using KNN or IterativeImputer, informed by `category`.
4. **`category`** — 11 unique values vs 10 documented. Find and merge the extra variant (whitespace/case issue).
5. **`recipe`** — Validate uniqueness (947 unique). Drop before modeling (ID only).
6. **Validation summary** — Markdown cell documenting every column: dtype, nulls, action taken, result.

### Exploratory Data Analysis
**Single-variable (4 charts, Plotly):**
1. Bar chart — recipe count by category
2. Histogram — calorie distribution
3. Pie/donut chart — high_traffic class balance
4. Bar chart — servings distribution (post-cleaning)

**Multi-variable (3 charts, Plotly):**
1. Grouped box plots — nutritional values by high_traffic status
2. Stacked/grouped bar chart — category vs high_traffic rate
3. Heatmap — correlation matrix of macros + high_traffic

Each chart followed by written findings connecting to the business question.

### Machine Learning
- **Problem**: Binary classification, precision ≥ 80% is the business KPI
- **Features**: One-hot encode `category`, StandardScaler on numerics, 80/20 stratified split
- **Four models**: Logistic Regression (baseline), Random Forest (cuml GPU), Gradient Boosting (sklearn), Neural Network (PyTorch feedforward)
- **Evaluation**: Precision, recall, F1, accuracy, confusion matrix, classification report for all four
- **Comparison**: Side-by-side metrics table, select best two for report
- **Business KPI**: Precision — "of recipes predicted high traffic, what % actually are?" Report against 80% target.

### Report
- Written narrative at every analysis step in the notebook
- Final summary with recommendations in the WRITTEN REPORT section
- Standalone `practical_exam_report.md` generated from notebook findings

## Grading Requirements Checklist

All 7 competencies must be "Sufficient" to pass:

1. **Data Validation** — validate + clean every column; prefer imputation over row removal
2. **Data Visualization** — ≥2 different single-variable chart types + ≥1 multi-variable chart
3. **Model Fitting** — baseline model + comparison model, both for classification
4. **Model Evaluation** — compare both models with appropriate metrics; describe what results show
5. **Business Focus** — state business goals, explain how analysis addresses them, give ≥1 recommendation
6. **Business Metrics** — define a KPI tied to the 80% precision goal; compare both models on it
7. **Communication** — written explanation for every analysis step; presentation narrative
