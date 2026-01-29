---
title: "XGBoost Notes - Part 2: Practical Usage"
date: "2026-01-20"
summary: "xgboost in practice - tuning, gpu, production, debugging"
description: "xgboost in practice - tuning, gpu, production, debugging"
toc: false
readTime: true
---

This is part 2 covering practical usage. [Part 1](/posts/xgboost-notes-part1) covers the internals and theory.

## Hyperparameter tuning

Grid search is a waste of time with 10+ hyperparameters. Use Optuna with TPE (Tree-structured Parzen Estimator). Converges in 35-100 trials vs 500+ for grid search.

```python
import optuna
from sklearn.model_selection import cross_val_score

def objective(trial):
    params = {
        'max_depth': trial.suggest_int('max_depth', 3, 10),
        'learning_rate': trial.suggest_float('learning_rate', 0.01, 0.3, log=True),
        'n_estimators': trial.suggest_int('n_estimators', 100, 1000),
        'min_child_weight': trial.suggest_int('min_child_weight', 1, 10),
        'subsample': trial.suggest_float('subsample', 0.5, 1.0),
        'colsample_bytree': trial.suggest_float('colsample_bytree', 0.5, 1.0),
        'reg_alpha': trial.suggest_float('reg_alpha', 1e-8, 100, log=True),
        'reg_lambda': trial.suggest_float('reg_lambda', 1e-8, 100, log=True),
        'gamma': trial.suggest_float('gamma', 1e-8, 100, log=True),
    }

    model = xgb.XGBClassifier(**params, early_stopping_rounds=50)

    # Use cross-validation
    scores = cross_val_score(
        model, X, y, cv=5, scoring='roc_auc',
        fit_params={'eval_set': [(X_val, y_val)]}
    )
    return scores.mean()

study = optuna.create_study(direction='maximize')
study.optimize(objective, n_trials=100)

print(study.best_params)
```

Use log scales for learning_rate, alpha, lambda - they span orders of magnitude.

### Tuning tips

1. **Start with defaults**: Get a baseline before tuning
2. **Fix n_estimators high, tune with early stopping**: Don't tune n_estimators directly
3. **Tune regularization last**: max_depth, learning_rate, subsample first
4. **Check if you're hitting bounds**: If best trials cluster at parameter limits, expand search space
5. **Use pruning**: Optuna can prune bad trials early

```python
# Pruning integration
def objective(trial):
    # ... params setup ...

    pruning_callback = optuna.integration.XGBoostPruningCallback(
        trial, 'validation-auc'
    )

    model = xgb.train(
        params, dtrain,
        evals=[(dval, 'validation')],
        callbacks=[pruning_callback]
    )

    return model.best_score
```

## GPU acceleration

Modern GPUs crush XGBoost training. Thousands of cores building histograms in parallel.

```python
params = {
    'tree_method': 'hist',
    'device': 'cuda',  # or 'cuda:0' for specific GPU
    'max_bin': 256
}

# That's it. Training now uses GPU.
model = xgb.train(params, dtrain)
```

For prediction (inference is also GPU-accelerated):

```python
# Model trained on GPU predicts on GPU by default
preds = model.predict(dtest)

# Force CPU prediction if needed
model.set_param({'device': 'cpu'})
preds = model.predict(dtest)
```

### GPU memory management

GPU memory is usually the bottleneck, not compute.

```python
# Reduce max_bin to save memory
params = {'max_bin': 128}  # default 256

# Use QuantileDMatrix for better memory efficiency
dtrain = xgb.QuantileDMatrix(X, label=y, max_bin=256)

# Monitor GPU memory
# nvidia-smi in another terminal
```

For data larger than GPU memory, use data iterators or external memory mode.

### Multi-GPU training

With Dask:

```python
from dask_cuda import LocalCUDACluster
from dask.distributed import Client
import xgboost as xgb

cluster = LocalCUDACluster()
client = Client(cluster)

# Dask arrays/dataframes
import dask.array as da
X_dask = da.from_array(X, chunks=(10000, -1))
y_dask = da.from_array(y, chunks=10000)

dtrain = xgb.dask.DaskDMatrix(client, X_dask, y_dask)

output = xgb.dask.train(
    client, params, dtrain,
    num_boost_round=100,
    evals=[(dtrain, 'train')]
)
model = output['booster']
```

