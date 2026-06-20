# LND-11: 24-Month Product Roadmap & Funding Strategy

---

## 1. 24-Month Macro Product Roadmap

LND-11's commercial development is organized into 4 distinct phases over a 24-month timeline, aligning product features with GTM readiness.

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
* **Product Features:** Ingest Copernicus 30m DEM, ALOS 12.5m DEM, IMD 100-year historical grids, and major hydrological networks. Map cadastral boundaries across 3 pilot states (Rajasthan, Gujarat, Madhya Pradesh). Build the automated PDF generation engine and launch the basic user portal.
* **GTM Focus:** Deliver transactional PDF reports to local solar consulting engineers and land brokers to test model performance and baseline assumptions.

### Phase 2: Spatial ML Scale & Enterprise API Launch (Months 7–12)
* **Product Features:** Train and deploy the GraphSAGE flood accumulation model on hydro-basin topology networks. Ingest Sentinel-1 SAR imagery logs to overlay true historical flooding anomalies. Launch the Developer REST API portal managed via Kong API Gateway (JWT, rate limiting, meter-billing). Expand cadastral database to 6 additional states.
* **GTM Focus:** Close paid pilot contracts with 5 Tier-1 Solar/Wind IPPs (Adani, ReNew, ACME, Tata Power) to integrate parcel screening into their land acquisition workflow.

### Phase 3: Climate Projections & Real Estate Module (Months 13–18)
* **Product Features:** Integrate downscaled CMIP6 climate model projections (SSP126, SSP245, SSP585) for 2030, 2050, and 2080. Deploy Bayesian Networks to predict groundwater depth decay trajectories based on local extraction trends. Develop the **Real Estate Suite**: Include micro-heat island metrics, surface drainage capacity checks, and soil swelling geotech indexes. Launch the GraphQL API interface. Expand cadastral database to cover 12 states.
* **GTM Focus:** Partner with regional chapters of CREDAI to launch LND-11 to tier-1 real estate township developers for site due diligence.

### Phase 4: Financial Underwriting & Compliance Portal (Months 19–24)
* **Product Features:** Build the **BFSI Underwriting Portal**: Create custom RBI-compliant physical climate risk disclosure report templates. Implement automated bulk screening tools: Allow financial institutions to upload CSV arrays of 10,000+ spatial coordinates or loan registries for batch risk evaluation. Integrate above-ground carbon stock baselines and NDVI vegetation loss alerting. Cadastral database expanded to 18 states.
* **GTM Focus:** Execute proof-of-concept integrations with 3 major private banks (HDFC, ICICI, Axis) and 2 housing finance companies.

---

## 2. 2-Developer MVP Build Plan (Months 1–6)

To build the Phase 1 MVP with a **team of two senior developers**, the roles and timelines are divided as follows:

### 2.1 Developer Division of Labor
* **Developer A: Geospatial Data & ML Engineer**
  * *Focus:* Raw geospatial data pipelines, coordinate systems (CRS) normalization, Cloud-Optimized GeoTIFF (COG) generation, SpatioTemporal Asset Catalog (STAC) configuration, training spatial ML models (XGBoost/LightGBM), and setting up model inference containers.
* **Developer B: Full-Stack & API Architect**
  * *Focus:* PostgreSQL/PostGIS setup, spatial table range partitioning, FastAPI API framework, Celery/Redis task queueing, automated PDF rendering (WeasyPrint), and the Next.js/Mapbox GL JS user dashboard.

### 2.2 Month-by-Month Developer Task Split

#### Month 1: Foundation & Data Ingestion
* **Developer A (GIS):** Ingest and clean primary DEMs (Copernicus GL-30, ALOS 12.5m) and IMD weather grids. Set up S3 directories and write translation scripts to convert raw geotiffs into COGs.
* **Developer B (Full-stack):** Initialize AWS Aurora PostgreSQL with PostGIS extension. Load administrative boundaries (States, Districts, Taluks). Scaffolding the FastAPI service and authentication models.
* *Milestone:* Database ready; raster blocks initialized; baseline server running.

#### Month 2: Cadastral Loading & Geometry Statistics
* **Developer A (GIS):** Build the parallel zonal statistics script using `rasterio.mask` and HTTP range requests. Configure the local STAC metadata database.
* **Developer B (Full-stack):** Ingest cadastral maps for the 3 pilot states (Rajasthan, Gujarat, Madhya Pradesh) into range-partitioned tables. Write geometry optimization queries (`ST_MakeValid`, spatial indexing).
* *Milestone:* Survey boundary polygon resolved automatically; zonal statistics computed in under 1 second.

