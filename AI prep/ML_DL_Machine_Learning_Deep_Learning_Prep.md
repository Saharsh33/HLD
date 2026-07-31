# ML & DL — Machine Learning & Deep Learning
### Interview Prep Notes — ZS / Fractal / Tiger Analytics / PwC
> **Depth**: Intermediate → Advanced | **Rounds**: R1 (30 min — ML heavy, Lin/Log regression focus) + R2 (60 min — Live Coding + Deep Dive)
> Ensemble methods (boosting, bagging) are **heavily tested**.
> Know bias-variance for every algorithm.
> OLS assumptions are **non-negotiable**.

---

# ═══════════════════════════════════════════════
# TOPIC 1: LINEAR REGRESSION — MOST ASKED IN R1
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

**What It Does:** Fits a hyperplane $\hat{y} = \beta_0 + \beta_1 x_1 + \ldots + \beta_p x_p$ that minimizes the sum of squared residuals.

**Geometric Intuition:**
```
y │         ●
  │       ● /
  │     ●  / ← regression line (minimizes
  │   ● / /     vertical distances²)
  │  ●/ /
  │ /● /
  │/ /●
  └──────────── x
     residual = yᵢ - ŷᵢ (vertical distance)
```

**OLS = Projection:** $\hat{y} = X(X^TX)^{-1}X^Ty$ — this projects $y$ onto the column space of $X$. The residuals $e = y - \hat{y}$ are **orthogonal** to the feature space.

### 5 OLS ASSUMPTIONS (MUST MEMORIZE — "LINE + H")

| # | Assumption | What It Means | What Happens When Violated | Fix |
|---|---|---|---|---|
| **L** | **Linearity** | $E[y \mid X] = X\beta$ (true relationship is linear) | Biased coefficients, poor predictions | Add polynomial/interaction terms, use non-linear models |
| **I** | **Independence** | Residuals are uncorrelated: $\text{Cov}(e_i, e_j) = 0$ | Underestimated SE → inflated significance | Time-series: use ARIMA; Spatial: spatial regression |
| **N** | **Normality** | Residuals $\sim N(0, \sigma^2)$ | Invalid CIs and p-values (coefficients still unbiased) | Transform Y, use robust SEs, large n (CLT helps) |
| **E** | **Equal Variance (Homoscedasticity)** | $\text{Var}(e_i) = \sigma^2$ for all $i$ | Inefficient estimates, wrong SEs | WLS, log-transform Y, robust standard errors |
| **+** | **No Multicollinearity** | Features are not perfectly correlated | Unstable $\beta$, huge SEs, $(X^TX)$ near-singular | VIF check (>5 suspect, >10 drop), PCA, drop features |

**Diagnosing Violations:**
```
Homoscedasticity ✅              Heteroscedasticity ❌
Residuals vs Fitted:             Residuals vs Fitted:

  ●  ●  ●  ●  ●  ●                    ●  ●  ●
● ● ● ● ● ● ● ● ● ●                ●  ●  ●
  ●  ●  ●  ●  ●  ●               ● ●  ●
─────────────────────────    ──● ●──────────────────
  ●  ●  ●  ●  ●  ●           ●                ●  ●
● ● ● ● ● ● ● ● ● ●                       ●  ●  ●  ●
  ●  ●  ●  ●  ●  ●                          ●  ●  ●  ●  ●
   Constant spread                 Fan/Cone shape = BAD
```

## 2. HOW IT LEARNS & METRICS (Simplified)

**The Math (Keep it simple):**
- **Objective:** Minimize **Mean Squared Error (MSE)**.
- **Closed-form Solution:** $\hat{\beta} = (X^TX)^{-1}X^Ty$. 
  *Interview tip:* This requires $X^TX$ to be invertible. Perfect multicollinearity breaks this (matrix becomes singular).

**Evaluation Metrics:**
- **R²:** How much variance in $Y$ is explained by $X$. (Closer to 1 is better).
- **Adjusted R²:** **Always mention this.** R² artificially increases when you add junk features. Adjusted R² penalizes useless features. Always use it to compare models.
- **MSE vs MAE:** 
  - MSE squares errors (heavily punishes large outliers).
  - MAE uses absolute errors (robust to outliers). Use MAE if your dataset has extreme outliers you don't want to over-correct for.

## 3. ALGORITHM WORKFLOW & TRADE-OFFS

```
[Raw Data] → [EDA + Check Assumptions] → [Handle Multicollinearity (VIF)]
      │                                            │
      ▼                                            ▼
[Feature Scaling]                         [Drop / Combine Features]
      │                                            │
      ▼                                            ▼
[Fit OLS: β̂ = (XᵀX)⁻¹Xᵀy]     OR     [Gradient Descent (if n large)]
      │
      ▼
[Residual Diagnostics: normality, homoscedasticity, independence]
      │
      ▼
[Model Evaluation: R², Adj-R², RMSE, AIC/BIC]
```

| Feature | Linear Regression | Polynomial Regression | Ridge/Lasso |
|---|---|---|---|
| Relationship | Linear | Non-linear | Linear + regularized |
| Interpretability | ★★★★★ | ★★★ | ★★★★ |
| Overfitting Risk | Low | High (high degree) | Low (penalized) |
| Multicollinearity | Breaks | Breaks worse | Handles it |

## 4. REAL-WORLD APPLICATIONS

- **Pricing Models**: House price prediction (features: sqft, bedrooms, location)
- **Sales Forecasting**: Revenue prediction from marketing spend
- **Risk Scoring**: Insurance premium calculation

**Pitfalls:**
- **Data Leakage**: Using future information as a feature (e.g., including "total_revenue" to predict "monthly_revenue")
- **Extrapolation**: Model is only valid within training data range
- **Outlier Sensitivity**: OLS minimizes squared errors → outliers have disproportionate influence → use robust regression or Huber loss

## 5. TOP INTERVIEW QUESTIONS

**Q1 (Easy):** _What are the assumptions of Linear Regression?_
> **LINE + H**: Linearity, Independence of errors, Normality of residuals, Equal variance (Homoscedasticity), No multicollinearity. Violations don't make the model "wrong" — they affect **inference** (CIs, p-values) more than predictions.

**Q2 (Medium):** _What happens if features are highly correlated?_
> **Multicollinearity**: $(X^TX)$ becomes near-singular → coefficients become unstable (huge SEs), signs may flip. Predictions can still be OK but interpretation breaks. **Fix**: Check VIF, remove/combine features, use Ridge (L2 keeps all features, shrinks coefficients).

**Q3 (Hard):** _When would you use MAE over MSE?_
> **MAE** when outliers are present and you don't want them to dominate the loss. **MSE** penalizes large errors quadratically → sensitive to outliers but better when large errors are truly costly. MAE gives **median** regression; MSE gives **mean** regression.

**Q4 (Follow-up):** _"Is Linear Regression a parametric or non-parametric model?"_
> **Parametric** — it assumes a fixed functional form ($y = X\beta + \epsilon$) with a finite number of parameters. Non-parametric models (KNN, decision trees) don't assume a fixed form.

**Red Flags:**
- Not knowing the 5 assumptions
- Thinking R² is sufficient for model evaluation (use Adj-R², AIC, residual plots)
- Not mentioning feature scaling isn't needed for OLS (but needed for gradient descent and regularized versions)

## 6. PRACTICAL CODING

```python
import numpy as np
import pandas as pd
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error, r2_score
import statsmodels.api as sm
from statsmodels.stats.outliers_influence import variance_inflation_factor

# --- Sklearn (Quick) ---
X = df[['sqft', 'bedrooms', 'age']].values
y = df['price'].values
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = LinearRegression()
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
print(f"R² = {r2_score(y_test, y_pred):.4f}")
print(f"RMSE = {np.sqrt(mean_squared_error(y_test, y_pred)):.2f}")
print(f"Coefficients: {dict(zip(['sqft','bedrooms','age'], model.coef_))}")

# --- Statsmodels (For inference — p-values, CIs) ---
X_sm = sm.add_constant(X)  # adds intercept
ols_model = sm.OLS(y, X_sm).fit()
print(ols_model.summary())  # gives R², Adj-R², p-values, CIs, F-stat

# --- VIF Check (Multicollinearity) ---
vif_data = pd.DataFrame()
vif_data["Feature"] = ['sqft', 'bedrooms', 'age']
vif_data["VIF"] = [variance_inflation_factor(X, i) for i in range(X.shape[1])]
print(vif_data)  # VIF > 5 → suspect, VIF > 10 → problematic
```

**Complexity:** Closed-form $O(np^2 + p^3)$ — $p^3$ for matrix inversion. GD: $O(np)$ per iteration.

**5-Bullet Quick Revision:**
1. **OLS** minimizes $\sum(y_i - \hat{y}_i)^2$. Closed form: $\hat{\beta} = (X^TX)^{-1}X^Ty$.
2. **5 Assumptions (LINE+H)**: Linearity, Independence, Normality, Equal Variance, No Multicollinearity.
3. **R² always increases** with more features → use **Adjusted R²** or **AIC/BIC** for model selection.
4. **Multicollinearity** → unstable coefficients, huge SEs. Check VIF. Fix with Ridge or drop features.
5. **Outliers disproportionately affect OLS** (squared loss). Consider MAE, Huber loss, or robust regression.

---

# ═══════════════════════════════════════════════
# TOPIC 2: LOGISTIC REGRESSION — MOST ASKED IN R1
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

**What It Does:** Models the **probability** of a binary outcome using the sigmoid function.

**Why Not Linear Regression for Classification?**
- LR outputs can be <0 or >1 → not valid probabilities
- Decision boundary is affected by outliers
- Residuals are heteroscedastic and non-normal

