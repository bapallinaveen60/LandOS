# LND-11: Database Schema & Spatial Indexing Strategy

---

## 1. PostGIS Relational Database Schema

The database architecture is built on PostgreSQL with the PostGIS extension. It manages user accounts, administrative hierarchies, cadastral boundaries, and cached risk ratings.

```
                  +--------------------------+
                  |    organizations         |
                  +--------------------------+
                               | 1
                               |
                               | 1..*
                  +--------------------------+
                  |         users            |
                  +--------------------------+
                               | 1
                               |
                               | 1..*
                  +--------------------------+
                  |       api_keys           |
                  +--------------------------+

+--------------------+     +--------------------+     +--------------------+
|       states       |     |     districts      |     |  taluks_tehsils    |
+--------------------+     +--------------------+     +--------------------+
          | 1                       | 1                        | 1
          |                         |                          |
          | 1..*                    | 1..*                     | 1..*
+--------------------+     +--------------------+     +--------------------+
|     districts      |     |   taluks_tehsils   |     |      villages      |
+--------------------+     +--------------------+     +--------------------+
                                                                 | 1
                                                                 |
                                                                 | 1..*
                                                      +--------------------+
                                                      | cadastral_parcels  |
                                                      +--------------------+
                                                                 | 1
                                                                 |
                                                                 | 1..*
                                                      +--------------------+
                                                      |   risk_assessments |
                                                      +--------------------+
```

### DDL Schema Definition

```sql
-- Enable PostGIS spatial extensions
CREATE EXTENSION IF NOT EXISTS postgis;
CREATE EXTENSION IF NOT EXISTS postgis_topology;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- 1. Tenant/Organization Table
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL UNIQUE,
    subscription_tier VARCHAR(50) NOT NULL CHECK (subscription_tier IN ('FREE', 'STARTER', 'GROWTH', 'ENTERPRISE')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 2. User Accounts Table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    role VARCHAR(50) NOT NULL CHECK (role IN ('ADMIN', 'ANALYST', 'VIEWER')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 3. API Keys Table
CREATE TABLE api_keys (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
    key_hash VARCHAR(255) NOT NULL UNIQUE,
    key_prefix VARCHAR(8) NOT NULL,
    name VARCHAR(100) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    rate_limit_per_minute INT DEFAULT 60,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP WITH TIME ZONE
);

-- 4. Administrative Boundaries: State
CREATE TABLE states (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    census_code VARCHAR(10),
    geom GEOMETRY(MultiPolygon, 4326) NOT NULL
);

-- 5. Administrative Boundaries: District
CREATE TABLE districts (
    id INT PRIMARY KEY,
    state_id INT NOT NULL REFERENCES states(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    census_code VARCHAR(10),
    geom GEOMETRY(MultiPolygon, 4326) NOT NULL,
    CONSTRAINT unique_state_district UNIQUE(state_id, name)
);

-- 6. Administrative Boundaries: Taluk / Tehsil
CREATE TABLE taluks_tehsils (
    id INT PRIMARY KEY,
    district_id INT NOT NULL REFERENCES districts(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    geom GEOMETRY(MultiPolygon, 4326) NOT NULL,
    CONSTRAINT unique_district_taluk UNIQUE(district_id, name)
);

-- 7. Administrative Boundaries: Village
CREATE TABLE villages (
    id INT PRIMARY KEY,
    taluk_id INT NOT NULL REFERENCES taluks_tehsils(id) ON DELETE CASCADE,
    name VARCHAR(150) NOT NULL,
    census_code VARCHAR(20),
    geom GEOMETRY(MultiPolygon, 4326) NOT NULL
);

-- 8. Cadastral Survey Numbers (Partitioned Table)
CREATE TABLE cadastral_parcels (
    id UUID NOT NULL DEFAULT uuid_generate_v4(),
    village_id INT NOT NULL REFERENCES villages(id) ON DELETE CASCADE,
    survey_number VARCHAR(100) NOT NULL,
    sub_division VARCHAR(50),
    area_hectares NUMERIC(10, 4),
    geom GEOMETRY(Polygon, 4326) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id, village_id) -- Partition key must be part of primary key
) PARTITION BY RANGE (village_id);

-- 9. Risk Assessments Cache Table
CREATE TABLE risk_assessments (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    parcel_id UUID NOT NULL,
    village_id INT NOT NULL,
    climate_risk_score INT NOT NULL CHECK (climate_risk_score BETWEEN 0 AND 100),
    water_security_score INT NOT NULL CHECK (water_security_score BETWEEN 0 AND 100),
    land_suitability_score INT NOT NULL CHECK (land_suitability_score BETWEEN 0 AND 100),
    esg_risk_score INT NOT NULL CHECK (esg_risk_score BETWEEN 0 AND 100),
    investment_grade VARCHAR(3) NOT NULL CHECK (investment_grade IN ('AAA', 'AA', 'A', 'BBB', 'BB', 'B', 'CCC')),
    
    -- Store structured probabilistic model outputs
    hazard_probabilities JSONB NOT NULL, 
    -- Format: {"flood": 0.12, "drought": 0.45, "heat": 0.08, "landslide": 0.02, "erosion": 0.15}
    
    confidence_intervals JSONB NOT NULL,
    -- Format: {"flood": [0.08, 0.16], "drought": [0.38, 0.52]}
    
    report_s3_url VARCHAR(512),
    generated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL, -- Cache invalidation constraint
    FOREIGN KEY (parcel_id, village_id) REFERENCES cadastral_parcels(id, village_id) ON DELETE CASCADE
);
```

