# Malakpet Constituency Voter Data Pipeline & Electoral Analysis

A data pipeline and analytical framework for booth-level electoral analysis of the Malakpet Assembly Constituency, Hyderabad, Telangana — built as a pilot on 20 of the constituency's ~300 polling booths, designed to scale to the full constituency.

## 1. Overview

This project extracts, structures, and analyzes voter and election-result data at the polling-booth level to support political strategy and campaign decision-making. It combines an automated document-extraction pipeline with an Excel-based analytical framework (seat grading, swing-voter classification, margin and turnout analysis) and an in-progress interactive Power BI dashboard.

Scope: 20-booth pilot within Malakpet constituency, covering the Saidabad mandal area. Elections analyzed: 2014, 2018, and 2023 Telangana Legislative Assembly elections.

## 2. Why Area-Level, Not Booth-Level

Polling booths are periodically delimited — redrawn, split, merged, or renumbered — by the Election Commission between election cycles. In this dataset specifically: 2014/2018 booth numbers include suffixed variants (e.g., 1A, 16A) that are consolidated or dropped by 2023, and some addresses that held 4 distinct booths in 2018 held only 3 by 2023. Raw 2014 Form 20 data also provides only numeric booth IDs with no name or address, making direct number-to-number matching across years unreliable.

Comparing "Booth #15" across three elections risks comparing three different underlying sets of voters, since the boundary assigned to that number may have shifted. The polling station venue (a specific community hall, school, or RWA building), by contrast, tends to remain stable across cycles even as the booth numbers and sub-boundaries within it change.

For this reason, all cross-election comparison in this project — seat grading, swing-voter classification, and historical voter estimation — is aggregated by polling station address/area, not by raw booth number. Booths with no reliably traceable historical match to a current address were excluded from trend analysis rather than estimated.

## 3. Pipeline Architecture

```
Raw electoral roll PDFs (scanned)
        ↓
Gemini Vision OCR extraction (Python)
        ↓
Structured booth-level voter data (JSON/CSV)
        ↓
Excel — cleaning, seat grading, swing analysis, 2023 turnout modeling
        ↓
Power BI — interactive dashboard (in progress)
```

## 5. Data Extraction: Challenges and Engineering Decisions

The extraction pipeline was not a straightforward build — several real constraints required active problem-solving:

Rate limits and mid-project deprecation: Initial builds on Gemini 2.5 Flash hit free-tier limits as low as ~20 requests/day, and the model was subsequently retired mid-project.
Local fallback attempt: Tested a local Paddle + Qwen2-VL + Llama vision-LLM extraction pipeline (Google Colab, free T4 GPU) as a rate-limit-free alternative. Hit repeated GPU out-of-memory errors with Qwen2-VL and a page-type misclassification issue. This approach is documented but parked, not the active path.
Resolution: Validated Gemini 3.5 Flash Lite's free tier against real request volume (117 requests across 12 booths, ~9.75 requests/booth) — comfortably within budget for the full 20-booth target. Pivoted back to a Gemini-based pipeline using this model.
Result: ~98% extraction accuracy across all 20 booths on the final pipeline run.

## 5. Analysis Methodology

Seat grading — Margin calculated per area as (party's votes − maximum votes of any other party), expressed in both absolute and percentage-of-turnout terms, averaged across election years with a recency-weighted blend (60% most recent election, 20%/20% for the two prior). Areas classified into four hold-strength categories (Weak/Average/Good/Strong) based on margin thresholds.

 Swing-voter classification — Areas classified as Consistent (same party won all 3 elections), Semi-swing (2 of 3), or Swing (3 different winners), based on aggregate area-level vote totals (see Section 2 for why area, not booth).

 Turnout analysis — Reported for the 2023 election only, calculated directly from actual electoral roll (e-roll) data rather than any estimated or modeled voter base. Turnout for 2014 and 2018 is intentionally excluded: reliable booth-level historical elector counts are not available at the scale needed to support a defensible turnout figure for those years (see Limitations). Margin and swing analysis for 2014/2018, which depend only on actual votes polled rather than an elector-count denominator, are unaffected by this exclusion.

(Power BI- Dashboard In Progress) Demographic–vote correlation — Booth/area-level demographic composition (from the extracted voter data) is correlated against party vote share (CORREL/SLOPE) to identify areas of demographic concentration associated with particular parties' vote strength. This is an ecological-level association, not an individual-level claim — see Limitations.

## 6. Known Data Limitations

Booth-level historical granularity: Form 17C (vote-count-per-booth) and detailed electoral roll data are not published by the Election Commission in a uniform, machine-readable format across election years. Historical PDFs use inconsistent booth numbering, and there is no public API for programmatic access. This is a constraint of India's public electoral-data infrastructure, not a gap in extraction capability — the pipeline achieved ~98% accuracy on all available, legible source material. Booths with no traceable historical match were excluded from trend analysis rather than estimated. For the same reason, turnout is reported for 2023 only, where reliable actual elector data exists.
Pilot scale: Analysis currently covers 20 of ~300 constituency booths (~7%). Correlation and margin findings should be treated as directional/illustrative until scaled to the full constituency.
2023-as-2028 proxy: Where 2023 data is used as a working base for near-term (2028) projection, this assumes no significant electorate composition change — a simplifying assumption stated explicitly, not a claim of predictive precision.
Ecological inference caveat: Demographic–vote correlations are area-level statistical associations. They do not establish individual voting behavior and may reflect confounding factors (urbanization, income, candidate-specific effects) that co-vary with the demographic variable studied.

## 7. Repository Structure

```
├── 01_raw_source_documents/
│   ├── electoral_rolls/        # REDACTED sample pages only — contains personal voter data
│   └── booth_results_pdfs/     # Full Form 20 scans, all 3 years — public data, no redaction needed
├── 02_pipeline/                # Gemini OCR extraction notebook/scripts
├── 03_pipeline_raw_output/     # Unmodified Excel output straight from the pipeline
├── 04_working_review/          # Lightly modified/reviewed master — not yet fully cleaned
├── 05_final_analysis/          # Cleaned master + final analysis sheets (Area Analysis, Seat Grading, Swing Voter)
├── 06_dashboard/               # Power BI roadmap and screenshots (in progress)
└── assets/                     # Diagrams, screenshots
```

## 9. Dashboard — In Progress

Planned structure:
```
Landing page: constituency map with booth-level pins, matched by polling station location name
Drillthrough → Current Demographics: booth-level voter segmentation
Drillthrough → Past Elections: area/booth-wise historical results and trends
Combined seat + area grading matrix
```
Status: Excel-side analysis complete. Power BI build in progress.

## 9. Tech Stack

Python · Gemini Vision API · Excel (advanced formulas, structured tables) · Power BI · DAX

## 10. Author

Saived kongara —  BBA , IBS Hyderabad (ICFAI) 

LinkedIn: www.linkedin.com/in/ksaived 

Email: saivedkongara@gmail.com


