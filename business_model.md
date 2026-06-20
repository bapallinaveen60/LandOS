# LND-11: Revenue Model & Business Projections

---

## 1. Product Lines & Pricing Model

LND-11 monetizes through a multi-tier product strategy tailored to developer scale and institutional integration:

```
+---------------------------------------------------------------------------------+
|                               Product Monetization                              |
+----------------------+----------------------------------+-----------------------+
| Transactional PDF    | Bulk Land Screening              | Enterprise APIs       |
| ₹4,000 / report      | ₹1,500 / parcel (min 10)         | ₹60,000 / mo base     |
| Retail / Developers  | IPP Site Selection               | Banks & NBFCs         |
+----------------------+----------------------------------+-----------------------+
```

### Product Definitions
1. **Single Parcel PDF Report (Transactional):**
   * **Target:** Land brokers, local real estate builders, individual land buyers, small-scale solar installation firms.
   * **Pricing:** **₹4,000 ($50 USD)** per report. Deliverable is an instantly generated PDF.
2. **Bulk Land Screening (Batch Processing):**
   * **Target:** Large-scale solar and wind Independent Power Producers (IPPs) seeking to screen thousands of hectares during preliminary bidding.
   * **Pricing:** **₹1,500 ($18 USD)** per parcel. Requires a minimum purchase of 10 parcels (e.g., screening a 200-acre site broken down by survey subdivisions).
3. **Enterprise API (Recurring Subscription):**
   * **Target:** Banking, financial services, insurance (BFSI), housing finance corporations (HFCs), and PropTech platforms (e.g., MagicBricks, 99acres).
   * **Pricing:** Flat subscription of **₹60,000 ($750 USD) / month** which includes 1,000 API queries. Overages are billed at **₹80 ($1.00 USD)** per query.
4. **Government & Smart Cities Licenses (SaaS Portal):**
   * **Target:** Urban development authorities (e.g., DDA, MHADA, MMRDA), state disaster management bodies.
   * **Pricing:** Annual site license of **₹25,00,000 ($30,000 USD)** for unlimited portal logins and cadastral overlay tools.

---

## 2. Indian Market Size (TAM, SAM, SOM)

The market calculations are tailored to the Indian economic footprint:

```
+-----------------------------------------------------------------------------+
|                          TAM: ₹8,000 Crores ($1B USD)                       |
|  (All Indian Mortgages, Real Estate, Infrastructure & Renewable Projects)   |
|                                                                             |
|            +----------------------------------------------------+           |
|            |            SAM: ₹1,200 Crores ($150M USD)          |           |
|            | (Tier-1/2 Real Estate, Utility Renewables, HFCs)  |           |
|            |                                                    |           |
|            |         +--------------------------------+         |           |
|            |         |  SOM: ₹120 Crores ($15M USD)   |         |           |
|            |         |  (3-Year Capture - Top IPPs &  |         |           |
|            |         |        Private Banks)          |         |           |
|            +---------+--------------------------------+---------+           |
+-----------------------------------------------------------------------------+
```

### 2.1 Total Addressable Market (TAM)
* **Definition:** The maximum potential value if 100% of mortgage underwriting, land registration, commercial infrastructure development, and renewable energy land deals in India utilized LND-11.
* **Calculation Metrics:**
  * ~10 million administrative land transactions occur annually in India. At an average due diligence spend of ₹4,000 per transaction = ₹4,000 Crores.
  * Over 5 million new home loans and agricultural mortgages are approved annually. At a pre-underwriting check fee of ₹800 per check = ₹400 Crores.
  * Over 15,000 large-scale solar, wind, transmission, and infrastructure site acquisitions are screened annually. Each requires an average of 1,000 parcel checks = ₹3,600 Crores.
  * **Total India TAM:** **₹8,000 Crores ($1 Billion USD)**.

### 2.2 Serviceable Addressable Market (SAM)
* **Definition:** The portion of the TAM that can be reached using LND-11's core digital portal, API products, and cadastral coverage.
* **Refinements:** Focuses exclusively on institutional transactions—excluding individual rural plot sales. Includes Tier-1/2 residential projects, major IPP solar developments (targeting India's 500 GW renewable energy target by 2030), and top-tier private banks/HFCs.
* **Total India SAM:** **₹1,200 Crores ($150 Million USD)**.

### 2.3 Serviceable Obtainable Market (SOM)
* **Definition:** The revenue we can capture by Year 3, assuming a conservative 10% penetration rate of our SAM by closing the top 20 solar/wind developers, 15 private banks/HFCs, and Tier-1 real estate developers.
* **Total India SOM:** **₹120 Crores ($15 Million USD)**.

---

## 3. Unit Economics & Gross Margins

LND-11 operates with typical high-margin software economics:

* **SaaS Margin Structure:**
  * **Selling Price (Average Blended Query):** ₹120 ($1.45)
  * **Cost of Goods Sold (COGS per Query):**
    * AWS processing compute (EC2/EKS spot instances): ₹4.00
    * SageMaker Serverless ML model inference: ₹3.00
    * Mapbox tile server and vector geocoding: ₹2.50
    * Raw weather and satellite API storage overhead: ₹1.50
    * **Total COGS:** **₹11.00 ($0.13 USD)**
  * **Gross Margin:** **90.8%**

---

## 4. 5-Year Revenue Projections (INR in Crores)

| Metric | Year 1 | Year 2 | Year 3 | Year 4 | Year 5 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Active Enterprise Customers** | 12 | 45 | 120 | 280 | 600 |
| **API Queries Processed (Millions)**| 1.2M | 5.5M | 15.0M | 40.0M | 100.0M |
| **SaaS/API Revenue** | ₹1.8 Cr | ₹8.2 Cr | ₹22.5 Cr | ₹60.0 Cr | ₹150.0 Cr |
| **Transactional PDF Report Rev.** | ₹0.6 Cr | ₹2.8 Cr | ₹8.0 Cr | ₹22.0 Cr | ₹50.0 Cr |
| **Govt Licenses Revenue** | ₹0.0 Cr | ₹1.0 Cr | ₹4.5 Cr | ₹12.0 Cr | ₹30.0 Cr |
| **Total Gross Revenue** | **₹2.4 Cr** | **₹12.0 Cr**| **₹35.0 Cr**| **₹94.0 Cr**| **₹230.0 Cr**|
| **Gross Margin (90%)** | ₹2.16 Cr | ₹10.8 Cr | ₹31.5 Cr | ₹84.6 Cr | ₹207.0 Cr |
| **Operating Expenses (R&D, SG&A)**| ₹3.00 Cr | ₹7.50 Cr | ₹15.00 Cr| ₹32.00 Cr| ₹75.00 Cr|
| **EBITDA** | **-₹0.84 Cr**| **₹3.30 Cr** | **₹16.50 Cr**| **₹52.60 Cr**| **₹132.00 Cr**|
| **EBITDA Margin** | -35.0% | 27.5% | 47.1% | 55.9% | 57.3% |
