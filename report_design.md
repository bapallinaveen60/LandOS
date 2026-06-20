# LND-11: Automated PDF Report Design Specification

---

## 1. Document Layout, Styling & Typography

To ensure that the reports are institutional grade and suitable for banking boards or international developers, the PDF generator follows strict layout rules using a Python-based reporting engine (WeasyPrint / ReportLab with CSS Paged Media):

* **Page Geometry:** Standard A4 size with $15\text{mm}$ margins.
* **Color System:**
  * Primary: Deep Navy Slate (`#0B192C`) - Cover pages, headers.
  * Secondary: Cobalt Blue (`#1E3E62`) - Accents, section lines.
  * Accent Red (High Risk): Carmine (`#D9534F`) - Risk warnings.
  * Accent Green (Low Risk): Emerald (`#2ECC71`) - Approval indicators.
  * Backgrounds: Warm Off-White (`#F9F9F9`) - Content container panels.
* **Typography:** 
  * Font Family: `Outfit` (Headers) and `Inter` (Body Text) loaded from Google Fonts.
  * Headings: Bold, primary color, size hierarchy: H1 = $24\text{pt}$, H2 = $16\text{pt}$, H3 = $12\text{pt}$.
  * Body text: $10\text{pt}$ Charcoal (`#333333`) with a $1.4$ line height.
* **Header/Footer Rules:**
  * Header: Displays "LND-11 Climate Risk Assessment" left-aligned, and the organization logo right-aligned in a $7\text{pt}$ muted grey font.
  * Footer: Displays "CONFIDENTIAL - [Date]" left-aligned, and "Page X of Y" right-aligned. The cover page suppresses all headers and footers.

---

## 2. 15-Page Report Section Structure

```
+-----------------------------------------------------------------------------+
|                               REPORT SECTIONS                               |
+-----------------------------------------------------------------------------+
|  [Page 1] Cover Page (Title, Map Centroid, Investment Grade Badge)          |
|  [Page 2] Section 1: Executive Summary (Key Metrics, Critical Warnings)     |
|  [Page 3] Section 2: Parcel Overview & Location Details                     |
|  [Page 4] Section 3: Topographic Analysis (DEM, Slope, Curvature Profiles)  |
|  [Page 5] Section 4: Hydrological Risk Modeling (Runoff, Basin Boundaries)  |
|  [Page 6] Section 5: Meteorology & Climate Baseline (Rain/Temp Trends)      |
|  [Page 7] Section 6: Flood Vulnerability Assessment (SAR backscatter maps) |
|  [Page 8] Section 7: Drought & Groundwater Assessment                       |
|  [Page 9] Section 8: Thermal Extremes & Heatwave Projections                 |
|  [Page 10] Section 9: Future Climate Forecasts (CMIP6: 2030, 2050, 2080)    |
|  [Page 11] Section 10: Environmental Constraints (Highways, Protected Area) |
|  [Page 12] Section 11: Soil Properties & Foundation Suitability             |
|  [Page 13] Section 12: ESG Impact & Carbon Stocks                           |
|  [Page 14] Section 13: Underwriting Recommendations & Mitigation Strategy   |
|  [Page 15] Section 14: Data Inventory, Citations, and Methodological Notes |
+-----------------------------------------------------------------------------+
```

### Detailed Section Definitions

* **Page 1: Cover Page**
  * Displays a full-bleed dark navy background.
  * Prominent text: "LND-11 CLIMATE INTELLIGENCE REPORT".
  * Sub-text: "Parcel Assessment: Survey No. 45, Khatkar, Rajasthan".
  * Floating badge: **INVESTMENT GRADE: AA** (rendered in green/gold depending on the rating).
  * High-resolution satellite thumbnail of the parcel polygon boundary centered.
* **Page 2: Executive Summary**
  * Three horizontal columns displaying:
    1. **Climate Risk Score:** $38/100$ (with color-coded gauge).
    2. **Water Security Score:** $42/100$ (with water droplet indicator).
    3. **Development Suitability:** $78/100$ (with crane/terrain indicator).
  * **Critical Flag Box:** Red border box displaying high-priority alerts:
    > **[!] CRITICAL WARNING:** Groundwater extraction in Jodhpur district is classified as "OVER-EXPLOITED" by CGWB. Local industrial pumping permits may face regulatory restrictions.
* **Page 3: Parcel Overview & Boundaries**
  * Table showing Lat/Lng, elevation range, perimeter, area (in Hectares and Bighas), and administrative hierarchy.
  * Topographic cross-section chart showing the elevation change across the longest diagonal of the parcel (derived from Copernicus 30m DEM).
