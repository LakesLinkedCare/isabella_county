# Lakes Linked Care — Michigan Rural Health Burden Analysis

**County-level DALY estimates for rural Michigan, built on open public data and a reproducible Python pipeline.**

**Live site:** https://lakeslinkedcare.github.io/isabella_county/  
**Contact:** Sergey Soshnikov, MD PhD · biososh@gmail.com · Central Michigan University  
**Status:** Active — Isabella, Clare, and Midland counties complete; Gratiot, Mecosta, Washtenaw in pipeline (2026)

---

## What This Is

This repository contains a reproducible Python pipeline and GitHub Pages website for computing planning-grade Disability-Adjusted Life Year (DALY) estimates for rural Michigan counties using exclusively open, public data sources (CDC PLACES, CDC WONDER, IHME GBD 2021, County Health Rankings, U.S. Census ACS).

The analysis is designed for:
- County Health Needs Assessment (CHNA) processes
- Grant application development (SAMHSA SOR, HRSA Rural Health, MDHHS, NIH R21)
- Academic collaboration and publication
- Public health education and planning

**These are planning-grade estimates, not formal GBD county burden estimates.** See [methods_supplement.html](methods_supplement.html) for full methodology, limitations, and citation guidance.

---

## Counties

| County | FIPS | DALYs/yr (frontier LE) | Status | Dashboard |
|--------|------|------------------------|--------|-----------|
| Isabella | 26073 | 15,140 (95% UI 14,032–16,206) | ✅ Complete v5.0 | [isabella.html](isabella.html) |
| Clare | 26035 | 8,198 (±20% uncertainty) | ✅ Complete v1.0 | [clare.html](clare.html) |
| Midland | 26111 | ~20,700 (derived est.) | ✅ Complete v1.0 | [midland.html](midland.html) |
| Gratiot | 26057 | — | 🔄 Q3 2026 | — |
| Mecosta | 26107 | — | 🔄 Q4 2026 | — |
| Washtenaw | 26161 | — | 🔄 Q4 2026 (urban reference) | — |

---

## Repository Structure

```
isabella_county/
│
├── index.html                          # Multi-county portal (landing page)
├── isabella.html                           # Isabella County dashboard
├── clare.html                          # Clare County dashboard
├── isabella_trends.html                         # Longitudinal trends page
│
├── methods_supplement.html             # Shared DALY framework + Python pipeline docs
├── isabella_methods.html  # Isabella-specific data quality tables
├── clare_methods.html                  # Clare-specific data quality tables
│
├── isabella_roi.html                            # Isabella Economic Impact analysis
├── clare_roi.html                               # Clare Economic Impact analysis
├── clare_ai_solutions.html             # Clare Interventions page
│
│── director.py                         # Pipeline orchestrator (entry point)
├── harvester.py                        # Data collection from 5 open APIs
├── daly_calculator.py                  # DALY computation + Monte Carlo
├── fact_checker.py                     # Math/epi/statistical validation
├── chatgpt_reviewer.py                 # GPT-4o peer review (requires OPENAI_API_KEY)
├── scientific_writer.py                # Narrative text generation
├── naghavi_age_sensitivity.py          # Age-banded YLL sensitivity analysis
│
├── isabella_county_config.json         # Computed Isabella estimates (pipeline output)
├── isabella_county_harvest.json        # Raw Isabella collected data (pipeline output)
├── clare_county_config.json            # Computed Clare estimates (pipeline output)
├── clare_county_harvest.json           # Raw Clare collected data (pipeline output)
│
├── isabella_fact_check_report.md/.json # Current Isabella fact check
├── isabella_review_report.md/.json     # Current Isabella peer review
├── isabella_written_content.md/.json   # Generated narrative content
├── pipeline_log.json                   # Timestamped pipeline run log
│
├── requirements.txt                    # Python dependencies
├── .gitignore
│
└── data/
    ├── README.md                       # Data catalogue + reproduction guide
    ├── places_2020_isabella.csv        # CDC PLACES by release year (2020–2025)
    ├── places_2021_isabella.csv
    ├── places_2022_isabella.csv
    ├── places_2023_isabella.csv
    ├── places_2024_isabella.csv        # Primary cross-sectional year
    ├── places_2025_isabella.csv
    ├── chr_isabella_all_years.csv      # County Health Rankings 2010–2024
    ├── cms_medicare_isabella.csv       # CMS Medicare utilization 2021
    ├── census_insurance_isabella.csv   # ACS insurance coverage 2022
    ├── usda_food_atlas_isabella.csv    # USDA Food Environment Atlas 2020
    ├── daly_longitudinal_v2.csv        # 7 conditions × 7 years (2017–2023)
    ├── daly_2026_projection_v2.csv     # OLS trend projection with 95% PI
    ├── isabella_combined_trends.csv    # Wide-format merged dataset
    ├── isabella_data_inventory.csv     # Source metadata catalogue
    ├── longitudinal_daly_v2.py         # Longitudinal model script
    └── build_combined.py               # Data merge script
```