**The Sigmoid (Logistic) Function:**
$$\sigma(z) = \frac{1}{1 + e^{-z}}, \quad z = \beta_0 + \beta_1 x_1 + \ldots + \beta_p x_p$$

```
P(y=1)
 1.0 │                    ●●●●●●●●●●
     │                  ●●
     │                ●●
 0.5 │- - - - - - - ●- - - - - - - - ← decision boundary (threshold)
     │            ●●
     │          ●●
 0.0 │●●●●●●●●●
     └──────────────────────────── z (log-odds)
```

**Interpreting the Output (Log-Odds):**
Linear models need an output that ranges from $(-\infty, +\infty)$. Probabilities are stuck between [0, 1]. Logistic regression solves this by predicting **Log-Odds**.
- **Odds:** $P / (1-P)$. (e.g., if P = 0.75, Odds = 0.75/0.25 = 3).
- **Log-Odds:** $\ln(\text{Odds})$.

*Coefficient meaning:* If $\beta_1 = 0.5$, a 1-unit increase in $X$ increases the *log-odds* by 0.5. To get the actual odds multiplier, exponentiate it: $e^{0.5} \approx 1.65$ (the odds increase by 65%).

## 2. THE LOSS FUNCTION (Why not MSE?)

**Log Loss (Binary Cross-Entropy):**
Logistic regression uses Log Loss instead of MSE. 

**Why? (Top Interview Question)** 
If you use MSE with a Sigmoid curve, the math creates a "wavy" (**non-convex**) loss surface. Gradient descent will get trapped in local minima. Log Loss creates a smooth, bowl-shaped (**convex**) surface, guaranteeing the model finds the global minimum.

```
MSE with Sigmoid:              Log Loss (Cross-Entropy):
    ╱╲                               ╲
   ╱  ╲    ╱╲                         ╲
  ╱    ╲  ╱  ╲                         ╲
 ╱      ╲╱    ╲                          ╲___
Non-convex (local minima)       Convex (single global minimum) ✅
```

*Note:* There is no closed-form solution for Logistic Regression. It must be solved using Gradient Descent (or similar solvers like L-BFGS).

## 3. ALGORITHM WORKFLOW & TRADE-OFFS

```
[Binary Classification Task]
       │
       ▼
[Feature Engineering + Scaling]    ← Scaling matters for gradient descent
       │
       ▼
[Fit Logistic Regression]          ← Solver: 'lbfgs' (default), 'saga' (large n)
       │
       ▼
[Choose Threshold]                 ← Default 0.5, but tune based on business need
       │
       ▼
[Evaluate: Confusion Matrix, ROC-AUC, Precision-Recall]
       │
       ▼
[Calibration Check]               ← Is predicted probability = actual frequency?
```

| Feature | Logistic Regression | SVM | Random Forest |
|---|---|---|---|
| Output | Probability | Distance from hyperplane | Probability (averaged votes) |
| Interpretability | ★★★★★ (coefficients = log-odds) | ★★ | ★★ |
| Non-linear boundary | No (unless feature engineering) | Yes (kernel trick) | Yes (inherently) |
| Scalability | ★★★★★ | ★★★ | ★★★★ |
| Handles multicollinearity | Poorly (use regularization) | Better | Robust |

## 4. REAL-WORLD APPLICATIONS

- **Churn Prediction**: P(customer leaves) given usage patterns
- **Credit Risk**: P(default) for loan approval — **interpretability is legally required**
- **Medical Diagnosis**: P(disease) given symptoms — calibrated probabilities matter
- **Click-Through Rate (CTR)**: P(click) for ad ranking

**Why Logistic Over Complex Models in Production:**
- Regulatory compliance (explainable AI)
- Low latency inference
- Easy to update with new data
- Well-calibrated probabilities out-of-box

## 5. TOP INTERVIEW QUESTIONS

**Q1 (Easy):** _What's the difference between Linear and Logistic Regression?_
> Linear: continuous output, MSE loss, OLS. Logistic: probability output via sigmoid, log-loss, gradient descent. Linear predicts values; Logistic predicts probabilities for classification.

**Q2 (Medium):** _How do you interpret coefficients in Logistic Regression?_
> $\beta_j$ = change in log-odds for 1-unit increase in $x_j$ (holding others constant). $e^{\beta_j}$ = odds ratio. Example: $\beta = 0.5$ → odds increase by factor of $e^{0.5} \approx 1.65$ (65% increase in odds).

**Q3 (Hard):** _Your Logistic Regression model has 90% accuracy but the business says it's useless. What went wrong?_
> Likely **class imbalance**. If 90% of data is class 0, predicting all 0s gives 90% accuracy but 0% recall for class 1. **Use**: Precision, Recall, F1, AUC-ROC, AUC-PR instead. Consider: SMOTE, class weights, threshold tuning.

**Q4 (Follow-up):** _"How would you choose the classification threshold?"_
> Default 0.5 is rarely optimal. Tune based on business cost:
> - **Fraud detection**: Lower threshold (e.g., 0.3) → catch more fraud (high recall), accept more false alarms
> - **Spam filter**: Higher threshold (e.g., 0.7) → avoid marking legitimate emails as spam (high precision)
> - Use **Precision-Recall curve** and pick threshold at desired operating point

**Red Flags:**
- Using accuracy for imbalanced datasets
- Not knowing the loss function (log-loss, not MSE)
- Confusing probability output with a hard classification

## 6. PRACTICAL CODING

```python
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import (classification_report, confusion_matrix,
                              roc_auc_score, roc_curve)
from sklearn.preprocessing import StandardScaler

# Feature scaling (important for logistic regression convergence)
scaler = StandardScaler()
X_train_sc = scaler.fit_transform(X_train)
X_test_sc = scaler.transform(X_test)

# Fit model (with L2 regularization by default)
model = LogisticRegression(C=1.0, solver='lbfgs', max_iter=1000,
                           class_weight='balanced')  # handles imbalance
model.fit(X_train_sc, y_train)

# Probabilities and predictions
y_proba = model.predict_proba(X_test_sc)[:, 1]  # P(class=1)
y_pred = (y_proba >= 0.5).astype(int)            # custom threshold

# Evaluation
print(classification_report(y_test, y_pred))
print(f"AUC-ROC: {roc_auc_score(y_test, y_proba):.4f}")

# Confusion Matrix
#                  Predicted
#                  0      1
# Actual  0    [ TN    FP ]
#         1    [ FN    TP ]
print(confusion_matrix(y_test, y_pred))

# Interpret coefficients
for feat, coef in zip(feature_names, model.coef_[0]):
    print(f"{feat}: β={coef:.3f}, OR={np.exp(coef):.3f}")
```

**5-Bullet Quick Revision:**
1. Logistic Regression uses **sigmoid** to output probabilities. Loss = **Binary Cross-Entropy** (convex, not MSE).
2. **Coefficients** are in **log-odds**. Exponentiate ($e^\beta$) to get **odds ratios**.
3. **No closed-form** — uses gradient descent. Same gradient form as linear regression: $X^T(\hat{p}-y)/n$.
4. **Threshold ≠ always 0.5** — tune based on business cost (precision vs recall trade-off).
5. For **imbalanced data**: use `class_weight='balanced'`, AUC-ROC/PR, and NEVER report only accuracy.

---

# ═══════════════════════════════════════════════
# TOPIC 3: DECISION TREES
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

**What It Does:** Recursively splits data into subsets using feature thresholds to create a tree of if-else decisions. Think of it as playing a game of "20 questions" with your data.

**How it chooses a split (The Math Simplified):**
The tree looks at all possible features and thresholds to find the split that makes the resulting child nodes as **pure** as possible.

- **For Classification:** It uses **Gini Impurity** or **Entropy**.
  *Intuition:* A node with 50 cats and 50 dogs is highly impure. A node with 100 cats and 0 dogs is perfectly pure. The tree greedily picks the split that drops the impurity the most (this drop is called **Information Gain**).
- **For Regression:** It uses **MSE** (Mean Squared Error) to group similar continuous values together.

**Gini vs Entropy (Common Interview Question):**
- Gini is computationally cheaper (no logarithms).
- Entropy tends to produce slightly more balanced trees.
- In practice, they perform almost identically. Sklearn's default is Gini.

```
                     [Is Income > 50K?]
                      /              \
                   Yes                No
                   /                    \
          [Age > 30?]            [Credit Score > 700?]
           /      \                 /           \
        Yes       No              Yes            No
        /          \              /                \
   [Approve]  [Review]     [Approve]          [Deny]
```

## 2. ALGORITHM WORKFLOW & TRADE-OFFS

**Greedy Algorithm (ID3 / CART):**
```
[Full Dataset]
      │
      ▼
[For each feature, find best split threshold]    ← exhaustive search
      │
      ▼
[Pick split with highest Information Gain / lowest Gini]
      │
      ▼
[Create child nodes]
      │
      ▼
[Recurse on each child]
      │
      ▼
[Stop when]: max_depth reached, min_samples_leaf, pure node, no improvement
```

**Overfitting Control:**
- **Pre-pruning**: Stop early (max_depth, min_samples_split, min_samples_leaf)
- **Post-pruning**: Grow full tree, then prune (cost-complexity pruning, $\alpha$ parameter)

| Pros | Cons |
|---|---|
| Highly interpretable (white box) | Prone to overfitting |
| Handles non-linear relationships | Unstable (small data change → different tree) |
| No feature scaling needed | Greedy → not globally optimal |
| Handles mixed data types | Biased toward features with many levels |

## 3. TOP INTERVIEW QUESTIONS

