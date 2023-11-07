# data-512-wildfire-project
This repo contains the code, README, and other files necessary to complete the Data 512 end-of-term project focused on wildfire smoke estimations and recommendations.


## Goal
This goal of this project is to analyze wildfire smoke impacts on Pahrump, Nevada (Population 42,471, Nye County) with the intent of generating insights to inform policy makers, city managers, city councils, and other civic institutions to make informed plans for future wildfire mitigation.

We will begin by analyzing historical fire data to establish in-city smoke estimates. Then we will use this data to predict smoke over the next 25 years (2024-2049). Finally, we will address the issue of:
### EKRC INSERT HUMAN CENTERED QUESTION HERE


## Licenses & API Information

The code used in this project is licensed under the MIT license - more information can be found in the license file in the top-level repo.

The USGS Wildland Fire Data resides in the Public Domain. Data from the U.S. Government are freely redistributable with proper metadata and source attribution. The U.S. Geological Survey is the originator of these data.


## Data File Information

### Raw/Input Data Files

USGS_Wildland_Fire_Combined_Dataset - This data contains the combined polygons and additional fields of matching fires found across 40 datasources. There are six keys which we describe in detail below:
    displayFieldName - Empty value
    gemoetryType - Single value of "esriGeometryPolygon"
    spatialReference - Dictionary of {'wkid': 102008, 'latestWkid': 102008}
    fieldAliases - This dictionary contains each of the 30 field variables as keys, and their "English translations" as values. As an example, 'USGS_Assigned_ID':'USGS Assigned ID'. The variable names and their aliases are straightforward. For more information on what each field means, please see the USGS provided metadata as this information is not provided in the data itself.
    fields - This key returns a list of dictionaries which provides the name of each field, its type (e.g., esriFieldTypeOID, esriFieldTypeInteger), its alias, and occasionally the length of the field (e.g., 100, 300000)
    features - This key returns what we will be using as the fire data. It is a list of 135,061 'attribute' dictionaries, each of which also contains a dictionary with the fields specified in "fields" or "fieldAliases". 

Wildland_Fire_Polygon_Metadata.xml - This file contains text metadata about the fire data. It will not be parsed and analyzed, but is an indespensible reference for data users.

  
### Intermediate Data Files Created During Runtime

fire_distances.csv - This file is created by calculating the distance between each fire in the USGS wildland fire combined data and Pahrump, NV. It has two columns:
    OBJECTID - the OBJECTID of the fire in question
    shortest_dist - the shortest distance between the edge of the fire and Pahrump

close_fires.csv - This file has the same fields as the above, but only contains fires within 1250 miles from town.

fire_features.csv - This file takes the relevent fire features for fires occurring after 1963 (inclusive) from the USGS wildland fire combined data which will later be used to create a smoke estimate. It has the following columns:
    objectid - the fire's objectid used to uniquely identify it
    Assigned_Fire_Type - what kind of fire was reported. Values include wildfire, likely wildfire, unknown - likely wildfire, prescribed fire, unknown - likely prescribed fire
    Fire_Year - the year the fire took place
    GIS_Acres - acres of the fire polygon calculated using the Calculate Geometry tool in ArcGIS pro
    Overlap_Within_1_or_2_Flag - areas that burned with >10% overlap of the current fire within 1 or 2 years of the current burn as determined by ArcGIS Tabulate Intersection Tool

filtered_fire_info.csv - This file combines the fire_distances and fire_features information above to keep select information only on the fires which are within 1250 miles from Pahrump and occurred after 1962 (inclusive). The columns are as follows:
    OBJECTID - the OBJECTID of the fire in question
    Assigned_Fire_Type - what kind of fire was reported. Values include wildfire, likely wildfire, unknown - likely wildfire, prescribed fire, unknown - likely prescribed fire
    Fire_Year - the year the fire took place
    GIS_Acres - acres of the fire polygon calculated using the Calculate Geometry tool in ArcGIS pro
    Overlap_Within_1_or_2_Flag - areas that burned with >10% overlap of the current fire within 1 or 2 years of the current burn as 
    shortest_dist - the shortest distance between the edge of the fire and Pahrump


