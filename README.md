#  Drug Delivery Analytics 

> *Decoding Nanoparticle Design for Precision Tumor Targeting*

##  Overview

This is a **comprehensive data analytics graduation project** developed as part of the **Digital Egypt Pioneers Initiative (DEPI) — Data Analysis Track**, under the supervision of **Eng. Mohamed Alemam**. It explores a curated dataset of nanoparticle biodistribution experiments drawn from published biomedical literature, investigating how physical design, surface chemistry, and targeting strategy affect tumor uptake and selectivity in cancer drug delivery.

The full pipeline was built in **Google Colab (Python)**, following the **Medallion Architecture (Bronze → Silver → Gold)**, and culminates in an interactive **Power BI dashboard** — first prototyped in **Figma** — designed to surface actionable insights for nanomedicine researchers and pharmaceutical developers.

**Key Focus Areas:**

- Nanoparticle physical design (size, shape, core material) vs. tumor uptake
- Zeta potential (surface charge) and its effect on selectivity
- Active vs. passive targeting strategy performance
- Tumor biology & organ-level biodistribution
- Surface coating / PEGylation & shell chemistry
- Time-point pharmacokinetics & dosage correlation
- Stimuli-responsive release mechanisms (pH, GSH, Heat, Light, Dual)
- Inorganic vs. organic nanoparticle comparison
- **AI Predictive Insights** — Tumor Retention & Targeting Selectivity models

---

## 👥 Project Team

| Name |
|---|
| Shahd Fekry Mohammed Shawky |
| Aalaa Moataz Ahmed Elsayed |
| Mohand Yasser Hussein Hanfy |
| Mohamed Mostafa Ahmed Mahmoud |
| Youssef Mohamed Saber Zedan |
| Yahya Abdel Nasser Abdel Naim |

**Supervisor:** Eng. Mohamed Alemam

---

##  Dataset

### Drug Delivery Dataset — Nanoparticle Biodistribution

