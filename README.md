# Group 7 — Beyond Raw Denial Rates (2024 HMDA)

**Big Data Analysis · Deliverable 3: Code and Reproducibility**  
**Group:** Hoang Thi Mai Anh (11230731), Nguyen Linh Anh (11230732), Do Thi Khanh Ly (see ID check below).  
**Institution:** National Economics University, Viet Nam.  
**Reporting year:** 2024.

This is a **submission snapshot**, assembled from the Group 7 Google Drive code and exported results and
paired with the exact PDF report supplied by the authors. The original model outputs have **not** been
recomputed during packaging. The `tools/verify_outputs.py` script validates the bundled saved outputs.

## 1. Directory structure

```text
project/
├── README.md                         # you are here
├── requirements.txt                  # measured Python dependency versions
├── report.pdf                        # the supplied final 10-page PDF, unmodified
├── src/                              # all 9 stage notebooks
│   ├── 00_ingest_csv_to_parquet.ipynb
│   ├── 01_clean_pandas.ipynb
│   ├── 01_clean_pyspark.ipynb
│   ├── 02_rq1_pandas.ipynb
│   ├── 03_benchmark.ipynb
│   ├── 04_rq2_pandas.ipynb
│   ├── 05_rq2_pyspark_check.ipynb
│   ├── 06_rq3_diagnostic.ipynb
│   └── 07_rq4_predictive.ipynb
├── results/
│   ├── cleaning/                     # facts and both cleaning logs
│   ├── rq1/                          # RQ1 standalone CSVs
│   ├── rq1_rq2_pandas/              # 37 analysis tables, manifest
│   ├── rq2_spark/                   # correctness and profiling
│   ├── rq3/                         # diagnostics, standardized gaps, bootstrap, LEI
│   ├── rq4/                         # UPDATED model tables and manifest
│   └── benchmark/                   # full 75-run experiment and environment
├── figures/
│   ├── rq1/                         # RQ1 maps/funnel/panel
│   ├── rq1_rq2_pandas/             # 17 exported research figures
│   ├── rq3/                         # diagnostic charts
│   ├── rq4/                         # predictive charts
│   ├── benchmark/                    # performance chart
│   └── report_panels/               # images used for ACM report
├── docs/                              # acquisition instructions and provenance
└── tools/verify_outputs.py           # verify saved output files (no raw data needed)
```

## 2. Data source, reporting year and acquisition

The data source is the **2024 public Loan/Application Register (LAR)** published by the
Federal Financial Institutions Examination Council (FFIEC) and Consumer Financial Protection Bureau (CFPB).
- Official publication page: https://ffiec.cfpb.gov/data-publication/2024
- Official field definitions: https://ffiec.cfpb.gov/documentation/publications/loan-level-datasets/lar-data-fields
- Instructor-provided ZIP: `2024_public_lar_one_year_csv.zip` (uncompressed CSV: 12,259,199 rows,
  99 variables, 4,636,628,799 bytes). These facts are recorded in `results/cleaning/dataset_facts.json`.
- Group Drive working folder: https://drive.google.com/drive/folders/1LWU1AKDHTmspApD4vBPanOcGi8MRv-If

**Raw CSV, converted raw Parquet, cleaned Parquet and individual-level predictive split/prediction Parquet are
intentionally excluded** from this submission to avoid unnecessary duplicate large public/derived datasets.
Download the raw ZIP via the instructor-provided course resource or the group's accessible shared Drive,
or obtain the corresponding annual public file from the official data publication page.
Put the file in the group Drive workspace as the notebook prompts specify.
To reproduce only analytical stages, the group Drive also contains `hmda_clean_v1/hmda_clean_v1.parquet`
(~47 MB), produced by the Pandas cleaning notebook.
The published outputs remain in `results/` and `figures/` even without the raw data.

## 3. Environment and installation

**Recorded runtime:** Google Colab, Python **3.13.18**, 2 CPUs, ~13.6 GB RAM; Pandas **2.2.3**,
PySpark **4.0.5**, PyArrow **23.0.1**, scikit-learn **1.6.1** for predictive models.
Use a clean session and **Java 17** for Spark. The notebooks are Colab-oriented and mount Google Drive;
they are not plug-and-play on a machine without the Colab `google.colab` module and the data paths.

For an equivalent Python environment:

