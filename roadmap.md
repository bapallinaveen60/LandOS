# LND-11: 24-Month Product Roadmap & Funding Strategy

---

## 1. 24-Month Granular Product Roadmap

LND-11's development is organized into 4 distinct phases over a 24-month timeline, aligning product features with GTM readiness.

```
       MONTHS 1-6                       MONTHS 7-12                     MONTHS 13-18                     MONTHS 19-24
+-----------------------+       +-----------------------+       +-----------------------+       +-----------------------+
| Phase 1: MVP & Core   | ----> | Phase 2: Scale APIs   | ----> | Phase 3: Real Estate  | ----> | Phase 4: BFSI Under-  |
| - Topography / Hydrology|      | - GraphSAGE Flood Model|      | - CMIP6 Projections   |      |   writing Suite       |
| - 3 States Cadastral  |       | - REST / API Gateway  |       | - Groundwater Decay   |       | - RBI Compliance Port |
| - Transactional PDFs  |       | - 6 States Expansion  |       | - 12 States Expansion |       | - 18 States Expansion |
+-----------------------+       +-----------------------+       +-----------------------+       +-----------------------+
```

### Phase 1: Core Engine & Transactional MVP (Months 1–6)
* **Product Features:**
  * Ingest Copernicus 30m DEM, ALOS 12.5m DEM, IMD 100-year historical meteorological grids, and major hydrological networks.
  * Map cadastral boundaries across 3 pilot states (Rajasthan, Gujarat, Madhya Pradesh) - chosen due to high concentration of utility solar developers.
  * Implement baseline XGBoost hazard classification models (Erosion, Slope stability, Landslide).
  * Build the automated PDF generation engine (WeasyPrint based) and launch the basic user portal.
* **GTM Focus:** Deliver transactional PDF reports to local solar consulting engineers and land brokers to test model performance and baseline assumptions.

### Phase 2: Spatial ML Scale & Enterprise API Launch (Months 7–12)
* **Product Features:**
  * Train and deploy the GraphSAGE flood accumulation model on hydro-basin topology networks.
  * Ingest Sentinel-1 SAR imagery logs to overlay true historical flooding anomalies.
  * Launch the Developer REST API portal managed via Kong API Gateway (JWT, rate limiting, meter-billing).
  * Expand cadastral database to 6 additional states (Maharashtra, Karnataka, Tamil Nadu, Andhra Pradesh, Telangana, Uttar Pradesh).
* **GTM Focus:** Close paid pilot contracts with 5 Tier-1 Solar/Wind IPPs (Adani, ReNew, ACME, Tata Power) to integrate parcel screening into their land acquisition workflow.

### Phase 3: Climate Projections & Real Estate Module (Months 13–18)
* **Product Features:**
  * Integrate downscaled CMIP6 climate model projections (SSP126, SSP245, SSP585) for 2030, 2050, and 2080.
  * Deploy Bayesian Networks to predict groundwater depth decay trajectories based on local extraction trends.
  * Develop the **Real Estate Suite**: Include micro-heat island metrics, surface drainage capacity checks, and soil swelling geotech indexes.
  * Launch the GraphQL API interface for custom UI query configurations.
  * Expand cadastral database to cover 12 states.
* **GTM Focus:** Partner with regional chapters of CREDAI to launch LND-11 to tier-1 real estate township developers for site due diligence.

### Phase 4: Financial Underwriting & Compliance Portal (Months 19–24)
* **Product Features:**
  * Build the **BFSI Underwriting Portal**: Create custom RBI-compliant physical climate risk disclosure report templates.
  * Implement automated bulk screening tools: Allow financial institutions to upload CSV arrays of 10,000+ spatial coordinates or loan registries for batch risk evaluation.
  * Integrate above-ground carbon stock baselines and NDVI vegetation loss alerting.
  * Cadastral database expanded to 18 states (covering ~90% of economic transactions in India).