| Attribute | Detail |
|---|---|
| **Dataset Name** | Supporting Information Excel 1.xlsx (Master_Sheet) |
| **Source** | [Zenodo Record 18327457](https://zenodo.org/records/18327457) |
| **Data Type** | Curated biomedical literature — experimental biodistribution records |
| **Total Records** | 2,374 experimental records (Gold layer) |
| **Unique Nanoparticles** | 84 distinct formulations |
| **Tumor Sites** | 12 distinct sites |
| **Organs Tracked** | 17 distinct organs/tissues |
| **Tumor Cell Lines** | 31 distinct cell lines |
| **Pipeline Stage** | Bronze → Silver → Gold → EDA (Python Analysis) |
| **Environment** | Google Colab |
| **File Format** | CSV (Gold Layer output) |

Key columns used throughout the analysis:

| Column | Type | Description |
|---|---|---|
| `NPs ID` | String | Unique identifier for each nanoparticle formulation |
| `NP_Class` | String | Organic or Inorganic |
| `Shape` | String | Physical shape (Sphere, Rod, Plate, etc.) |
| `Size (nm)` / `Size_Category` | Float / String | Particle diameter; binned Small / Optimal Range / Large |
| `Zeta Potential (mv)` / `Zeta_Category` | Float / String | Surface charge; Negative / Near Neutral / Positive |
| `Tumor Site` / `Tumor Cell` | String | Anatomical tumor location / cell line used |
| `Targeting Strategy` | String | Active or Passive |
| `HAS_PEG` / `Shell Type` | Boolean / String | PEGylation status and coating material |
| `Responsive release` | String | Stimulus-responsive release mechanism |
| `INPs_Core` | String | Core material for inorganic nanoparticles |
| `Time point (h)` | Integer | Hours post-administration at measurement |
| `Administration Dosages (mg/kg)` | Float | Dose per kg of body weight |
| `Tumor_%ID` / `Liver_%ID` / `Spleen_%ID` | Float | % of injected dose found in each tissue |
| `Selectivity_Index` | Float | Ratio of tumor uptake to liver uptake |
| `Targeting_Efficiency` | String | Ordinal tier: Very Poor → Exceptional |

---

##  Tools & Technologies

| Component | Technology | Purpose |
|---|---|---|
| **Raw Data Source** | Microsoft Excel | Original `Drug_Delivery_Dataset.xlsx` (Master_Sheet) |
| **Data Engineering** | Google Colab + Python (pandas) | End-to-end Medallion pipeline execution |
| **Data Architecture** | Medallion Architecture (Bronze → Silver → Gold) | Structured, analysis-ready pipeline |
| **Data Modeling** | Star Schema (1 Fact + 7 Dimensions) | Optimized relational design for Power BI |
| **EDA & Visualization** | Python — pandas, matplotlib, seaborn | 12-section exploratory analysis notebook |
| **Machine Learning** | Predictive models for tumor retention & selectivity | AI Predictive Insights dashboard page |
| **Dashboard Design** | Figma | UI/UX wireframing & layout planning before Power BI build |
| **Visualization / BI** | Power BI | 10-page interactive dashboard |
| **Cloud Alternative (future)** | Microsoft Fabric | Considered for scalable PySpark-based Medallion execution |

---

##  Architecture & Steps

### Step 1: Data Pipeline (Medallion Architecture)

```
Bronze Layer → Silver Layer → Gold Layer
  (Raw)          (Cleaned)      (Star Schema)
```

####  Bronze Layer — `01_Bronze_Layer_DrugDelivery_Colab.ipynb`

- Raw `Drug_Delivery_Dataset.xlsx` (Master_Sheet) read as-is via `pandas.read_excel`
- No transformation, no cleaning — captures the exact source state
- Output: `DrugDelivery_Bronze.csv` saved to Google Drive (`Bronze_Layer` folder)

####  Silver Layer — `02_Silver_Layer_DrugDelivery.ipynb`

Six targeted transformations applied to produce a clean, analysis-ready dataset:

| # | Action | Detail |
|---|---|---|
| 1 | Remove invalid records | Dropped 2 unreliable NP records (`INPS-P24`, `ONPS-P30`) |
| 2 | Impute missing Tumor Size (mm³) | Median imputation — robust to outliers |
| 3 | Drop Tumor selectivity column | Fully null, redundant for downstream analysis |
| 4 | Impute missing Administration Dosages | Random Forest regression (ML imputation) from size, shape, targeting strategy, etc. |
| 5 | Drop PDI column | High missing rate, not usable |
| 6 | Handle missing INPs_Core | Categorical fill — organic NPs filled with `"No_Core"` |
| 7 | Engineer `Time_Phase` feature | Early (0–24h) / Intermediate (24–96h) / Late (>96h) |

- Output: `DrugDelivery_Silver.csv` — **2,374 rows × 25 columns**

####  Gold Layer — `03_Gold_Layer_DrugDelivery.ipynb`

- Star schema built from the cleaned Silver dataset — **1 Fact table + 7 Dimension tables**
- Surrogate keys assigned to every dimension table for optimized joins
- Row counts and null-FK checks validated post-build
- Output: 8 CSV files exported to Google Drive, Power BI-ready

---

### Step 2: Data Modeling (Star Schema)

| Table | Type | Key Column(s) | Description |
|---|---|---|---|
| `Fact_Biodistribution` | Fact | `Fact_ID` (PK), 7 FK keys | All quantitative measures — Tumor_%ID, Liver_%ID, Spleen_%ID, Selectivity_Index |
| `Dim_Nanoparticle` | Dimension | `NP_Key` (PK) | NP_Class, INPs_Core, Shape, Size, Size_Category, Zeta Potential, HAS_PEG, Shell Type — 84 unique records |
| `Dim_Tumor` | Dimension | `Tumor_Key` (PK) | Tumor Cell, Tumor Site, Tumor Size (mm³) — 45 unique records |
| `Dim_Time` | Dimension | `Time_Key` (PK) | Time point (h) + Time_Phase — 29 unique time points |
| `Dim_Targeting_Strategy` | Dimension | `Targeting_Key` (PK) | Active or Passive — 2 unique values |
| `Dim_Organ` | Dimension | `Organ_Key` (PK) | 17 distinct organs/tissues |
| `Dim_Administration` | Dimension | `Administration_Key` (PK) | Administration Dosages (mg/kg) — 42 unique values |
| `Dim_Efficiency` | Dimension | `Efficiency_Key` (PK) | Ordinal ratings: Very Poor → Exceptional — 6 unique values |

---

### Step 3: Exploratory Data Analysis (EDA)

Performed entirely in Python (Google Colab) on the joined flat table (Fact + 7 Dimensions, sanity-checked for null FKs). Organized into 12 categories:

| # | Category | Focus |
|---|---|---|
| 0 | Setup | Load all 8 Gold tables, join into one analysis-ready flat table |
| 1 | Overview & Dataset Summary | KPI cards: counts, averages, key ratios |
| 2 | Nanoparticle Physical Design | Size, Zeta potential, Shape, Core material |
| 3 | Targeting Strategy Performance | Active vs. Passive |
| 4 | Tumor Biology & Organ Targeting | Tumor sites, cell lines, organ biodistribution |
| 5 | Surface Coating & Shell Analysis | Shell Type, PEGylation |
| 6 | Time Point & Dosage Analysis | Temporal dynamics, dosage relationships |
| 7 | Responsive Release Mechanisms | Stimulus-responsive release behavior |
| 8 | Selectivity & Efficiency Performance | Selectivity_Index, Targeting_Efficiency |
| 9 | Inorganic vs. Organic NPs Comparison | NP_Class comparison |
| 10 | Cross-Dimensional & Advanced Insights | Multi-factor combinations |
| 11 | Data Quality & Coverage | Remaining nulls, representation imbalance |

> **Note:** Zeta Potential has a 25.7% missing-value rate — all Zeta-related charts are based on the 1,763 records with available values, excluding the 611 missing records for that section only.

---

##  Power BI Dashboard — 10 Pages

Built directly on the Gold Layer star schema, following the exact analytical flow of the EDA notebook. First prototyped in **[Figma](https://www.figma.com/design/CQiLykbYdeZVe9vt1HBnZS/Drug-Delivery?node-id=0-1&t=RT0lO9uKsoaP39fH-1)** to define layout, navigation, and color scheme before implementation.

| Page | Title | Theme / Focus |
|---|---|---|
| 1 | Overview | Dataset Summary & KPI Snapshot |
| 2 | Nanoparticle Physical Design Properties | Size, Shape & Core Material vs. Tumor Uptake |
| 3 | Zeta Potential — Dedicated Analysis | Surface Charge & Targeting Performance |
| 4 | Targeting Strategy | Active vs. Passive Deep-Dive |
| 5 | Tumor Biology & Organ Targeting | Organ & Tumor Site Biodistribution |
| 6 | Surface Coating & Shell Analysis (PEG) | PEGylation & Shell Chemistry |
| 7 | Time Point & Dosage Analysis | Pharmacokinetics Over Time |
| 8 | Responsive Release Mechanisms | Stimuli-Responsive Release Performance |
| 9 | Inorganic vs. Organic NPs Comparison | NP Class Head-to-Head Benchmark |
| 10 | AI Predictive Insights | Tumor Retention & Selectivity ML Models |

**Design language:** dark, near-black canvas with glowing green molecular imagery; fixed left-hand navigation rail; consistent single-hue green gradients across bar charts and heatmaps — chosen to reinforce the nanoparticle research domain while keeping dense cross-dimensional visuals legible.

---

##  Key KPIs & Headline Insights

### Overview
| KPI | Value |
|---|---|
| Unique Nanoparticles (NPs ID) | 84 |
| Unique Tumor Sites | 12 |
| Average Tumor %ID | 1.31 |
| Average Liver %ID | 17.16 |
| Average Selectivity Index | 0.30 |
| Targeting Strategy Split | Passive 86.27% / Active 13.73% |
| NP_Class Split | Inorganic 57.92% / Organic 42.08% |
| PEGylated Formulations | 55.01% |
| Most Represented Tumor Site | Breast (684 records); least: Sarcoma (44) |
| Targeting Efficiency | Poor (631) & Moderate (573) dominate; Exceptional rare (88) |

### Physical Design
| KPI | Value |
|---|---|
| Small Size % | 76.14% of formulations |
| Optimal Range Size % | 12.50% |
| Large Size % | 11.36% |
| Highest Uptake by Size | Small (1.73) vs. Large (0.80) |
| Highest Uptake by Shape | Rod (1.68) vs. Plate (0.45) |
| Dominant Inorganic Core | Gold (975 records) |

### Zeta Potential
| KPI | Value |
|---|---|
| Near Neutral Zeta % | 56.21% |
| Highest Tumor %ID by Zeta | Near Neutral (1.41) vs. High Negative (0.82) |
| Highest Selectivity Index | High Negative (1.49) — far above Negative (0.36), Near Neutral (0.29), Positive (0.05) |

### Targeting Strategy
| KPI | Value |
|---|---|
| Avg Tumor %ID — Passive | 1.42 vs. Active 0.65 (more than double) |
| Avg Selectivity Index — Passive | 0.33 vs. Active 0.09 |
| Active Targeting Efficiency | Skews toward Poor/Very Poor (63.8% combined) |

### Tumor Biology
| KPI | Value |
|---|---|
| Unique Organs Measured | 17 |
| Unique Tumor Cell Lines | 31 |
| Top Tumor Site by Uptake | Ovary (3.2), Sarcoma (2.6), Prostate (2.2) |
| Top Tumor Site by Selectivity | Skin (0.96) |
| Most Represented Cell Line | 4T1 (355 records) |
| Top Off-Target Organs | Liver (16), Blood (15) |
| Liver_%ID vs Selectivity_Index | r = -0.243 (weak negative correlation) |

### Surface Coating & PEG
| KPI | Value |
|---|---|
| PEG % | 55.01% |
| No Stealth Effect % | 29.57% |
| HPMA % / Cellulose % | 4.84% / 2.61% |
| Highest Selectivity by Shell | HPMA (1.4) vs. PEG (0.2) |
| Highest Uptake by Shell | HPMA (3.2), Cellulose (2.3) |
| Non-PEGylated vs. PEGylated Selectivity | 0.40 vs. 0.21 |

### Time Point & Dosage
| KPI | Value |
|---|---|
| Peak Tumor %ID (time series) | 15.00 (mid-timeline) |
| Early (≤6h) vs. Late (≥24h) Uptake | 1.40 vs. 1.35 (near-identical) |
| Dosage vs. Tumor_%ID Correlation | r = 0.019 (essentially none) |
| Dosage vs. Liver_%ID Correlation | r = -0.234 (weak negative) |

### Responsive Release
| KPI | Value |
|---|---|
| No Trigger Records | 1,529 (majority) |
| Leading Trigger Type | pH-responsive (539) |
| Highest Selectivity | No Trigger (0.37) — ahead of Light (0.28), GSH (0.26) |
| Highest Combo | No-Trigger + Passive targeting → 0.39 selectivity (matrix peak) |

### Inorganic vs. Organic
| KPI | Value |
|---|---|
| Avg Selectivity Index | Organic 0.32 vs. Inorganic 0.28 |
| Avg Liver_%ID | Inorganic 21 vs. Organic 12 |
| Avg Spleen_%ID | Inorganic 3.0 vs. Organic 1.3 |
| Shared Top Tumor Site | Breast (411 Inorganic / 273 Organic) |
| Organic-Dominant Site | Ovary (169 Organic vs. 81 Inorganic) |

---

##  AI Predictive Insights

Two supervised models were built to predict nanoparticle biodistribution behavior from design features:

| Model | Accuracy | Precision | Recall | F1-Score |
|---|---|---|---|---|
| **Tumor Retention Model** | 62.1% | 63.7% | 60.4% | 61.8% |
| **Targeting Selectivity Model** | 59.4% | 60.9% | 55.1% | 55.8% |

**Top drivers of tumor uptake:** Size level, Shape, Administration dosage, Zeta potential, NP class
**Top drivers of selectivity:** Tumor site, NP class, Zeta potential, Time point, Shape

> Moderate accuracy (~60%) reflects the inherent biological variance of *in vivo* tumor delivery data — consistent with the broader nanomedicine literature.

---

##  SMART Research Questions

| # | Question | KPIs Measured |
|---|---|---|
| 1 | What are the overall biodistribution averages, and how many unique nanoparticles/tumor sites does the dataset cover? | Avg Tumor_%ID, Avg Liver_%ID, Total NPs, Total Tumor Sites |
| 2 | Which nanoparticle size, shape, and core material result in the highest tumor uptake? | Tumor_%ID, Size Category, Shape, INPs_Core |
| 3 | Does Zeta Potential category affect tumor uptake and selectivity? | Zeta Category, Tumor_%ID, Selectivity Index |
| 4 | Does active targeting deliver more drug to tumors than passive targeting? | Targeting Strategy, Tumor_%ID, Targeting Efficiency |
| 5 | Which tumor sites/cell lines accumulate the most drug, and which organs show highest biodistribution? | Tumor Site, Tumor Cell, Organ Biodistribution |
| 6 | Does PEGylation improve tumor uptake and selectivity vs. non-PEGylated NPs? | HAS_PEG, Shell Type, Tumor_%ID, Selectivity Index |
| 7 | At which time point does tumor uptake peak, and does higher dosage improve delivery? | Time Point (h), Tumor_%ID, Dosage (mg/kg) |
| 8 | Which responsive release mechanism leads to the best tumor selectivity? | Responsive Release, Selectivity Index, Targeting Efficiency |
| 9 | Do inorganic nanoparticles outperform organic ones in selectivity and liver accumulation? | NP Class, Selectivity Index, Liver_%ID, Tumor Site |

---

##  Project File Structure

```
Drug_Delivery_Project/
├── Medallion_Architecture/
│   ├── Bronze/
│   │   └── 01_Bronze_Layer_DrugDelivery_Colab.ipynb   # Raw Excel ingestion → CSV
│   ├── Silver/
│   │   └── 02_Silver_Layer_DrugDelivery.ipynb         # Cleaning + Feature Engineering
│   └── Gold/
│       └── 03_Gold_Layer_DrugDelivery.ipynb           # Star schema (1 Fact + 7 Dims)
├── EDA/
│   └── Drug_Delivery_EDA.ipynb                        # 12-section Python EDA
├── Dashboard/
│   ├── Drug_Delivery_Dashboard.pbix                   # Power BI dashboard (10 pages)
│   └── Figma_Prototype_Link.md                        # Figma design reference
├── Documentation/
│   └── Drug_Delivery_Documentation.pdf                # Full project documentation
└── README.md
```

---

##  How to Run

### Prerequisites
- Google Colab (or local Jupyter) with Python 3 + `pandas`, `matplotlib`, `seaborn`, `scikit-learn`
- Power BI Desktop
- Google Drive access (pipeline storage backend)

### Execution Steps

**Step 1 — Run Bronze Layer**
```bash
# Open in Google Colab
Medallion_Architecture/Bronze/01_Bronze_Layer_DrugDelivery_Colab.ipynb
```
Reads the raw `Drug_Delivery_Dataset.xlsx` and exports `DrugDelivery_Bronze.csv`.

**Step 2 — Run Silver Layer**
```bash
Medallion_Architecture/Silver/02_Silver_Layer_DrugDelivery.ipynb
```
Applies cleaning, imputation, and `Time_Phase` feature engineering → `DrugDelivery_Silver.csv` (2,374 × 25).

**Step 3 — Run Gold Layer**
```bash
Medallion_Architecture/Gold/03_Gold_Layer_DrugDelivery.ipynb
```
Builds the star schema (1 Fact + 7 Dimensions) → 8 CSV files, Power BI-ready.

**Step 4 — Run EDA Notebook**
```bash
EDA/Drug_Delivery_EDA.ipynb
```
Produces the full 12-section exploratory analysis.

**Step 5 — Open Power BI Dashboard**
```
# Connect Power BI Desktop to the Gold layer CSV tables
# Open the .pbix file and refresh the data source
```

---

##  Key Findings & Insights

- **Passive targeting outperforms active targeting** in both tumor uptake (1.42 vs. 0.65) and selectivity (0.33 vs. 0.09) — a counterintuitive but consistent pattern across the dataset.
- **Near-neutral zeta potential** achieves the highest tumor uptake, while **high-negative zeta** particles — despite lower uptake — show a striking Selectivity Index of 1.49.
- **HPMA shell coating** delivers both the highest selectivity (1.4) and tumor uptake (3.2) of any shell type, outperforming standard PEG.
- **PEGylation** shows a nuanced trade-off: non-PEGylated particles actually show higher mean tumor uptake and nearly double the selectivity of PEGylated ones — challenging the assumption that PEG universally improves targeting.
- **Dosage has essentially no correlation** with tumor uptake (r = 0.019), suggesting delivery efficiency is driven by design, not dose.
- **No-trigger (non-responsive) release mechanisms** unexpectedly outperform all stimuli-responsive alternatives in average selectivity.
- **Organic NPs edge out inorganic NPs in selectivity** (0.32 vs. 0.28), while inorganic NPs accumulate nearly 2× more in the liver and spleen — a clear accumulation/specificity trade-off.
- **AI models** achieve ~60% accuracy predicting tumor retention and selectivity, with size, shape, zeta potential, and tumor site emerging as the dominant predictive drivers.

---

##  Learning Outcomes

| Skill | Application |
|---|---|
| **Data Pipeline Design** | Medallion Architecture (Bronze → Silver → Gold) in Google Colab |
| **Data Modeling** | Star schema with 1 fact table and 7 dimension tables |
| **Data Cleaning** | Median & Random Forest (ML) imputation, invalid record removal, feature engineering |
| **Exploratory Data Analysis** | 12-section Python EDA with matplotlib/seaborn across 84 nanoparticles |
| **Machine Learning** | Predictive modeling for tumor retention & targeting selectivity |
| **Data Visualization** | 10-page interactive Power BI dashboard with slicers, heatmaps, and KPI cards |
| **Dashboard Design** | Figma wireframing for layout, color palette, and storytelling flow |
| **Domain Translation** | Turning biomedical literature data into an analysis-ready dimensional model |

---

##  Data Source & Attribution

**Dataset:** Supporting Information Excel 1.xlsx — curated nanoparticle biodistribution data from published biomedical literature, hosted on [Zenodo (Record 18327457)](https://zenodo.org/records/18327457)

**Tools:** Google Colab (Python/pandas) · Power BI · Figma · Microsoft Excel

**Initiative:** Digital Egypt Pioneers Initiative (DEPI) — Data Analysis Track, in partnership with eYouth and the Egyptian Ministry of Communications and Information Technology

---

##  Summary

This **Drug Delivery Analytics — Nanoparticle Biodistribution Dashboard** delivers:

 **Complete 3-Layer Data Pipeline** — Bronze, Silver, Gold via Python notebooks in Google Colab
 **10 Interactive Power BI Dashboard Pages** — Overview through AI Predictive Insights
 **2,374 Biodistribution Records** across 84 nanoparticle formulations, 12 tumor sites, and 17 organs
 **Figma-First Design Process** — full UI/UX prototype before Power BI implementation
 **AI Predictive Layer** — Tumor Retention & Targeting Selectivity models (~60% accuracy)
 **9 SMART Research Questions** answered end-to-end, from raw Excel to actionable dashboard insight
