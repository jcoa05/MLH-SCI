# Pragmatic Stacked Ensemble ML for Resource-Constrained Clinical Research

![R](https://img.shields.io/badge/R-%3E%3D4.4.0-blue?logo=r&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)
![Status](https://img.shields.io/badge/Status-Work%20in%20Progress-yellow)

> **Demonstrating that ensemble machine learning is achievable on standard consumer hardware**

This repository contains a complete, reproducible implementation of stacked ensemble machine learning designed specifically for researchers operating under computational constraints. The workflow executes entirely on mid-range laptop hardware (8 GB RAM, no GPU) without requiring cloud computing, high-performance clusters, or specialized infrastructure.

---

## Project Status

This project is under active development. Current status:

| Component | Status |
|-----------|--------|
| Core ML workflow | Complete |
| Stacked ensemble construction | Complete |
| SHAP/DALEX explainability | Complete |
| Visualization outputs | In progress |
| Manuscript | In progress |

The manuscript is being prepared for submission to *Machine Learning: Health*.

---

## Motivation

Machine learning has transformed clinical prognostic modeling, yet advanced methods remain largely inaccessible to researchers in low- and middle-income countries (LMICs) and other resource-constrained settings. The barriers are well documented: limited computing infrastructure, insufficient data availability, and shortages of trained scientists.

Stacked ensemble learning, which combines predictions from multiple diverse algorithms through meta-learning, offers improved prediction accuracy and robustness. However, the computational demands of training multiple base learners with cross-validation and hyperparameter optimization have rendered stacking impractical for many research teams.

This project addresses a critical knowledge gap: **Which adaptations are necessary and sufficient to make stacked ensembles feasible on standard hardware?** We provide transparent documentation of every methodological trade-off, enabling researchers to make informed decisions about adapting these techniques to their own constraints.

---

## Key Features

- **Consumer hardware execution**: Complete workflow runs on 8 GB RAM laptops without GPU acceleration
- **Overnight completion**: Total execution time under 8 hours for full pipeline
- **Five diverse base learners**: Elastic Net, Random Forest, XGBoost, SVM, and MARS
- **Non-negative LASSO meta-learner**: Automatic base learner selection with interpretable weights
- **Two-stage evaluation**: Approximately unbiased performance estimates without nested CV overhead
- **Comprehensive explainability**: Permutation importance (DALEX) and approximate SHAP values (fastshap)
- **Strict reproducibility**: Complete `renv.lock` for exact package version management
- **Detailed feasibility metrics**: Memory consumption, timing breakdowns, deployment footprint

---

## Repository Structure

```
MLH-SCI/
├── mlh.Rmd                 # Main analysis workflow (literate R Markdown)
├── data.csv                # Simulated SCI cohort dataset (N=1,500)
├── renv.lock               # Exact package versions for reproducibility
├── renv/                   # renv library management
├── article_metrics.rds     # Compiled feasibility metrics (generated)
├── README.md               # This file
└── LICENSE                 # MIT License
```

---

## Requirements

### Hardware (Minimum)
- **CPU**: Any modern multi-core processor (tested on Intel Core i5-1335U)
- **RAM**: 8 GB minimum
- **Storage**: 2 GB free space for packages and outputs
- **GPU**: Not required

### Software
- **R**: Version 4.4.0 or higher
- **Operating System**: Windows, macOS, or Linux

All R package dependencies are managed through `renv` and will be installed automatically.

---

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/jcoa05/MLH-SCI.git
cd MLH-SCI
```

### 2. Open the project in RStudio

Open `MLH-SCI.Rproj` (if available) or set your working directory to the cloned folder.

### 3. Restore the environment

The first code chunk in `mlh.Rmd` will automatically restore all package dependencies from `renv.lock`. Alternatively, run manually in R:

```r
# Install renv if not available
if (!requireNamespace("renv", quietly = TRUE)) {
  install.packages("renv")
}

# Restore exact package versions
renv::restore()
```

This ensures you have identical package versions to those used in development, guaranteeing reproducibility.

---

## Usage

### Running the Complete Workflow

Open `mlh.Rmd` in RStudio and knit the document, or run chunks sequentially. The workflow will:

1. Load and preprocess the simulated dataset
2. Define and tune five base learners using grouped cross-validation
3. Construct the stacked ensemble with non-negative LASSO meta-learner
4. Evaluate performance using independent two-stage CV
5. Generate explainability outputs (permutation importance, SHAP values)
6. Compile feasibility metrics

**Expected runtime**: 2-4 hours on the minimum hardware specification.

### Control Parameters

Key parameters can be adjusted in the "Control knobs" chunk:

```r
cv_folds = 5              # Number of CV folds
cv_repeats = 2            # Number of CV repeats
trees_n = 500             # Trees per forest/boosting ensemble
grid_size = 15            # Hyperparameter configurations per algorithm
bootstrap_iters = 100     # Bootstrap iterations for CIs
shap_nsim = 10            # SHAP Monte Carlo samples
```

Reducing these values decreases runtime at the cost of optimization thoroughness and estimate precision.

---

## Methodology Overview

### Pragmatic Adaptations

The workflow incorporates four deliberate adaptations trading theoretical optimality for computational tractability:

| Adaptation | Trade-off | Justification |
|------------|-----------|---------------|
| **Grouped repeated CV** (not nested) | Minor hyperparameter optimization bias | Reduces ensemble constructions from 125 to 25 |
| **Constrained hyperparameter grids** | Potentially suboptimal configurations | 70-80% reduction in search space |
| **Two-stage evaluation** | Stage 1 metrics carry optimization bias | Eliminates meta-learner leakage in Stage 2 |
| **Approximate SHAP** (Monte Carlo) | Approximation error in feature attributions | ~50x computational speedup |

### Base Learners

| Algorithm | Engine | Key Tuned Parameters |
|-----------|--------|---------------------|
| Elastic Net | glmnet | Penalty, L1/L2 mixture |
| Random Forest | ranger | mtry, min_n |
| XGBoost | xgboost | Tree depth, learning rate, min_n, loss reduction, sample size |
| SVM (RBF) | kernlab | Cost, sigma |
| MARS | earth | Number of terms, interaction degree |

### Meta-Learner

Non-negative LASSO regression performs automatic base learner selection by driving irrelevant coefficients to zero while ensuring all retained members contribute positively to predictions.

---

## Feasibility Benchmarks

Results from execution on Intel Core i5-1335U (8 GB RAM):

| Metric | Value |
|--------|-------|
| Total workflow time | < 3 hours |
| Peak memory (RSS) | < 6 GB |
| Total training episodes | ~1,500 |
| Selected ensemble members | Variable (LASSO-determined) |
| Single-patient inference | < 50 ms |
| Serialized model size | < 100 MB |

These benchmarks demonstrate feasibility for overnight execution on consumer hardware.

---

## Limitations

### Methodological
- **Simulated data**: Results reflect controlled conditions; real-world data introduces additional preprocessing burden
- **Approximate SHAP**: Monte Carlo sampling introduces approximation error compared to exact Shapley values
- **Constrained search**: Hyperparameter grids may miss optimal configurations
- **No nested CV**: Stage 1 base learner metrics carry optimization bias (Stage 2 ensemble metrics are approximately unbiased)

### Generalizability
- **Single dataset**: Feasibility demonstrated on one simulated cohort; generalization to other domains requires validation
- **Hardware variation**: Timing results will vary across different processor architectures
- **R ecosystem**: Implementation is R-specific; Python users would need adaptation

### Scope
- **Regression only**: Current implementation targets continuous outcomes; classification would require modifications
- **No missing data handling**: Simulated data is complete; real applications require imputation strategies

---

## Citation

If you use this work, please cite:

```bibtex
@article{mlh_sci_2025,
  title={Pragmatic implementation of stacked ensemble machine learning in 
         resource-constrained clinical research: a methodological demonstration},
  author={[Authors]},
  journal={Machine Learning: Health},
  year={2025},
  note={Manuscript in preparation}
}
```

A DOI will be added upon publication.

---

## Contributing

This project is under active development. Contributions, suggestions, and issue reports are welcome.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit changes (`git commit -am 'Add improvement'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

Please ensure any code contributions maintain compatibility with the 8 GB RAM constraint.

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

---

## Acknowledgments

This work aims to democratize access to advanced machine learning methods for researchers operating under resource constraints, particularly in low- and middle-income countries where disease burden is disproportionately high but computational infrastructure remains limited.

---

## Contact

For questions or collaboration inquiries, please open an issue on this repository.