* **GTM Focus:** Execute proof-of-concept integrations with 3 major private banks (HDFC, ICICI, Axis) and 2 housing finance companies (HDFC, LIC HFL).

---

## 2. Customer Acquisition Channels & GTM Details

LND-11 bypasses expensive generic advertising in favor of targeted B2B conversion loops:

1. **The Spatial Audit Program (Land-Screening Trial):**
   * *Strategy:* Offer utility solar developers a free "Climate Vulnerability Audit" of their past 3 successful and 1 failed project sites. By showing them how our GraphSAGE models would have flag-alerted the flooding that caused their actual insurance claims, we prove the platform’s value using their own historical data.
2. **Cadastral Freemium Land Hooks:**
   * *Strategy:* Provide free access to cadastral boundaries and basic topography for any parcel search (up to 10 searches / month). Land brokers and local engineers will use this tool as their daily default, creating a natural bottom-up conversion funnel to paid, deep-dive risk reports.
3. **Thought Leadership via District Climate Hazard Briefings:**
   * *Strategy:* Publish quarterly regional reports (e.g., *"Groundwater & Flood Risk Analysis of Solar Projects in Western Rajasthan"*). Distribute these to Chief Risk Officers and ESG Committee members at major financial institutions and funds.

---

## 3. Funding Strategy & Capital Milestones

To fuel this roadmap, LND-11 requires **$2.75M in total early-stage capital** to execute the pipeline before reaching EBITDA positive operations.

```
       PRE-SEED ROUND                     SEED ROUND                        SERIES A
       $250k (Angel)                    $2.5M (VCs)                       $10M (Institutional)
+-------------------------+       +-------------------------+       +-------------------------+
| Months 1-6 Milestones:  | ----> | Months 6-18 Milestones: | ----> | Month 18+ Milestones:   |
| - Ingest static DEMs    |       | - Map 12 Indian states  |       | - Complete national map |
| - Map 3 states (Cad.)   |       | - Deploy GraphSAGE model|       | - Integrate 5 banks     |
| - Launch PDF MVP        |       | - Sign 20 IPP contracts |       | - Launch SEA/Africa GTM |
| - Secure 2 solar pilots |       | - Reach ₹2.4 Crore ARR  |       | - Reach ₹35 Crore ARR   |
+-------------------------+       +-------------------------+       +-------------------------+
```

### 3.1 Pre-Seed Stage ($250,000 Angel Round)
* **Timeline:** Months 1–6
* **Key Milestones:**
  * Complete core backend ingestion engine.
  * Map cadastral boundaries across 3 pilot states (Rajasthan, Gujarat, Madhya Pradesh).
  * Build the PDF report generator and secure 2 signed letters of intent (LOI) from solar developers.
* **Use of Funds:** 70% Product/Engineering (GIS developers, weather data storage), 30% GTM/Operations.

### 3.2 Seed Stage ($2,500,000 VC Round)
* **Timeline:** Months 6–18 (Targeting Sequoia, Accel, or climate-tech specific funds like Avaana Capital).
* **Key Milestones:**
  * Expand cadastral database to 12 states.
  * Deploy the GraphSAGE flood model and CMIP6 climate scenarios.
  * Launch the API product and sign 20+ active enterprise contracts.
  * Reach an Annual Recurring Revenue (ARR) run rate of **₹2.4 Crores ($300k USD)**.
* **Use of Funds:** 50% Engineering & ML, 30% Business Development (Enterprise sales directors, customer success), 20% Cloud Infrastructure & High-resolution data licensing.

### 3.3 Series A ($10,000,000 Expansion Round)
* **Timeline:** Months 18+
* **Key Milestones:**
  * Complete national cadastral map coverage across India.
  * Secure 5 major private banks for mortgage integration.
  * Launch global expansion strategy targeting highly vulnerable, high-growth developing markets (Southeast Asia and East Africa).
  * Reach a gross revenue run rate of **₹35 Crores (~$4.2 Million USD)**.
