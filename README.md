# Global UAV & Flying Drone Patent Landscape (2000–2022)
**Innovation Management — Group 14 (Flying Drones)**

This repository contains the data extraction, semantic filtering, and validation pipeline for analyzing the global Unmanned Aerial Vehicle (UAV) technological sector using **EPO PATSTAT (`PROD`)**.

The current milestone on `main` freezes **Stage 1 (High-Recall Baseline Extraction)** and **Stage 2 (Semantic Refinement & Validation)**, providing a clean, validated core corpus ready for the upcoming landscape analysis steps.

---

## Current Sprint & Team Allocation
**Deadline:** Thursday, November 26th

* **Step 1 — Describe the Field (`Satine`)**
  * Analyze market concentration and whether the sector is consolidating.
  * Identify key incumbents versus new entrants.
  * Determine which technological subfields are growing fastest.
  * Evaluate the Political, Economic, Social, and Technological (PEST) forces shaping the industry.
* **Step 2 — Competition & Harmonization**
  * **Map Competition (`Michel`):** Competitive positioning and assignee dynamics.
  * **Harmonization (`Francesco`, `Riym`):** Cross-stream integration, alignment, and dataset consistency.
* **Step 3 — Score the Present (`Casteddu`)**
  * Assess the current technological maturity and competitive standing of the sector.

---

## Repository Structure

```text
Innovation-Management-Group-14/
├── .gitignore
├── README.md
├── notebooks/
│   ├── 01_baseline_extraction.ipynb
│   ├── 02_semantic_filtering_and_audit.ipynb
│   └── 03_manual_validation_sampling.ipynb
├── data/
│   └── flying_drones_stage2_clean.parquet
└── validation/
    └── drone_validation_sample_100.xlsx
```

---

## Folder & File Guide

### 1. `notebooks/`
Sequential Jupyter notebooks executed on the EPO Technology Intelligence Platform (TIP):
* **`01_baseline_extraction.ipynb`**: Connects to PATSTAT `PROD` via SQLAlchemy ORM and executes a multi-clause boolean retrieval query combining core/secondary CPC & IPC codes (`B64U`, `B64C`, `G05D`, etc.) with domain anchor keywords and exclusion filters. Runs initial missingness and filing-year diagnostics before exporting the raw baseline.
* **`02_semantic_filtering_and_audit.ipynb`**: Loads the baseline corpus alongside pre-computed 384-dimensional sentence embeddings (`all-MiniLM-L6-v2`). Truncates filings after `2022-12-31` to eliminate right-truncation lag, computes cosine similarity against a gold-standard `B64U` seed centroid (**Strategy A**, threshold $\ge 0.65$), validates domain precision via a **CPC Subclass Migration Audit**, and exports the final Stage 2 corpus.
* **`03_manual_validation_sampling.ipynb`**: Loads the frozen Stage 2 dataset, draws a reproducible random sample of `100` patents (`random_state=42`), splits them evenly across all `5` team members (`20` records each), and exports the annotation sheet. Includes a post-annotation evaluation cell to compute empirical corpus precision and the **95% Wilson Score Confidence Interval**.

### 2. `data/`
* **`flying_drones_stage2_clean.parquet`**: The finalized, high-precision Stage 2 dataset (~23k patent applications filed between `2000-01-01` and `2022-12-31` with centroid similarity $\ge 0.65$). Load directly in your notebooks with:
  ```python
  df = pd.read_parquet("data/flying_drones_stage2_clean.parquet")
  ```
* **External Large Files (Excluded via `.gitignore`):** The raw SQL baseline (`flying_drones_baseline_prod.parquet`, ~220k records) and dense embedding matrix (`flying_drones_embeddings.npy`, ~339 MB) exceed GitHub's file size limits.
  * [Download `flying_drones_embeddings_final.npy` from Google Drive](https://drive.google.com/file/d/1S1HsmdiZ-A4__a6OZcK5CF5TqVoRkWOF/view?usp=drive_link)

### 3. `validation/`
* **`drone_validation_sample_100.xlsx` (or `.csv`)**: The 100-patent manual audit sheet pre-assigned to `Member_1` through `Member_5`. Each member reviews their 20 assigned titles, abstracts, and CPC codes, filling in `is_relevant` (`1` = relevant flying drone technology, `0` = false positive) and optional `review_notes`.

---

## Methodology & Validation Summary

* **Right-Truncation Cutoff (`2000–2022`):** Filings from 2023 onward are excluded to prevent statutory 18-month publication windows and PATSTAT database ingestion delays from artificially depressing recent filing trends.
* **Semantic Filtering (`Strategy A >= 0.65`):** Dense vector similarity against a 15-patent `B64U` core centroid isolates direct UAV architectures from peripheral keyword matches.
* **CPC Subclass Migration Audit:** Post-filtering, the top 5 core aeronautical and autonomous control subclasses (**`B64U`**, **`B64C`**, **`B64D`**, **`G05D`**, **`B64F`**) expanded from **46.3%** of the baseline to **72.4%** of the retained corpus, while generic software (`G06F`), computer vision (`G06V`), and cellular telecom (`H04W`) noise dropped substantially.
