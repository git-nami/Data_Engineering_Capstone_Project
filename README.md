# Weather_Effect_On_US_Immigration_Capstone:
### Project Summary:
As part of this project we will model the immigration and weather data of United States to understand the impact of temperature on the travel pattern of passengers coming to the United states over the past years for the past 50 years.

### Scope:
As part of this project we will model the immigration and weather data of the United States to understand the impact of temperature on the travel pattern of passengers coming to the United States over the past years for the past 50 years i.e since 1970s. More specifically we want to answer the questions below:
Temperature influence over the years on the choice of destination city in the U.S.
Temperature influence over the years on the choice of arrival date of travellers aggregated across the various date dimensions i.e. year, month, quarter etc.

### Source Datasets:
Below are the datasets that we will used across the project.
1. i94_apr16_sub.sas7bdat: This immigration data comes from the US National Tourism and Trade Office and contains information present on the I94 form which is the arrival/departure report card for the international visitors.
2. GlobalLandTemperaturesByCity.csv: This dataset contains the world temperature recorded over various years for various cities across countries. We would only be looking at the temperature data of United States since 1970s
3. I94PORT_lookup.csv: This file is generated based on the dictionary provided for i94port column in the I94_SAS_Labels_Descriptions.SAS file. Since we are only interested in US port of entry city and states, non US port of entries such as Canada, Mexico , Belgium etc as well as invalid, collapsed, no port code data have been dropped while creating the file.
4. I94CIT_I94RES_lookup.csv: This file is generated based on dictionary provided for i94cit and i94res column in the I94_SAS_Labels_Descriptions.SAS file.

### Target tables:
Below are the target table details:
1. Fact Table : fact_immigration
2. Dimension Tables: dim_temperature, dim_date
3. Format: parquet

### Conceptual Data Model:
1. fact_immigration: The table will contain the immigration details for travellers travelling in US.
i94_cicid
i94_country_origin
i94_country_residence
i94_port_code
i94_arrival_date
i94_dep_date
i94_mode_travel
i94_visa_type
i94_gender
i94_age
year
2. dim_temperature: The table will contain the temperature for USA across various cities post 1970.
i94_port_code
city
state
date
avg_temperature
avg_temperature_un
latitude
longitude
year
3. dim_date : The table will contain various date dimensions for the arrival dates from the immigration dataset.
date
year
month
day
week
weekday
quarter

### Data dictionary:
1. fact_immigration: The table will contain the immigration details for travellers travelling in the US.
i94_cicid = unique id assigned to each row obtained from i94_apr16_sub.sas7bdat(cicid)
i94_country_origin = coutry of origin obtained from I94CIT_I94RES_lookup.csv(value)
i94_country_residence = country of residence obtained from I94CIT_I94RES_lookup.csv(value)
i94_port_code = 3 character i94 port code specifying the port of entry in the USA obtained from i94_apr16_sub.sas7bdat(i94port)
i94_arrival_date = Arrival date in the USA obtained from i94_apr16_sub.sas7bdat(arrdate)
i94_dep_date = Departure date from the USA i94_apr16_sub.sas7bdat(depdate)
i94_mode_travel = Mode of travel obtained from i94_apr16_sub.sas7bdat(i94mode)
i94_visa_type = Visa type issued obtained from i94_apr16_sub.sas7bdat(i94visa)
i94_gender = Gender of the traveller obtained from i94_apr16_sub.sas7bdat(gender)
i94_age = Age of the traveller obtained from i94_apr16_sub.sas7bdat(i94bir and biryear)
year = Year of arrival date
2. dim_temperature: The table will contain the temperature for USA across various cities post 1970.
i94_port_code = 3 character i94 port code from the immigration dataset obtained from I94PORT_lookup.csv(code)
city = City where the temperature was recorded obtained from GlobalLandTemperaturesByCity.csv(city).
state = State details for the i94 port from the immigration dataset obtained from I94PORT_lookup.csv(state)
date = Date when the temperature was recorded obtained from GlobalLandTemperaturesByCity.csv(dt).
avg_temperature = Average temperature in the city obtained from GlobalLandTemperaturesByCity.csv(AverageTemperature).
avg_temperature_un = Average temperature uncertainity in the city obtained from GlobalLandTemperaturesByCity.csv(AverageTemperatureUncertainty).
latitude = latitude where ther temperature was recorded obtained from GlobalLandTemperaturesByCity.csv(Latitude).
longitude = longitude where the temperature was recorded obtained from GlobalLandTemperaturesByCity.csv(Longitude).
year = year of date when the tempertature was recorded.
3. dim_date : The table will contain various date dimensions for the arrival and deperature dates from the immigration dataset.
date = arrival date obtained from i94_apr16_sub.sas7bdat(arrdate)
year = year of arrival date
month = month of arrival date
day = day of arrival date
week = week of the year of arrival date
weekday = day of the week of arrival date
quarter = quarter of arrival date

### ETL Pipeline Design:
The ETL pipeline will leverage Spark's in memory processing capabilities which is very fast and will loads the data as follows:
1. Create a SparkSession and read the i94_apr16_sub.sas7bdat, GlobalLandTemperaturesByCity.csv, I94PORT_lookup.csv and I94CIT_I94RES_lookup.csv input files to create spark dataframes.
2. Select the desired columns for each of the target tables and perform the required transformations.
3. Save the transformed dataframe with the required partitioning in parquet format.

### Detail ETL Mapping Design:
1. fact_immigration:
--------------------
1. Read the following datasets and convert it into spark dataframe: i94_apr16_sub.sas7bdat, I94PORT_lookup.csv, I94CIT_I94RES_lookup.csv
2. Get the actual country names for the country of origin and country of residence codes values by joining i94_apr16_sub.sas7bdat and I94CIT_I94RES_lookup.csv dataframes.
3. Filter out the invalid port of entries by joining i94_apr16_sub.sas7bdat and I94PORT_lookup.csv dataframes.
4. Replace the mode of the travel, visa type codes with the actual values.Filter out the mode of travel values where mode of travel = land, air, sea.
5. Get the age of the travelling using the age and the birth year information.
6. Get the gender and cicid of the traveller.
7. Convert the arrival date and the departure date from SAS numeric format to yyyy-mm-dd format and filter out the invalid records.
8. Select the required columns to create the fact table and write it in parquet format partitioned by year of arrival date and i94 port code.
2. dim_temperature:
--------------------
1. Read the GlobalLandTemperaturesByCity.csv file and convert to spark dataframe
2. Check the temperature column for null values and filter out records where the average temperature columns value is null.
3. Check the other required column for null values
4. Join GlobalLandTemperaturesByCity.csv with the I94PORT_lookup.csv dataframes to get the i94 port of entry code and state based on the city column.
5. Select the required columns to create the temperature dimension table and write in parquet format partitioned by year of the date when the temperature was recorded.
3. dim_date:
--------------------
1. Get the arrival date from immigration fact table and extract various date dimensions such as year, month, day, week etc.
2. Create the date dimension table and write it in parquet format partitioned by year of arrival date.