Or with Ray:

```python
from xgboost_ray import RayDMatrix, train

train_set = RayDMatrix(X, y)
result = train(
    params, train_set,
    num_boost_round=100,
    num_actors=4  # distribute across 4 workers
)
```

## XGBoost vs LightGBM vs CatBoost

Quick take:
- **XGBoost**: Default choice. Best docs, most mature.
- **LightGBM**: Faster training, leaf-wise growth. Use for >10M rows.
- **CatBoost**: Native categorical support. Less tuning needed.

In practice I use LightGBM for most things because it's faster and has a nicer api. XGBoost when I need the ecosystem or specific features.

### Technical differences

**Tree growth strategy:**
- XGBoost: Level-wise (breadth-first). Grows all leaves at same depth before going deeper.
- LightGBM: Leaf-wise (best-first). Grows leaf with highest gain. Can create unbalanced trees.
- CatBoost: Oblivious trees. Same split condition at each level.

Leaf-wise is usually faster and can be more accurate, but risks overfitting on small data.

**Categorical handling:**
- XGBoost: Optimal partitioning (recent), one-hot
- LightGBM: Optimal partitioning
- CatBoost: Ordered target encoding (best for high cardinality)

**Missing values:**
- All three learn optimal direction
- CatBoost treats missing as a separate category

**Default regularization:**
- CatBoost has more aggressive defaults, often works well out-of-box
- XGBoost/LightGBM need more tuning

## SHAP notes

Built-in feature importance (gain/cover/frequency) is biased toward high cardinality features. SHAP gives you proper Shapley values with theoretical guarantees.

```python
import shap

# TreeExplainer is fast for tree models
explainer = shap.TreeExplainer(model)
shap_values = explainer.shap_values(X_test)

# Global importance
shap.summary_plot(shap_values, X_test, plot_type='bar')

# Beeswarm (shows distribution)
shap.summary_plot(shap_values, X_test)

# Single prediction explanation
shap.waterfall_plot(shap.Explanation(
    values=shap_values[0],
    base_values=explainer.expected_value,
    data=X_test.iloc[0]
))

# Dependence plot (feature vs SHAP value)
shap.dependence_plot('feature_name', shap_values, X_test)
```

But honestly, feature importance reflects prediction contribution, not causation. Don't overinterpret it.

### SHAP interaction values

For understanding feature interactions:

```python
shap_interaction = explainer.shap_interaction_values(X_test)
# Shape: (n_samples, n_features, n_features)

# Plot interaction between two features
shap.dependence_plot(
    ('feature1', 'feature2'),
    shap_interaction, X_test
)
```

Expensive to compute but useful for understanding non-additive effects.

## Production notes

- Use native format (`model.save_model('model.json')`) not pickle - version compatibility is better
- Sparse vs dense vectors matter - SparseVectors treat zeros as missing, DenseVectors treat them as zeros. Inconsistency between training/serving will silently break things.
- Ray Serve handles scaling properly if you need it. Flask doesn't scale.

### Model serialization

```python
# JSON format (human readable, larger)
model.save_model('model.json')

# Binary format (smaller, faster)
model.save_model('model.ubj')  # Universal Binary JSON

# Load
model = xgb.Booster()
model.load_model('model.json')

# Sklearn wrapper
import joblib
joblib.dump(xgb_sklearn_model, 'model.joblib')
```

Avoid pickle for XGBoost models. Version compatibility is a nightmare.

### Feature consistency

The silent killer in production. Training and serving must have:
- Same feature order
- Same feature names
- Same handling of missing/zeros
- Same preprocessing

```python
# Save feature names with model
model.feature_names = feature_list
model.save_model('model.json')

# At inference, validate
assert list(X_test.columns) == model.feature_names
```

Better yet, use a feature store or package preprocessing with the model.

### Serving options

**Simple (low traffic):**
```python
from fastapi import FastAPI
import xgboost as xgb

app = FastAPI()
model = xgb.Booster()
model.load_model('model.json')

@app.post('/predict')
def predict(features: dict):
    dmatrix = xgb.DMatrix([list(features.values())])
    return {'prediction': float(model.predict(dmatrix)[0])}
```

**Production (high traffic):**
- Ray Serve for auto-scaling
- Triton Inference Server for GPU serving
- ONNX export for cross-platform deployment

