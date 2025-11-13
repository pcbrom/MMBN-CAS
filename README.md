# Blockchain Maturity Model for Sustainable Agri-Food Supply Chain Businesses

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.17603492.svg)](https://doi.org/10.5281/zenodo.17603492)

End-to-end pipeline for simulation, psychometric analysis, and equating of a 22-item ordinal instrument (Likert 1–5). This repository includes:

- Synthetic response generation with an LLM for maturity profiles (novice, intermediate, advanced).
- Reliability, dimensionality (PA/EFA/CFA), and IRT calibration (GRM) in R.
- Equating and classification by cut scores, plus complementary analyses in Python.


Flow overview
- 1) `simulador.ipynb`: builds a response agenda and calls the API (OpenAI) to generate item/persona/profile responses, saving `outputs/mmbncas_llm_raw.jsonl`.
- 2) `analise_simulacao.R`: reads JSONL, computes reliability (α, ω), runs PA/EFA/CFA, calibrates a GRM (mirt), and exports parameters and scores: `outputs/grm_item_parameters_mmab_ncas.csv`, `outputs/mmbncas_llm_with_theta.csv`, plus figures.
- 3) `equalizacao.ipynb`: re-estimates `theta_hat` via MLE from GRM parameters, applies cut points `[-0.20, 0.40]` for profiles (novice/intermediate/advanced), and generates diagnostics/plots (Wright map, standardized residuals, categorical divergence, PCA/clustering). It may produce `outputs/mmbncas_llm_with_theta_scored.csv`.


Items and dimensions (used in the analyses)
- Items: `q1` … `q22`, responses in 1–5 (ordinal).
- Hypothesized dimensions:
  - Governance & Strategy: `q1–q6`
  - Operational Integration: `q7–q12`
  - Sustainability & Scalability: `q13–q22`
- Profiles and cut points for classification: `CUTS = [-0.20, 0.40]`, `LABELS = ["novice", "intermediate", "advanced"]`.


Requirements
- Python 3.10+ and R 4.2+ (recommended)
- Python (core in `requirements.txt`):
  - `python-dotenv`, `pandas`, `tqdm`, `openai`, `rpy2`, `numpy`, `tenacity`
  - Extra packages used in notebooks: `matplotlib`, `seaborn`, `scipy`, `scikit-learn`, `factor_analyzer`, `jupyter`
- R packages: `tidyverse`, `psych`, `GPArotation`, `lavaan`, `mirt`


Quick setup
1) Python
   - Create a virtual environment and install the core dependencies:
     - `python -m venv .venv && source .venv/bin/activate` (Linux/Mac)
     - `python -m venv .venv && .venv\Scripts\activate` (Windows)
     - `pip install -r requirements.txt`
   - For notebooks: `pip install jupyter matplotlib seaborn scipy scikit-learn factor_analyzer`
2) R
   - From R/RStudio: `install.packages(c("tidyverse","psych","GPArotation","lavaan","mirt"))`
3) Credentials (to run the LLM simulator)
   - Create a `.env` file with: `OPENAI_API_KEY=your_token_here`


How to run
- Option A — Reproduce with existing artifacts (no API costs):
  1. Skip `simulador.ipynb` and use the existing `outputs/mmbncas_llm_raw.jsonl`.
  2. Run `analise_simulacao.R` (RStudio or `Rscript analise_simulacao.R`).
  3. Open and run `equalizacao.ipynb` (ensure `PARAM_PATH` and `RESP_PATH` point to the files in `outputs/`).

- Option B — Full pipeline (incurs API costs):
  1. `simulador.ipynb`: configure `.env`, execute cells to generate `outputs/mmbncas_llm_raw.jsonl`. Defaults: 20 replicas × 3 profiles × 60 respondents; personas and per-profile theta distributions are defined in the notebook.
  2. `analise_simulacao.R`: runs reliability (α, ω), PA/EFA (Spearman, ML+Promax), CFA (DWLS with `lavaan`), and IRT GRM (`mirt`). Exports:
     - `outputs/grm_item_parameters_mmab_ncas.csv` (a, b1..b4)
     - `outputs/mmbncas_llm_with_theta.csv` (consolidated dataset with items and EAP-estimated `theta`)
     - Figures: item/test information, ICC grid, etc.
  3. `equalizacao.ipynb`: reads GRM parameters and responses, estimates `theta_hat` (MLE), computes SE via information, classifies by cut points, and generates:
     - `outputs/wright_map.png`, `outputs/standardized_residuals.png`, `outputs/categorical_divergence.png`, `outputs/pca_clustering_analysis.png`
     - A dataset with `theta_hat` and predicted profile (e.g., `outputs/mmbncas_llm_with_theta_scored.csv`)

Important notes
- Cost/time: `simulador.ipynb` makes OpenAI API calls. Use the precomputed artifacts in `outputs/` to avoid costs.
- Column `theta`: in the original JSONL, `theta` is the imposed/drawn latent trait; in the R pipeline, the consolidated file stores the estimated `theta` (EAP). The equating notebook treats `theta`, when present, as imposed/reference for evaluation (RMSE, correlation). Confirm semantics before comparing estimates.
- Parallelization: `equalizacao.ipynb` uses `ProcessPoolExecutor` to speed up batch `theta_hat` estimation.


Repository structure (key files)
- `simulador.ipynb` — LLM-based simulation and response collection.
- `analise_simulacao.R` — R pipeline: parsing, reliability, PA/EFA/CFA, GRM.
- `analise_simulacao.ipynb` — complementary analyses in Python (α/ω, ordinal EFA, R bridge via `rpy2`).
- `equalizacao.ipynb` — equating/MLE estimation, diagnostics, and visualizations.
- `outputs/` — generated artifacts (JSONL, CSVs, figures). There is also a copy of `mmbncas_llm_with_theta_scored.csv` at the repository root.
- `material/` — notes and supporting materials (`material/nota.txt` lists Python packages and writing notes).
- `MMBN-CAS.Rproj` — RStudio project file.
- `LICENSE` — repository license.


Expected results/artifacts (examples)
- `outputs/mmbncas_llm_raw.jsonl` — responses per respondent (uuid), profile, persona, `theta`, and JSON blob with `q1..q22`.
- `outputs/mmbncas_llm_with_theta.csv` — consolidated dataset with items and GRM EAP `theta`.
- `outputs/grm_item_parameters_mmab_ncas.csv` — IRT GRM parameters (a, b1..b4) by item.
- Figures: `grid_icc_grm.png`, `item_information.png`, `test_information.png`, `wright_map.png`, `standardized_residuals.png`, `categorical_divergence.png`, `pca_clustering_analysis.png`.


License
- See `LICENSE` for terms of use and redistribution.