**Q1:** _How do you prevent a Decision Tree from overfitting?_
> Pre-pruning (max_depth, min_samples_split, min_samples_leaf) or post-pruning (cost-complexity pruning). Or better yet, use **ensembles** (Random Forest, Gradient Boosting) which aggregate many trees.

**Q2:** _Decision Tree vs. Linear Regression — when would you pick each?_
> DT when relationship is non-linear, interactions exist, and you need interpretability for non-technical stakeholders. LR when relationship is approximately linear, you need statistical inference (p-values, CIs), and the model must be deployable with minimal compute.

**Complexity:** Training: $O(n \cdot p \cdot \text{depth})$. Prediction: $O(\text{depth})$.

---

# ═══════════════════════════════════════════════
# TOPIC 4: BAGGING & RANDOM FORESTS
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

> Ensemble methods (bagging vs boosting) are a top interview topic.

**Bagging (Bootstrap Aggregating):**
- **Goal:** Reduce **variance** (stabilize unstable models like decision trees)
- **Method:** Train multiple models on **bootstrap samples**, then aggregate predictions

**What is a Bootstrap Sample?**
Sampling **with replacement** from your dataset. Imagine your data is a bag of 5 balls [A, B, C, D, E]. A bootstrap sample of size 5 might be [A, A, C, D, D] — you put each ball back before drawing the next one. Some items appear multiple times, some don't appear at all.

Why do this? Each bootstrap sample is **slightly different**, so each trained model is slightly different. Averaging these different models smooths out the instability (variance) of any single model.

```
Original Dataset D (n samples)
      │
      ├──→ Bootstrap Sample 1 (n samples, with replacement) → Tree 1 → Pred₁
      ├──→ Bootstrap Sample 2 (n samples, with replacement) → Tree 2 → Pred₂
      ├──→ Bootstrap Sample 3 (n samples, with replacement) → Tree 3 → Pred₃
      │    ...
      └──→ Bootstrap Sample B (n samples, with replacement) → Tree B → Pred_B
                                                                    │
                                                                    ▼
                                          Classification: Majority Vote
                                          Regression: Average
```

**Key Insight:** Each bootstrap sample contains ~63.2% unique data points. Here's why: probability of a specific point NOT being picked in one draw = $(1 - 1/n)$. After $n$ draws: $(1-1/n)^n \approx 1/e \approx 0.368$. So ~36.8% of data is **left out** = **Out-of-Bag (OOB)** samples → this is a **free validation set** (no need for a separate held-out set)!

### Random Forest = Bagging + Feature Randomness

**Extra Trick:** At each split, consider only a **random subset of features** ($m = \sqrt{p}$ for classification, $m = p/3$ for regression).

**Why?** If one feature is very strong, all bagged trees will use it at the root → trees are **correlated** → averaging correlated things doesn't reduce variance much. Random feature selection **decorrelates** trees → much better variance reduction.

```
Bagging:                              Random Forest:
All trees can use all features        Each split uses random √p features

Tree 1: [F₃ > 5?]                    Tree 1: [F₇ > 3?]      (from {F₂,F₅,F₇})
Tree 2: [F₃ > 5?]   ← correlated!   Tree 2: [F₁ > 8?]      (from {F₁,F₃,F₈})
Tree 3: [F₃ > 5?]                    Tree 3: [F₅ > 2?]      (from {F₄,F₅,F₆})
                                              ← decorrelated! ✅
```

## 2. MATHEMATICS THAT MATTERS

**Variance Reduction:**
- For $B$ independent models with variance $\sigma^2$: $\text{Var}(\text{avg}) = \sigma^2/B$
- For correlated models (correlation $\rho$): $\text{Var}(\text{avg}) = \rho\sigma^2 + \frac{(1-\rho)\sigma^2}{B}$
- RF reduces $\rho$ (through feature randomness) → lower variance than plain bagging

**Feature Importance (Mean Decrease Impurity):**
$$\text{Importance}(F_j) = \sum_{\text{trees}} \sum_{\text{splits on } F_j} \frac{n_{\text{node}}}{n} \cdot \Delta \text{Impurity}$$

**Permutation Importance (more reliable):**
$$\text{Importance}(F_j) = \text{Score}_{\text{original}} - \text{Score}_{\text{permuted } F_j}$$

## 3. ALGORITHM WORKFLOW & TRADE-OFFS

| Feature | Decision Tree | Bagging | Random Forest |
|---|---|---|---|
| Bias | Low | Low | Low |
| Variance | **High** | Reduced | **Most Reduced** |
| Interpretability | ★★★★★ | ★★★ | ★★★ |
| Overfitting Risk | High | Lower | **Lowest** |
| Correlated Trees? | N/A | Yes (problem!) | No (feature randomization) |
| OOB Evaluation | No | Yes | Yes |

## 4. TOP INTERVIEW QUESTIONS

**Q1 (Easy):** _What is the difference between Bagging and Random Forest?_
> Both use bootstrap sampling + aggregation. RF adds **random feature selection at each split** to decorrelate trees. This extra randomness reduces variance further.

**Q2 (Medium):** _Random Forest has 500 trees and still overfits. What do you do?_
> 1. Increase `min_samples_leaf` / decrease `max_depth` (constrain individual trees)
> 2. Decrease `max_features` (more randomness → less overfitting)
> 3. Check if n_estimators is too low (unlikely at 500)
> 4. Feature selection — too many noisy features
> 5. Check for data leakage

**Q3 (Hard):** _Does increasing number of trees in Random Forest cause overfitting?_
> **No!** This is a key insight. Adding more trees **never** overfits (unlike boosting). Each tree sees a different bootstrap sample + random features. More trees → better averaging → lower variance. The model converges but doesn't overfit. (However, computation increases linearly.)

## 5. PRACTICAL CODING

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report

rf = RandomForestClassifier(
    n_estimators=500,         # number of trees
    max_depth=10,             # prevent overfitting
    max_features='sqrt',      # √p features per split
    min_samples_leaf=5,       # minimum samples in leaf
    oob_score=True,           # use OOB for validation
    class_weight='balanced',  # handle imbalance
    random_state=42,
    n_jobs=-1                 # parallelize
)
rf.fit(X_train, y_train)

print(f"OOB Score: {rf.oob_score_:.4f}")  # Free validation!
print(f"Test Accuracy: {rf.score(X_test, y_test):.4f}")

# Feature Importance
importances = pd.Series(rf.feature_importances_, index=feature_names)
print(importances.sort_values(ascending=False).head(10))
```

**5-Bullet Quick Revision:**
1. **Bagging** reduces variance by averaging bootstrap-sampled models. RF adds **feature randomness** to decorrelate trees.
2. Each bootstrap sample uses ~63.2% of data. Remaining ~36.8% = **OOB** (free validation).
3. `max_features = √p` (classification), `p/3` (regression). Lower = more randomness = less variance.
4. **Adding more trees to RF NEVER overfits** — unlike boosting. It just converges.
5. Feature importance: Use **permutation importance** (model-agnostic, unbiased) over default impurity-based (biased toward high-cardinality features).

---

# ═══════════════════════════════════════════════
# TOPIC 4B: CROSS-VALIDATION (Essential Foundation)
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

**Why We Need It:** You can't evaluate a model on the same data it was trained on (that just measures memorization). You need to test on **unseen data**. A simple train-test split wastes data and gives a noisy estimate. Cross-validation gives a **more reliable** estimate by using ALL data for both training and validation.

### K-Fold Cross-Validation (Most Common: K=5 or K=10)

```
Dataset split into 5 folds:

 Fold 1: [████ TEST ████] [    TRAIN    ] [    TRAIN    ] [    TRAIN    ] [    TRAIN    ]
 Fold 2: [    TRAIN    ] [████ TEST ████] [    TRAIN    ] [    TRAIN    ] [    TRAIN    ]
 Fold 3: [    TRAIN    ] [    TRAIN    ] [████ TEST ████] [    TRAIN    ] [    TRAIN    ]
 Fold 4: [    TRAIN    ] [    TRAIN    ] [    TRAIN    ] [████ TEST ████] [    TRAIN    ]
 Fold 5: [    TRAIN    ] [    TRAIN    ] [    TRAIN    ] [    TRAIN    ] [████ TEST ████]

 Final Score = average of 5 fold scores ± std dev
```

Each fold serves as the test set exactly once. Every data point gets used for both training (4 times) and validation (1 time).

**Variants:**

| Method | Description | When to Use |
|---|---|---|
| **K-Fold** | Split data into K equal parts, rotate test set | Default choice |
| **Stratified K-Fold** | Same, but preserves class ratio in each fold | **Imbalanced data** (always use this for classification) |
| **Leave-One-Out (LOOCV)** | K = n (each sample is its own test set) | Very small datasets |
| **Time-Series Split** | Train on past, test on future (no shuffling!) | **Time-series data** (prevents look-ahead bias) |
| **Repeated K-Fold** | Run K-Fold multiple times with different random splits | When you need lower variance in the estimate |

**⚠️ Critical Rules:**
- **Never leak information from test fold to train fold.** Fit all preprocessing (scaling, imputation, encoding) on the TRAINING folds only. `Pipeline` in sklearn handles this automatically.
- **Don't use CV score as the final performance estimate.** Use CV for model selection/hyperparameter tuning, then evaluate the chosen model on a completely held-out test set.

```python
from sklearn.model_selection import cross_val_score, StratifiedKFold
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression

# Correct way: preprocessing inside CV pipeline
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('model', LogisticRegression())
])

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(pipe, X, y, cv=cv, scoring='roc_auc')
print(f"AUC: {scores.mean():.4f} ± {scores.std():.4f}")
```

---

# ═══════════════════════════════════════════════
# TOPIC 5: BOOSTING (AdaBoost, Gradient Boosting, XGBoost, LightGBM)
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

**Boosting = Sequential Ensemble that reduces BIAS (and variance).**

Unlike bagging (parallel, reduces variance), boosting trains models **sequentially**, with each new model focusing on the **mistakes of the previous ones**.

### AdaBoost
```
Round 1: Train weak learner on full data → misclassifies some points
                    │
