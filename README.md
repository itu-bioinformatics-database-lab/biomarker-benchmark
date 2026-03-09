
# When Complexity Doesn’t Pay — Benchmarking Deep Learning and Ensemble Methods for Biomarker Discovery

A fully reproducible benchmark comparing **27 feature selection strategies** (single and ensemble) and **11 classifiers** across **three multi-omics disease cohorts** (ROSMAP, PSP, BRCA). We evaluate predictive performance, stability, and **external biological validation** against HMDD, CTD (genes & pathways), GeneCards, and EWAS-ATLAS.

> **Companion paper:** *When Complexity Doesn’t Pay: Benchmarking Deep Learning and Ensemble Methods for Biomarker Discovery*. 

---

## 🔍 What’s inside

* **End-to-end notebooks** to:

  * prepare & clean data,
  * run the benchmark across single/dual/triple-omics,
  * aggregate ranks with ensembles,
  * generate publication-ready figures/tables.
* **Modular Python package (`notebooks/modules/…`)** for feature selection, rank aggregation, ML training/evaluation, functional enrichment, and plotting.
* **Deep baselines:** wrapped, working code for **MORE** and **MOGONET** (kept separate for fair comparisons).
* **Processed artifacts** for figures and tables used in the manuscript.

---

## 📦 Repository layout (high-level)

```
Bench/
  ├─ 7.Prepare Data For Publicatio Ready Plots.ipynb
  ├─ 8.Plotting.ipynb
MOGONET/
  ├─ main_mogonet.py, train_test.py, utils.py, models.py, feat_importance.py
  ├─ mogenet-feature-ranks.csv
MORE/
  ├─ Code/{main_MORE.py, train_test.py, utils.py, models.py, feat_importance.py, ...}
  ├─ more-feature-ranks.csv
RScripts/
  ├─ BoxPlots.R, pheatap.R
notebooks/
  ├─ 1. Prepare All Validation Datasets.ipynb
  ├─ 2. Data Cleaning.ipynb
  ├─ 3. Data Preprocessing.ipynb
  ├─ 4. Prepare Datasets for Experiments.ipynb
  ├─ 5. BenchMarking Pipeline.ipynb
  ├─ 6. Prepare Results For Visualization.ipynb
  ├─ 7.Prepare Data For Publicatio Ready Plots.ipynb
  ├─ 8.Plotting.ipynb
  ├─ BenchPipeline{Single,Dual,Trio}{Brca,Mayo,Rosmap}.{py,sh}
  └─ modules/
      ├─ featureSelection/ (MRA.py, RRA.py, TA.py, ensemble_feature_selection.py)
      ├─ differentialAnalysis/ (t-test, limma, mannwhitneyu, boruta, svm_rfe, shap, lime, ...)
      ├─ machineLearning/ (classification.py)
      ├─ functionalEnrichment/ (gProfiler.py, miR_to_Targets.py)
      ├─ dataVisualization/ (plots.py, dimensionality_reduction.py)
      ├─ benchmarker.py, benchmark_utils.py, marble.py, more_mogonet_utils.py
      └─ utils.py, logger.py, stats.py, exception.py, verbose_config.py
artifacts/
  ├─ hm27_probe_ids.json
  └─ mirBase_to_mirTARBase.json
```

> **Note:** Temporary `__pycache__`, `.ipynb_checkpoints`, and `logs/` are included for transparency but are not required to reproduce results.

---

## 🧪 Datasets

* **ROSMAP (AD)** – miRNA, mRNA, methylation
* **PSP (MayoRNASeq)** – mRNA, proteomics, metabolomics
* **BRCA (TCGA via UCSC Xena)** – miRNA, mRNA, methylation

You’ll need to obtain raw data from the original sources (Synapse, UCSC Xena, etc.) and place them as expected by the notebooks. Each cohort’s **download/formatting** is scripted in `notebooks/1.*` and `notebooks/2–4.*`. Paths are parameterized inside notebooks and `notebooks/modules/verbose_config.py`.

---

## ⚙️ Environment

We used **Python 3.11**. Create a clean environment:

```bash
conda create -n biomarker-bench python=3.11 -y
conda activate biomarker-bench

# core
pip install -r requirements.txt
```

> If you use R plots, install the R packages listed in `RScripts/BoxPlots.R` and `pheatap.R`.

---

## 🚀 Quick start (end-to-end)

