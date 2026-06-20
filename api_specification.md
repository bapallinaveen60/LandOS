# LND-11: API Product Specification

---

## 1. REST API Specification

LND-11 exposes its climate intelligence through a versioned REST API. All requests require authentication via an `Authorization: Bearer <API_KEY>` header.

### 1.1 GET `/api/v1/risk/summary`
Retrieves climate risk indices for a specific survey number and village, or latitude/longitude coordinates.

#### Request Query Parameters
* `state` (string, optional): State name.
* `district` (string, optional): District name.
* `taluk` (string, optional): Taluk/Tehsil name.
* `village` (string, optional): Village name.
* `survey_number` (string, optional): Survey number.
* `lat` (float, optional): Latitude (WGS84). Required if administrative fields are absent.
* `lng` (float, optional): Longitude (WGS84). Required if administrative fields are absent.

#### Example Request
```http
GET /api/v1/risk/summary?state=Rajasthan&district=Jodhpur&taluk=Tiwari&village=Khatkar&survey_number=45 HTTP/1.1
Host: api.lnd11.com
Authorization: Bearer lnd_live_8f3a1290bc89d4fe
Accept: application/json
```

#### Example Response (`200 OK`)
```json
{
  "request_id": "req_88a9df21-c244-4b5b-a81d-e9fb94f1c97a",
  "timestamp": "2026-06-21T01:15:00Z",
  "parcel_info": {
    "survey_number": "45",
    "village": "Khatkar",
    "district": "Jodhpur",
    "state": "Rajasthan",
    "area_hectares": 12.4582,
    "centroid": {
      "latitude": 26.5412,
      "longitude": 73.1245
    }
  },
  "composite_scores": {
    "climate_risk_score": 38,
    "water_security_score": 42,
    "land_suitability_score": 78,
    "esg_risk_score": 18,
    "investment_grade": "A"
  },
  "hazard_probabilities": {
    "flood_risk": {
      "probability": 0.12,
      "confidence_interval": [0.08, 0.16],
      "risk_level": "LOW"
    },
    "drought_risk": {
      "probability": 0.58,
      "confidence_interval": [0.51, 0.65],
      "risk_level": "HIGH"
    },
    "heat_risk": {
      "probability": 0.72,
      "confidence_interval": [0.66, 0.78],
      "risk_level": "CRITICAL"
    },
    "landslide_risk": {
      "probability": 0.01,
      "confidence_interval": [0.00, 0.02],
      "risk_level": "NEGLIGIBLE"
    },
    "soil_erosion_risk": {
      "probability": 0.22,
      "confidence_interval": [0.18, 0.26],
      "risk_level": "MODERATE"
    }
  },
  "cache_hits": false
}
```

### 1.2 POST `/api/v1/risk/polygon`
Allows users to upload custom GeoJSON polygons for properties where cadastral maps are not registered.

#### Request Body
```json
{
  "geometry": {
    "type": "Polygon",
    "coordinates": [
      [
        [73.1240, 26.5410],
        [73.1250, 26.5410],
        [73.1250, 26.5420],
        [73.1240, 26.5420],
        [73.1240, 26.5410]
      ]
    ]
  },
  "buffer_meters": 100,
  "generate_pdf_report": true
}
```

#### Example Response (`202 Accepted`)
```json
{
  "job_id": "job_091bc8d3-3f11-420a-98ad-2bb0639d1b09",
  "status": "PROCESSING",
  "estimated_time_seconds": 45,
  "status_url": "https://api.lnd11.com/api/v1/jobs/job_091bc8d3-3f11-420a-98ad-2bb0639d1b09"
}
```

---

## 2. GraphQL API Specification

GraphQL is recommended for dashboard integrations, allowing the UI to pull only the metadata, scores, or future climate intervals required.

### Schema Definition
```graphql
type Centroid {
  latitude: Float!
  longitude: Float!
}

type ParcelInfo {
  surveyNumber: String
  village: String!
  district: String!
  state: String!
  areaHectares: Float!
  centroid: Centroid!
}

type CompositeScores {
  climateRiskScore: Int!
  waterSecurityScore: Int!
  landSuitabilityScore: Int!
  esgRiskScore: Int!
  investmentGrade: String!
}

type HazardMetric {
  probability: Float!
  confidenceInterval: [Float!]!
  riskLevel: String!
}

type HazardProbabilities {
  floodRisk: HazardMetric!
  droughtRisk: HazardMetric!
  heatRisk: HazardMetric!
  landslideRisk: HazardMetric!
  soilErosionRisk: HazardMetric!
}

type ClimateProjections {
  year: Int!
  scenario: String!
  tempAnomalyMax: Float!
  precipitationAnomalyPct: Float!
  floodProbabilityShift: Float!
}

type ClimateRiskReport {
  id: ID!
  parcelInfo: ParcelInfo!
  compositeScores: CompositeScores!
  hazardProbabilities: HazardProbabilities!
  projections(scenarios: [String!], years: [Int!]): [ClimateProjections!]!
  reportPdfUrl: String
}

type Query {
  getParcelReport(
    state: String!
    district: String!
    taluk: String!
    village: String!
    surveyNumber: String!
  ): ClimateRiskReport
  
  getCustomPolygonReport(
    geojson: String!
  ): ClimateRiskReport
}
```

---

## 3. Pricing, Rate Limits & Enterprise Plans

LND-11 enforces a tiered monetization strategy using Kong API gateway policies:

```
+---------------------------------------------------------------------------------+
|                                 Pricing Tiers                                   |
+----------------------+----------------------------------+-----------------------+
| Developer Tier       | Growth Tier                      | Enterprise Tier       |
| Free - 10 lookups/mo | $750/mo - 1,000 lookups/mo       | Custom - Volume API   |
| 3 req / min limit    | 60 req / min limit               | 500+ req / min limit  |
+----------------------+----------------------------------+-----------------------+
```

### Rate Limits & SLA Matrix

| Plan | Base Monthly Price | Monthly Lookup Limit | Over-limit Unit Cost | Rate Limit (Throttle) | SLA Guarantees | Support Channel |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Developer (Free)** | ₹0 | 10 lookups | N/A | 3 requests / min | None | Forum |
| **Starter** | ₹20,000 ($250 USD) | 250 lookups | ₹120 / lookup | 10 requests / min | 99.0% Uptime | Email (24hr response) |
| **Growth** | ₹60,000 ($750 USD) | 1,000 lookups | ₹80 / lookup | 60 requests / min | 99.9% Uptime | Slack Channel (4hr SLA) |
| **Enterprise** | Custom | Custom (Commitment) | ₹30 - ₹50 / lookup | 500+ requests / min | 99.99% Uptime | Dedicated TAM, PagerDuty |

### Enterprise Billing Constraints & Webhooks
Enterprise users (e.g., banks conducting bulk mortgage reviews) can configure **Webhooks** to receive asymmetric callback payloads when bulk calculations finish, preventing open HTTP connections:

```http
POST /your-endpoint-webhook HTTP/1.1
Host: partner-systems.com
Content-Type: application/json

{
  "event": "report.completed",
  "job_id": "job_091bc8d3-3f11-420a-98ad-2bb0639d1b09",
  "result": {
    "parcel_id": "8f2a2811-9a1b-419b-bcfa-192a839da4ef",
    "investment_grade": "AA",
    "climate_risk_score": 21,
    "pdf_report_url": "https://s3.ap-south-1.amazonaws.com/lnd11-reports/generated/2026/report_job_091bc8d3.pdf"
  }
}
```
