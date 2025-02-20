---
title: "Modelling in 2024"
date: "2024-12-20"
summary: "some modelling tips in 2024"
description: "some modelling tips in 2024"
toc: false
readTime: true
---

Reminder of an old modelling tricks that never die, even in 2025:

- Linear regs (l1/l2/elasticnet) as a starter. Focus on feature engineering here.
- Quantile Regression is also cool
- LightGBM for pretty much everything as it’s fast and accurate.
- CatBoost if you have many categorical features.
- Random Forest if your OOS variance is too high from LightGBM.
- XGBoost when you’re just curious.
- You can slap SHAP ontop of these but realistically it won't help too much.
- Ensemble multiple models and average the preds using postprocessing techniques like weighted avg, median, etc, as well as run feature neutralisation.
