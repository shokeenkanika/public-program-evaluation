# Delhi Bus Service Analysis: Policy Overview and a Prediction Study using ML tools 

## Background

Delhi's public transport system is spatially uneven. Some wards have good access to rapid transit and dense road networks. Others depend on limited bus coverage. This analysis builds a ward-level dataset to examine how neighborhood characteristics relate to bus service provision.

We face a practical constraint: ward boundaries and counts change over time. Our cleaning process prioritizes geographic consistency by joining all data sources to a single set of ward polygons. Results should be interpreted as applying to these specific ward units.

## Research Questions

### Question 1: What factors explain bus service intensity?

We model bus service intensity as a function of several ward characteristics, controlling for district-level differences. Our outcome variable combines three standardized measures: stop density, route density, and daily trips. 

**Predictors include:**

- Stop and route density (from transit schedules and ward area)
- Population density (imputed at the district median where missing)
- Vehicle ownership (total two-wheeler and four-wheeler registrations at the nearest Regional Transport Office)
- Distance to nearest RTO (in meters)
- Nighttime lights (mean radiance as a proxy for economic activity)
- Metro access (distance to nearest metro station in kilometers)
- Road density (kilometers of drivable roads per square kilometer)
- District fixed effects

### Question 2: Where would new service investments have the highest impact?

Using our fitted model, we run a scenario analysis. We ask: if low-service wards were brought up to a target service level (the 75th percentile of stop and route density), where would we see the largest gains relative to cost?

**Our approach:**

1. Set target thresholds at the 75th percentile for stops per km² and routes per km²
2. Calculate the required increases in stops and routes for each ward
3. Use the regression model to estimate predicted gains in bus service intensity
4. Convert improvements to return on investment using cost assumptions:
   - Stop or shelter construction: ₹4-5 lakh (we use the midpoint)
   - Route operating costs: approximated from per-kilometer figures and assumed daily utilization, annualized for one year
5. Report ROI as index gain per ₹1 lakh and ₹ lakh per 0.01 index gain

This is a screening tool for prioritization discussions, not a final capital planning model.

## Methods

### Model choice

We use Elastic Net regression with cross-validation. This approach suits our data because several predictors are highly correlated (stop density, route density, road density, and nighttime lights). Elastic Net handles collinearity better than standard regression while still identifying the most important features. It reduces overfitting when we add district fixed effects.

The model uses 10-fold cross-validation to select the optimal regularization strength from a grid of candidate values.

### Outcome transformation

We model the log-transformed bus service intensity to reduce skew. The analysis uses an 80/20 train-test split. We report model fit using RMSE, MAE, and R² in both transformed and original units. For visualization, we clip negative predictions to zero.

Note that bus service intensity is a constructed index, so results are interpreted in index units rather than direct trip counts.

### What this analysis does and does not do

This is an associational analysis designed to explain patterns and support prioritization. It is not a causal study. We do not have the timing variation, natural experiments, or panel structure needed for causal identification. The code is structured to allow future extensions if additional data become available.

## Code Structure

### metro_stations.ipynb

Retrieves Delhi Metro station data (names, latitude, longitude) from an ArcGIS server using paginated requests. Removes duplicate stations by averaging coordinates. Outputs a ready-to-use data frame to avoid repeated API calls.

### data_cleaning.ipynb

Builds the ward-level analysis dataset (`data/clean/analysis_dataset_ward_level.csv`) through the following steps:

**Transit data:** Loads GTFS combined table (linking stop times, trips, routes, and stops). Spatially joins stops to wards to calculate ward totals for stops, routes, and daily trips. Standardizes these measures and combines them into a bus service intensity index.

**Population:** Loads ward population and calculates population density.

**Vehicle ownership:** Filters vehicle registration records to two-wheelers and four-wheelers. Aggregates counts by RTO. Assigns each ward to its nearest RTO and records the distance in meters.

**Nighttime lights:** Calculates mean radiance per ward from VIIRS satellite imagery.

**Metro access:** Measures distance from ward centroid to nearest metro station in kilometers.

**Road density:** Uses OpenStreetMap road network data to calculate kilometers of drivable roads per square kilometer within each ward.

**District effects:** Creates district indicator variables.

**Final output:** Merges all sources into a single flat dataset and writes to CSV.

### analysis.ipynb

Loads the cleaned ward dataset. Engineers density variables (stops per km², routes per km²). Imputes missing population density using district medians and adds a missingness indicator. Includes district fixed effects.

Fits an Elastic Net model with cross-validated hyperparameters. Reports model performance and ranks predictors by coefficient magnitude (Question 1).

Runs the ROI scenario analysis (Question 2). Raises stop and route density to the 75th percentile for low-service wards. Ranks wards by modeled gain per rupee invested.

# public-program-evaluation

# Data Sources 

(1) Delhi RTO-wise Vehicle Registrations Data for 2021 (manual download)
https://data.opencity.in/dataset/delhi-rto-wise-vehicle-registrations-data
saved as: data/raw/2021.csv 

(2) Open City, Transit Data, New Delhi based on GTFS specifications (manual download)
https://otd.delhi.gov.in/data/static/
saved as: /data/raw/Delhi_Bus_Transit
manually imported from .txt and aggregated into one excel, saved as: data/processed/Delhi_Bus_GTFS.xlsx

(3) Open City, Population Data, ward-level 2021 (manual download)
https://data.opencity.in/dataset/delhi-census-2011-data/resource/372c35ad-ae9e-418f-a2e6-faa3351de767
saved as: /data/raw/Census 2011 Ward-Level Population Data (OpenCity).csv

(4) Delhi Spatial Boundaries (via query)
https://projects.datameet.org/Municipal_Spatial_Data/, 
https://github.com/datameet/Municipal_Spatial_Data/tree/master/Delhi

(5) Delhi Districts (via query)
https://github.com/HindustanTimesLabs/shapefiles,
https://github.com/HindustanTimesLabs/shapefiles/blob/master/city/delhi/district/delhi_1997-2012_district.json

(6) Metro Station Proximity (via query from ArcGIS)
https://services3.arcgis.com/9nfxWATFamVUTTGb/arcgis/rest/services/Spatial_Accessibility_of_the_Delhi_Metro_Network_WFL1/FeatureServer/1/query
see: code/metro_stations.ipynb 

(7) VIIRS Nighttime Lights from Earth Observation Group 
https://eogdata.mines.edu/products/vnl/#annual_v2
saved as: /data/raw/VNL_v21_npp_201204-201212_global_vcmcfg_c202205302300.average.dat.tif

# Data Handling 

data/processed/Delhi_Bus_GTFS.xlsx -> code/data_cleaning.ipynb -> data/processed/gtfs_combined.xlsx

data/processed/gtfs_combined.xlsx -> code/data_cleaning.ipynb -> data/clean/analysis_dataset_ward_level.csv

data/raw/2021.csv -> code/data_cleaning.ipynb -> data/clean/analysis_dataset_ward_level.csv

data/raw/Census 2011 Ward-Level Population Data (OpenCity).csv -> code/data_cleaning.ipynb -> data/clean/analysis_dataset_ward_level.csv
