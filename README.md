# CPS Labor Force Validation Study

Reproducing BLS labor force statistics from IPUMS Current Population Survey microdata and validating against published benchmarks.

---

## Overview

This project validates that I can compute official U.S. labor force statistics (unemployment rate, labor force participation, employment-population ratio) using CPS microdata before scaling to custom demographic analysis. The validation exercise covers:

- **National estimates** (civilian noninstitutional population, age 16+)
- **Veteran and nonveteran estimates** (age 18+, split by VETSTAT)
- **Survey weight selection** for different population definitions

**Completion Date:** October 2026  
**Status:** ✅ Complete — Ready to extend to 2020–2026 time series

---

## Objectives

1. Reproduce labor force statistics that BLS publishes using person-level CPS microdata from IPUMS, before using the data for further analysis
2. Confirm which survey weight variable is needed for each population (civilian, veteran, nonveteran)
3. Document methodological decisions for future reproducibility

---

## Data and Sources

| Source | Description | Notes |
|--------|-------------|-------|
| **IPUMS CPS** | Basic monthly extract, March 2024 (27 variables) | Data not included in this repository per IPUMS terms; instructions below |
| **BLS Benchmarks** | Not seasonally adjusted figures from Table A-1 and veteran series reports | Retrieved 10/13/26; BLS revises SA series each January |

**Universe Definitions:**
- **National:** Civilian noninstitutional population, age 16+
- **Veteran status:** Civilian noninstitutional population, age 18+ (split by VETSTAT: 1=No service, 2=Yes)

---

## Key Finding: Weight Selection Matters

| Population | Best Weight | Validation Result |
|------------|-------------|-------------------|
| **National (age 16+)** | `COMPWT` | Within 0.01 pp of BLS unemployment rate |
| **Veterans (age 18+)** | `WTVET` | Within 0.2% of BLS levels (except unemployed +1.4%) |
| **Nonveterans (age 18+)** | `WTVET` | Within 0.06% of BLS levels |

Using the standard household weight (`WTFINL`) produced estimates **7–11% below BLS for veterans**, even though the unemployment rate matched due to ratio cancellation of errors.

---

## Results

### National (Headline) Estimates — Age 16+

| Measure | This Analysis (COMPWT) | BLS (NSA) | Difference |
|---------|------------------------|-----------|------------|
| Population | 267,884,190 | 267,884K | 0.0% |
| Labor Force | 167,960,219 | 167,960K | 0.0% |
| Unemployed | 6,603,802 | 6,604K | -0.03% |
| **Unemployment Rate** | **3.93%** | **3.9%** | **-0.01 pp** |

**✅ Validation Successful:** Using composite weight (`COMPWT`) reproduces BLS figures within rounding tolerance.

---

### Veterans and Nonveterans — Age 18+

| Weight | Veterans Population | Veterans LF | Nonveterans Pop | Nonveterans LF |
|--------|---------------------|-------------|-----------------|----------------|
| `WTFINL` | -6.77% | -10.73% | +0.54% | +1.13% |
| `COMPWT` | -6.74% | -11.09% | +0.52% | +0.61% |
| `WTVET` | **+0.00%** | **-0.13%** | **+0.03%** | **+0.02%** |

*Abbreviations: pop = civilian noninstitutional population; LF = labor force (employed + unemployed)*

**Conclusion:** `WTVET` (veteran weight) is required for veteran-status subgroups. Standard weights substantially understate veteran labor force counts.

---

## Methodology Notes

### Weight Choice Depends on Population
- `COMPWT` reproduces national estimates (civilian, age 16+)
- `WTVET` (controlled to VA veteran totals) reproduces veteran and nonveteran figures
- Never assume one weight works for all populations

### Seasonal Adjustment
- Microdata match BLS's **not seasonally adjusted (NSA)** series by construction
- Seasonally adjusted figures are **not a valid benchmark** for a single month of microdata
- Record retrieval dates for every benchmark (BLS revisions occur each January)

### Levels vs. Rates
- **Rates alone can mislead:** Veteran unemployment rate under `WTFINL` matched BLS (3.0%) even though every level was 7–11% too low, because weight errors cancel in a ratio
- **Population totals are the control check:** Matching the population confirms the universe and weights before any labor force statistic is examined
- Note: WTVET veteran population matches BLS by construction (it is the control total), so it is not an independent test

---

## Open Items

| Item | Status | Next Step |
|------|--------|-----------|
| Nonveteran population under WTVET is ~66K (0.03%) above BLS | Under investigation | Check codebook for NIU exclusions |
| Small remaining gaps in unemployed counts (veterans +1.4%, nonveterans +0.06%) | Minor, within tolerance | Document as acceptable variance |

---

## Reproducing the Extract

To recreate the IPUMS extract used in this analysis:

1. Visit [IPUMS CPS Extract Builder](https://cps.ipums.org/cps-action/select/variables/)
2. Select samples: **March 2024** (Basic Monthly)
3. Variables: `YEAR`, `MONTH`, `SAMPLE`, `CPSIDP`, `SERIAL`, `PERNUM`, `AGE`, `SEX`, `RACE`, `HISPAN`, `EDUC`, `MARST`, `VETSTAT`, `LABFORCE`, `EMPSTAT`, `WHYUNEMP`, `DURUNEMP`, `POPSTAT`, `WTFINL`, `COMPWT`, `WTVET`
4. Format: CSV or fixed-width
5. Download and place in `data/raw/` (not committed to Git)

---

## Repository Structure
cps-labor-force/
├── notebooks/
│   └── validate_bls_march_2024.ipynb
├── data/
│   └── raw/                           # .gitignored
├── README.md                          # Human-readable documentation
├── requirements.txt                   # Machine-readable dependencies
└── .gitignore                         # Excludes data files

---

## Technologies Used

- **Python** 3.13
- **pandas** 3.0.6 — Data manipulation
- **Jupyter** — Exploratory analysis and documentation

---

## Author

**Stephanie Chan, PhD** - Public Policy Researcher & Data Analyst  
Former Senior Research Analyst, U.S. Department of Labor  
[Github Profile] ( https://github.com/stephanie-s-chan)
---

## License

MIT License — Free for personal and educational use. For commercial licensing, contact author.