Round 2: ↑ weights on misclassified → train new learner on reweighted data
                    │
Round 3: ↑ weights on remaining errors → train again
                    │
         ...continue for T rounds...
                    │
Final:   Weighted majority vote: F(x) = sign(Σ αₜ · hₜ(x))

αₜ = learner weight (higher for more accurate learners)
```

### Gradient Boosting (GBM)
**Key Idea:** Instead of reweighting, fit each new tree to the **residuals** (negative gradient of the loss).

```
Step 0: F₀(x) = ȳ (start with mean)
Step 1: r₁ = y - F₀(x)  →  Fit tree h₁ to residuals  →  F₁(x) = F₀(x) + η·h₁(x)
Step 2: r₂ = y - F₁(x)  →  Fit tree h₂ to residuals  →  F₂(x) = F₁(x) + η·h₂(x)
   ...
Step T: F_T(x) = F₀(x) + η·Σ hₜ(x)

η = learning rate (shrinkage) — smaller = slower learning = better generalization
```

**Gradient Boosting in General:**
- Works with ANY differentiable loss function
- Each tree fits the negative gradient of the loss w.r.t. the current prediction
- For MSE: negative gradient = residuals (that's why it's intuitive)
- For log-loss: negative gradient = $(y - \hat{p})$ — same as logistic regression!

### XGBoost vs. LightGBM vs. CatBoost

| Feature | XGBoost | LightGBM | CatBoost |
|---|---|---|---|
| Tree Growth | Level-wise (balanced) | **Leaf-wise** (fastest) | Symmetric (balanced) |
| Speed | Fast | **Fastest** | Moderate |
| Categorical Features | Needs encoding | Supports natively | **Best native support** |
| Missing Values | Learns optimal direction | Learns optimal direction | Uses stats-based approach |
| Regularization | L1 + L2 on weights | L1 + L2 on weights | Ordered boosting (reduces overfit) |
| Best For | General purpose | Large datasets, speed | Categorical-heavy data |

**Tree Growth Strategies:**
```
Level-wise (XGBoost):              Leaf-wise (LightGBM):
        [root]                           [root]
       /      \                         /      \
     [L1]    [L1]                    [L1]      [L1]
    /  \    /   \                   /   \
  [L2][L2][L2] [L2]              [L2] [L2]
  All nodes at level                 ↑ Grows deepest leaf
  expanded equally                   (highest gain) → faster
```

## 2. MATHEMATICS THAT MATTERS

**XGBoost Objective:**
$$\text{Obj} = \sum_{i=1}^{n} L(y_i, \hat{y}_i) + \sum_{t=1}^{T} \Omega(f_t)$$

**Regularization term:**
$$\Omega(f) = \gamma T + \frac{1}{2}\lambda \sum_{j=1}^{T} w_j^2$$
- $T$ = number of leaves, $w_j$ = leaf weights
- $\gamma$ = penalty for tree complexity (more leaves)
- $\lambda$ = L2 regularization on leaf weights

**Second-Order Approximation (Taylor expansion — why XGBoost is fast):**
$$\text{Obj}^{(t)} \approx \sum_{i=1}^{n}\left[g_i f_t(x_i) + \frac{1}{2}h_i f_t^2(x_i)\right] + \Omega(f_t)$$
- $g_i = \frac{\partial L}{\partial \hat{y}_i}$ (first derivative = gradient)
- $h_i = \frac{\partial^2 L}{\partial \hat{y}_i^2}$ (second derivative = Hessian)
- Using 2nd order info → better split decisions and convergence than vanilla GBM

## 3. ALGORITHM WORKFLOW & TRADE-OFFS

**Bagging vs. Boosting — THE Comparison Table:**

| Aspect | Bagging (RF) | Boosting (XGBoost) |
|---|---|---|
| Training | **Parallel** (independent trees) | **Sequential** (each tree depends on previous) |
| Reduces | **Variance** | **Bias** (and variance) |
| Base Learner | Full/deep trees | **Shallow trees** (stumps to depth 3-6) |
| Overfitting | Resistant (more trees = better) | **CAN overfit** (more trees can hurt) |
| Learning Rate | N/A | Critical hyperparameter (η) |
| Speed | Faster (parallelizable) | Slower (sequential) |
| Missing Values | Doesn't handle | XGBoost/LightGBM handle natively |

**Key Hyperparameters (XGBoost/LightGBM):**

| Parameter | Effect | Typical Range |
|---|---|---|
| `n_estimators` | Number of boosting rounds | 100-1000+ |
| `learning_rate` (η) | Shrinkage per step | 0.01-0.3 |
| `max_depth` | Tree depth | 3-8 (XGB), -1 for LGBM |
| `min_child_weight` | Min samples per leaf | 1-10 |
| `subsample` | Row sampling ratio | 0.6-0.9 |
| `colsample_bytree` | Feature sampling ratio | 0.6-0.9 |
| `reg_alpha` (L1) | Lasso regularization | 0-1 |
| `reg_lambda` (L2) | Ridge regularization | 1 (default) |

**Tuning Strategy:** Lower learning rate + more trees generally works better. Use **early stopping** to find optimal n_estimators.

## 4. REAL-WORLD APPLICATIONS

- **Kaggle Champion**: XGBoost/LightGBM win ~70%+ of tabular data competitions
- **Fraud Detection**: Class imbalance + complex patterns → XGBoost with `scale_pos_weight`
- **Churn Prediction**: Feature importance from boosting → actionable business insights
- **CTR Prediction**: LightGBM for speed at scale (billions of rows)

## 5. TOP INTERVIEW QUESTIONS

**Q1 (Easy):** _What is the difference between Bagging and Boosting?_
> Bagging: parallel, independent models, reduces **variance** (RF). Boosting: sequential, each model corrects previous errors, reduces **bias** (XGBoost). Bagging uses deep trees; Boosting uses shallow trees.

**Q2 (Medium):** _Why does XGBoost handle missing values?_
> XGBoost learns the optimal direction for missing values during training. At each split, it sends missing values left AND right, measures the gain for each, and picks the better direction. This is learned from data, not imputed.

**Q3 (Hard):** _You're choosing between Random Forest and XGBoost. How do you decide?_
> **RF**: When you need robustness without much tuning, low risk of overfitting, parallel training matters, and you have limited time for hyperparameter search.
> **XGBoost**: When you need maximum predictive power, can afford tuning time, have imbalanced/complex data, and need built-in missing value handling. XGBoost typically wins on accuracy but requires more careful tuning.

**Q4 (Follow-up):** _"Can boosting overfit? How do you prevent it?"_
> **Yes!** Unlike RF, adding more trees in boosting CAN overfit. Prevent with:
> 1. **Early stopping** (monitor validation loss, stop when it increases)
> 2. Lower **learning rate** (η = 0.01-0.1)
> 3. **Regularization** (gamma, lambda, alpha)
> 4. **Subsampling** (subsample < 1.0, colsample_bytree < 1.0) — adds randomness like bagging
> 5. Limit **max_depth** (3-6)

## 6. PRACTICAL CODING

```python
import xgboost as xgb
import lightgbm as lgb
from sklearn.model_selection import cross_val_score

# --- XGBoost ---
xgb_model = xgb.XGBClassifier(
    n_estimators=500,
    learning_rate=0.05,
    max_depth=5,
    min_child_weight=3,
    subsample=0.8,
    colsample_bytree=0.8,
    reg_lambda=1.0,
    scale_pos_weight=10,  # for imbalanced data (ratio neg/pos)
    eval_metric='auc',
    early_stopping_rounds=50,
    random_state=42,
    n_jobs=-1
)
xgb_model.fit(X_train, y_train,
              eval_set=[(X_val, y_val)],
              verbose=50)

# --- LightGBM ---
lgb_model = lgb.LGBMClassifier(
    n_estimators=1000,
    learning_rate=0.05,
    num_leaves=31,        # leaf-wise: control complexity here, NOT max_depth
    min_child_samples=20,
    subsample=0.8,
    colsample_bytree=0.8,
    reg_alpha=0.1,
    reg_lambda=1.0,
    is_unbalance=True,    # for imbalanced data
    random_state=42,
    n_jobs=-1
)
lgb_model.fit(X_train, y_train,
              eval_set=[(X_val, y_val)],
              callbacks=[lgb.early_stopping(50), lgb.log_evaluation(50)])
```

**5-Bullet Quick Revision:**
1. **Bagging = parallel, reduces variance. Boosting = sequential, reduces bias.** Both are ensemble methods.
2. Gradient Boosting fits each new tree to **residuals** (negative gradient of loss).
3. XGBoost uses **2nd-order Taylor expansion** (gradient + Hessian) for faster, better splits.
4. **Boosting CAN overfit** — use early stopping, low learning rate, regularization. RF cannot overfit with more trees.
5. LightGBM is faster (leaf-wise growth) but can overfit more easily. XGBoost is more stable (level-wise growth).

---

# ═══════════════════════════════════════════════
# TOPIC 6: GRADIENT DESCENT
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

**What It Does:** Iteratively minimizes a function by moving in the direction of steepest descent (negative gradient).

$$\theta_{t+1} = \theta_t - \eta \cdot \nabla J(\theta_t)$$
- $\theta$ = parameters, $\eta$ = learning rate, $\nabla J$ = gradient (direction of steepest ascent)

```
J(θ)
  │\
  │ \
  │  \        ╱
  │   \      ╱
  │    \    ╱
  │     \  ╱
  │      \/  ← global minimum (we want to reach here)
  │
  └────────────── θ

  Step: θ_new = θ_old - η × slope
  Negative slope → move right. Positive slope → move left.