```python
# ONNX export
from onnxmltools import convert_xgboost
from skl2onnx.common.data_types import FloatTensorType

initial_type = [('input', FloatTensorType([None, n_features]))]
onnx_model = convert_xgboost(model, initial_types=initial_type)

with open('model.onnx', 'wb') as f:
    f.write(onnx_model.SerializeToString())
```

## Common pitfalls

1. **Data leakage in time series**: Don't use future data. Use time-based splits, not random.

2. **Overfitting with early stopping on training set**: Always use separate validation set.

3. **Ignoring class imbalance**: Use scale_pos_weight or custom loss.

```python
# For binary classification with 10:1 imbalance
params = {'scale_pos_weight': 10}
```

4. **Feature scaling**: XGBoost doesn't need it (trees split on thresholds), but your preprocessing pipeline might.

5. **Sparse/dense confusion**: Train and predict with same format.

6. **Forgetting to tune**: Defaults are decent but rarely optimal. At minimum tune max_depth and learning_rate.

7. **Too many trees with high learning rate**: Either many trees + low LR, or few trees + high LR. Not both.

## When to use deep learning instead

- Multi-modal data (images + tabular, text + tabular)
- When you have time to tune architectures and want to write a paper

For pure tabular, start with XGBoost. Add deep learning to ensemble if you have time. Kaggle winners do 3-layer stacking: multiple XGBoost configs -> XGBoost + NN + others -> weighted average.

### The ensemble recipe

What actually wins competitions:

```
Layer 1: Diverse base models
- XGBoost with different hyperparameters (5-10 variants)
- LightGBM variants
- CatBoost
- Neural net (TabNet, simple MLP)
- Random Forest

Layer 2: Meta-learner
- Another XGBoost or linear model
- Trained on Layer 1 out-of-fold predictions

Layer 3 (optional): Simple average
- Weighted mean of Layer 2 outputs
```

For production, this is usually overkill. Single well-tuned XGBoost is 95% of the way there.

## The repository (if you're curious)

The XGBoost GitHub repo is actually well-organized. ~27k stars, 640+ contributors.

Language breakdown:
- C++ (43%): Core algorithm
- Python (21%): Main API
- CUDA (18%): GPU kernels
- R, Java/Scala (~15%): Language bindings

Key directories:
- `src/`: Core C++ implementation, tree builders, objectives
- `python-package/`: Sklearn-compatible API, DMatrix, visualization
- `jvm-packages/`: Spark integration
- `plugin/`: Extensible architecture

Building from source (if you need bleeding edge):

```bash
git clone --recursive https://github.com/dmlc/xgboost
cd xgboost
cmake -B build -S . -DCMAKE_BUILD_TYPE=RelWithDebInfo -GNinja
cd build && ninja
```

The `--recursive` is important - pulls in submodules for dependencies.

## State of XGBoost (2025-2026)

Recent developments worth noting:

- **Native categorical support** is now stable (1.6+)
- **GPU training** is mature and basically just works
- **Federated learning** support for privacy-preserving training
- **Better Spark/Dask integration** for distributed training
- **Quantile DMatrix** for memory-efficient large datasets

The ecosystem is stable. Not much revolutionary happening, which is fine - it's a mature tool that works.

What's still annoying:
- Version compatibility between pickle serialized models
- Categorical handling still not as good as CatBoost
- Documentation for advanced features (custom objectives, LTR) could be better

## Real-world patterns

Some patterns I've seen work in production:

### Credit risk / fraud detection

Classic XGBoost territory. High cardinality categoricals (merchant IDs, device fingerprints), lots of missing data, need for interpretability.

What works:
- Shallow trees (depth 3-5) with strong regularization
- Custom loss with asymmetric costs (false negatives expensive)
- Monotonic constraints on features like income, credit score
- SHAP for model cards and regulatory compliance

### Time series features for tabular models

XGBoost doesn't do time series natively, but you can engineer features:

```python
# Lag features
df['value_lag_1'] = df.groupby('entity')['value'].shift(1)
df['value_lag_7'] = df.groupby('entity')['value'].shift(7)

# Rolling stats
df['value_rolling_mean_7'] = df.groupby('entity')['value'].transform(
    lambda x: x.rolling(7).mean()
)

# Time-based features
df['day_of_week'] = df['timestamp'].dt.dayofweek
df['month'] = df['timestamp'].dt.month
df['is_weekend'] = df['day_of_week'].isin([5, 6])
```

