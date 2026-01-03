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
data/processed/gtfs_combined.xlsx,
data/raw/2021.csv, 
data/raw/Census 2011 Ward-Level Population Data (OpenCity).csv -> code/data_cleaning.ipynb -> data/clean/analysis_dataset_ward_level.csv