### Cleaned Data Files

annual_smoke_estimate.csv - This file contains the annual smoke estimates for Pahrump, NV. 0 would indicate no smoke all year. Higher values indicate worse smoke over the course of the year. Columns are as follows:
    Fire_Year - Year the fires occurred in
    Annual_Smoke_Estimate - Annual smoke estimate created by averaging individual smoke estimates over the fire year
    


## Known Data Issues and Special Considerations

### USGS_Wildland_Fire_Combined_Dataset Issues & Considerations

   Fire perimeter data is never 100% accurate, and data originating prior to 1980 especially underestimates the actual number of fires. Boundaries should be assumed to be approximate area burned. It is impossible to know how many fires were missed,  misattributed, or mismapped in the original, and now combined, datasets. Circular fires indicated by a circle-ness ratio approaching 1 (>= 0.98) may be lightning strikes or fires where the true boundary was unknown and a user buffered a point.
    The data creators assume land can only burn once per year (usually true), thus if an area is listed as burned twice in one year it is counted as one burn. Fires which burned in the same year within 500m of each other were also counted as the same fire, potentially reducing total fire count. Entries with identical fire boundaries in subsequent years are flagged as they likely represent the same fire in two datasets with a mislabeled year. Wildfires without years were removed from the dataset by the creators as much as possible.
    Per documentation, "Prescribed_Burn_Notice Prescribed fire data in this dataset represents only a fraction of the area burned in prescribed burns across all years due to lack of reporting, particularly on private lands. The missing prescribed burn data becomes more pronounced further back in time, particularly in the southeastern U.S.; however, errors and omissions still occur through the most recent years in this dataset."
    Again, per documentation, "All fires that were clearly labeled as wildfires or prescribed fires were labeled as such. Remaining fires in wildfire datasets were labeled as "likely wildfire". Fires from the Monitoring Trends in Burn Severity Dataset marked as "Unknown" were labeled as "Unknown - Likely Wildfire" if a wildfire report existed for these fires. Otherwise, they were labeled as "Unknown - Likely Prescribed Fire". Additional fields were added to each dataset to create common fields across all original datasets. These fields included Fire Source, Fire Tier, Fire Name, Fire Code, Fire Year, multiple fire dates, ignition source (human or natural), map method and notes."
    Because data was combined from multiple sources, field entries may not exactly match across them. Given considerable concern about data quality, all attributes returned for a single fire are captured with parenthetical numbers indicating how many time the entry appeared. To use an example from the documentation, "if 10 records contained a polygon for the Soda wildfire the script looped through the fire names and assembled them in the Fire_Name field count. Using this example, let's say that 5 rows had a fire name of "Soda", 3 had "SODA", and two had "soda". The resulting Fire_Name attribute in the USGS_Wildland_Fire_Combined_Dataset for that row would appear like this: "Soda (5), SODA (3), soda (2)" where the parenthetical values represent the count of times that attribute appeared." Dates are also combined into a single field to avoid favoring any single date. They use the same parenthetical numberical indication for the number of times the date appeared. Date types may not be identical.
    Fire data in our source was only collected until 2020, so there will be a meaningful data gap between 2020 and 2023.

### fire_distances.csv Issues & Considerations
   Our current fire distance calculator only runs for fires with a "ring" geometry. 35 fires in the USGS_Wildland_Fire_Combined_Dataset have "curve ring" shapes. Due to their incompatability with our processing methods we will discard them from the data.
   
### annual_smoke_estimate.csv Considerations
   We calculate the previously burned acreage per fire in our smoke estimate to discount the amount of smoke it will product. Unfortunately not all entries are able to be parsed uniformly. In the event that the percentage of fires with unreadable pre-burned acres is less than 10%, we will assume no-preburned acres for those fires. Otherwise, the data_processing program will print a warning message to the user.
    
