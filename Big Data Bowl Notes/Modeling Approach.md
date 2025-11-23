# 1) Paper-ready argument (concise, defendable)

**Context & goal.**  
We do **not** intend to predict the binary completion label perfectly on every frame. Our goal is to produce a probabilistic signal over time (per-frame completion probability while the ball is in flight) that is: (a) interpretable, (b) stable (not wildly oscillatory), (c) reasonably calibrated, and (d) discriminative in aggregate (e.g., distinguishes plays/receivers who raise probability vs not). Small changes in model logloss or AUC are less relevant than the _behavior_ of the probability trace.

**Why manual conservative tuning can be justified.**  
Classical automated hyperparameter optimization (grid/cross-val minimizing logloss) optimizes for _pointwise reduction of logloss_ on held-out binary outcomes. That frequently pushes models to become overconfident (very near 0/1) to minimize logloss when the dataset has strong label signal or imbalanced labels. For our use-case this can be counterproductive: we want meaningful probability _gradients_ and reasonable uncertainty, not extreme 0/1 outputs that mask temporal nuance. Manually choosing conservative hyperparameters (shallow trees, small learning rate, subsampling, regularization) is a principled way to enforce parsimony and smoothing of the predicted probability surface.

**How to make this rigorous and acceptable scientifically.**

1. State the objective explicitly (probability trace utility, not per-frame binary accuracy).
    
2. Use conservative hyperparameters chosen a priori based on domain knowledge and signal/noise characteristics (document choices and motivation).
    
3. Validate with _task-aligned diagnostics_ (calibration plots, Brier score, reliability diagrams, temporal smoothness metrics, aggregated play-level statistics like your AUC-sum) rather than only binary classification metrics.
    
4. Report sensitivity: show what happens if you relax/strengthen regularization (e.g., three model settings: conservative / moderate / aggressive) and report diagnostics.
    
5. Discuss limitations: manual tuning trades off some statistical optimality for interpretability/stability.
    

**Takeaway sentence suitable for a methods section:**

> “Because our target is a time-varying probabilistic score (the completion probability trace) rather than per-frame binary classification accuracy, we deliberately favor a conservative XGBoost configuration (shallow trees, low learning rate, and regularization) to produce stable, well-behaved probability trajectories. We validate these models using calibration and task-level aggregate metrics rather than relying solely on pointwise classification metrics.”


