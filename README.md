# Data 512 Project: Wildfire Analysis

### Project Goal

The goal of this project is to analyze the impact of wildfires in the U.S, specifically near Murfreesboro, TN. Using wildfire data from [here](https://www.sciencebase.gov/catalog/item/61aa537dd34eb622f699df81), this project estimated the smoke impact of wild fires within 650 miles of Murfreesboro, TN and compares this estimate to AQI data from US EPA Air Quality System (AQS) API. Additonally, a predictive model predicts the smoke impact estimates until 20250. The fires we are between the years 1961-2021 and we define wildfire season as May 1 - October 31. The analysis is continued further to inlcude public health into the smoke impact estimate. IHME data on chronic respiratory deaths is added into the model to predict the smoke impact, including this public health impact, till 2050.

### Licensing
The starter code is adpated from Dr. McDonald's example notebooks, wp_page_info_example.ipynb and wp_ores_liftwing_example.ipynb which are licensed [CC-BY](https://creativecommons.org/licenses/by/4.0/). The following changes were made:
- Added my own API key information
- Added my own city data

All other code not designated as starter code from Dr. McDonald is created by Elizabeth Holden and licensed [CC-BY](https://creativecommons.org/licenses/by/4.0/).

### Repository Structure

```plaintext
Wildfire-Analysis/
├── examples/
    ├──epa_air_quality_history_example.ipynb
    └──wildfire_geo_proximity_example.ipynb
├── generated_data/
    ├──distance_df.csv
    ├──smoke_impact_fires_650miles.csv
    └──year_mean.csv
├── raw_data/
    └── IHME_deaths.csv
├── notebooks/
    ├──wildfire.ipynb
    ├──aqi.ipynb
    ├──analysis.ipynb
│   └──smoke_impact_extension.ipynb
├── wildfire/
    ├──Reader.py
    ├──Wildfire_short_sample_2024.json
    └──__init__.py
├── docs
    ├── Project Part 1 Write-up.pdf
    ├── HCD-Presentation.pptx
    └── Project Part 4 Final Report.pdf
├── LICENSE
└── README.md
```
### Data Sources
- Wildfire data: The wildfire data comes from [here](https://www.sciencebase.gov/catalog/item/61aa537dd34eb622f699df81). The file is too large to upload to the repository, but the file used in analysis was the USGS_Wildland_Fire_Combined_Dataset.json file from the GeoJSON download.
- AQI data: The AQI data came from the US EPA Air Quality System (AQS) API.
- Chronic Respiratory Illness Death data: The chronic respiratory illness death data comes from IHME's Global Burden of Disease Study. The link to access this data is [here](https://vizhub.healthdata.org/gbd-compare/). You must use the visualization tool to filter for the state of Tennessee, all age groups, both genders, the death metric, and for the disease.

### Files
- examples/ This directory contains files provided by Dr. David McDonald
- generated_data/ This contains intermediate files generated by the notebooks
    - distance_df.csv: Contains data about fires within 1800 miles of Murfreesboro. The file contains the following columns:
        - year (type int64)
        - size (type float64)
        - distance (type float64)
    - smoke_impact_fires_650miles.csv: Contains smoke impact estimates for fires within 650 miles of Murfreesboro. This file contains the following columns:
        - year (type int64)
        - size (type float64)
        - distance (type float64)
        - smoke_impact_estimate (type float64)
    - year_mean.csv: Contains avialble AQI estimates from monitoring station closest to Murfreesboro, TN. This file contains the following columns:
        - year (type float64)
        - max_aqi (type float64)
- raw_data/ This contains data downloaded directly from a source
    - IHME_deaths.csv: Contains hronic respiratory illness deaths per year for Tenessee from IHME. This file contains the following columns:
        - Location (type object)
        - Year (type float64)
        - Age (type object)
        - Sex (type object)
        - Cause of death or injury (type object)
        - Measure (type object)
        - Value (type float64)
        - Lower bound (type float64)
        - Upper bound (type float64)
- notebooks/ This contains notebooks used for the project
    - wildfire.ipynb: This notebook collects and processes all wildfire data and calculates smoke impact estimate. This produces the distance_df.csv file and the smoke_impact_fires_650miles.csv file.
    - aqi.ipynb: This notebook collects and processes all AQI data and calculates AQI estimates. This produces the year_mean.csv file.
    - analysis.ipynb: This notebook contains all the visualizations and analysis between smoke impact estimate and AQI, inlcuding predictive model
    - smoke_impact_extension.ipynb: This notebook contains all the visualizations, analysis, and modeling for the smoke impact with IHME chronic respiratory illness data
- wildfire/ This directory is from Dr. McDonald which contains Reader.py module to help read in the geojson wildfire data
- docs/ This directory contains documents required for this project

#### Model
ARIMA was used for modeling in analysis.ipynb and in smoke_impact_extension.ipynb, documentation for this model can be found [here](https://en.wikipedia.org/wiki/Autoregressive_integrated_moving_average) and [here](https://www.statsmodels.org/stable/generated/statsmodels.tsa.arima.model.ARIMA.html). Linear regression was used in smoke_impact_extention.ipynb, implimentation documentation can be found [here](https://scikit-learn.org/1.5/modules/generated/sklearn.linear_model.LinearRegression.html).


### Considerations
#### Distance:
Distance is calculated as the average distance of all perimeter points to the city, not quite what the centroid would be, but it is probably fairly close.
#### Smoke Impact Estimate:
The smoke impact estimate calculation was based off of the [Inverse Square Law](https://en.wikipedia.org/wiki/Inverse-square_law) which states that the '"intensity" of a specified physical quantity is inversely proportional to the square of the distance from the source of that physical quantity.' The calculation is $intensity \times \left(\frac{1}{distance^2}\right)$. However, after discussing with Sarah K., to ensure that the calculation is never dividing 0, I added 1 to the distance, so my calculation for each fire smoke impact estimate is $intensity \times \left(\frac{1}{(distance + 1)^2}\right)$. After calculating the smoke impact estimate for each fire, I created the yearly estimate by summing all the smoke impact estimates for each year. This calculation accounts for a decrease in value for when the fire is further away but also accounts for an increase in value when a fire is buring more acres.
#### AQI Estimate:
My county only had 1 monitoring station that had data for SO2, NO2, and O2 for the years 1988-2012. After reading [this](https://document.airnow.gov/technical-assistance-document-for-the-reporting-of-daily-air-quailty.pdf) techincal document about how the AQI is calculated, I understood if mutltiple particles had a AQI score, the highest score out of all the available data was taken as the AQI score for that period. Taking this into account, I gathered all AQI data for the fire seasons, May 1 through October 31, years that were availble. Next, if a day had mutiple AQI values for the same day, I took the average of them to get a value for every day for each particle. Then I took the max of all the particles to get a daily AQI value. Then I took the average per year to get the AQI estimate for the year. This might not be reflective fo the true AQI value for my county since I did not have access to all the data points due to lack of monitoring station data.
#### Comparing Smoke Impact Estimate and AQI Estimate:
To compare the smoke impact estimate and AQI estimate, I plotted both of these values. Overall, both estimates are slightly moving up and to the right as years go on, but they do not have the same peaks, increases, or decreases. Additonally, I computed the correlation between the smoke impact estimate and aqi estimate and got that the value 0.279829 which indicates a slight positive correlation. Since the smoke impact estimate is a sum of all values per fire and AQI is an average for every day across a year, these values are hard to compare. Not only is the calculation making these values hard to compare, they are composed of different aspects. The AQI estimate that I had is based off of SO2, NO2, and O2, and NO2 is the only particle that is present in smoke, so the other 2, SO2 and O2, might not be a good indicator of smoke.
#### Modeling with Chronic Respiratory Illness Data:
Many assumptions were made during the analysis of smoke impact with the chronic respiratory illness deaths:
- The IHME Global Burden of Disease data for chronic respiratory deaths that was used is for the whole state of Tennesse, so I am assuming that the same trend applies to the city of Murfreesboro as well.
- The IHME Global Burden of Disease data for chronic respiratory deaths that was used is only from 1990-2020, so the smoke impact estimate model including chronic respiratory deaths is from 1990-2050.
- The IHME Global Burden of Disease data for chronic respiratory deaths tracks all deaths from chronic respiratory illnesses, not just those that may be related to wildfire smoke exposure. Chronic respiratory diseases can be caused by lots of factors, like genetics, smoking, or other environmental pollutants, so it’s hard to pinpoint how much wildfire smoke specifically contributes to these deaths. This means that any conclusions drawn from this analysis should be taken with caution since other causes could be at play.
- I have also assumed when predicting the chronic respiratory illness deaths that they will follow the same trend as they did historically, and keep increasing. However, due to factors like medical advances, this might not hold.