* **Page 4: Topography Analysis**
  * Elevation and slope distribution histograms.
  * Aspect wheel (showing the percentage of slopes facing North, South, East, West - critical for solar panel angle efficiency).
* **Page 5: Hydrological Assessment**
  * Drainage network map showing secondary streams, water bodies, and catchment flow lines.
  * Computed Topographic Wetness Index (TWI) map highlighting where rainwater naturally pools during storm surges.
* **Page 6: Meteorology & Historic Baselines**
  * Charts showing 50-year monthly precipitation climatology and temperature ranges.
  * Line graph tracking the increase in extreme rainfall events (>100mm/day) over the last 3 decades.
* **Page 7: Flood Risk Analysis**
  * Flood probability distribution curve with confidence intervals.
  * Sentinel-1 SAR-derived historical flood inundation map from recent monsoon seasons.
* **Page 8: Drought & Groundwater Stress**
  * Time-series chart tracking the Standardized Precipitation Index (SPI) indicating drought intervals.
  * Groundwater level decay curve showing annual decline in local monitoring wells.
* **Page 9: Thermal Risk & Heatwave Projections**
  * Projected annual days with temperatures above $45^\circ\text{C}$.
  * Solar Irradiance baseline (GHI - Global Horizontal Irradiance) - useful for solar developers.
* **Page 10: Future Climate Projections (CMIP6)**
  * A comparative table showing metrics across three scenarios (SSP126, SSP245, SSP585) for 2030, 2050, and 2080.
  * Features predicted metrics: Max Temperature shifts, Precipitation anomalies, and Flood probability shifts.
* **Page 11: Regulatory & Development Constraints**
  * Buffer maps indicating distances to the nearest Forest Reserve, Wildlife Sanctuary, National Highway, and Railway Line.
  * Zoning classification verification.
* **Page 12: Soil Geotech Properties**
  * Soil class breakdown (e.g., Clayey Sand).
  * Infiltration, permeability rates, and clay swelling indices (indicates structural foundation hazard).
* **Page 13: ESG Compliance Metrics**
  * Estimated above-ground carbon stock.
  * Baseline NDVI index tracking trends over the last 5 years.
* **Page 14: Underwriting Recommendations**
  * Detailed SWOT analysis of the land from an environmental perspective.
  * Suggested mitigation engineering checklists (e.g., "Build retaining walls on the northern boundary to mitigate landslide potential").
* **Page 15: Methodology & Metadata**
  * System metadata, data freshness timestamps, models version numbers (e.g., `LND-GNN-v2.1`), and reference citations.

---

## 3. Visual Layout Mockups

### Executive Summary Dashboard Panel Mockup (Page 2)

```
=============================================================================
                          LND-11 CLIMATE RISK INFERENCE
=============================================================================
 SURVEY NUMBER: 45 | VILLAGE: Khatkar | TEHSIL: Tiwari | DISTRICT: Jodhpur
 Area: 12.45 Hectares | Coordinates: 26.5412° N, 73.1245° E
-----------------------------------------------------------------------------
                      *** INVESTMENT GRADE: A ***
-----------------------------------------------------------------------------
   [ CLIMATE RISK ]          [ WATER SECURITY ]      [ SUITABILITY SCORE ]
       38 / 100                   42 / 100                 78 / 100
    [==>        ]              [====>      ]            [=======>   ]
      (Moderate)                (Vulnerable)              (Highly Fit)
-----------------------------------------------------------------------------
                           CRITICAL THREAT FLAGS
 [!] GROUNDWATER DECAY: Level falling at 0.45m/year. Restrictive boring.
 [!] THERMAL ANOMALY: Max temperatures projected to exceed 48°C by 2035.
=============================================================================
```

### CMIP6 Future Projections Metric Table Mockup (Page 10)

```
-----------------------------------------------------------------------------
                    CMIP6 PROJECTIONS FOR 2030 & 2050
-----------------------------------------------------------------------------
 Year | Scenario | Max Temp Shift | Rainfall Change | Flood Prob. Multiplier
------+----------+----------------+-----------------+------------------------
 2030 | SSP1-2.6 | +0.6 °C        | +2.4 %          | 1.05x
 2030 | SSP2-4.5 | +0.8 °C        | +4.1 %          | 1.12x
 2030 | SSP5-8.5 | +1.2 °C        | +6.8 %          | 1.25x
 2050 | SSP1-2.6 | +1.1 °C        | +3.8 %          | 1.15x
 2050 | SSP2-4.5 | +1.8 °C        | +8.2 %          | 1.34x
 2050 | SSP5-8.5 | +2.9 °C        | +14.5 %         | 1.68x
-----------------------------------------------------------------------------
* Baselines are calculated relative to historical averages from 1990-2020.
```
