# About This Guide
This guide collects opinionated, practical rules for writing Data Science code in Python. It derives of years of practise and falling into pitfalls, creating unreadable, unperformant and complex code where simpler techniques would be more appropriate. It focuses specifically on Pandas, NumPy, and Scikit-learn and the patterns, pitfalls, and conventions that general-purpose style guides do not address.

## What it covers: 

vectorization, DataFrame safety, sorting, hyperparameter management, reproducibility, data leakage prevention, null value handling, and model evaluation.

## What it does not cover: 

general Python style (see PEP8), code structure and documentation conventions (see the Google Python Style Guide), or framework-specific deep learning patterns (PyTorch, TensorFlow, etc.).
This guide is not a checklist to enforce in code review. It is a reference and a set of defaults that reflect current best practices and the reasoning behind them. Every rule can be broken when the situation calls for it. Understanding why a rule exists is more valuable than following it blindly.