```

### Variants

| Variant | Batch Size | Per-step Cost | Convergence | Noise |
|---|---|---|---|---|
| **Batch GD** | Entire dataset ($n$) | $O(n)$ | Smooth, slow | None |
| **Stochastic GD (SGD)** | 1 sample | $O(1)$ | Noisy, fast | High (can escape local minima) |
| **Mini-batch GD** | $b$ samples (32-512) | $O(b)$ | Best of both | Moderate |

### Learning Rate — The Most Critical Hyperparameter

```
Too Small:                  Just Right:                Too Large:
    ╲                          ╲                        ╱╲╱╲╱╲
     ╲                          ╲                      ╱      ╲
      ╲                          ╲╱  ← converges      ╱  diverges!
       ╲  ← very slow
        ╲
```

### Advanced Optimizers

| Optimizer | Key Idea | When to Use |
|---|---|---|
| **SGD + Momentum** | Accumulates velocity → faster convergence | Simple, well-understood |
| **RMSprop** | Per-parameter adaptive LR (divides by running avg of squared gradients) | RNNs |
| **Adam** | Momentum + RMSprop combined | **Default choice** for deep learning |
| **AdamW** | Adam + decoupled weight decay | Transformers, modern DL |

**Adam Update Rule:**
$$m_t = \beta_1 m_{t-1} + (1-\beta_1) g_t \quad \text{(1st moment — momentum)}$$
$$v_t = \beta_2 v_{t-1} + (1-\beta_2) g_t^2 \quad \text{(2nd moment — adaptive LR)}$$
$$\hat{m}_t = \frac{m_t}{1-\beta_1^t}, \quad \hat{v}_t = \frac{v_t}{1-\beta_2^t} \quad \text{(bias correction)}$$
$$\theta_{t+1} = \theta_t - \frac{\eta}{\sqrt{\hat{v}_t} + \epsilon} \hat{m}_t$$

Defaults: $\beta_1 = 0.9$, $\beta_2 = 0.999$, $\epsilon = 10^{-8}$

## 2. TOP INTERVIEW QUESTIONS

**Q1:** _Why use SGD over Batch GD?_
> SGD is faster (updates after each sample), uses less memory, and the noise helps escape shallow local minima. But it's noisier → use learning rate scheduling or momentum for stability.

**Q2:** _What is the vanishing gradient problem?_
> In deep networks with sigmoid/tanh activations, gradients get multiplied through layers. Since sigmoid derivative max = 0.25, after many layers: $0.25^{L} \to 0$. Fix: ReLU activation, batch normalization, skip connections (ResNet), LSTM/GRU for RNNs.

**Q3:** _When would you NOT use Adam?_
> When you need better generalization (SGD + momentum often generalizes better on final test performance). When training is very long (Adam can converge to sharp minima). For large-scale vision models, SGD is often preferred.

---

# ═══════════════════════════════════════════════
# TOPIC 7: REGULARIZATION (L1, L2, ElasticNet)
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

**Why Regularize?** When a model has too many features or too much flexibility, it starts fitting noise in the training data instead of the actual pattern. Regularization adds a **penalty term** to the loss function that discourages overly complex (large coefficient) models.

Think of it as telling the model: "I want you to minimize errors, BUT I'll also punish you for having large weights. So find a solution that's good enough WITHOUT going overboard."

| Type | Penalty | Effect on Weights | Geometry | Best For |
|---|---|---|---|---|
| **L1 (Lasso)** | $\lambda \sum \vert \beta_j \vert$ | Drives weights to **exactly 0** | Diamond constraint | **Feature selection** (sparse models) |
| **L2 (Ridge)** | $\lambda \sum \beta_j^2$ | Shrinks weights **toward 0** (never exactly 0) | Circle constraint | **Multicollinearity** handling |
| **ElasticNet** | $\alpha\lambda\sum \vert \beta_j \vert + \frac{(1-\alpha)\lambda}{2}\sum\beta_j^2$ | Mix of both | Blend | Correlated features + sparsity |

**Geometric Intuition:**
```
      β₂                        β₂
       │   /                      │   /
       │  / OLS                   │  / OLS
       │ /  solution              │ /  solution
       │╱                        │╱
  ─────◆──── β₁             ────(●)──── β₁
       │╲                        │╲
       │ L1 diamond              │ L2 circle
       │ (hits corners =         │ (touches smoothly =
       │  zero weights)          │  small weights)
```

L1's diamond has **corners on axes** → the OLS contour is more likely to hit a corner → some $\beta_j = 0$ → **automatic feature selection**.

## 2. MATHEMATICS THAT MATTERS

**Ridge:**
$$\hat{\beta}_{\text{ridge}} = (X^TX + \lambda I)^{-1}X^Ty$$
- Adding $\lambda I$ makes $(X^TX + \lambda I)$ **always invertible** → solves multicollinearity
- $\lambda \to 0$: OLS. $\lambda \to \infty$: all coefficients → 0 (underfitting)

**Lasso:** No closed form (L1 is non-differentiable at 0) → solved with coordinate descent or subgradient methods.

**Bayesian Interpretation:**
- L2 = Gaussian prior on weights: $\beta_j \sim N(0, \frac{1}{\lambda})$
- L1 = Laplace prior on weights: peaked at 0 → encourages sparsity

## 3. TOP INTERVIEW QUESTIONS

**Q1:** _When would you use Lasso over Ridge?_
> When you suspect many features are irrelevant and want automatic **feature selection** (Lasso sets coefficients to 0). Ridge when all features are potentially relevant but some are correlated.

**Q2:** _What happens to bias and variance as λ increases?_
> **Bias ↑, Variance ↓**. High λ → heavy regularization → simpler model → more bias but less overfitting. There's an optimal λ (found via cross-validation) that minimizes total error.

```python
from sklearn.linear_model import Ridge, Lasso, ElasticNet
from sklearn.model_selection import GridSearchCV

# Ridge with CV
ridge = Ridge()
params = {'alpha': [0.01, 0.1, 1.0, 10.0, 100.0]}
grid = GridSearchCV(ridge, params, cv=5, scoring='neg_mean_squared_error')
grid.fit(X_train, y_train)
print(f"Best α: {grid.best_params_['alpha']}")

# Lasso (for feature selection)
lasso = Lasso(alpha=0.1)
lasso.fit(X_train, y_train)
selected = [f for f, c in zip(feature_names, lasso.coef_) if c != 0]
print(f"Selected features: {selected}")
```

**5-Bullet Quick Revision:**
1. **L1 (Lasso)** = sparsity / feature selection. **L2 (Ridge)** = shrinkage / handles multicollinearity.
2. Ridge has a **closed form**: $(X^TX + \lambda I)^{-1}X^Ty$. Lasso does not (uses coordinate descent).
3. **↑ λ = ↑ bias, ↓ variance**. Optimal λ found via cross-validation.
4. **Bayesian view**: L2 = Gaussian prior, L1 = Laplace prior on weights.
5. **ElasticNet** = best of both when features are correlated AND you want sparsity.

---

# ═══════════════════════════════════════════════
# TOPIC 8: BIAS-VARIANCE TRADEOFF
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

$$\text{Total Error} = \text{Bias}^2 + \text{Variance} + \text{Irreducible Noise}$$

| Component | Definition | Source |
|---|---|---|
| **Bias** | Error from wrong assumptions (underfitting) | Model too simple |
| **Variance** | Error from sensitivity to training data (overfitting) | Model too complex |
| **Irreducible Error** | Noise inherent in data | Can't be reduced |

```
Error
  │╲                     ╱
  │  ╲  Total Error    ╱
  │   ╲    ╱╲        ╱
  │    ╲ ╱    ╲    ╱
  │     X  sweet  ╲╱  Variance
  │   ╱  ╲ spot
  │ ╱     ╲
  │╱ Bias² ╲─────────
  └───────────────────── Model Complexity
  Simple ←────────────→ Complex
```

**Algorithm Spectrum:**

| Algorithm | Bias | Variance | Notes |
|---|---|---|---|
| Linear Regression | High | Low | Underfits non-linear data |
| **KNN (K=1)** | **Low (≈0)** | **Very High** | **OVERFITS** — memorizes training data |
| KNN (K=n) | High | Low | Predicts mean of all data → underfits |
| Decision Tree (deep) | Low | High | Memorizes training set |
| Random Forest | Low | **Low** | Bagging reduces variance |
| Boosted Trees | **Low** | Low-Med | Boosting reduces bias |
| Neural Network (large) | Low | High | Needs regularization (dropout, etc.) |

> K=1 **OVERFITS (low bias, high variance).** Each prediction = its nearest neighbor, essentially memorizing the training data. Noisy points directly affect predictions.

## 2. TOP INTERVIEW QUESTIONS

**Q1:** _K=1 in KNN: overfit or underfit?_
> **Overfit.** Training error = 0 (each point is its own nearest neighbor). Test error is high because the model captures noise. Increasing K smooths the decision boundary → reduces variance but increases bias.

**Q2:** _How does model complexity relate to bias-variance?_
> ↑ Complexity = ↓ Bias + ↑ Variance. Goal: find the sweet spot (minimum total error). Use cross-validation to estimate generalization error.

**Q3:** _How does Random Forest achieve low bias AND low variance?_
> Individual deep trees have low bias (complex) but high variance. Averaging many decorrelated trees (via bootstrap + feature randomness) reduces variance while keeping bias low. It's the best of both worlds.

---

# ═══════════════════════════════════════════════
# TOPIC 9: SVM & KERNEL TRICK
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

**Objective:** Find the hyperplane with the **maximum margin** separating classes.

```
        │            ● ●           Support Vectors
  ○     │     ●    ●               (closest points to boundary)
     ○  │  ●     ●                       │
  ○   ○ │● ●   ●                         ▼
     ○  │  ● ●                     ○ ←──│margin│──→ ●
  ○   ○ │   ●                            │
        │                      ────────[hyperplane]────────
  Class 0  Class 1
