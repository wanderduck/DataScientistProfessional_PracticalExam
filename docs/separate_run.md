# Practical Exam Report: Predicting High-Traffic Recipes for Tasty Bytes

**Prepared for:** Head of Data Science
**Analyst:** Data Scientist
**Date:** March 2026

---

## 1. Data Validation

The dataset contains 947 recipes across 8 columns. I validated every column against the provided data dictionary and performed cleaning where necessary. No rows were removed — all issues were resolved through imputation and transformation to preserve as much data as possible.

### Column-by-Column Validation

**`recipe`** — Numeric unique identifier. All 947 values are unique with no nulls. This column was excluded from modeling features since it carries no predictive information.

**`calories`, `carbohydrate`, `sugar`, `protein`** — These four numeric nutritional columns each contained 52 null values, and the nulls occurred in the same 52 rows. Rather than dropping these rows (which would lose 5.5% of the data), I imputed the missing values using a KNN Imputer (k=5). To make the imputation category-aware, I one-hot encoded the `category` column as an auxiliary feature during imputation, so that missing nutritional values were filled based on similar recipes within the same category. After imputation, I confirmed all values were non-negative and within plausible ranges.

**`category`** — Character column expected to contain 10 unique recipe categories. I found 11 unique values after stripping whitespace. The unexpected 11th category is "Chicken Breast," which exists alongside "Chicken." I kept both categories as-is because the distinction appears intentional in the data — "Chicken Breast" may represent a specific sub-type with different nutritional and traffic characteristics. Merging them without domain confirmation could introduce noise.

**`servings`** — Documented as numeric but stored as object dtype with 6 unique values. I converted all values to numeric integers. No nulls were present after conversion.

**`high_traffic`** — The target variable. Rows where the recipe generated high traffic are marked "High"; the remaining 373 rows are NaN. I filled NaN values with "Not High" and then mapped the column to binary: "High" → 1, all else → 0. The resulting class distribution is 574 high-traffic recipes (60.6%) and 373 not-high-traffic recipes (39.4%) — reasonably balanced for classification without requiring resampling techniques.

---

## 2. Exploratory Analysis

### Single-Variable Findings

**Calorie Distribution** — The calorie distribution is right-skewed with a median of approximately 289 and a mean of approximately 436 calories per serving. Most recipes fall under 600 calories, with a long tail extending to a maximum of around 3,633 calories. This skew is expected — most recipes are moderate in calories, but a few high-calorie outliers exist (likely rich desserts or large-portion dishes).

**Category Distribution** — The 11 recipe categories are reasonably balanced. No single category dominates the dataset, which means the model has adequate representation across recipe types to learn category-specific patterns.

**Servings Distribution** — The most common serving sizes are 4 and 6, reflecting typical household meal portions.

**Class Balance** — The 60.6% / 39.4% split between high and not-high traffic is moderate. This slight imbalance did not require oversampling or undersampling — stratified splitting during train/test division was sufficient to preserve the ratio.

### Multi-Variable Findings

**Nutritional Features vs. Traffic** — Grouped box plots of each nutritional feature (calories, carbohydrate, sugar, protein) split by high-traffic status showed weak individual separation. The distributions for high-traffic and not-high-traffic recipes overlap substantially. This indicates that no single nutritional metric is a strong standalone predictor of traffic.

**Category vs. Traffic Rate** — Traffic rates vary significantly across categories. Vegetable, Potato, and Chicken recipes tend to generate high traffic more frequently, while Beverages show a lower traffic rate. This finding suggests that recipe category is the strongest individual predictor of traffic — which aligns with intuition, as homepage visitors likely respond more to meal-type recipes than drinks.

**Correlation Analysis** — A correlation matrix confirmed that individual nutritional features have weak linear correlation with the target. Category-based features carry the most predictive signal for traffic classification.

---

## 3. Model Development

### Problem Type

This is a **binary classification** problem. The target variable `high_traffic` has two classes (1 = high traffic, 0 = not high traffic), and the business goal is to predict which class a recipe belongs to before featuring it on the homepage.

### Feature Engineering

I constructed 15 features total:
- **5 numeric features:** calories, carbohydrate, sugar, protein, and servings — all scaled using StandardScaler to normalize their ranges
- **10 categorical features:** one-hot encoded from the `category` column using drop-first encoding to avoid multicollinearity

The data was split 80/20 into training (757 recipes) and test (190 recipes) sets using stratified sampling to preserve the class distribution in both sets.

### Baseline Model: Logistic Regression

I selected Logistic Regression as the baseline because it is interpretable, fast to train, and provides a reliable benchmark for binary classification tasks. It assumes a linear decision boundary, which sets a floor — if more complex models cannot meaningfully beat it, the additional complexity is not justified.

### Comparison Models

I fitted three comparison models to evaluate whether additional complexity improves prediction:

1. **Random Forest** — An ensemble of decision trees that captures non-linear relationships and feature interactions. Trained using GPU-accelerated cuML.
2. **Gradient Boosting** — A sequential ensemble method that iteratively corrects errors from previous trees.
3. **Neural Network (PyTorch)** — A feedforward network with 3 layers and dropout regularization, trained for 200 epochs. This tests whether deep learning can extract patterns that tree-based and linear models miss.

