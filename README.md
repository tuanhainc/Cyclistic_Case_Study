# Background
## About the company
In 2016, Cyclistic launched a successful bike-share offering. Since then, the program has grown to a fleet of 5,824 bicycles that are geotracked and locked into a network of 692 stations across Chicago. The bikes can be unlocked from one station and returned to any other station in the system anytime.  
The director of marketing believes the company’s future success depends on maximizing the number of annual memberships. Therefore, the company wants to understand how casual riders and annual members use Cyclistic bikes differently. Cyclistic will use these insights for a marketing program development aiming to transform casual into annual riders.
## Business task
This report investigates Cyclistic’s historical trip data, including 12-month datasets across different years. All data aggregation, cleaning, and manipulation are recorded here. Ultimately, the paper uses data insights and compelling visualization to answer the main question:  

  **-	How do annual members and casual riders use Cyclistic bikes differently?**
  
# Preparation
## Metadata summary
There are 38 CSV files from Cyclistic’s trip data ranging between April 2020 to June 2023, representing each month of according year. Each file includes 13 variables. Below is the summary of the variables:

| Field Name       | Summary                                       | Example                     |
|------------------|-----------------------------------------------|-----------------------------|
| ride_id          | ID attached to each trip taken               | "110A0B8E360392EC"         |
| rideable_type    | Type of bike                                  | "classic_bike", "docked_bike", "electric_bike" |
| started_at       | Day and time trip started, in CST            | "2021-08-28 11:57:03"      |
| ended_at         | Day and time trip ended, in CST              | "2021-08-28 12:35:22"      |
| start_station_name  | Name of station where trip originated       | "Wells St & Concord Ln"    |
| start_station_id    | ID of station where trip originated         | "TA1308000050", "chargingstx5", etc. |
| end_station_name    | Name of station where trip terminated       | "Shedd Aquarium"           |
| end_station_id      | ID of station where trip terminated         | "15544", "TA1307000130", etc. |
| start_lat        | Latitude of station where trip originated    | "41.940055846999996"       |
| start_lng        | Longitude of station where trip originated   | "-87.652963877"           |
| end_lat          | Latitude of station where trip terminated    | "41.954383"                |
| end_lng          | Longitude of station where trip terminated   | -87.648043                 |
| member_casual    | "Casual" is a rider who purchased a 24-Hour Pass; "Member" is a rider who purchased an Annual Membership | "member", "casual" |


Note that there were many incomplete fields, data missing most often from station name, ID, and latitude variables, as well as many fields that were likely invalid. This section, however, remains only a quick preview of the data. Further data cleaning is discussed in the next section.

## Merging data
After making sure every file contains a consistent number of fields, I proceeded to merge all the files into one aggregated file for cleaning and analysis. I aggregated the 36 CSV files into one consistent table using Power Query and exported them in a large CSV format file by the external tool Dax Studio. The CSV file was later imported into Google Cloud Storage and uploaded to SQL BigQuery. The total number of records for the aggregated data is 12,159,101 rows.
## Summary
  -	36 CSV files, representing historical trip data from April 2020 through June 2023 were collected and merged. The aggregated data has a total of 12,159,101 rows.
  -	Data preview revealed many empty rows, and missing data from station name, ID, and latitude variables.
  -	Data was trimmed during the merging process to eliminate extra spaces.

# Process
SQL Query: [Data Cleaning and Processing](https://github.com/tuanhainc/Cyclistic/blob/main/SQL%20Query%20-%20Data%20Cleaning%20and%20Processing)
## Data cleaning
This section inspects data potential faulty format, typos, missing values, and duplication. The cleaning strategy includes removing unnecessary values, duplication, and anomalies, using clear format, and converting fields into appropriate data types.  
With the ride_id variable having only unique values, any duplication of ride_id would be removed. Trips with no end destination, in this case, having no station altitude, name, and ID that is impossible to retrieve using the remaining information, would also be removed. The schema for cleaned data trip data would be as below:

| Field Name         | Data Type  | Mode      |
|--------------------|------------|-----------|
| ride_id            | STRING     | REQUIRED  |
| rideable_type      | STRING     | REQUIRED  |
| started_at         | TIMESTAMP  | REQUIRED  |
| ended_at           | TIMESTAMP  | REQUIRED  |
| start_station_name | STRING     | NULLABLE  |
| start_station_id   | STRING     | NULLABLE  |
| end_station_name   | STRING     | NULLABLE  |
| end_station_id     | STRING     | NULLABLE  |
| start_lat          | FLOAT      | NULLABLE  |
| start_lng          | FLOAT      | NULLABLE  |
| end_lat            | FLOAT      | NULLABLE  |
| end_lng            | FLOAT      | NULLABLE  |
| member_casual      | STRING     | REQUIRED  |

Total records of 240,533 rows were removed during this process. It is worth noting that I decided not to remove every row with missing values in the field of station name, ID, and altitude.

## Data process
In this section, the data is further processed to add new fields for analysis. I dissected the 'started_at' and 'ended_at' fields and broke them down into year, month, day, weekday, and hour components. This segmentation serves the purpose of facilitating a more convenient analysis of rider behaviors across various timeframes.  
Regarding location, it is trickier to make use of station information given the mismatch across the names, IDs, and altitude values. However, there is no null value for the altitude variables after removing trips with no end destination (rows with null values for all four variables: name, ID, and altitude values). Therefore, I only focused on the latitude and longitude variables moving forward. These variables would be rounded up to three decimal places for the sake of consistency, and processing time reduction while still adequately reflecting station locations (the accuracy of 3 decimal places is 111 meters).  
Duration is a new field added by subtracting the started time and the ended time. Duration is expressed in minutes and is the time interval of a single trip. Rows with negative or zero value duration, which does not make sense, were removed.  
Distance is measured in meters, and is the length from the start station to the end station. This was calculated based on the coordinates of the start and end stations. Although this measurement does not truly reflect the actual distance traveled by a rider, it could be useful in investigating customers’ behavior by looking at its relationship with travel duration.

## Summary
  -	Rows having duplicate trip IDs were removed.
  -	Rows with no end destination were removed.
  -	Rows where “started_at” is greater than “ended_at” were removed.
  -	A total of 240,533 rows were removed during the cleaning process.
  -	Year, month, day, weekday, and hour components were added derived from “started_at” and “ended_at”
  -	Trip duration field was added.
  -	Distance between start and end stations was added.

# Analyze
Tools used in this section are SQL in combination with Tableau for data analysis and visualization.

## Trip duration
SQL Query:  
[Mean Trip Duration](https://github.com/tuanhainc/Cyclistic/blob/main/SQL%20Query%20-%20Calculate%20Median)  
[Ride Count Percentage in Different Duration Length](https://github.com/tuanhainc/Cyclistic/blob/main/SQL%20Query%20-%20Ride%20Count%20Percentage%20per%20Trip%20Duration%20Length)

![image](https://drive.google.com/file/d/1_djY3hbP4muS1-nkTpLwTfVwBwX7n3i3/view?usp=drive_link)

Significantly, the mean trip duration for casual riders is much higher than for member riders, especially for docked bikes. A considerable gap between the mean and median means duration of casual trips is more scattered with more extreme values. Yet, it is safe to say that casual riders tend to have longer ride trips.