```

**Hard Margin vs. Soft Margin:**
- **Hard margin**: No misclassification allowed → fails with non-separable data
- **Soft margin**: Allow some misclassification controlled by $C$
  - Large $C$ → less tolerance for misclassification (may overfit)
  - Small $C$ → more tolerance (may underfit)

### Kernel Trick

**Problem:** Data isn't linearly separable in original space.
**Solution:** Map to higher-dimensional space where it IS separable.

```
Original Space (2D):              Feature Space (3D):
    ○ ○ ○                                    ● ●
   ○ ● ● ○                              ● ● ● ●
  ○ ● ● ● ○                           ─────────── ← linear separator
   ○ ● ● ○                              ○ ○ ○
    ○ ○ ○                            ○ ○ ○ ○ ○ ○
Not linearly separable!          Linearly separable in higher dim!
```

**The Trick:** Never actually compute the transformation $\phi(x)$. Instead, use **kernel functions** $K(x_i, x_j) = \phi(x_i) \cdot \phi(x_j)$ that compute the dot product in the higher-dimensional space directly.

| Kernel | Formula | Use Case |
|---|---|---|
| Linear | $K(x,y) = x \cdot y$ | Linearly separable data |
| Polynomial | $K(x,y) = (x \cdot y + c)^d$ | Moderate non-linearity |
| **RBF (Gaussian)** | $K(x,y) = \exp(-\gamma \lVert x-y \rVert^2)$ | **Default**, most flexible |
| Sigmoid | $K(x,y) = \tanh(\alpha x \cdot y + c)$ | Rarely used (not always valid) |

**RBF parameter $\gamma$:**
- Large $\gamma$ → each point has small influence → complex boundary → **overfit**
- Small $\gamma$ → each point has large influence → smooth boundary → **underfit**

## 2. TOP INTERVIEW QUESTIONS

**Q1:** _Explain the kernel trick in simple terms._
> Instead of explicitly mapping data to a higher-dimensional space (expensive), kernel functions compute the similarity (dot product) in that space directly using the original features. This makes SVM efficient even with infinite-dimensional feature spaces (like RBF kernel).

**Q2:** _SVM vs. Logistic Regression — when to use which?_
> **SVM**: Small-medium datasets, high-dimensional sparse data (text), non-linear boundaries (with kernels). **LR**: Need probability output, interpretable model, large datasets (scales better), online learning.

**Q3:** _Does SVM work well for large datasets?_
> **No** — training complexity is $O(n^2)$ to $O(n^3)$ for kernel SVM. For large n, prefer linear SVM (`LinearSVC`), logistic regression, or tree-based methods.

---

# ═══════════════════════════════════════════════
# TOPIC 10: METRICS (Precision, Recall, F1, ROC, AUC)
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

**Confusion Matrix:**
```
                     Predicted
                   Pos      Neg
Actual  Pos     [ TP       FN ]     ← Recall = TP/(TP+FN)
        Neg     [ FP       TN ]     ← Specificity = TN/(TN+FP)
                  ↑
          Precision = TP/(TP+FP)
```

**Key Metrics:**

| Metric | Formula | Optimize When | Example |
|---|---|---|---|
| **Accuracy** | $(TP+TN)/(TP+TN+FP+FN)$ | Balanced classes | General classification |
| **Precision** | $TP/(TP+FP)$ | FP is costly | Spam filter (don't mark real email as spam) |
| **Recall (Sensitivity/TPR)** | $TP/(TP+FN)$ | FN is costly | **Cancer/Fraud detection** (don't miss positive cases) |
| **F1 Score** | $2 \cdot \frac{P \cdot R}{P + R}$ | Need balance of P and R | Imbalanced datasets |
| **Specificity (TNR)** | $TN/(TN+FP)$ | FP is costly | Drug testing |
| **FPR** | $FP/(FP+TN) = 1 - \text{Specificity}$ | ROC curve x-axis | — |

**ROC Curve & AUC:**
```
TPR (Recall)
  1.0│     ●●●●●●●●●●
     │   ●●
     │  ●●    ← ROC curve
     │ ●● 
     │●●          Area Under = AUC
     │●           AUC = 0.5 → random
 0.0│●            AUC = 1.0 → perfect
    └──────────────
   0.0   FPR    1.0
```

**ROC-AUC vs. PR-AUC:**
- **ROC-AUC**: Use for balanced datasets. Can be misleading with heavy imbalance.
- **PR-AUC (Precision-Recall)**: Use for **imbalanced datasets** — more informative when negatives dominate.

## 2. TOP INTERVIEW QUESTIONS

**Q1:** _When would F1 be misleading?_
> When costs of FP and FN are very different. F1 weighs them equally. Use **Fβ** where $\beta > 1$ emphasizes recall (e.g., $F_2$ for fraud) and $\beta < 1$ emphasizes precision.

**Q2:** _AUC = 0.5. What does that mean?_
> Model performs no better than random guessing. The ROC curve follows the diagonal.

**Q3:** _Your model has 99% accuracy on a fraud dataset (1% fraud). Is it good?_
> **No!** A dummy classifier predicting all "not fraud" gets 99% accuracy. Check recall for the fraud class (probably 0%). Use F1, PR-AUC, or class-specific metrics.

---

# ═══════════════════════════════════════════════
# TOPIC 11: MISSING DATA IMPUTATION
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

**Types of Missingness:**

| Type | Meaning | Example | Implication |
|---|---|---|---|
| **MCAR** | Missing Completely At Random | Sensor randomly fails | Safe to impute or drop |
| **MAR** | Missing At Random (given observed data) | High-income people skip income question | Can impute using other features |
| **MNAR** | Missing Not At Random | Sick people skip health survey | **Most dangerous** — imputation can bias results |

**Imputation Methods:**

| Method | When To Use | Pros | Cons |
|---|---|---|---|
| **Drop rows** | MCAR, small % missing | Simple | Loses data, biased if not MCAR |
| **Mean/Median** | Quick baseline | Fast | Reduces variance, ignores correlations |
| **Mode** | Categorical | Fast | Same as above |
| **KNN Imputer** | MAR, local patterns | Captures local structure | Slow for large data, sensitive to K |
| **Iterative (MICE)** | MAR, complex patterns | Models feature relationships | Slow, complex |
| **Indicator variable** | When missingness is informative | Preserves signal | Adds features |

```python
from sklearn.impute import SimpleImputer, KNNImputer
from sklearn.experimental import enable_iterative_imputer
from sklearn.impute import IterativeImputer

# Mean/Median imputation
imp = SimpleImputer(strategy='median')
X_imputed = imp.fit_transform(X_train)

# KNN Imputer
knn_imp = KNNImputer(n_neighbors=5)
X_imputed = knn_imp.fit_transform(X_train)

# MICE (Iterative)
mice_imp = IterativeImputer(max_iter=10, random_state=42)
X_imputed = mice_imp.fit_transform(X_train)
```

**⚠️ Critical Rule:** Fit imputer on **training data only**, then transform test data. Never fit on full dataset (data leakage!).

---

# ═══════════════════════════════════════════════
# TOPIC 12: KNN (K-Nearest Neighbors)
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

**Lazy Learner:** No training phase — stores the entire dataset and computes at prediction time.

$$\hat{y} = \text{majority vote of } K \text{ nearest neighbors (classification)}$$
$$\hat{y} = \text{average of } K \text{ nearest neighbors (regression)}$$

**Distance Metrics:**
- **Euclidean**: $\sqrt{\sum(x_i - y_i)^2}$ — most common
- **Manhattan**: $\sum|x_i - y_i|$ — better for high dimensions
- **Cosine**: $1 - \frac{x \cdot y}{\|x\|\|y\|}$ — for text/NLP

**K's Effect on Bias-Variance:**
```
K=1:                    K=n:                     K=optimal:
●○●○●○●○               ●●●●●●●●                 ●●○○●●●●
Very complex            Predict majority class    Balanced
boundary                everywhere                boundary

