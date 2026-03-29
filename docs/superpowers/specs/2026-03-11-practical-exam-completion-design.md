# Practical Exam Completion — Design Spec

## Goal

Complete the three empty sections of `pacticalExam_submission.ipynb` (Data Cleaning, EDA, Machine Learning) and generate the written report. All 7 grading competencies must be satisfied.

## Data Cleaning

### Target: `high_traffic`
- 373 NaN values. Map "High" → 1, everything else → 0.
- Rationale: data description states only high traffic is marked. Missing means the recipe was featured but did not generate high traffic (not that it was never featured).

### Nutritional columns: `calories`, `carbohydrate`, `sugar`, `protein`
- 52 nulls in each (identical rows missing all four).
- Impute using `sklearn.impute.IterativeImputer` or `KNNImputer`.
- Include `category` as a feature in the imputer (encode first) so imputation is category-aware.
- Rationale: rubric penalizes row removal; category-aware imputation is more accurate than global median.

### `servings`
- Object dtype, 6 unique values. Inspect for non-numeric entries (e.g., "4 as a snack").
- Extract leading integer, convert to numeric.

### `category`
- 11 unique values but docs specify 10. Investigate the 11th — likely whitespace, case, or variant spelling.
- Merge into the correct canonical category.

### `recipe`
- Unique ID (947 unique, 947 rows). Validate, then exclude from feature set.

### Validation Summary
- Markdown cell after cleaning summarizing: column name, original dtype, null count, cleaning action, result dtype, post-clean null count.
- Re-run `.info()` and `.describe()` on the cleaned DataFrame to demonstrate analysis-ready state.

## Exploratory Data Analysis

### Single-Variable Charts (4, Plotly)
1. **Bar chart** — recipe count by category. Shows data distribution across recipe types.
2. **Histogram** — calorie distribution. Shows spread, skewness, outliers in the most intuitive nutritional feature.
3. **Pie/donut chart** — high_traffic class balance (1 vs 0). Grounds the reader in the target distribution before modeling.
4. **Bar chart** — servings distribution post-cleaning. Demonstrates data quality work, shows common serving sizes.

### Multi-Variable Charts (3, Plotly)
1. **Grouped box plots** — each nutritional column split by high_traffic. Shows which features differentiate high-traffic recipes.
2. **Stacked/grouped bar chart** — high_traffic rate by category. Shows which recipe categories drive traffic.
3. **Heatmap** — correlation matrix of all numeric features + encoded high_traffic. Shows macro-to-traffic relationships.

### Written Findings
Each chart followed by a markdown cell with:
- What the chart shows
- Key observations
- Connection to the business question ("Can we predict high traffic?")

## Machine Learning

### Problem Statement
Binary classification. Target: `high_traffic` (1/0). Business KPI: precision >= 80%.

### Preprocessing
- One-hot encode `category` (drop_first=True)
- StandardScaler on numeric features (calories, carbohydrate, sugar, protein, servings)
- Train/test split: 80/20, stratified on `high_traffic`, random_state=42

### Models (fit all four, report best two)

1. **Logistic Regression** (sklearn) — baseline. Simple, interpretable, fast.
2. **Random Forest Classifier** (cuml GPU) — ensemble, handles non-linear relationships.
3. **Gradient Boosting Classifier** (sklearn) — sequential ensemble, often best performer.
4. **PyTorch Neural Network** — feedforward classifier (2-3 hidden layers, ReLU, sigmoid output, BCELoss). Leverages GPU setup.

### Evaluation
For each model:
- Classification report (precision, recall, F1, support)
- Confusion matrix (visualized with Plotly heatmap)
- Accuracy score

### Model Comparison
- Side-by-side DataFrame of metrics across all four models
- Written analysis of trade-offs (precision vs recall, complexity vs performance)
- Logistic Regression is always retained as the baseline in the final two-model report. The best-performing alternative (by precision) is the comparison model.

### Business Metrics
- **KPI definition**: Precision of high-traffic predictions — "Of recipes the model predicts as high traffic, what percentage actually generate high traffic?"
- **Target**: >= 80% (from product manager's request)
- **Report**: KPI values for both selected models, with interpretation
- **Recommendation**: Which model to deploy, what precision to expect, next steps

## Written Report

### In-Notebook
- Markdown narrative at every analysis step
- Final WRITTEN REPORT section summarizing: business goals, approach, key findings, KPI values, recommendations

### Standalone Report (`docs/practical_exam_report.md`)
- Generated from notebook findings
- Structured per the exam requirements: data validation, EDA, model development, model evaluation, business metrics, recommendations

## Grading Competency Mapping

| Competency | Where Addressed |
|---|---|
| Data Validation | Data Cleaning section — all columns validated, imputation over removal |
| Data Visualization | EDA — 4 single-variable (3 chart types) + 3 multi-variable charts |
| Model Fitting | ML — baseline (LogReg) + comparison, both classification |
| Model Evaluation | ML — side-by-side metrics, written comparison |
| Business Focus | Business goals stated, analysis connected, recommendations given |
| Business Metrics | Precision KPI defined, both models compared against 80% target |
| Communication | Written explanation at every step, narrative structure |

## Out of Scope
- **Presentation slides** — the exam requires an 8-10 slide presentation. This spec covers the notebook and written report only. Slides will be prepared separately after analysis is complete.

## Technical Constraints
- Plotly for all visualizations
- cuml for Random Forest (GPU-accelerated, direct import). **Fallback**: if cuml fails at runtime due to CUDA version mismatch, use sklearn's `RandomForestClassifier` instead.
- PyTorch for neural network (cu13 libs preloaded)
- cudf.pandas enabled for DataFrame acceleration
- Python 3.13, uv package manager