---

## Quick Start

```bash
# Clone
git clone https://github.com/lakeslinkedcare/isabella_county
cd isabella_county

# Install dependencies
pip install -r requirements.txt

# Add OpenAI API key (only needed for chatgpt_reviewer.py step)
echo "OPENAI_API_KEY=sk-..." > .env

# Run pipeline for Isabella County
python3 director.py --county isabella --steps harvest,calculate,factcheck

# Run pipeline for Clare County
python3 director.py --county clare --steps harvest,calculate,factcheck

# Run pipeline for Midland County
python3 director.py --county midland --steps harvest,calculate,factcheck

# Regenerate longitudinal model
cd data && python3 longitudinal_daly_v2.py
```

---

## Pipeline Architecture

```
director.py
    │
    ├─→ 1. harvester.py          → *_county_harvest.json
    ├─→ 2. daly_calculator.py    → *_county_config.json + data/daly_longitudinal_v2.csv
    ├─→ 3. fact_checker.py       → *_fact_check_report.md/.json
    ├─→ 4. chatgpt_reviewer.py   → *_review_report.md/.json
    ├─→ 5. html_generator.py     → *.html dashboards            [planned Q3 2026]
    └─→ 6. deploy.py             → GitHub Pages push             [planned Q3 2026]
```

Full pipeline documentation: [methods_supplement.html → Section 2](methods_supplement.html)

---

## Data Sources

| Source | Version | Used for |
|--------|---------|----------|
| [CDC PLACES](https://www.cdc.gov/places) | 2024 (2022 BRFSS) | YLD prevalence inputs |
| [CDC WONDER](https://wonder.cdc.gov) | 2020–22 (3-yr pooled) | YLL mortality counts |
| [IHME GBD 2021](https://ghdx.healthdata.org) | 2021 | Disability weights + frontier LE |
| [County Health Rankings](https://countyhealthrankings.org) | 2010–2024 | YPLL trend scaling |
| [U.S. Census ACS](https://census.gov) | 2022 5-yr | Population denominators, SDOH |
| [USDA Food Atlas](https://ers.usda.gov) | 2020 | Food security SDOH context |
| [CMS Medicare GVF](https://data.cms.gov) | 2021 | Healthcare utilization context |

All sources are publicly available at no cost. See [data/README.md](data/README.md) for download URLs, API endpoints, and what to commit vs. download at runtime.

---

## DALY Methodology Summary

**DALY = YLL + YLD**

- **YLL** (Years of Life Lost): CDC WONDER mortality × rural adjustment factor × (frontier LE 89.1 yrs − mean age at death)
- **YLD** (Years Lived with Disability): CDC PLACES prevalence × IHME GBD 2021 disability weight
- **Uncertainty:** Monte Carlo n=10,000 (Poisson for mortality, Beta for prevalence and DW, Uniform for rural adjustment)
- **Two LE standards reported:** WHO GHE frontier 89.1 yrs (primary) and Michigan observed 78.6 yrs (planning sensitivity)

Full equations, parameter tables, and limitation disclosure: [methods_supplement.html](methods_supplement.html)

---

## Key Results (June 2026)

**Isabella County (FIPS 26073, pop. 64,565)**  
Total burden: **15,140 DALYs/yr** (95% UI 14,032–16,206), frontier LE  
Top conditions: Cancer #1 · CVD #2 · SUD #3 (frontier standard)  
Economic burden: $1.03B/yr (VSL method) · $939M/yr (Human Capital)

**Clare County (FIPS 26035, pop. 30,013)**  
Total burden: **8,198 DALYs/yr** (±20% uncertainty), frontier LE  
Top conditions: Cancer #1 · CVD #2 · SUD #3 (frontier standard)  
Designation: Medically Underserved Area (MUA); opioid mortality ~47/100k

**Midland County (FIPS 26111, pop. 82,884)**  
Total burden: **15,837 DALYs/yr** (MI LE) / ~20,700 (frontier LE) — derived prevalence estimates  
Top conditions: Cancer #1 · Mental Health #2 · SUD #3  
Economic burden: $982M/yr (HC) · $7.2B/yr (VSL) · Dow Chemical industrial economy context  
⚠️ Prevalence values derived from Michigan state rates — direct CDC PLACES pull pending

---

## Citation

> Soshnikov S. County-Level Disease Burden Analysis in Rural Michigan: DALY Framework, Data Sources, and Reproducible Python Pipeline. *Lakes Linked Care*, 2026. Available at: https://lakeslinkedcare.github.io/isabella_county/

---

## License

Code: MIT License  
Data outputs: CC BY 4.0 (with attribution to original sources — CDC, IHME, RWJ Foundation, U.S. Census Bureau, USDA)  
Raw source data is governed by the respective agencies' terms of service.

---

*Sergey Soshnikov, MD PhD · Central Michigan University · June 2026*