Low Bias                High Bias                 Sweet Spot
High Variance           Low Variance              
→ OVERFIT              → UNDERFIT                 → Just Right
```

**K=1 → OVERFIT (memorizes training data, 0% training error)**

## 2. TOP INTERVIEW QUESTIONS

**Q1:** _Why is feature scaling essential for KNN?_
> KNN uses distance-based calculations. If one feature has range [0, 1000] and another [0, 1], the first feature dominates the distance. StandardScaler or MinMaxScaler equalizes feature contributions.

**Q2:** _What's the curse of dimensionality and how does it affect KNN?_
> In high dimensions, all points become equidistant → "nearest neighbor" loses meaning. Distance between nearest and farthest neighbor converges. Fix: dimensionality reduction (PCA), feature selection.

**Complexity:** Training: $O(1)$ (just stores data). Prediction: $O(nd)$ where $n$ = training samples, $d$ = dimensions. Use KD-tree or Ball-tree for speedup.

---

# ═══════════════════════════════════════════════
# TOPIC 13: MLE & MAP
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

**MLE (Maximum Likelihood Estimation):**
> "What parameters make the observed data MOST LIKELY?"

$$\hat{\theta}_{MLE} = \arg\max_\theta P(D|\theta) = \arg\max_\theta \prod_{i=1}^{n} P(x_i|\theta)$$

Take log (for computational convenience):
$$\hat{\theta}_{MLE} = \arg\max_\theta \sum_{i=1}^{n} \log P(x_i|\theta)$$

**MAP (Maximum A Posteriori):**
> "What parameters are most probable GIVEN the data AND prior belief?"

$$\hat{\theta}_{MAP} = \arg\max_\theta P(\theta|D) = \arg\max_\theta P(D|\theta) \cdot P(\theta)$$

**MLE vs. MAP:**

| Feature | MLE | MAP |
|---|---|---|
| Prior | None (uniform/uninformative) | Yes — regularization effect |
| Overfitting risk | Higher (no constraint) | Lower |
| With Gaussian prior | N/A | = **L2 Regularization (Ridge)** |
| With Laplace prior | N/A | = **L1 Regularization (Lasso)** |
| With infinite data | Converges to true θ | Converges to MLE (prior washes out) |

**Example — MLE for Gaussian:**
Given data $x_1, \ldots, x_n$ from $N(\mu, \sigma^2)$:
$$\hat{\mu}_{MLE} = \bar{x} = \frac{1}{n}\sum x_i, \quad \hat{\sigma}^2_{MLE} = \frac{1}{n}\sum(x_i - \bar{x})^2$$
Note: MLE of variance is **biased** (divides by $n$, not $n-1$). Unbiased estimator uses $n-1$.

## 2. TOP INTERVIEW QUESTIONS

**Q1:** _Connection between MLE and cross-entropy loss?_
> Minimizing cross-entropy loss = Maximizing log-likelihood for Bernoulli/categorical distributions. They are mathematically equivalent. This is why logistic regression uses log-loss.

**Q2:** _When would MAP be better than MLE?_
> Small datasets (prior provides regularization), when prior knowledge exists (domain expertise), and when you want to avoid overfitting. With enough data, MAP converges to MLE.

---

# ═══════════════════════════════════════════════
# TOPIC 14: NLP BASICS (TF-IDF, Embeddings, Transformers)
# ═══════════════════════════════════════════════

## 0. TEXT PREPROCESSING & TOKENIZATION

Before any advanced modeling, text must be cleaned and broken down.

**1. Basic Cleaning:**
- **Lowercasing**: "Apple" and "apple" become the same.
- **Stop word removal**: Removing common words like "the", "and", "is" (useful for TF-IDF, but **bad** for Transformers, which need full context).
- **Stemming vs. Lemmatization**:
  - *Stemming*: Crudely chops off word endings. "Running" → "Run". "Caring" → "Car". (Fast, but often produces non-words).
  - *Lemmatization*: Uses a dictionary to find the root word. "Better" → "Good". "Running" → "Run". (Slower, but grammatically correct).

**2. Tokenization:**
How do we split a sentence into pieces (tokens)?
- **Word Tokenization**: Split by spaces. *Problem*: Huge vocabulary size, and how do you handle new/made-up words (Out-of-Vocabulary or OOV)?
- **Character Tokenization**: Split into letters. *Problem*: Too granular. A model has to learn that 'c', 'a', 't' means cat. Sequence lengths become huge.
- **Subword Tokenization (BPE - Byte Pair Encoding):** *The modern standard (used in GPT, BERT).* It starts with characters and iteratively merges the most frequent pairs. 
  - Common words remain whole: "happy" → ["happy"]
  - Rare words are split: "unhappiness" → ["un", "happi", "ness"]
  - *Advantage*: Handles OOV words perfectly and keeps the vocabulary size manageable (e.g., 50k tokens).

## 1. TEXT REPRESENTATION: FROM BAG-OF-WORDS TO TF-IDF

Before a model can process text, it needs to be converted into numbers. 

**Bag-of-Words (BoW):**
The simplest approach. You create a vocabulary of all unique words. Each document is a vector counting how many times each word appears.
- *Problem:* "The" might appear 100 times, but it carries no meaning. Rare words like "quantum" might appear once but define the document's topic. BoW treats all counts equally.

**TF-IDF (Term Frequency - Inverse Document Frequency)**
TF-IDF fixes the BoW problem by balancing two competing metrics:

1. **TF (Term Frequency):** How often does the word appear in *this specific* document? (Local importance)
   $\text{TF} = \frac{\text{count of word in doc}}{\text{total words in doc}}$
2. **IDF (Inverse Document Frequency):** How rare is the word across *all* documents? (Global rarity)
   $\text{IDF} = \log\left(\frac{\text{total number of documents}}{\text{documents containing the word}}\right)$

$$\text{TF-IDF} = \text{TF} \times \text{IDF}$$

**Intuition Example:**
Imagine an article about astronomy in a corpus of 1000 random articles.
- "The" appears frequently (High TF), but it's in all 1000 documents (IDF = $\log(1000/1000) = 0$). TF-IDF = 0.
- "Galaxy" appears frequently in this article (High TF) and is rare globally (e.g., in only 5 docs, IDF = $\log(1000/5) \approx 5.3$). TF-IDF is High!
TF-IDF automatically highlights the words that make a document unique.

## 2. WORD EMBEDDINGS (Dense Vectors)

TF-IDF produces sparse vectors (mostly zeros) where the size equals the vocabulary size (e.g., 50,000 dimensions). More importantly, it has **no semantic understanding**. The distance between "king" and "queen" is the same as "king" and "apple".

**Word Embeddings (e.g., Word2Vec)** map words to dense, low-dimensional vectors (e.g., 300 dimensions). 
The core philosophy (Distributional Hypothesis): *"You shall know a word by the company it keeps."*

**Word2Vec (Skip-gram model) Intuition:**
Imagine a sliding window moving across text.
Sentence: "The cute dog barked loudly"
Center word: "dog". Context words: ["The", "cute", "barked", "loudly"]

We train a shallow neural network on a fake task: "Given the center word 'dog', predict the context words."
To get good at this, the network must learn that "dog" and "cat" often share the same context ("cute", "barked", "meowed"). 
Once training is done, we throw away the prediction part. The **internal weights** of the network become our Word Embeddings! Because "dog" and "cat" had to predict similar context words, their resulting vectors end up very close in the 300-D space.

*Famous property:* Semantic math works! $\vec{\text{King}} - \vec{\text{Man}} + \vec{\text{Woman}} \approx \vec{\text{Queen}}$

## 3. THE TRANSFORMER ARCHITECTURE & SELF-ATTENTION

Word2Vec is static: the vector for "bank" (river) and "bank" (money) is exactly the same.
**Transformers (like BERT and GPT)** solved this by creating *contextual* embeddings. "Bank" gets a different vector depending on the surrounding words.

### The Magic of Self-Attention
Self-attention asks: "When processing a specific word, how much focus (attention) should I pay to every other word in the sentence?"

**The Query, Key, Value (Q, K, V) Analogy:**
Think of retrieving information from a database or a library.
- **Query (Q):** What I'm looking for (e.g., "Subject of this verb").
- **Key (K):** What each word offers (e.g., "I am a noun").
- **Value (V):** The actual meaning/content of the word.

For the sentence: *"The animal didn't cross the street because it was too tired."*
What does "it" refer to? The animal or the street?
- The word "it" produces a **Query**.
- "animal" and "street" produce **Keys**.
- The model computes the dot product of the Query of "it" with all Keys. The Query for "it" will match highly with the Key for "animal" (because of "tired").
- We then take a weighted sum of the **Values** of all words, heavily weighted toward "animal". The new contextual embedding for "it" now contains the meaning of "animal"!

### Key Transformer Components
1. **Multi-Head Attention:** Instead of doing self-attention once, we do it multiple times in parallel (heads). One head might focus on grammar (subject-verb), another on adjectives, etc.
2. **Positional Encoding:** Transformers process all words simultaneously (unlike RNNs). They have no sense of word order. We must inject a positional signal (usually sine/cosine waves) into the word embeddings so the model knows "A dog bit John" is different from "John bit a dog".

**BERT vs GPT:**
- **BERT (Encoder only):** Bidirectional. Reads the whole sentence at once. Trained by masking out words ("The [MASK] sat on the mat") and predicting them. Great for classification, NER, understanding.
- **GPT (Decoder only):** Autoregressive. Reads left-to-right. Trained to predict the *next* word. Great for generation (ChatGPT).


## 4. NLP EVALUATION METRICS

How do we score models that generate text (like translation or summarization)? Accuracy doesn't work because there are many valid ways to write the same thing.

- **BLEU (Bilingual Evaluation Understudy):** Used mostly for **Translation**. It measures *Precision*. How many of the n-grams (1-word, 2-word chunks) in the *generated* text actually appeared in the *reference* text?
  - *Critique:* It penalizes you if you use a valid synonym that wasn't in the reference.
- **ROUGE (Recall-Oriented Understudy for Gisting Evaluation):** Used mostly for **Summarization**. It measures *Recall*. How many of the n-grams in the *human reference* summary were captured by the *generated* summary?
- **Perplexity:** Used for Language Models (like GPT). It measures how "surprised" the model is by a real test set. Lower perplexity = better model (it assigned high probability to the real text).

## 5. MODERN LLM PARADIGMS

When you have a base model (like GPT-4 or LLaMA) but need it to know about your company's private data, you have two choices:

1. **Fine-Tuning:**
   - *What:* Actually changing the model's internal weights by training it on your specific examples (Q&A pairs, instructions).
   - *Best for:* Teaching the model a **new tone**, **format**, or **style** (e.g., teaching it to reply like a pirate, or output strict JSON).
   - *Bad for:* Memorizing facts. It's expensive and models still hallucinate.

2. **RAG (Retrieval-Augmented Generation):**
   - *What:* You don't change the model. Instead, when a user asks a question, you search your private database (using vector embeddings) for the relevant document, paste that document into the prompt, and say: "Answer the question using ONLY this text."
   - *Best for:* **Factual accuracy**, preventing hallucinations, and citing sources. It's much cheaper and you can update the database instantly without retraining the model.

---

# ═══════════════════════════════════════════════
# TOPIC 15: DEEP LEARNING ARCHITECTURES
# ═══════════════════════════════════════════════

## 1. CNN (Convolutional Neural Network)

**Core Use Case:** Images, spatial data.
Before CNNs, we fed images into standard neural networks by flattening them into a 1D line of pixels. This destroyed spatial relationships (pixels next to each other were now far apart).

CNNs preserve the 2D grid using **Convolutions**.

**The Convolution Operation:**
Imagine a small 3x3 grid (a **filter** or **kernel**) sliding across the image like a magnifying glass. 
At each step, it multiplies its own numbers by the pixel values underneath it and sums them up.
- Early layers have filters that act as edge detectors (vertical edges, horizontal edges).
- Deeper layers combine these edges to detect shapes (circles, corners).
- Even deeper layers detect complex objects (faces, cars).

**Why CNNs are brilliant:**
1. **Parameter Sharing:** A filter that learns to detect a cat ear in the top-left corner uses the *exact same weights* to detect a cat ear in the bottom-right corner. This drastically reduces the number of parameters compared to standard networks.
2. **Translation Invariance:** A cat is recognized as a cat no matter where it is in the image.

**Pooling (Max Pooling):**
Periodically, we downsample the image (e.g., taking the maximum value in every 2x2 patch). This reduces computation and makes the network even more robust to slight shifts and distortions in the image.

## 2. RNN & LSTM (Recurrent Neural Networks)

**Core Use Case:** Sequential data (Text, Time Series, Audio), where order matters and length is variable.

**The Vanilla RNN:**
Standard networks have no memory. An RNN has a loop: it processes step 1, produces an output, and passes a **Hidden State** (memory) to step 2. 
At step 2, it looks at the new input AND the memory from step 1.
*Flaw:* **The Vanishing Gradient Problem.** In a long sentence, the gradient (learning signal) gets multiplied by a small number over and over as it travels back through time. By the time it reaches the beginning of the sentence, it's virtually zero. The RNN forgets early words.

**LSTM (Long Short-Term Memory) to the rescue:**
LSTMs fix the memory problem by introducing a "Cell State" — a conveyor belt running straight through the entire sequence. Information can flow down this belt easily.
It uses **Gates** (small neural networks outputting values between 0 and 1) to control what enters and leaves the belt:
1. **Forget Gate:** Decides what old information to throw away. (e.g., "The subject was singular, but I just saw a period. Forget the singular subject.")
2. **Input Gate:** Decides what new information to add to the belt.
3. **Output Gate:** Decides what part of the memory to output for this specific time step.
LSTMs were the king of NLP before Transformers took over.

## 3. GANs (Generative Adversarial Networks)

**Core Use Case:** Generating highly realistic new data (faces, art, deepfakes).

GANs are a fascinating setup where two neural networks fight against each other in a zero-sum game.

1. **The Generator (The Forger):** Starts with completely random noise and tries to transform it into a realistic image (e.g., a human face).
2. **The Discriminator (The Detective):** Looks at images and tries to classify them as "Real" (from the actual training data) or "Fake" (created by the Generator).

**The Training Loop:**
- The Generator creates a batch of fake images.
- The Discriminator is trained on a mix of real and fake images to tell them apart.
- Then, we train the Generator. Its goal is to maximize the Discriminator's mistake rate. It learns: "When I changed these pixels, the Discriminator thought it was real!"
- Over time, the Discriminator gets better at spotting fakes, which forces the Generator to produce even more photorealistic images to fool it.

*Challenges:* They are notoriously hard to train. If the Discriminator gets too good too fast, the Generator gets no useful feedback. If the Generator finds one specific image that always fools the Discriminator, it will only generate that one image (**Mode Collapse**).

## 4. TOP INTERVIEW QUESTIONS

**Q1:** *CNN vs. RNN vs. Transformer — when to use which?*
> **CNN**: Spatial data (images). **RNN/LSTM**: Temporal/Sequential data where order matters (time series). **Transformers**: Modern NLP and increasingly vision. They replace RNNs because they can process whole sequences in parallel (no unrolling through time) and handle long-range dependencies perfectly via self-attention.

**Q2:** *Why do we need Positional Encoding in Transformers but not RNNs?*
> RNNs inherently understand sequence because they process data one step at a time (t=1, t=2...). Transformers process the entire sequence simultaneously in parallel. Without positional encoding added to the embeddings, a Transformer would treat a sentence as an unordered bag of words.

**Q3:** *What is the purpose of Pooling in a CNN?*
> Pooling (like Max Pooling) reduces the spatial dimensions (width and height) of the representation. This reduces the number of parameters and computation in the network. More importantly, it provides spatial translation invariance—the exact location of a feature (like an edge) becomes less important than its rough relative location.


# ═══════════════════════════════════════════════
# TOPIC 16: DENSITY ESTIMATION (KDE, K-Means)
# ═══════════════════════════════════════════════

## 1. KDE (Kernel Density Estimation)

**What It Does:** Non-parametric way to estimate the probability density function.

$$\hat{f}(x) = \frac{1}{nh}\sum_{i=1}^{n} K\left(\frac{x - x_i}{h}\right)$$
- $K$ = kernel function (usually Gaussian)
- $h$ = bandwidth (smoothness parameter)
  - Small $h$ → spiky, noisy estimate (overfitting)
  - Large $h$ → oversmoothed, loses detail (underfitting)

## 2. K-Means Clustering

```
[Choose K centers randomly]
       │
       ▼
