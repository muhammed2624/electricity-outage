# Voltix data: what's real, what changed, what's still missing

## What's new

`data/processed/discos_recent_quarterly.csv` — 66 rows: 11 DisCos x 6 quarters
(2023/Q3, 2023/Q4, 2024/Q1, 2024/Q2, 2024/Q3, 2024/Q4), each with Energy
Received (GWh), Billing Efficiency (%), and ATC&C Loss (%). Every number is
transcribed directly from NERC's own published quarterly reports (nerc.gov.ng)
— not estimated, not interpolated, not fabricated. Sources are cited per-row in
the `source_report` column and listed in full at the top of
`src/build_recent_quarterly.py`.

This closes most of the gap between the original training data (ends Sep 2022)
and today (mid-2026), with the exceptions below.

## Why this isn't merged into the training set

The original `master_discos_monthly.csv` is **monthly**. NERC no longer
publishes that monthly workbook format — its quarterly reports give the same
metrics but per **quarter**. A 3-month trailing z-score (the core of the proxy
label) means something different computed on quarters than on months, so
mixing the two granularities into one training matrix would quietly change
what "recent trend" means for part of the data without saying so.

Instead, this is a **separate, clearly-labeled table** you can use for:
- The app's evaluation/context panel — showing where each DisCo stands most
  recently, alongside (not blended with) the model's Sep-2022-trained
  predictions.
- The capstone report's Scope & Limitations section — you can now say
  precisely how stale the training data is *and* show real, current numbers
  side by side, which is a stronger, more honest position than either hiding
  the gap or silently patching over it.
- Future work: retraining a second, quarterly-cadence model once enough
  quarters accumulate to support a trailing window (needs roughly 8+ quarters
  of consistent reporting to do this properly — you have 5 real quarters now
  plus whatever NERC publishes going forward).

## Known remaining gap — say this explicitly in the report

**2022/Q4 through 2023/Q2** (three quarters) are not covered. The NERC
quarterly reports for that period exist publicly but weren't pulled in this
pass. **2024/Q4 onward** is also not yet covered — NERC's most recent
published quarterly report at the time of writing was 2024/Q3.

Two honest ways to close this further:
1. Pull the missing NERC quarterly PDFs (2022/Q4, 2023/Q1, 2023/Q2, and
   anything published after 2024/Q3) and extend `build_recent_quarterly.py`
   the same way — same table structure, same per-row sourcing.
2. Check nerc.gov.ng's Resources page directly for the newest report before
   the investor pitch, since NERC publishes on a lag and a newer quarter may
   already be out.

## What did NOT change

- The core monthly training data, the proxy label logic, and the trained
  model are untouched — this is additive, not a replacement.
- No synthetic or interpolated numbers were added anywhere. Every value in
  `discos_recent_quarterly.csv` traces back to a specific NERC table, cited
  by row.