---

## 2. Table Partitioning Strategy

Because India contains over 300 million cadastral survey parcels, a single monolithic table would severely degrade PostGIS spatial index performance (`GIST` index operations on trees that exceed RAM size slow down logarithmically). 

### Partitioning by Geography (Village Range Partitions)
We partition `cadastral_parcels` dynamically using RANGE partitioning based on `village_id`.
* **State IDs** are partitioned into distinct ranges. For example:
  * Rajasthan Villages: `village_id` from `100000` to `199999`
  * Gujarat Villages: `village_id` from `200000` to `299999`
  * Maharashtra Villages: `village_id` from `300000` to `399999`

```sql
-- Example Partition Creation
CREATE TABLE cadastral_parcels_rajasthan PARTITION OF cadastral_parcels
    FOR VALUES FROM (100000) TO (200000);

CREATE TABLE cadastral_parcels_gujarat PARTITION OF cadastral_parcels
    FOR VALUES FROM (200000) TO (300000);
```

---

## 3. PostGIS Indexing & Query Optimizations

### Spatial Indices
Without proper indexing, bounding box intersections require sequential table scans. We apply GiST (Generalized Search Tree) spatial indexing to all geometry fields:

```sql
-- Spatial indexing for administrative tables
CREATE INDEX idx_states_geom ON states USING GIST (geom);
CREATE INDEX idx_districts_geom ON districts USING GIST (geom);
CREATE INDEX idx_taluks_geom ON taluks_tehsils USING GIST (geom);
CREATE INDEX idx_villages_geom ON villages USING GIST (geom);

-- Local spatial indexing on partitioned tables
CREATE INDEX idx_cadastral_geom_rajasthan ON cadastral_parcels_rajasthan USING GIST (geom);
CREATE INDEX idx_cadastral_geom_gujarat ON cadastral_parcels_gujarat USING GIST (geom);
```

### Administrative Search B-Tree Indexes
To resolve queries like `State -> District -> Taluk -> Village -> Survey Number` rapidly:

```sql
CREATE INDEX idx_districts_state ON districts(state_id);
CREATE INDEX idx_taluks_district ON taluks_tehsils(district_id);
CREATE INDEX idx_villages_taluk ON villages(taluk_id);
CREATE INDEX idx_cadastral_search ON cadastral_parcels(village_id, survey_number);
```

### PostGIS Query Pattern Example
Resolving survey boundaries and checking for river overlaps within a 500m buffer zone:

```sql
SELECT 
    p.id AS parcel_id, 
    p.survey_number,
    ST_Area(p.geom::geography) / 10000.0 AS area_hectares,
    ST_Distance(p.geom::geography, r.geom::geography) AS distance_to_river_meters
FROM 
    cadastral_parcels p
JOIN 
    villages v ON p.village_id = v.id
JOIN 
    taluks_tehsils t ON v.taluk_id = t.id
JOIN 
    districts d ON t.district_id = d.id
CROSS JOIN LATERAL (
    -- KNN (K-Nearest Neighbors) spatial index join for the closest river
    SELECT geom FROM major_rivers 
    ORDER BY geom <-> p.geom 
    LIMIT 1
) r
WHERE 
    d.name = 'Jodhpur' 
    AND v.name = 'Khatkar' 
    AND p.survey_number = '45';
```

---

## 4. GeoParquet Geospatial Lakehouse Architecture

While transaction meta-data and parcel shapes reside in PostGIS, massive temporal observation arrays (e.g., 20 years of daily satellite NDVI anomalies, 100 years of daily IMD gridded rainfall) are kept in a cost-effective, high-performance Data Lakehouse using **GeoParquet** on Amazon S3.

### S3 Directory Structure

```
s3://lnd11-geolake/
├── environmental/
│   └── ndvi_timeseries/
│       ├── state=rajasthan/
│       │   ├── year=2024/
│       │   │   └── data_chunk_001.parquet
│       │   └── year=2025/
│       │       └── data_chunk_002.parquet
├── meteorology/
│   └── historical_imd_grids/
│       ├── variable=precipitation/
│       │   └── decade=2010_2020/
│       │       └── imd_ppt_10_20.parquet
```

### GeoParquet Advantage
Unlike traditional Shapefiles or massive GeoJSONs, GeoParquet embeds coordinate system metadata (WGS84, CRS codes) directly into the Parquet file schema while preserving columnar vector representation. This allows **Apache Spark, DuckDB, or AWS Athena** to perform vectorized spatial filters without loading the entire dataset into memory:

```python
import duckdb

def query_historical_rainfall_duckdb(parcel_wkt, parquet_s3_path):
    """
    Leverages DuckDB spatial extensions to query a GeoParquet rain grid.
    Only loads data blocks intersecting with the parcel's bounding box.
    """
    con = duckdb.connect()
    con.execute("INSTALL spatial; LOAD spatial;")
    
    query = f"""
        SELECT 
            year, 
            month, 
            avg(ppt_value) as mean_ppt
        FROM read_parquet('{parquet_s3_path}')
        WHERE ST_Intersects(
            ST_GeomFromWKT('{parcel_wkt}'), 
            ST_GeomFromWKT(geom_wkt)
        )
        GROUP BY year, month
        ORDER BY year, month;
    """
    return con.execute(query).fetchdf()
```
