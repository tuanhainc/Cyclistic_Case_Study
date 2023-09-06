# Background
## About the company
In 2016, Cyclistic launched a successful bike-share offering. Since then, the program has grown to a fleet of 5,824 bicycles that are geotracked and locked into a network of 692 stations across Chicago. The bikes can be unlocked from one station and returned to any other station in the system anytime.
The director of marketing believes the company’s future success depends on maximizing the number of annual memberships. Therefore, the company wants to understand how casual riders and annual members use Cyclistic bikes differently. Cyclistic will use these insights for a marketing program development aiming to transform casual into annual riders.
Business task
This report investigates Cyclistic’s historical trip data, including 12-month datasets across different years. All data aggregation, cleaning, and manipulation are recorded here. Ultimately, the paper uses data insights and compelling visualization to answer the main question:
  
  **-	How do annual members and casual riders use Cyclistic bikes differently?**
  
Preparation
Metadata summary
There are 38 CSV files from Cyclistic’s trip data ranging between April 2020 to June 2023, representing each month of according year. Each file includes 13 variables. Below is the summary of the variables: 
Field Name	Summary	Example
ride_id	ID attached to each trip taken	“110A0B8E360392EC”
rideable_type	Type of bike	“classic_bike”, “docked_bike”, “electric_bike”
started_at	Day and time trip started, in CST	“2021-08-28 11:57:03”
ended_at	Day and time trip ended, in CST	“2021-08-28 12:35:22”
start_station_name	Name of station where trip originated	“Wells St & Concord Ln”
start_station_id	ID of station where trip originated	“TA1308000050”, “chargingstx5”, etc.
end_station_name	Name of station where trip terminated	“Shedd Aquarium”
end_station_id	ID of station where trip terminated	“15544”, “TA1307000130”, etc.
start_lat	Latitude of station where trip originated	“41.940055846999996”
start_lng	Longitude of station where trip originated	“-87.652963877”
end_lat	Latitude of station where trip terminated	“41.954383”
end_lng	Longitude of station where trip terminated	-87.648043
member_casual	"Casual" is a rider who purchased a 24-Hour Pass; "Member" is a rider who purchased an Annual Membership	“member”, “casual”

Note that there were many incomplete fields, data missing most often from station name, ID, and latitude variables, as well as many fields that were likely invalid. This section, however, remains only a quick preview of the data. Further data cleaning is discussed in the next section.