Important: Use time-based train/val/test splits, not random. Leakage will kill you.

### Recommendations / ranking

XGBoost's LTR objectives work well for:
- Search ranking
- Product recommendations
- Ad relevance

The key is proper group handling - samples within a query compete against each other, not across queries.

### Ensemble strategies that actually work

In competitions:

```python
# Simple but effective
final_pred = 0.4 * xgb_pred + 0.3 * lgb_pred + 0.3 * catboost_pred

# Slightly fancier - train a meta-model
from sklearn.linear_model import Ridge

meta_features = np.column_stack([xgb_oof, lgb_oof, catboost_oof])
meta_model = Ridge().fit(meta_features, y_train)
final_pred = meta_model.predict(np.column_stack([xgb_test, lgb_test, catboost_test]))
```

In production, single model is usually fine. Ensembling adds complexity for marginal gains.

## Debugging XGBoost models

When things go wrong:

### Model not learning
- Check if labels are correct (common mistake: wrong column)
- Learning rate too low + not enough trees
- All features have zero importance (data issue)

### Overfitting
- Train AUC >> validation AUC
- Add regularization: increase lambda, gamma, decrease max_depth
- Add stochasticity: subsample, colsample

### Underfitting
- Train AUC is low
- Increase max_depth, decrease regularization
- Check if features are informative at all

### Debugging feature importance

```python
# Quick sanity check
importance = model.get_score(importance_type='gain')
sorted_importance = sorted(importance.items(), key=lambda x: x[1], reverse=True)

# Top features
for feat, score in sorted_importance[:10]:
    print(f"{feat}: {score:.4f}")

# If one feature dominates, it might be leakage
# If all features are similar, model might not be learning much
```

### Prediction debugging

```python
# Single prediction explanation
import shap
explainer = shap.TreeExplainer(model)
shap_values = explainer.shap_values(X_test.iloc[[0]])

# See which features pushed prediction up/down
for feat, val in zip(X_test.columns, shap_values[0]):
    if abs(val) > 0.01:  # only significant contributions
        print(f"{feat}: {val:+.4f}")
```

## Links & References

