# Harborline Outfitters: CRM Data Cleaning & Audit
## r-datacleaning-project

Auditing and cleaning a messy, never-reviewed CRM export in R : turning a raw customer extract into an analysis-ready dataset for regional segmentation and churn modeling, with every cleaning decision documented and justified.

## Business Context

Harborline Outfitters' analytics team wants to build a regional customer segmentation and, eventually, a churn model. Both depend on values for *where* a customer is, *how much* they earn, and *how much* they've spent, but the CRM extract has never been audited.

This project treats that extract as a real, imperfect dataset: duplicate customer records with disagreeing values, dates written in three different layouts, currency fields stored as punctuated text, placeholder codes standing in for missing values, and a handful of flatly impossible values (an age of 203, a lifetime spend of nearly $2 million).

## What's in This Repo

| File | Description |
|---|---|
| `harborline-crm-data-cleaning.Rmd` | The full R Markdown source: cleaning steps, code, and the reasoning behind each decision. |
| `harborline-crm-data-cleaning.html` | The knitted report (open this to see the full analysis and output without running any code.) |
| `Harborline_Customers_RAW.csv` | The raw input: 205 customer records, 12 columns |
| `output/harborline_customers_clean.csv` | The cleaned, analysis-ready dataset (200 rows). |
| `output/harborline_data_dictionary.csv` | Column-by-column definitions for the clean file, including which fields were imputed or flagged. |

## The Cleaning Process

The raw extract is worked through in seven stages, each addressing a specific data-quality risk before the next stage builds on it:

1. **Load and inventory** : Every column is read as text first, so nothing gets silently misinterpreted before it's been inspected. A full fault inventory catalogs every issue found, by column and row count.

2. **Duplicates & keys** : Whitespace is trimmed, exact-duplicate rows are dropped, and customer IDs with disagreeing values are resolved using a stated, defensible rule (kept the record with the higher lifetime spend, since spend is cumulative and can't legitimately decrease).

3. **Types & formats** : Three different date layouts are parsed in a single pass; sentinel codes (`"unknown"`, `999`, `-999`, `"N/A"`) are converted to real missing values *before* any numeric conversion, so they can't silently distort a mean or a range.

4. **Categorical standardization** : Region, state, and channel labels are collapsed to their canonical forms; the newsletter opt-in flag is converted from eight-plus text encodings into a proper logical column.

5. **Range & logic validation** : Values are checked against actual business rules (age 18–100, non-negative income, at least one order), an extreme outlier is cleared rather than capped, and three cross-column contradictions are resolved (an order predating signup, a date after the extract date, a region that disagrees with its own state).

6. **Missing-data strategy** : Four different strategies for four different columns, chosen deliberately: region is *derived* from state (not a guess), age gets a simple median, income gets a region-aware median, and total spend — the actual outcome metric — is left missing and flagged rather than estimated.

7. **Verification & handoff** : Every rule claimed along the way is checked with real assertions, and a row-reconciliation ledger accounts for exactly where the dataset went from 205 rows to 200.

## Key Results

- **205 → 200 rows**: 3 exact duplicates removed, 2 conflicting customer records resolved down to one each.
- **3 date formats** (`YYYY-MM-DD`, `MM/DD/YYYY`, `DD-Mon-YYYY`) parsed correctly in a single pass, with zero parse failures.
- **1 extreme outlier** cleared: a $1.95M lifetime-spend value roughly 440× the next-highest figure, tied to a customer with only two orders.
- **7 rows** had a region that contradicted their own state and were corrected; **4 rows** had an order dated before signup; **3 rows** had a date after the extract cutoff; all resolved without deleting the customer record.
- Applying `drop_na()` would have thrown away **18% of the dataset** (36 of 200 rows). Instead, each column's missing values are handled on their own terms, and the outcome metric (`total_spent`) is never imputed.

## Reproducing This

1. Clone or download this repo.
2. Open `harborline-crm-data-cleaning.Rmd` in RStudio, with `Harborline_Customers_RAW.csv` in the same folder.
3. Click **Knit** (requires the `tidyverse` and `scales` packages).
4. The report renders to HTML, and the clean dataset + data dictionary are written to `output/`.

## Tools

R, tidyverse (`dplyr`, `stringr`, `lubridate`, `readr`), R Markdown / `knitr`.
