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

![image](https://github.com/tuanhainc/Images/blob/9d434b7cc319aff4088e12425f0606e558c65131/mean_trip_duration.png)

Significantly, the mean trip duration for casual riders is much higher than for member riders, especially for docked bikes. A considerable gap between the mean and median means duration of casual trips is more scattered with more extreme values. Yet, it is safe to say that casual riders tend to have longer ride trips.

![image](https://github.com/tuanhainc/Images/blob/main/percentage_of_ride.png)

The reason for the much higher mean trip duration of casual riders could be explained by more-than-30-minute trips accounting for nearly 25% of total casual rides. Generally, member riders tend to prefer short-length trips.

![image](https://github.com/tuanhainc/Images/blob/main/trip_duration.png)

Since the majority of trips in our dataset are completed in under 60 minutes, I have chosen to narrow my analysis to records with durations of less than 110 minutes, as indicated in the graph above. When examining trip durations by bike type, it becomes evident that casual customers exhibit a pronounced preference for electric bikes, particularly for trips under 30 minutes.

## Seasonality
![image](https://github.com/tuanhainc/Images/blob/main/Commuting.png)

For members, spikes in total rides occur between 6-8 AM. Suggesting a significant number of riders commuting to work using Cyclistic membership.

![image](https://github.com/tuanhainc/Images/blob/main/Commuting2.png)

The previous presumption can be proved by checking the ride count distribution only on the weekends. The behaviors in the two groups remain remarkably similar.

## Attitude and distance
![image](https://github.com/tuanhainc/Images/blob/main/Attitude.png)

To see any relationship with locations, I plugged in the altitude variables of start stations. The concentration of starting points was distributed heavily along the coastline for member type. The same happened for casual customer counterparts but is slightly more scattered inland.  Generally, there is not much to say about the differences in locations between the two customer types here.

![image](https://github.com/tuanhainc/Images/blob/main/Attitude2.png)

However, a closer look at the data on weekends and weekdays reveals an interesting pattern. Member rides are denser on weekends, while casual rides follow the opposite trend. It is not yet to assume anything since the visualization uses the count of rides of two customer types proportionally with the total ride count of the whole data set.

![image](https://github.com/tuanhainc/Images/blob/main/Weekendandweekday.png)

To understand the usage in a more appropriately proportional manner, I used the percentage of ride count per weekday compared to the total count for a particular customer type. This reveals a surge in casual riders during weekends, indicating a preference for weekend rides, while members tend to favor weekday rides.

![image](https://github.com/tuanhainc/Images/blob/main/scatterplot.png)

The relationship between ride duration and distance does not appear to follow a linear regression pattern, likely because the meter variable represents the distance between start and end stations, which may not always align with the actual distance traveled. Interestingly, it seems there is a ceiling in the scatter plot at trips with around 1500-minute (24-hour) duration. Above the ceiling, presented only casual rides with docked bike type. In other words, casual riders are the ones renting bikes for trips lasting more than a day, and they exclusively choose docked bikes for such journeys.  
Another intriguing anomaly in the scatter plot is the vertical wall of plots for both customer types at zero-meter distance. I would calculate the percentage of ride count at zero distance compared to the total ride count of each customer type.

![image](https://github.com/tuanhainc/Images/blob/main/percentagereturntrips.png)

Zero-meter distance indicates that the riders returned to where they started at the end of the trip. A higher percentage of such return trips for casual customers could be attributed to their unfamiliarity with various bike return locations. However, the difference is insignificant and is not worth extra examination.

## Key insights
  -	**Trip duration:** Casual riders tend to opt for longer trips more compared to their member counterparts. Trips that are more than 1 day in duration are mostly from casual customers. 
  -	**Morning Commute Patterns:** Most riders who use Cyclistic for their daily work commutes are member riders.
  -	**Rideable type: **Casual riders have a stronger preference for electric bikes. Most docked bike users are casual customers.
  -	**Weekday vs. Weekend: **Members tend to favor weekdays for their rides, while casual customers have a stronger preference for weekend rides.

# Recommendation
To further answer the question of **“Why would casual riders buy Cyclistic annual memberships?”** necessitates extra output. For example, unique rider ID information is crucial to understanding the correlation and causation relationship between two customer types. From that, a strategic digital media campaign to influence casual riders to become members can be proposed.



