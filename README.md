# Malakpet Constituency Voter Data Pipeline & Electoral Analysis

A data pipeline and analytical framework for booth-level electoral analysis of the Malakpet Assembly Constituency, Hyderabad, Telangana — built as a pilot on 20 of the constituency's ~300 polling booths, designed to scale to the full constituency.

🔗 **[View Live Interactive Tableau Public Dashboard](#)** 

---

## 1. Overview
This project extracts, structures, and analyzes voter and election-result data at the polling-booth level to support political strategy and campaign decision-making. It combines an automated document-extraction pipeline with an Excel-based analytical framework (area grading, volatility classification, and margin analysis) and a three-view interactive Tableau Public dashboard.

* **Scope:** 20-booth pilot within Malakpet constituency, covering the Saidabad mandal area.
* **Elections analyzed:** 2014, 2018, and 2023 Telangana Legislative Assembly elections.
* **Client lens:** Parameterized in Excel for any party and demonstrated here from the BJP perspective to evaluate hold strength, vulnerability, and expansion pockets.

---

## 2. Why Area-Level, Not Booth-Level
Polling booths are periodically delimited — redrawn, split, merged, or renumbered — by the Election Commission between election cycles. In this dataset specifically:
* 2014/2018 booth numbers include suffixed variants (e.g., `1A`, `16A`) that are consolidated or dropped by 2023.
* Some addresses that held 4 distinct booths in 2018 held only 3 by 2023.
* Raw 2014 Form 20 data provides only numeric booth IDs with no station name or address, making direct number-to-number matching across years unreliable.

Comparing "Booth #15" across three elections risks comparing three different underlying sets of voters, since the boundary assigned to that number may have shifted. The **polling station venue** (a specific community hall, school, or RWA building), by contrast, tends to remain stable across cycles even as the booth numbers and sub-boundaries within it change.

For this reason, all cross-election comparison in this project — area grading, volatility classification, and historical trend analysis — is aggregated by **polling station address/area**, not by raw booth number. Booths with no reliably traceable historical match to a current address were excluded from trend analysis rather than estimated.

---

## 3. Pipeline Architecture
```text
Raw electoral roll PDFs (scanned)
        ↓
Gemini Vision OCR extraction (Python)
        ↓
Structured booth-level voter data (JSON/CSV)
        ↓
Excel — cleaning, area grading, volatility analysis, 2023 turnout modeling
        ↓
Tableau Public — three-view interactive dashboard
```

---

## 4. Data Extraction: Challenges and Engineering Decisions
The extraction pipeline required active engineering trade-offs to handle real-world API and hardware constraints:

* **Rate limits and mid-project deprecation:** Initial builds on `Gemini 2.5 Flash` hit free-tier limits as low as ~20 requests/day, and the model was subsequently retired mid-project.
* **Local fallback attempt:** Tested a local `PaddleOCR + Qwen2-VL + Llama` vision-LLM extraction pipeline (Google Colab, free T4 GPU) as a rate-limit-free alternative. Hit repeated GPU out-of-memory errors with Qwen2-VL and a page-type misclassification issue. This approach is documented but parked, not the active path.
* **Resolution:** Validated `Gemini 3.5 Flash Lite`'s free tier against real request volume (117 requests across 12 booths, ~9.75 requests/booth) — comfortably within budget for the full 20-booth target. Pivoted back to a Gemini-based pipeline using this model.
* **Result:** **~98% extraction accuracy** across all 20 booths on the final pipeline run.

---

## 5. Analysis Methodology
* **Area grading (BJP margin lens):** Margin calculated per polling area as `(BJP votes − maximum votes of any other party)`, expressed in both raw votes and percentage-of-turnout terms (`BJP Margin %`). Averaged across election years using a recency-weighted blend (**60%** for 2023, **20%** each for 2018 and 2014). Areas are classified into four hold-strength categories (**Weak / Average / Good / Strong**) based on margin thresholds.
* **Polling-area volatility classification:** Areas classified as **Consistent** (same party won all 3 elections), **Semi-swing** (2 of 3), or **Swing** (3 different winners), based on aggregate area-level vote totals (see Section 2 for why area, not booth).
* **Turnout analysis:** Reported for the 2023 election only, calculated directly from verified electoral roll (e-roll) data rather than an estimated or modeled voter base. Turnout for 2014 and 2018 is intentionally excluded because reliable booth-level historical elector counts are unavailable at scale. Margin and volatility metrics for 2014/2018 depend only on votes polled and remain unaffected.
* **Demographic–vote correlation:** Area-level demographic composition and household structure metrics (derived from extracted e-roll data) are correlated against party vote share (`CORREL`/`SLOPE`) to identify demographic concentration patterns associated with party strength.

---

## 6. Known Data Limitations
* **Booth-level historical granularity:** Form 17C (vote-count-per-booth) and detailed electoral rolls are not published by the ECI in a uniform, machine-readable format across cycles, and no public API exists. Booths without a traceable historical match were excluded rather than estimated, and turnout is reported for 2023 only.
* **Pilot scale:** Analysis currently covers 20 of ~300 constituency booths (~7%) in the Saidabad area. Correlation and margin findings should be treated as directional/illustrative until scaled to the full constituency.
* **2023-as-2028 proxy:** Where 2023 e-roll data is used as a working base for near-term strategy, it assumes minimal electorate composition shift — a simplifying assumption stated explicitly rather than a predictive claim.
* **Ecological inference caveat:** Demographic–vote correlations represent area-level statistical associations, not individual-level voting behavior, and may reflect co-varying factors such as urbanization, income, or candidate-specific effects.

---

## 7. Repository Structure
```text
├── 01_raw_source_documents/
│   ├── electoral_rolls/        # REDACTED sample pages only — contains personal voter data
│   └── booth_files/            # Full Form 20 scans, all 3 years — public data, no redaction needed
├── 02_pipeline/                # Gemini OCR extraction notebook/scripts
├── 03_pipeline_raw_output/     # Unmodified Excel output straight from the pipeline
├── 04_working_review/          # Malakpet_2014/2018/2023_pdf_to_xlsx.xlsx (raw PDF→Excel per year)
│                               # + Cleaned_Raw_Output.xlsx (flags fixed, intermediate review)
├── 05_final_analysis/          # Cleaned master + final analysis sheets (Area Analysis, Area Grading, Volatility)
├── 06_dashboard/               # Tableau Public dashboard link, workbook notes, and calculated field docs
├── assets/                     # Pipeline diagrams and Tableau dashboard screenshots
├── requirements.txt            # Python dependencies for 02_pipeline
└── README.md
```

---

## 8. Setup
```bash
pip install -r requirements.txt
```
`pdf2image` also requires the `poppler-utils` system package (`apt-get install poppler-utils` on Linux, `brew install poppler` on macOS). The extraction notebook expects a Gemini API key stored in a local `.env` file (excluded via `.gitignore`) and loaded via `python-dotenv`.

---

## 9. Interactive Tableau Public Dashboard
The three-view Tableau Public dashboard consolidates spatial, historical, and demographic layers into a single campaign decision tool:
* **View 1 — Map & Performance:** Constituency map with polling-station pins showing hold-strength grades and area volatility classifications.
* **View 2 — Vote Share & Margin:** Multi-cycle (2014–2023) vote-share trends and calculated `BJP Margin %` across polling areas.
* **View 3 — Demographics:** Booth-area voter segmentation, household structure (`Household Type` calculated field), and demographic concentration metrics.

---

## 10. Tech Stack
`Python` · `pandas` · `pdf2image` · `Gemini Vision API` · `Excel` · `Tableau Public`



