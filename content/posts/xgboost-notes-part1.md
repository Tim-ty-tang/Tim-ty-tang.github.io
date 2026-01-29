---
title: "XGBoost Notes - Part 1: Internals"
date: "2026-01-10"
summary: "xgboost internals - math, tree methods, regularization"
description: "xgboost internals - math, tree methods, regularization"
toc: false
readTime: true
---

Quick notes on XGBoost after going through the codebase and some papers, it's been a while i worked on traditional tabular ML so i want a revisit. Mostly for my own reference, but maybe useful if you're trying to understand why this thing works so well.

This is part 1 covering the internals. [Part 2](/posts/xgboost-notes-part2) covers practical usage and production.

## Why XGBoost is still relevant

In the age of LLMs, tree-based models still dominate tabular data. Research (and experience) shows XGBoost beats deep learning on 8 of 11 tabular benchmarks and that includes benchmarks from papers proposing the deep learning architectures themselves. To this day XGBoost is still the king.

The reasons are pretty intuitive:
- Trees naturally handle irregular, non-smooth relationships (tabular data is full of these)
- Trees ignore uninformative features by just not splitting on them
- Works great with 1K-100K samples (deep learning needs way more) (sub 10k is a very common space)
- Sensible defaults actually work

## The actual trick: Newton-Raphson in function space

Traditional gradient boosting is gradient descent in function space. XGBoost is Newton-Raphson instead - it uses both first and second derivatives.

The objective at step t:

```
obj^(t) = Σᵢ l(yᵢ, ŷᵢ^(t-1) + fₜ(xᵢ)) + Ω(fₜ)
```

Taylor expand to second order:

```
obj^(t) ≈ Σᵢ [l(yᵢ, ŷᵢ^(t-1)) + gᵢfₜ(xᵢ) + ½hᵢfₜ²(xᵢ)] + Ω(fₜ)
```

Where:
- gᵢ = ∂l/∂ŷ (first derivative, the gradient)
- hᵢ = ∂²l/∂ŷ² (second derivative, the hessian)

The key insight: after removing constants, the objective only depends on gᵢ and hᵢ. This means XGBoost works with ANY twice-differentiable loss function - you just need to provide gradient and hessian.

For a leaf j with sample set Iⱼ, the optimal weight is:

```
w*ⱼ = -Σᵢ∈Iⱼ gᵢ / (Σᵢ∈Iⱼ hᵢ + λ)
```

And the gain from splitting is:

```
Gain = ½[GL²/(HL+λ) + GR²/(HR+λ) - (GL+GR)²/(HL+HR+λ)] - γ
```

Where GL, HL are gradient/hessian sums for left child, GR, HR for right. This is computed for every possible split point.

The γ term is min_split_loss - if Gain < 0, don't split. This is how gamma does pruning.

## Tree methods (this confused me for a while)

XGBoost has multiple tree building algorithms:

- **exact**: Scans every data point. Accurate but slow. Good for small data.
- **hist**: Bins features into 256 buckets, builds histograms. This is the fast one.
- **approx**: Re-bins before each tree using hessian weights. Middle ground.
- **gpu_hist**: Histogram method on GPU. 10-50x speedup on large data.

Use `hist` for most things. Use `gpu_hist` if you have a GPU and >1M rows.

The histogram approach is why LightGBM was such a big deal - same idea, bins continuous features so you iterate through bins instead of raw data. O(n_bins) instead of O(n log n).

### How histogram actually works

Instead of sorting features and scanning all values:

1. **Binning**: Map continuous features to discrete bins (default 256). This happens once before training via quantile sketch.

2. **Histogram construction**: For each node, accumulate gradient/hessian sums per bin. This is O(n) where n is samples in node.

3. **Split finding**: Iterate through bins (not data points) to find best split. O(max_bin) per feature.

4. **Histogram subtraction trick**: For sibling nodes, compute one histogram and derive the other by subtraction from parent. Cuts histogram construction in half.