### Core Documentation
- [XGBoost Official Docs](https://xgboost.readthedocs.io/)
- [XGBoost Parameters Reference](https://xgboost.readthedocs.io/en/stable/parameter.html)
- [XGBoost GitHub](https://github.com/dmlc/xgboost)
- [XGBoost FAQ](https://xgboost.readthedocs.io/en/latest/faq.html)

### Papers
- [XGBoost: A Scalable Tree Boosting System (Chen & Guestrin, KDD 2016)](https://www.kdd.org/kdd2016/papers/files/rfp0697-chenAemb.pdf)
- [LightGBM Paper (NeurIPS)](https://papers.nips.cc/paper/6907-lightgbm-a-highly-efficient-gradient-boosting-decision-tree)
- [CatBoost Paper (arXiv)](https://arxiv.org/abs/1706.09516)
- [Tabular Data: Deep Learning is Not All You Need (arXiv)](https://arxiv.org/abs/2106.03253)
- [Why Tree-Based Models Outperform Deep Learning (NeurIPS 2022)](https://arxiv.org/abs/2207.08815)
- [Tabular Models Benchmark 2026](https://research.aimultiple.com/tabular-models/)
- [Comprehensive Benchmark of ML and DL on Structured Data (ScienceDirect 2025)](https://www.sciencedirect.com/science/article/pii/S0925231225020090)

### Hyperparameter Tuning
- [Optuna - Hyperparameter Optimization Framework](https://optuna.org/)
- [Ultimate Guide to XGBoost Parameter Tuning (Random Realizations)](https://randomrealizations.com/posts/xgboost-parameter-tuning-with-optuna/) - Excellent in-depth guide
- [Bayesian Optimization of XGBoost with Optuna](https://xgboosting.com/bayesian-optimization-of-xgboost-hyperparameters-with-optuna/)
- [XGBoost Tuning with Optuna - Kaggle Grandmaster Guide (Forecastegy)](https://forecastegy.com/posts/xgboost-hyperparameter-tuning-with-optuna/)
- [Using Optuna to Optimize XGBoost (Official Optuna Blog)](https://medium.com/optuna/using-optuna-to-optimize-xgboost-hyperparameters-63bfcdfd3407)
- [GPU-Accelerated XGBoost with Bayesian Tuning](https://qfournier.github.io/blog/2025/xgboost/)

### GPU Acceleration
- [XGBoost GPU Support (Official)](https://xgboost.readthedocs.io/en/stable/gpu/index.html)
- [Make XGBoost 46x Faster with GPU (Medium)](https://medium.com/data-science-collective/stop-waiting-make-xgboost-46x-faster-with-one-parameter-change-78839bf732c0)
- [Accelerating XGBoost with GPU Step-by-Step](https://medium.com/zackary-yen/accelerating-xgboost-with-gpu-a-step-by-step-guide-3660cc3bb840)
- [RAPIDS Zero-Code-Change GPU Acceleration (NVIDIA)](https://developer.nvidia.com/blog/rapids-brings-zero-code-change-acceleration-io-performance-gains-and-out-of-core-xgboost/)
- [GPU-Accelerate Model Training with CUDA-X (NVIDIA)](https://developer.nvidia.com/blog/how-to-gpu-accelerate-model-training-with-cuda-x-data-science/)

### SHAP & Interpretability
- [SHAP Documentation](https://shap.readthedocs.io/)
- [Introduction to Explainable AI with Shapley Values (SHAP)](https://shap.readthedocs.io/en/latest/example_notebooks/overviews/An%20introduction%20to%20explainable%20AI%20with%20Shapley%20values.html)
- [Interpretable Machine Learning with XGBoost (Scott Lundberg)](https://medium.com/data-science/interpretable-machine-learning-with-xgboost-9ec80d148d27)
- [SHAP Values vs Feature Importance (Medium)](https://medium.com/biased-algorithms/shap-values-vs-feature-importance-ba6b91c16319)
- [XGBoost Feature Importance Computed 3 Ways (MLJAR)](https://mljar.com/blog/feature-importance-xgboost/)
- [SHAP Chapter - Interpretable ML Book (Christoph Molnar)](https://christophm.github.io/interpretable-ml-book/shap.html)
- [Gentle Introduction to SHAP for Tree-Based Models](https://machinelearningmastery.com/a-gentle-introduction-to-shap-for-tree-based-models/)

### Production Deployment
- [Deploying XGBoost with Ray Serve (Anyscale)](https://www.anyscale.com/blog/deploying-xgboost-models-with-ray-serve)
- [XGBoost Deployment Strategies (MoldStud)](https://moldstud.com/articles/p-xgboost-deployment-strategies-and-challenge-solutions)
- [XGBoost Deployment Best Practices (Restack)](https://www.restack.io/p/xgboost-deployment-answer-best-practices-cat-ai)
- [Build, Train, Deploy XGBoost on Cloud AI Platform (Google Codelabs)](https://codelabs.developers.google.com/codelabs/xgb-caip-e2e)
- [Deploy XGBoost in Pure Python (GitHub)](https://github.com/mwburke/xgboost-python-deploy)

### Comparisons & Benchmarks
- [XGBoost vs Deep Learning for Tabular Data (Analytics India Mag)](https://analyticsindiamag.com/deep-tech/deep-learning-xgboost-or-both-what-works-best-for-tabular-data/)
- [Why Tree-Based Models Beat Deep Learning (Medium)](https://medium.com/geekculture/why-tree-based-models-beat-deep-learning-on-tabular-data-fcad692b1456)
- [TabNet vs XGBoost Comparison (ResearchGate)](https://www.researchgate.net/publication/392509412_A_Comparative_Study_of_TabNet_and_XGBoost_for_Tabular_Data_Classification)
- [TabR vs XGBoost (IEEE)](https://ieeexplore.ieee.org/document/10802905/)

### Tutorials & Guides
- [XGBoost Python Guide 2025 (PyPI Tutorial)](https://generalistprogrammer.com/tutorials/xgboost-python-package-guide)
- [XGBoost Complete Guide (IGMGuru)](https://www.igmguru.com/blog/xgboost)
- [Optimizing ML with XGBoost Strategies](https://www.numberanalytics.com/blog/optimizing-ml-with-xgboost-strategies)
- [XGBoost Tutorial for Drug Development (PMC)](https://pmc.ncbi.nlm.nih.gov/articles/PMC11895769/)
