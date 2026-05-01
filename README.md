# BACE1 Binding Affinity Prediction with Scaffold-Based Evaluation

A QSAR modelling pipeline for predicting the binding affinity of small 
molecules against BACE1 (Beta-secretase 1), a key target in Alzheimer's 
disease research. Built using data from ChEMBL, molecular fingerprints 
via RDKit, and two tuned ML models: Random Forest and LightGBM.

The central idea this project investigates is how honestly we're 
evaluating  binding affinity prediction models. Most QSAR benchmarks use 
random train/test splits, which inflate performance by leaking structural 
information between training and test sets. This project implements a 
Bemis-Murcko scaffold split as a more rigorous alternative, by grouping 
molecules by their core ring systems and linkers, then ensuring the test set 
contains only scaffolds unseen during training. This better simulates the 
challenge of predicting activity for new chemical matter. Results are compared 
directly against a random split to quantify the difference.

In drug discovery, a model that performs well on random splits but poorly 
on scaffold splits has memorised chemical series rather than learned 
generalisable structure-activity relationships. 


## Results

**Random Forest**
- Random split: RMSE 0.716 | MAE 0.538 | R² 0.647
- Scaffold split: RMSE 0.800 | MAE 0.628 | R² 0.372

**LightGBM**
- Random split: RMSE 0.698 | MAE 0.510 | R² 0.664
- Scaffold split: RMSE 0.771 | MAE 0.600 | R² 0.417

The performance drop from random to scaffold split is consistent across 
both models. R² roughly halves and RMSE increases noticeably. This gap 
reflects the difficulty of generalising to new chemical scaffolds 
unseen during training.

The potency-stratified analysis shows the models perform best in the 
medium potency range (pIC50 6-8) where training data is most dense, and 
struggle more at the extremes.


## Key Findings

- Scaffold splits produce substantially lower R² than random splits,
  confirming that random split benchmarks overestimate real-world
  generalisation
- LightGBM consistently outperforms Random Forest across all metrics 
  and both split types
- Both models show a systematic underprediction bias on scaffold splits, 
  pulling predictions toward the training set mean when encountering 
  structurally novel compounds


## Pipeline

1. **Data Extraction** - ChEMBL API query for all BACE1 IC50 measurements
2. **Data Cleaning** — unit standardisation, missing value removal, 
   duplicate handling via median aggregation
3. **Molecular Featurisation** — Morgan fingerprints (radius=2, 2048 bits) 
   via RDKit
4. **Dataset Splitting** — Bemis-Murcko scaffold split vs random split (80/20)
5. **Model Training** — Random Forest and LightGBM with RandomizedSearchCV 
   hyperparameter tuning (5-fold CV)
6. **Evaluation and Analysis** — RMSE, MAE, R² across both splits; residual 
   distributions, feature importance, RMSE by potency range


## Dataset

- **Source:** ChEMBL (target: CHEMBL4822)
- **Measurement type:** IC50 (converted to pIC50)
- **Final dataset:** 8,079 unique compounds after cleaning and deduplication
- **Potency range:** pIC50 2.54 – 10.96 (mean 6.74)
- **Unique scaffolds:** 2,852


## Background

BACE1 (Beta-site Amyloid Precursor Protein Cleaving Enzyme 1) is a 
protease involved in the production of amyloid-beta peptides implicated 
in Alzheimer's disease. It has been extensively studied as a drug target. 
Its well-curated ChEMBL dataset has made it a standard benchmark for 
computational methods development, and inhibiting BACE1 has been a major 
focus of Alzheimer's drug discovery for over two decades.


## Planned Expansions on the project

- **Graph Neural Network** — replace Morgan fingerprints with a molecular 
  graph representation and compare GNN performance against fingerprint-based 
  models on both split types
- **Multi-target evaluation** — extend the pipeline to multiple ChEMBL 
  targets to assess whether the scaffold split performance gap is consistent 
  across different protein families