```python
# Pseudocode for histogram split finding
for feature in features:
    hist = build_histogram(feature, gradients, hessians)
    for bin_idx in range(max_bin):
        GL, HL = hist[:bin_idx].sum()
        GR, HR = hist[bin_idx:].sum()
        gain = compute_gain(GL, HL, GR, HR, lambda_, gamma)
        if gain > best_gain:
            best_split = (feature, bin_idx)
```

The max_bin parameter controls memory/accuracy tradeoff. 256 is usually fine. LightGBM uses 255 by default (one less bit, slightly faster).

### Cache-aware optimizations

XGBoost does some nice systems stuff:

- **Block structure**: Data stored in compressed sparse column (CSC) format in memory blocks
- **Cache prefetching**: For exact method, prefetches next data into CPU cache while processing current
- **Out-of-core**: Can handle datasets larger than RAM by loading blocks from disk

The `max_cached_hist_node` parameter limits histogram cache size. Cache grows exponentially with tree depth, so deep trees can blow memory.

## Sparsity-aware split finding

Real data is sparse. One-hot encoded categoricals, missing values, zeros - lots of sparsity.

XGBoost handles this elegantly:

1. **Default direction**: For each split, learn whether missing/sparse values should go left or right
2. **Only scan non-missing**: Skip zeros/missing in the split finding loop
3. **Store both directions**: Try sending missing left AND right, pick better gain

```python
# During training, for each split:
gain_missing_left = compute_gain(GL + G_missing, HL + H_missing, GR, HR)
gain_missing_right = compute_gain(GL, HL, GR + G_missing, HR + H_missing)
default_direction = LEFT if gain_missing_left > gain_missing_right else RIGHT
```

This is why XGBoost handles missing values automatically - it learns which branch is better. No imputation needed.

Important gotcha: This means training with missing values and inference with zeros can behave differently. Be consistent.

## The regularization math

The regularization term Ω(f) in XGBoost is:

```
Ω(f) = γT + ½λΣⱼwⱼ²
```

Where:
- T = number of leaves
- wⱼ = weight of leaf j
- γ = min_split_loss (complexity penalty per leaf)
- λ = L2 regularization on weights

There's also α for L1:

```
Ω(f) = γT + ½λΣⱼwⱼ² + αΣⱼ|wⱼ|
```

How they interact:
- **λ (reg_lambda)**: Shrinks leaf weights, reduces gain values. Higher λ = more conservative splits.
- **α (reg_alpha)**: Promotes sparsity in leaf weights. Pushes small weights to zero.
- **γ (gamma)**: Explicit pruning threshold. Split only if Gain > γ.

The combination: λ reduces gain across the board, γ sets cutoff. Together they're quite powerful.

## Regularization that actually matters

From looking at Kaggle winners (2024-2025), the regularization params that matter:

```python
params = {
    'max_depth': 3-5,        # shallow trees
    'reg_alpha': 10-150,     # L1 on leaf weights
    'reg_lambda': 10-150,    # L2 on leaf weights
    'gamma': 25-100,         # min split loss (pruning)
    'subsample': 0.5,        # row sampling
    'colsample_bytree': 0.5  # column sampling
}
```

The stochastic stuff (subsample, colsample) adds randomness like random forests. gamma sets the pruning threshold, lambda/alpha reduce the gain values. They work together.

### Stochastic regularization breakdown

Multiple levels of randomness:

- **subsample**: Row sampling per tree. 0.8 means 80% of rows per tree.
- **colsample_bytree**: Column sampling per tree. Computed once when tree starts.
- **colsample_bylevel**: Column sampling per depth level. Re-samples at each level.
- **colsample_bynode**: Column sampling per split. Most granular, re-samples at each node.

These multiply: if colsample_bytree=0.5 and colsample_bylevel=0.5, you get 25% of features at each level.

My take: subsample + colsample_bytree is usually enough. The bylevel/bynode stuff adds complexity for marginal gains.

## DMatrix and data loading

DMatrix is XGBoost's internal data structure. Worth understanding because it affects memory and speed.