### FINAL EPA ESTIMATE CONSIDERATIONS
   EPA AQI data is available year round. However, as mentioned in our epa_comparison code, because we are comparing it with our smoke estimate which is primarily gathered in fire season (May 1st - Oct 31st) we will limit our AQI data to information taken May 1st - Oct 31st annually. Additionally, while some stations may produce granular AQI measurements (e.g., on the hourly scale), "The Air Quality Index is based on daily air quality summaries, specifically daily maximums or daily averages. It is not valid to use shorter-term (e.g. hourly) data to calculate an AQI value." [Technical Assistance Document for the Reporting of Daily Air Quality – the Air Quality Index (AQI)](https://www.airnow.gov/sites/default/files/2020-05/aqi-technical-assistance-document-sept2018.pdf) Due to this standard, we will only use the 24-HR BLK AVG AQI measurement for each gas/particulate.
    After pulling the data we find that some sensors do not collect any data (e.g., sensors 0001, 0002, 0003, 0004), and our local sensors only collect information on particulate matter with a diameter of 10 microns or less (PM10 Total 0-10um STP, code 81102). Per the [California Air Resources Board](https://ww2.arb.ca.gov/smokereadyca#:~:text=Particles%20from%20smoke%20can%20be,pass%20directly%20into%20the%20bloodstream), "Particles from smoke can be very small (with diameters of 2.5 micrometers and smaller)" so we are still capturing some wildfire air quality impacts even with dimished gas/particulate reporting.

## Anaysis Reproduction Steps

1. To create file smoke estimates we need historical fire data. That information is provided by the USGS at this site: https://www.sciencebase.gov/catalog/item/61aa537dd34eb622f699df81. Download GeoJSON Files.zip “Wildland Fire Polygons Fire Feature Data Open Source GeoJSON Files”. This may take a few minutes. 

2. Unzip the downloaded zip file. Inside the contained "GeoJSON Exports" folder you will see combined and merged datasets. Per the USGS website, "These datasets were created by combining 40 different, published wildland fire data sources. Each one of these data sources has a different spatial scale, spatial resolution, and time period for their particular wildland fire dataset. The purpose of these new datasets is to combine these disparate wildfire datasets, using a common set of attributes, into a single set of polygons with a single fire boundary for each fire. This dataset is intended to create a more comprehensive fire dataset than the existing datasets while eliminating duplication of fire polygons and attributes." We will be using only the "combined" datasets to avoid the duplication of fires in the "merged" datasets.

3. Copy or move the "Wildland_Fire_Polygon_Metadata.xml" into the "raw_data" folder of your repo. The Metadata file does not contain analyzable data, but is useful for understanding the raw data more thoroughly. 

4. Save your "USGS_Wildland_Fire_Combined_Dataset.json" file to the directory above your parent directory with the raw data, clean data, code etc. We used GitHub to manage code and the JSON is larger than their file systems will allow for. For this reason, we must reduce the data first, then save the data in an intermediate_data folder (output of data_acquisition code below).

5. Scripts in the data_filtering.ipynb will require both the [Pyproj](https://pyproj4.github.io/pyproj/stable/index.html), the [geojson](https://pypi.org/project/geojson/) module and the wildfire user module. Pyproj and geojson can be installed via pip. The wildfire user module should be downloaded from the course website, unzipped, and moved into the folder pointed to by your PYTHONPATH system variable.

6. Run the data_acquisition script located in scr/ to select information on fires in 1963-2023 which were within 1250 miles from Pahrump, NV.

7. Run the data_processing script located in scr/ to create the fire smoke estimators for Pahrump. Significant assumptions and judgements went in to defining and calculating the smoke estimates, these decisions are captured in the "Purpose" section of the script.

8. If you're interested in understanding how well our smoke estimate compares to the EPA's Air Quality Index, check out the epa_comparison script in scr/.