---

## 4. Model Evaluation

I evaluated all four models on the held-out test set using precision, recall, F1 score, and accuracy. Given the business requirement of correctly predicting high traffic 80% of the time, **precision is the primary evaluation metric** — it measures how often a predicted "high traffic" recipe actually generates high traffic.

### Results

| Model | Precision | Recall | F1 Score | Accuracy |
|---|---|---|---|---|
| Neural Network | 84.7% | 72.2% | 77.9% | 75.3% |
| Logistic Regression (baseline) | 84.5% | 75.7% | 79.8% | 76.8% |
| Random Forest | 80.6% | 75.7% | 78.0% | 74.2% |
| Gradient Boosting | 79.5% | 77.4% | 78.4% | 74.2% |

### What the Comparison Shows

The Neural Network and Logistic Regression both exceed the 80% precision target comfortably, at 84.7% and 84.5% respectively. Random Forest also clears the threshold at 80.6%. Gradient Boosting falls just short at 79.5%.

Logistic Regression delivers the best overall balance among the high-precision models — it matches the Neural Network on precision (within 0.2 percentage points) while achieving notably better recall (75.7% vs. 72.2%) and the highest F1 score (79.8%) and accuracy (76.8%) of any model.

The more complex ensemble models (Random Forest and Gradient Boosting) do not improve on the baseline. Gradient Boosting trades precision for recall, catching 77.4% of actual high-traffic recipes but dropping below the 80% precision target. This confirms that a simple linear decision boundary is well-suited to this dataset — additional model complexity does not pay off.

---

## 5. Business Metric Definition

### KPI: Precision of High-Traffic Predictions

The business needs to "correctly predict high traffic recipes 80% of the time." This maps directly to **precision** — the proportion of recipes predicted as high-traffic that actually generate high traffic when featured.

**Why precision over accuracy?** Featuring a recipe predicted as high-traffic that turns out to be low-traffic has a direct cost: the homepage fails to drive the 40% traffic increase that popular recipes generate, which means fewer subscriptions. Precision specifically measures the reliability of "high traffic" predictions, which is the actionable output the product team will use.

### Initial KPI Values

Based on current test set performance:

| Model | Precision (KPI) | Meets 80% Target? |
|---|---|---|
| Neural Network | 84.7% | Yes |
| Logistic Regression | 84.5% | Yes |
| Random Forest | 80.6% | Yes |
| Gradient Boosting | 79.5% | No |

Three of the four models meet or exceed the 80% precision target. The top two models — Neural Network and Logistic Regression — both achieve approximately 85% precision, meaning that for every 10 recipes the model recommends as "high traffic," between 8 and 9 will actually generate high traffic when featured. Gradient Boosting narrowly misses the target at 79.5%.

### Monitoring Recommendation

The product team should track precision on a rolling basis as new recipes are featured and actual traffic outcomes are observed. If precision drops below 80%, the model should be retrained with updated data. I also recommend tracking recall alongside precision — a model that is precise but misses too many high-traffic recipes leaves potential traffic gains on the table.

---

## 6. Final Summary and Recommendations

### Summary

After validating and cleaning the dataset of 947 recipes, I trained four classification models. The Neural Network leads on precision at 84.7%, while Logistic Regression delivers the strongest overall performance (84.5% precision, 75.7% recall, 79.8% F1, 76.8% accuracy). Three of the four models exceed the 80% precision target. Recipe category emerged as the strongest predictor of traffic — categories like Vegetable, Potato, and Chicken are associated with higher traffic rates, while Beverages tend to underperform.

### Recommendations

1. **Deploy Logistic Regression for production.** While the Neural Network has the highest precision (84.7%), Logistic Regression is nearly identical (84.5%) and is simpler, faster, fully interpretable, and easier to maintain. It also has better recall (75.7% vs. 72.2%), a higher F1 score (79.8% vs. 77.9%), and the best accuracy (76.8%) — making it the strongest all-around model. Both exceed the 80% precision target.

2. **Prioritize category in recipe selection.** When multiple recipes score similarly in the model, favor categories with historically high traffic rates (Vegetable, Potato, Chicken) and deprioritize categories with lower rates (Beverages). This uses the strongest signal in the data.

3. **Collect additional features.** The current feature set achieves the 80% target, but nutritional features alone are weak predictors. Future data collection should consider features such as preparation time, cost per serving, seasonal relevance, and ingredient count. These could push precision higher and improve recall.

4. **Establish a feedback loop.** Track the precision KPI weekly as the model is used in production. Log which recipes were recommended, which were featured, and the resulting traffic. This data will enable ongoing model retraining and performance monitoring against the 80% threshold.

5. **A/B test the model against manual selection.** Run a controlled comparison between the model's recommendations and the product manager's current manual picks. This will quantify the real-world impact of model-driven recipe selection on traffic and subscriptions, providing a concrete ROI for the data science team's work.