```python
import xgboost as xgb

# Basic loading
dtrain = xgb.DMatrix(X_train, label=y_train)

# With feature names (useful for SHAP later)
dtrain = xgb.DMatrix(X_train, label=y_train, feature_names=feature_cols)

# Enable categorical support (newer feature, pretty nice)
dtrain = xgb.DMatrix(X_train, label=y_train, enable_categorical=True)

# From pandas with automatic type inference
dtrain = xgb.DMatrix(df[feature_cols], label=df['target'])
```

For large datasets, use QuantileDMatrix - it computes quantiles for histogram binning during construction:

```python
# More memory efficient for hist/gpu_hist
dtrain = xgb.QuantileDMatrix(X_train, label=y_train, max_bin=256)
```

### Data iterators for huge datasets

When data doesn't fit in memory:

```python
class MyIterator(xgb.DataIter):
    def __init__(self, file_paths):
        self.file_paths = file_paths
        self.idx = 0
        super().__init__()

    def next(self, input_data):
        if self.idx >= len(self.file_paths):
            return 0  # no more data

        # Load one chunk
        df = pd.read_parquet(self.file_paths[self.idx])
        input_data(data=df[features], label=df['target'])
        self.idx += 1
        return 1  # has more data

    def reset(self):
        self.idx = 0

# Use with QuantileDMatrix
it = MyIterator(file_paths)
dtrain = xgb.QuantileDMatrix(it)
```

This streams data in chunks. Good for datasets >> RAM.

## Early stopping done right

Early stopping is critical but often misused.

```python
# Basic early stopping
model = xgb.train(
    params,
    dtrain,
    num_boost_round=1000,
    evals=[(dval, 'validation')],
    early_stopping_rounds=50
)

# Best iteration stored here
print(model.best_iteration)
print(model.best_score)
```

Common mistakes:

1. **Using training set for early stopping**: Overfit city. Always use held-out validation.

2. **early_stopping_rounds too small**: 10 rounds is way too aggressive. Use 50-100.

3. **Forgetting to use best_iteration at inference**:
```python
# Wrong - uses all trees
preds = model.predict(dtest)

# Right - uses trees up to best iteration
preds = model.predict(dtest, iteration_range=(0, model.best_iteration + 1))
```

4. **Not considering metric direction**: Some metrics are higher-better (AUC), some lower-better (logloss). XGBoost usually figures it out but check.

### Cross-validation with early stopping

```python
cv_results = xgb.cv(
    params,
    dtrain,
    num_boost_round=1000,
    nfold=5,
    early_stopping_rounds=50,
    metrics=['auc', 'logloss'],
    seed=42
)

# cv_results is a DataFrame with train/test metrics per round
optimal_rounds = cv_results['test-auc-mean'].idxmax()
```

This is the proper way to find optimal boosting rounds.

## Custom objectives

Since XGBoost only needs gradient and hessian, writing custom losses is straightforward:

```python
def custom_loss(y_pred, dtrain):
    y_true = dtrain.get_label()

    # Your gradient: dl/d(y_pred)
    grad = your_gradient_function(y_true, y_pred)

    # Your hessian: d²l/d(y_pred)²
    hess = your_hessian_function(y_true, y_pred)

    return grad, hess

# Use it
params = {'disable_default_eval_metric': True}
model = xgb.train(params, dtrain, obj=custom_loss)
```

Example - focal loss for imbalanced classification:

```python
def focal_loss(y_pred, dtrain, gamma=2.0):
    y_true = dtrain.get_label()
    p = 1 / (1 + np.exp(-y_pred))  # sigmoid

    # Focal loss gradient
    grad = p - y_true + gamma * (
        y_true * p * (1 - p) * np.log(p + 1e-8) -
        (1 - y_true) * p * (1 - p) * np.log(1 - p + 1e-8)
    )

    # Focal loss hessian (approximation)
    hess = p * (1 - p)
    hess = np.maximum(hess, 1e-6)  # numerical stability

    return grad, hess
```

Custom eval metrics work similarly:

```python
def custom_eval(y_pred, dtrain):
    y_true = dtrain.get_label()
    score = your_metric(y_true, y_pred)
    return 'custom_metric', score

model = xgb.train(params, dtrain, evals=[(dval, 'val')], feval=custom_eval)
```