```bash
python -m venv .venv
# Activate the virtual environment as appropriate for your operating system
python -m pip install -r requirements.txt
```

For Google Colab: upload/open each `.ipynb`, put/shortcut the shared folder at
`/content/drive/MyDrive/hmda`, use **Runtime → Restart session and run all**, and follow the
individual notebook's setup cells. Do **not** mix extra PySpark 3.x installs with the specified 4.0.5
in the same runtime. Local Spark also needs a working Java installation.

## 4. Reproduction order (source files in `src/`)

| Stage | Notebook | Purpose | Main results |
|---|---|---|---|
| 00 | `00_ingest_csv_to_parquet.ipynb` | Raw ZIP/CSV ingestion to columnar Parquet | `hmda_parquet`, `dataset_facts.json` |
| 01a | `01_clean_pandas.ipynb` | R1–R12 cleaning with Pandas | `hmda_clean_v1.parquet`, `cleaning_log.json` |
| 01b | `01_clean_pyspark.ipynb` | Independent Spark cleaning | Spark Parquet, `cleaning_log_spark.json` |
| 02 | `02_rq1_pandas.ipynb` | State, MSA, region, loan type and intervals | `results/rq1/`, `figures/rq1/` |
| 03 | `03_benchmark.ipynb` | Same seven aggregations at 25/50/100% × five configs × five runs | `results/benchmark/` |
| 04 | `04_rq2_pandas.ipynb` | Regional financial profiles, DTI, product mix and gaps | `results/rq1_rq2_pandas/` |
| 05 | `05_rq2_pyspark_check.ipynb` | Independently compare Pandas/Spark RQ2 tables; run after 04 | `results/rq2_spark/` |
| 06 | `06_rq3_diagnostic.ipynb` | Shared-support direct adjustment, bootstrap, CLTV/LEI sensitivity | `results/rq3/` |
| 07 | `07_rq4_predictive.ipynb` | Same train/test split; sklearn/Spark ML, A/B/C comparison | `results/rq4/` |

The cleaned cohort used by stages 02–07 has **3,140,857** observations, including **276,604** denials.
For state-level geography, the 50 states + D.C. reference covers **3,119,941** observations and **268,873**
denials (8.6179%). National whole-cohort comparisons use 8.8066%.
Primary RQ3 common support uses a **different** reference (4.9917%) and must not be compared by
subtracting unadjusted full-state rates from shared-support adjusted estimates.

The report's descriptive and diagnostic analysis uses **Pandas and PySpark**, with 37/37 RQ1/RQ2 tables
reported equivalent; RQ3 includes seven independent checks. The performance experiment is a separate
**75-run** comparison, not the RQ4 single-fit training times.

## 5. Checking the bundled saved outputs, without rerunning 12 million rows

```bash
python tools/verify_outputs.py
```

This validates the JSON notebooks, cohort totals, independent correctness summaries,
the 75-run benchmark grid, 80/20 predictive split and RQ4 A/B metrics. It does **not** replace rerunning
the notebooks on the original HMDA data and does not attest to causal interpretation.
Run with the same data and environment for full computational reproduction.

## 6. Privacy, exclusions and result versions

- Do not commit personal Google credentials, mount tokens or private data. The included notebooks may
  open standard Google Drive authorization prompts at runtime; the package contains no shared credentials.
- Financial invalid/missing values follow `docs/cleaning_rules_v1.md`; records are not deleted simply
  because one variable is missing. Demographic attributes are not model predictors.
- The latest saved RQ4 results matching **this report** are included. In particular, the scikit-learn
  Random Forest test ROC-AUC is **0.7872 (Model A) vs 0.7914 (Model B)**. An older archived RQ4 ZIP with
  0.7820/0.7821 has deliberately been excluded. RQ4 individual-level `.parquet` predictions/splits from
  the earlier run are likewise omitted to avoid mixing versions. Recreate them from stage 07 if needed.
- The reported diagnostic learner/model comparisons are not causal proof of geographic discrimination.
- The **report PDF is copied byte-for-byte** from the author's uploaded `Group7_Final_ACM_GitHub.pdf`.

## 7. Original output and code provenance

See `docs/SOURCE_PROVENANCE.md` for original Drive locations and the difference between sourced
files and newly authored packaging instructions. SHA-256 checksums are in `docs/FILE_MANIFEST.csv`.
