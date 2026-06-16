# Functional Evidence Calibration

Calibrate a **functional assay score** (or any in-silico predictor) into ACMG/AMP clinical
evidence strengths, using ClinVar as the clinical truth set and the ClinGen-SVI local
calibration method ([Pejaver et al. 2022](https://doi.org/10.1016/j.ajhg.2022.10.013)).

The worked example uses **AlphaMissense** scores for **LDLR** because they are public — no
real functional data is distributed here. Swap in your own dataset by editing one parameters
cell.

## What it does

1. **`notebooks/01_process_clinvar.ipynb`** — download NCBI ClinVar `variant_summary.txt.gz`,
   filter to germline GRCh38 coding SNVs with a non-conflicting Pathogenic/Benign assertion,
   parse the protein change, and write a tidy per-substitution label table (`cs_simple`).
2. **`notebooks/02_calibrate_functional_data.ipynb`** — align your score to the ClinVar labels,
   check class separation, set or estimate the class prior `alpha`, run calibration + inference,
   and attach ACMG evidence strengths (`ACMG18`/`ACMG20`) to every variant.

## Repo layout

```
functional-evidence-calibration/
├── notebooks/
│   ├── 01_process_clinvar.ipynb
│   └── 02_calibrate_functional_data.ipynb
├── data/
│   ├── raw/                      # downloaded ClinVar releases (gitignored)
│   ├── clinvar/                  # processed label tables (LDLR example bundled)
│   └── example/                  # example functional input (LDLR AlphaMissense)
└── results/example_output/       # example calibration + inference outputs
```

## Setup

```bash
pip install -r requirements.txt

# Required: ClinGen-SVI local calibration code
git clone https://github.com/pejaverlab/clingen-svi-comp_calibration.git

# Optional: only if you want to ESTIMATE alpha (the class prior) from the data
git clone https://github.com/Dzeiberg/dist_curve.git
pip install -e dist_curve
# dist_curve also needs a trained estimator weights file (model.hdf5) from that project;
# point ESTIMATOR_MODEL_PATH at it in notebook 02.
```

> **Compatibility:** the calibration tool targets Python 3.10–3.11 and NumPy < 1.25. Its
> argparse setup fails on Python 3.14, and it depends on legacy NumPy behaviour removed in
> 1.25. The pins in `requirements.txt` (NumPy 1.24.4 / SciPy 1.10.1) are known-good.

## Usage

1. Run `01_process_clinvar.ipynb` to (re)build the ClinVar table. The repo already ships an
   **LDLR** example subset (`data/clinvar/clinvar_coding_GRCh38_2025-07_LDLR.csv.gz`), so you
   can skip this if you only want to reproduce the example.
2. Open `02_calibrate_functional_data.ipynb`, edit the **Parameters** cell (gene, functional
   data path, score column, `ALPHA` / `ESTIMATE_ALPHA`), and run all cells.

### Bringing your own functional data

Edit the **Parameters** cell and the functional-data parser in notebook 02. The merge needs
`gene`, `aapos`, `aaref`, `aaalt` columns plus your score column. Higher scores are assumed to be
**more pathogenic** — negate a reversed assay before calibrating.

## Data sources & citation

- **ClinVar** — NCBI, `variant_summary.txt.gz`
  (<https://ftp.ncbi.nlm.nih.gov/pub/clinvar/tab_delimited/>).
- **AlphaMissense** (example scores) — Cheng et al., *Science* 2023; licensed CC BY-NC-SA 4.0 by
  Google DeepMind. Example data included here for non-commercial demonstration only.
- **Calibration method** — Pejaver et al., *Am J Hum Genet* 2022;
  [`pejaverlab/clingen-svi-comp_calibration`](https://github.com/pejaverlab/clingen-svi-comp_calibration).
- **Class-prior estimation** — [`Dzeiberg/dist_curve`](https://github.com/Dzeiberg/dist_curve)
  (Zeiberg et al., AAAI 2020).

## License

Code and notebooks are released under the [MIT License](LICENSE). Bundled example data is
licensed separately — the AlphaMissense example is CC BY-NC-SA 4.0 (non-commercial); ClinVar-derived
data is public domain. See [LICENSE](LICENSE) for details.
