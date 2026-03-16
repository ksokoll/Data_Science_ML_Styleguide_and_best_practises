# Data Science / ML Style guides and best practises

This repository contains reference guides for writing production-grade Data Science and ML code in Python.

## 1. Data Science Style Guide

An opinionated, practical guide for writing Data Science code in Python, focusing specifically on Pandas, NumPy, and Scikit-learn. It derives from years of practice and the pitfalls that general-purpose style guides do not address: vectorization, DataFrame safety, sorting, hyperparameter management, reproducibility, data leakage prevention, null value handling, and model evaluation.

Not a checklist for code review but a reference for understanding why certain patterns exist and what goes wrong when you ignore them.

## 2. Software Engineering at Google: ML/AI Insights

A chapter-by-chapter extraction of the most relevant ideas from *Software Engineering at Google* (Winters, Manshreck, Wright, 2020), filtered for E2E AI/ML engineering: Data Engineering, Data Science, ML Engineering, Solution Architecture, CI/CD, Serving, and Dependency Management.

Where the style guide focuses on how to write individual pieces of ML code correctly, this guide focuses on how to maintain, test, and operate ML systems over time and at scale like Hyrum's Law applied to feature pipelines, the test pyramid applied to training infrastructure, trunk-based development applied to model releases.
