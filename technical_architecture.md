# LND-11: Technical Architecture & Spatial Data Pipeline

---

## 1. Geospatial Data Ingestion & Layer Design

The system integrates 50+ heterogeneous geospatial layers across different formats (GeoTIFF, Shapefile, GeoJSON, NetCDF) and spatio-temporal resolutions. 

```
+-----------------------------------------------------------------------------------+
|                              Data Ingestion Engine                                |
+------------------------+------------------------+---------------------------------+
| Raster Inputs          | Vector Inputs          | Climate Projections             |
| (DEM, NDVI, Soil Moist)| (Rivers, Roads, Cad.)  | (CMIP6 SSP Scenarios)           |
+-----------+------------+-----------+------------+---------------+-----------------+
            |                        |                            |
            v                        v                            v
  GDAL / Rasterio          GeoPandas / PyGEOS             Dask / Xarray
            |                        |                            |
            +------------------------+----------------------------+
                                     |
                                     v
                       CRS Normalization (EPSG:4326)
                                     |
                +--------------------+--------------------+
                |                                         |
                v                                         v
     S3 COGs (Cloud Optimized)               PostGIS Cadastral DB
```

### Data Layer Inventory

| Domain | Dataset Name | Native Resolution | File Format | Update Frequency | Source Agency |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Topography** | Copernicus DEM | 30m (GLO-30) | Cloud-Optimized GeoTIFF | Static | ESA / Copernicus |
| | ALOS PALSAR DEM | 12.5m | GeoTIFF | Static | JAXA |
| | Terrain Slope & Aspect | Derived (12.5m) | COG (Cloud-Optimized GeoTIFF) | Static | LND-11 Internal |
| | Topographic Wetness Index | Derived (12.5m) | COG | Static | LND-11 Internal |
| **Hydrology** | HydroSHEDS Drainage Basin | 90m (3-arcsec) | Vector Shapefile | Static | WWF / USGS |
| | National Water Bodies | Variable | Vector (PostGIS) | Monthly | Bhuvan / NRSC |
| | Groundwater Depth & Quality | Station level | CSV / Interpolated | Bi-annual | CGWB India |
| | Major/Minor River Networks | Variable | Vector (Spatial Index) | Static | NRSC |
| **Meteorology** | IMD Gridded Rainfall | 0.25° x 0.25° | NetCDF / Raster | Daily | IMD |
| | IMD Gridded Temperature | 1° x 1° | NetCDF | Daily | IMD |
| | ERA5 Reanalysis (CAPE, TCWV) | 0.25° (~31km) | NetCDF | Monthly | ECMWF |
| **Climate Change** | CMIP6 Downscaled Projections | ~25km (Downscaled to 1km) | NetCDF / Zarr | Static (Decadal runs) | IPCC / NASA NEX-GDDP |
| **Environmental**| Sentinel-2 L2A (LULC, NDVI) | 10m | COG / Multi-spectral | 5-day cycle | ESA |
| | MODIS Soil Moisture (SMAP) | 9km (Downscaled to 1km) | HDF5 / NetCDF | Daily | NASA |
| | Vegetation Health Index (VHI) | 250m | GeoTIFF | 8-day cycle | NOAA |
| **Soil** | National Soil Map (NBSS&LUP) | 1:250,000 | Vector Shapefile | Static | NBSS&LUP |
| | Soil Organic Carbon | 250m | GeoTIFF | Annual | ISRIC World Soil |

---

## 2. Geospatial Processing & Zonal Statistics Engine

To calculate parcel-level risk, LND-11 matches the parcel's cadastral boundary (vector polygon) with these 50+ layers.

### Coordinate Reference System (CRS) Normalization
All incoming datasets are reprojected to a common geodetic coordinate system (**EPSG:4326 - WGS84**) for spatial storage. During geometric operations (e.g., computing precise parcel area, slope gradients, or buffers), geometries are dynamically projected to the local **UTM (Universal Transverse Mercator) Zone** (Zones 42N to 46N cover India) to prevent distortion:
$$\text{UTM Zone} = \lfloor \frac{\text{Longitude} + 180}{6} \rfloor + 1$$