#### Month 3: Model Training & Core REST API
* **Developer A (GIS/ML):** Train and calibrate XGBoost models for soil erosion and terrain slope hazards. Construct the topological hydrology graph from HydroSHEDS. Deploy inference to AWS SageMaker/Triton.
* **Developer B (Full-stack):** Build the REST API risk endpoints. Implement Celery workers and Redis broker to handle asynchronous calculation tasks.
* *Milestone:* Model outputs (scores and 95% confidence intervals) accessible via REST endpoints.

#### Month 4: Map Dashboard & PDF Document Engine
* **Developer A (GIS/ML):** Run historical Sentinel-1 SAR flood audits for validation. Ingest CMIP6 warming scenarios (SSP245, SSP585) downscaled to 1km resolution.
* **Developer B (Full-stack):** Develop the Next.js frontend UI dashboard. Integrate Mapbox GL JS to render vector tiles of cadastral boundaries dynamically. Write the CSS and HTML templates for the WeasyPrint PDF generator.
* *Milestone:* User dashboard complete; interactive vector map active; automated PDF reports generating.

#### Months 5–6: Integration, Latency Tuning, and Pilots
* **Developer A (GIS/ML):** Optimize Triton Inference Server concurrency to support bulk requests. Build fallback prediction routes for missing raster tiles.
* **Developer B (Full-stack):** Configure Kong API Gateway for rate limiting and billing metering. Optimize PostgreSQL queries and configure cache storage. Integrate frontend modules with final API endpoints.
* *Milestone:* Fully operational, production-ready LND-11 platform launched with 2 pilot solar clients.

### 2.3 Key Technical Shortcuts
1. **TiTiler for Map Tiles:** Do not build a custom tile server. Use **TiTiler** (FastAPI tiler) to render S3 COGs as Mapbox layers dynamically.
2. **DuckDB Spatial for Ad-hoc Queries:** Use DuckDB's spatial extensions to process large vector files (like national soil maps) on S3 without having to import them all into PostgreSQL.
3. **Use Premade Admin Databases:** Avoid cleaning government shapefiles manually; use pre-built socioeconomic and administrative datasets.

---

## 3. Customer Acquisition Channels & GTM Details

LND-11 bypasses expensive generic advertising in favor of targeted B2B conversion loops:

1. **The Spatial Audit Program (Land-Screening Trial):**
   * *Strategy:* Offer utility solar developers a free "Climate Vulnerability Audit" of their past 3 successful and 1 failed project sites. By showing them how our GraphSAGE models would have flag-alerted the flooding that caused their actual insurance claims, we prove the platform’s value using their own historical data.
2. **Cadastral Freemium Land Hooks:**
   * *Strategy:* Provide free access to cadastral boundaries and basic topography for any parcel search (up to 10 searches / month). Land brokers and local engineers will use this tool as their daily default, creating a natural bottom-up conversion funnel to paid, deep-dive risk reports.
3. **Thought Leadership via District Climate Hazard Briefings:**
   * *Strategy:* Publish quarterly regional reports (e.g., *"Groundwater & Flood Risk Analysis of Solar Projects in Western Rajasthan"*). Distribute these to Chief Risk Officers and ESG Committee members at major financial institutions and funds.

---

## 4. Funding Strategy & Capital Milestones

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

### 4.1 Pre-Seed Stage ($250,000 Angel Round)
* **Timeline:** Months 1–6
* **Key Milestones:**
  * Complete core backend ingestion engine.
  * Map cadastral boundaries across 3 pilot states (Rajasthan, Gujarat, Madhya Pradesh).
  * Build the PDF report generator and secure 2 signed letters of intent (LOI) from solar developers.
* **Use of Funds:** 70% Product/Engineering (GIS developers, weather data storage), 30% GTM/Operations.

### 4.2 Seed Stage ($2,500,000 VC Round)
* **Timeline:** Months 6–18 (Targeting Sequoia, Accel, or climate-tech specific funds like Avaana Capital).
* **Key Milestones:**
  * Expand cadastral database to 12 states.
  * Deploy the GraphSAGE flood model and CMIP6 climate scenarios.
  * Launch the API product and sign 20+ active enterprise contracts.
  * Reach an Annual Recurring Revenue (ARR) run rate of **₹2.4 Crores ($300k USD)**.
* **Use of Funds:** 50% Engineering & ML, 30% Business Development (Enterprise sales directors, customer success), 20% Cloud Infrastructure & High-resolution data licensing.

### 4.3 Series A ($10,000,000 Expansion Round)
* **Timeline:** Months 18+
* **Key Milestones:**
  * Complete national cadastral map coverage across India.
  * Secure 5 major private banks for mortgage integration.
  * Launch global expansion strategy targeting highly vulnerable, high-growth developing markets (Southeast Asia and East Africa).
  * Reach a gross revenue run rate of **₹35 Crores (~$4.2 Million USD)**.