## Monotonic constraints

Sometimes you know a feature should only increase (or decrease) the prediction. Income should increase loan approval probability, age should decrease term life insurance risk, etc.

```python
params = {
    'monotone_constraints': '(1,-1,0,0,1)'  # 5 features
    # 1 = increasing, -1 = decreasing, 0 = no constraint
}

# Or with feature names (cleaner)
params = {
    'monotone_constraints': {
        'income': 1,
        'age': -1,
        'credit_score': 1
    }
}
```

This is enforced during tree building - splits that violate monotonicity are rejected.

Super useful for:
- Regulatory compliance (explain why higher income = higher approval)
- Sanity checking (catch weird model behavior)
- Incorporating domain knowledge

## Interaction constraints

Control which features can interact in the tree:

```python
# Features 0,1,2 can interact with each other
# Features 3,4 can interact with each other
# But 0,1,2 cannot interact with 3,4
params = {
    'interaction_constraints': '[[0,1,2],[3,4]]'
}
```

Use case: You have user features and item features in a recommendation model. Maybe you want to prevent user features from interacting directly.

Honestly I rarely use this. The interpretability vs performance tradeoff usually isn't worth it.

## Categorical feature support

Native categorical handling is relatively new (XGBoost 1.6+). Before this everyone one-hot encoded.

```python
# Mark categorical columns
dtrain = xgb.DMatrix(X, label=y, enable_categorical=True)

# Or with pandas categorical dtype
df['category_col'] = df['category_col'].astype('category')
dtrain = xgb.DMatrix(df[features], label=df['target'], enable_categorical=True)

params = {
    'tree_method': 'hist',  # required for native categoricals
    'max_cat_to_onehot': 4  # one-hot if <= 4 categories, partition otherwise
}
```

For high cardinality categoricals (>4 categories by default), XGBoost uses optimal partitioning instead of one-hot. This finds the best subset of categories for each split.

CatBoost still does categoricals better (ordered target encoding), but native XGBoost categoricals are decent and simpler than manual encoding.

## Learning to rank

XGBoost has built-in ranking objectives. Useful for search, recommendations, etc.

```python
params = {
    'objective': 'rank:ndcg',  # or 'rank:map', 'rank:pairwise'
    'eval_metric': 'ndcg@10',
    'lambdarank_num_pair_per_sample': 10
}

# Group data by query - critical!
dtrain = xgb.DMatrix(X, label=relevance_labels)
dtrain.set_group(group_sizes)  # [5, 10, 8] = 3 queries with 5, 10, 8 docs
```

The group parameter tells XGBoost which samples belong to the same query. Ranking loss is computed within groups.

Objectives:
- **rank:pairwise**: RankNet-style pairwise loss
- **rank:ndcg**: LambdaMART optimizing NDCG (most common)
- **rank:map**: LambdaMART optimizing MAP (binary relevance only)

Tip: Start with rank:ndcg, it's the most robust.

## Comparison with Random Forest

Since both are tree ensembles, worth understanding the tradeoffs:

**Random Forest:**
- Trees built independently (parallelizable)
- Each tree sees bootstrap sample
- Final prediction: average of trees
- Lower variance, can't overfit as easily
- Generally more stable with less tuning

**XGBoost:**
- Trees built sequentially (each corrects previous)
- Each tree fits residuals/gradients
- Final prediction: sum of trees
- Can achieve lower bias with proper tuning
- More prone to overfitting, needs regularization

When to use RF over XGBoost:
- Very noisy data where XGBoost overfits
- Limited tuning time (RF defaults are good)
- Need uncertainty estimates (RF variance across trees)
- Embarrassingly parallel training needed

In practice, XGBoost usually wins on accuracy with proper tuning. But RF is a good sanity check baseline.

## Links & References