### Cloud-Optimized GeoTIFFs (COGs) and SpatioTemporal Asset Catalog (STAC)
Raster datasets are stored in Amazon S3 as COGs. This allows the processing layer to perform **HTTP Range Requests**, downloading only the specific pixels covering the land parcel polygon instead of reading a multi-gigabyte raster file. The metadata is queried using a STAC API hosted on AWS.

### The In-Memory Extraction Pipeline
We use Python's multiprocessing wrappers over `rasterio` and `fiona` (optimized with C-libraries GDAL and GEOS) to compute zonal statistics in parallel:

```python
import rasterio
import rasterio.mask
import numpy as np
from shapely.geometry import shape

def extract_zonal_stats(geojson_polygon, cog_s3_url):
    """
    Performs fast zonal statistics over an S3 Cloud-Optimized GeoTIFF.
    Uses HTTP Range Requests to read only the intersecting pixels.
    """
    geom = shape(geojson_polygon)
    
    # Configure rasterio environment for high-performance AWS S3 access
    with rasterio.Env(
        GDAL_DISABLE_READDIR_ON_OPEN='EMPTY_DIR',
        CPL_VSIL_CURL_ALLOWED_EXTENSIONS='.tif',
        GDAL_HTTP_MERGE_CONSECUTIVE_RANGES='YES'
    ):
        with rasterio.open(cog_s3_url) as src:
            # Mask the raster with the parcel geometry
            try:
                out_image, out_transform = rasterio.mask.mask(src, [geom], crop=True)
                # Filter out no-data values
                nodata = src.nodata if src.nodata is not None else -9999
                valid_data = out_image[0][out_image[0] != nodata]
                
                if len(valid_data) == 0:
                    return {"mean": None, "std": None, "min": None, "max": None}
                
                return {
                    "mean": float(np.mean(valid_data)),
                    "std": float(np.std(valid_data)),
                    "min": float(np.min(valid_data)),
                    "max": float(np.max(valid_data)),
                    "p90": float(np.percentile(valid_data, 90))
                }
            except ValueError:
                # Polygon does not overlap raster bounds
                return {"mean": None, "std": None, "min": None, "max": None}
```

---

## 3. Survey Number Intelligence Engine Workflow

The Survey Number Intelligence Engine converts a legal administrative query (e.g., *Survey No. 45, Village: Khatkar, District: Jodhpur, Rajasthan*) into a georeferenced spatial polygon and executes risk modeling:

```
[User Request] ---> 1. Administrative Resolver (PostGIS query)
                          |
                          v (Georeferenced Polygon)
                    2. Spatial Index Lookup (STAC API / R-Tree)
                          |
                          v (Intersected Raster Tiles / Vector Geometries)
                    3. Parallel Zonal Stats (GDAL/Rasterio/Dask)
                          |
                          v (Structured Feature Vectors)
                    4. AI Inference Engine (XGBoost/GNN)
                          |
                          v (Probabilistic Scores + Confidence Intervals)
                    5. Composite Index Calculator
                          |
                          +---> 6a. REST/GraphQL Response
                          |
                          +---> 6b. PDF Generator (Celery Queue -> S3 URL)
```

1. **Administrative Resolution:** The engine queries a PostGIS table containing the administrative hierarchy of India down to the cadastral survey number level. A spatial index (`GIST` index) returns the boundaries of the parcel.
2. **Spatial Index Intersection:** The parcel geometry is intersected with static and dynamic vectors using an R-Tree index. In parallel, it queries the STAC API to find the paths of the relevant COG slices (DEM, Slope, NDVI, Soil Moisture) on S3.
3. **Zonal Feature Extraction:** Workers run parallel zonal statistics (mean, variance, skewness) to extract topographic indices, meteorological histories, and vegetation metrics.
4. **AI Inference:** The extracted feature vector is fed to our machine learning inference engine (running on AWS SageMaker / Triton), executing XGBoost (for tabular features) and spatial Graph Neural Networks (to capture downstream hydrological relationships).
5. **Score Aggregation:** The probabilistic outputs of the ML models are aggregated through our mathematical Composite Index Framework to yield standardized scores.
6. **Result Delivery:**
   * For APIs: Returns JSON payloads containing risk scores, probabilities, and confidence intervals under 2 seconds.
   * For UI/Reports: Enqueues a PDF generation task to Celery workers, assembling the report in PDF format, uploading it to S3, and delivering a signed URL.