[Assign each point to nearest center]  ←──┐
       │                                    │
       ▼                                    │
[Update centers = mean of assigned points]──┘
       │                              (repeat until convergence)
       ▼
[Final K clusters]
```

**Objective:** Minimize within-cluster sum of squares (WCSS / Inertia):
$$J = \sum_{k=1}^{K} \sum_{x_i \in C_k} \|x_i - \mu_k\|^2$$

**Limitations:**
- Assumes **spherical, equal-sized** clusters
- Sensitive to **initialization** → use K-Means++ (smarter init)
- Sensitive to **outliers** → consider K-Medoids
- Must specify $K$ → use **Elbow Method** or **Silhouette Score**

```python
from sklearn.cluster import KMeans

# Elbow Method
inertias = []
for k in range(1, 11):
    km = KMeans(n_clusters=k, init='k-means++', random_state=42)
    km.fit(X)
    inertias.append(km.inertia_)

# Plot inertias vs k → look for "elbow" (diminishing returns)
```

---

# ═══════════════════════════════════════════════
# TOPIC 17: MATRIX FACTORIZATION (Recommendation Systems)
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS

**Problem:** User-Item rating matrix $R$ (m×n) is very sparse. Predict missing entries.

**Solution:** Factorize $R \approx U \cdot V^T$
- $U$ (m×k) = user latent factors
- $V$ (n×k) = item latent factors
- $k$ = number of latent dimensions (e.g., 50-200)

**Objective:**
$$\min_{U,V} \sum_{(i,j) \in \text{observed}} (r_{ij} - \mathbf{u}_i^T \mathbf{v}_j)^2 + \lambda(\|U\|^2 + \|V\|^2)$$

**Predict:** $\hat{r}_{ij} = \mathbf{u}_i^T \mathbf{v}_j$

**Types:**
- **Collaborative Filtering**: Based on user-item interactions (matrix factorization)
- **Content-Based**: Based on item features (e.g., genre, description)
- **Hybrid**: Combines both

---

# ML & DL MASTER REVISION TABLE

| Topic | Key Formula / Concept | #1 Interview Trap | Quick Fact |
|---|---|---|---|
| **Linear Regression** | $\hat{\beta} = (X^TX)^{-1}X^Ty$ | 5 Assumptions: LINE+H | R² always ↑ with more features → use Adj R² |
| **Logistic Regression** | Loss = Binary Cross-Entropy | NOT MSE (non-convex) | Coefficients = log-odds, $e^\beta$ = odds ratio |
| **Decision Trees** | Gini = $1 - \sum p_k^2$ | Greedy → not globally optimal | Overfit easily → use ensembles |
| **Random Forest** | Bagging + feature randomness | More trees NEVER overfit | OOB score = free validation |
| **Boosting** | Sequential, fits residuals | CAN overfit (unlike RF) | Early stopping is essential |
| **Gradient Descent** | $\theta = \theta - \eta \nabla J$ | Vanishing gradient problem | Adam = default for DL |
| **Regularization** | L1 = sparsity, L2 = shrinkage | L2 = Gaussian prior, L1 = Laplace | Ridge has closed form, Lasso doesn't |
| **Bias-Variance** | Error = Bias² + Variance + Noise | **KNN K=1 → OVERFIT** | RF = low bias + low variance |
| **SVM** | Max margin + kernel trick | $O(n^2)$–$O(n^3)$ for kernel | RBF kernel = default |
| **Metrics** | F1 = 2PR/(P+R) | 99% accuracy ≠ good (imbalance!) | Use PR-AUC for imbalanced data |
| **MLE/MAP** | MAP with Gaussian prior = Ridge | MLE variance estimate is biased | Cross-entropy loss = negative log-likelihood |
| **NLP** | TF-IDF → Word2Vec → BERT | BERT = contextual, W2V = static | Transformers replace RNNs (parallelizable) |
| **DL Architectures** | CNN=spatial, RNN=sequential | Vanishing gradient → LSTM | Dropout = DL regularization |