1. **Prepare validation references**
   Open and run: `notebooks/1. Prepare All Validation Datasets.ipynb`

2. **Clean & preprocess cohorts**
   Run sequentially:

* `notebooks/2. Data Cleaning.ipynb`
* `notebooks/3. Data Preprocessing.ipynb`
* `notebooks/4. Prepare Datasets for Experiments.ipynb`

3. **Run the benchmark**
   Use a pipeline script (single/dual/triple; cohort-specific):

```bash
# examples
python notebooks/BenchPipelineSingleBrca.py
python notebooks/BenchPipelineDualRosmap.py
python notebooks/BenchPipelineTrio.py
```

This produces cross-validated performance, ranked features, and panel-wise outputs under `notebooks/` and `Bench/`.

4. **Aggregate results & make figures/tables**

```bash
# collect & normalize results for plotting
jupyter nbconvert --to notebook --execute notebooks/6. Prepare Results For Visualization.ipynb
# publication-ready data wrangling
jupyter nbconvert --to notebook --execute notebooks/7.Prepare Data For Publicatio Ready Plots.ipynb
# final figures
jupyter nbconvert --to notebook --execute notebooks/8.Plotting.ipynb
```

Key outputs:

* `Bench/Cross_Validation_Results.csv` – all CV metrics
* `Bench/Selected_Biomarker_Panels.csv` – top-k panels per selector
* `Bench/plot_data_{BRCA,MayoRNASeq,ROSMAP}.csv` – figure sources
* `Bench/table_{BRCA,MayoRNASeq,ROSMAP}.csv` – manuscript tables

---

## 🧠 Methods at a glance

* **Single rankers (15):** t-test, Mann–Whitney U, LASSO/Ridge/Elastic Net, Boruta, RF-FI/RF-PFI, XGB-FI/XGB-PFI, SVM-RFE, SHAP, LIME, MORE-Ranker, MOGONET-Ranker.
* **Ensemble rankers (13):** mean/median/geometric mean **rank/weight**, min/max/TA (threshold algorithm), **MRA**, **Stuart**, **RRA**.
* **Classifiers (11):** Logistic Regression, SVM, MLP, Decision Tree, Random Forest, XGBoost, CatBoost, AdaBoost, Gradient Boosting, **MORE**, **MOGONET**.

External validation: HMDD (miRNAs), CTD genes & CTD-pathways, GeneCards (genes), EWAS-ATLAS (CpGs). Validation helpers live under `notebooks/modules/functionalEnrichment/`.

---



> We **do not** blend MORE/MOGONET into ensembles to keep tiers separable (single → ensemble → deep). Use their ranks as standalone comparators.

---

## 🔁 Reproducibility notes

* **CV:** 5-fold stratified. Seeds are set within notebooks/modules (see `verbose_config.py` / `benchmarker.py`).
* **Panels:** evaluated at `k ∈ {10,20,…,100}` and “All” (200/400/600 per single/dual/triple).
* **Preprocessing:** modality-aware variance filters, ANOVA+FDR, correlation constraint (PC1 < 50%), min-max scaling; log10 for proteomics/metabolomics.
* **Class imbalance:** BRCA tumor downsampled to ~1.2× controls (see `4.*` notebook).
* **Validation build:** scripts in `notebooks/1.*` pull and standardize HMDD/CTD/GeneCards/EWAS-ATLAS; pathway validation uses g:Profiler → CTD cross-reference.


## **Interactive Benchmark Explorer:**  
If you run the benchmarking pipeline provided in this repository, you can explore your generated results using the interactive benchmark explorer:  
> **[Benchmark Explorer](https://github.com/CyrilleMesue/biomarker-benchmark-results-explorer)**  
>  
> The explorer allows interactive inspection of benchmark outputs including model performance metrics, feature selection results, heatmaps, and validation summaries.

---

## 📜 Citation

If you use this code or results, please cite the paper:

> *When Complexity Doesn’t Pay: Benchmarking Deep Learning and Ensemble Methods for Biomarker Discovery.* (2025). 

---


## 🙌 Acknowledgements

We thank the maintainers of **CTD, HMDD, GeneCards, EWAS-ATLAS, g:Profiler**, and the data providers behind **ROSMAP, TCGA-BRCA (UCSC Xena), and MayoRNASeq (PSP)**.

---

## ✉️ Contact

For questions, issues, or collaboration requests, please open a GitHub issue or reach out to the corresponding author listed in the paper.