### Core Documentation
- [XGBoost Official Docs](https://xgboost.readthedocs.io/)
- [XGBoost Parameters Reference](https://xgboost.readthedocs.io/en/stable/parameter.html)
- [Tree Methods Explained](https://xgboost.readthedocs.io/en/stable/treemethod.html)
- [XGBoost GitHub](https://github.com/dmlc/xgboost)

### Papers
- [XGBoost: A Scalable Tree Boosting System (Chen & Guestrin, KDD 2016)](https://www.kdd.org/kdd2016/papers/files/rfp0697-chenAemb.pdf) - The original paper
- [Tabular Data: Deep Learning is Not All You Need (arXiv)](https://arxiv.org/abs/2106.03253) - XGBoost vs deep learning benchmark
- [Why do tree-based models still outperform deep learning on tabular data? (NeurIPS 2022)](https://arxiv.org/abs/2207.08815) - Explains why trees win on irregular patterns
- [Tabular Models Benchmark 2026](https://research.aimultiple.com/tabular-models/) - Recent benchmark comparisons

### Histogram Algorithm
- [XGBoost's Fast Histogram Method (Laurae)](https://medium.com/data-design/xgboosts-new-fast-histogram-tree-method-hist-a3c08f36234c)
- [Histograms for Efficient Gradient Boosting (Juliano Garcia)](https://robotenique.github.io/posts/gbm-histogram/)
- [Configure XGBoost Histogram Tree Method](https://xgboosting.com/configure-xgboost-histogram-tree-method-tree_methodhist/)
- [Fast Histogram GitHub Issue #1950](https://github.com/dmlc/xgboost/issues/1950) - Original implementation discussion

### Missing Values & Sparsity
- [XGBoost, Missing Values, and Sparsity (arfer.net)](https://arfer.net/w/xgboost-sparsity) - Deep dive into sparsity handling
- [How XGBoost Handles Sparsities (Medium)](https://medium.com/hypatai/how-xgboost-handles-sparsities-arising-from-of-missing-data-with-an-example-90ce8e4ba9ca)
- [Handling Missing Data in XGBoost](https://www.aryanupadhyay.com/post/handling-missing-data-in-xgboost)

### Custom Objectives
- [Custom Objective and Evaluation Metric (Official Tutorial)](https://xgboost.readthedocs.io/en/stable/tutorials/custom_metric_obj.html)
- [Advanced Usage of Custom Objectives](https://xgboost.readthedocs.io/en/latest/tutorials/advanced_custom_obj.html)
- [Building a Tunable Custom Objective (AppsFlyer)](https://medium.com/appsflyerengineering/building-a-tunable-and-configurable-custom-objective-function-for-xgboost-d3ced8967809)

### Monotonic Constraints
- [Monotonic Constraints Tutorial (Official)](https://xgboost.readthedocs.io/en/stable/tutorials/monotonic.html)
- [Understanding Monotonic Constraints in XGBoost](https://medium.com/@ompanda/understanding-monotonic-constraints-in-xgboost-models-for-improved-interpretability-212e3723cf30)
- [Monotonic Constraints: Missteps, Pitfalls, and Remedies](https://medium.com/@xwang222/monotonic-constraints-in-ml-missteps-pitfalls-and-remedies-53182d87cea8)
- [LDA-XGB Framework for Fair Lending (arXiv)](https://arxiv.org/html/2410.19067v1) - Regulatory compliance applications

### Learning to Rank
- [Learning to Rank Tutorial (Official)](https://xgboost.readthedocs.io/en/stable/tutorials/learning_to_rank.html)
- [Learning to Rank with XGBoost and GPU (NVIDIA)](https://developer.nvidia.com/blog/learning-to-rank-with-xgboost-and-gpu/)
- [LambdaMART Explained (Shaped Blog)](https://www.shaped.ai/blog/lambdamart-explained-the-workhorse-of-learning-to-rank)
- [From RankNet to LambdaMART (OLX Engineering)](https://tech.olx.com/from-ranknet-to-lambdamart-leveraging-xgboost-for-enhanced-ranking-models-cf21f33350fb)
- [XGBoost Learning to Rank in Python (Forecastegy)](https://forecastegy.com/posts/xgboost-learning-to-rank-python/)

---

Continue to [Part 2: Practical Usage & Production](/posts/xgboost-notes-part2) for tuning, GPU acceleration, production deployment, and real-world patterns.