---

## 4. Scalable System Architecture Blueprint

LND-11 relies on a cloud-native, microservices-based infrastructure deployed on AWS, utilizing Kubernetes (EKS) for container orchestration, and a hybrid data lake/relational database design:

```
                                  [ Next.js Frontend / API Client ]
                                                 |
                                                 v
                                    [ AWS Route 53 / ALB ]
                                                 |
                                                 v
                                    [ API Gateway (Kong) ]
                                                 |
                       +-------------------------+-------------------------+
                       |                                                   |
                       v                                                   v
           [ EKS: FastAPI Backend ]                            [ EKS: GraphQL Engine ]
             (Auth, Cadastral API)                               (Dynamic Data Query)
                       |                                                   |
           +-----------+-----------+                                       |
           |                       |                                       |
           v                       v                                       v
    [ Redis Cache ]     [ Aurora PostgreSQL / PostGIS ] <----------+-------+
    (Session, Tasks)     - Cadastral Boundaries                    |
                         - Spatial Indexes (GIST)                  |
                         - Metadata / User Accounts                |
                                                                   |
  [ Celery Task Broker (Redis/RabbitMQ) ]                          |
           |                                                       |
           v                                                       |
  [ EKS: Spatial Processing Workers (GeoPandas, GDAL, Rasterio) ] -+
           |
           +---------------------+---------------------+
           |                     |                     |
           v                     v                     v
    [ Amazon S3 COGs ]   [ SageMaker Inference ]  [ Athena / GeoParquet ]
     - Raster Data Lake   - XGBoost Models         - Historical Weather
     - PDF Reports        - PyTorch GNNs           - Satellite Time Series
```

### Cloud Components Detail

* **Data Storage Layer:**
  * **Aurora PostgreSQL (PostGIS enabled):** Stores transactional data, user profiles, API keys, administrative hierarchy, and cadastral survey-number boundary geometries. Optimized with local spatial indices (`CREATE INDEX idx_cadastral_geom ON cadastral_table USING GIST (geom);`).
  * **Amazon S3 (Data Lake):** Acts as the raster repository (COGs) and document store (PDF reports). Historical satellite runs (NDVI timeseries) are stored as Apache Parquet datasets to utilize Athena for partition queries.
  * **GeoParquet & AWS Athena:** Large historical vector files (e.g., country-wide soil maps, decade-long weather stations) are stored in GeoParquet format on S3 and queried serverless-ly using Athena.
* **Processing & AI Layer:**
  * **FastAPI Microservices:** Lightweight, asynchronous web framework running on EKS pods to manage user accounts, coordinate PDF requests, and serve API requests.
  * **Celery workers (Dask-enabled):** Handles heavy spatial intersections and raster masking. Workers run on Spot Instances, automatically scaling up based on SQS queue length.
  * **AWS SageMaker Serverless Inference:** Hosts pre-trained XGBoost, LightGBM, and GraphSAGE models. Cold-start times are mitigated using provisioned concurrency for enterprise routes.
* **Client & Edge Routing:**
  * **Next.js & React:** Served via CloudFront.
  * **Mapbox GL JS:** Renders vector tiles of cadastral boundaries dynamically on the client side, overlaying raster risk heatmaps generated by our tile server (Tiler/TiTiler).
  * **Kong API Gateway:** Manages API keys, JWT validation, rate-limiting, and usage billing meters.
