---
Title: "Cyclistic Case Study"
Author: "Lakshmi Kant"
Date: "`11-Feb-2023`"
Output:
  pdf_document: default
  html_document: default
---
![image](https://user-images.githubusercontent.com/43418706/235446898-ab20faf9-0221-4ad0-a890-04ebb0bdf1b7.png)

## Cyclistic Case Study with R

### Background Scenario

Cyclistic is an imaginary bike-sharing company that operates in Chicago, Illinois. My assignment was to work as a data analyst on the marketing team.

Cyclistic has two types of customers: annual members and casual riders. The director of marketing believes that the company's future success depends on maximizing the number of members. She asks the marketing team to design a new strategy to convert casual riders into annual members based on an analysis of how casual riders and annual members use Cyclistic bikes differently. The new strategy should focus on digital media like email, social media and other channels.


### Business Questions

1. How do annual members and casual riders use Cyclistic bikes differently?
2. Why would casual riders buy Cyclistic annual memberships?
3. How can Cyclistic use digital media to influence casual riders to become members?

*My Hypothesis: Members and casual customers are fundamentally different and their purposes for using Cyclistic are fundamentally different. Members tend to be locals who commute or use a bikeshare for errands and occasional transportation and casual riders tend to be tourists who are using a bikeshare to sightsee. It might be very difficult to convert casual riders into those wanting a membership.*


### Stakeholders

* Director of Marketing
* Cyclistic Executive team

### Setting up my environment
```{r setup, include=FALSE}
```
```{r}
tinytex::install_tinytex() #to export to pdf
```


```{r message=FALSE, warning=FALSE}
library(tidyverse)
library(ggplot2)
library(lubridate)
library(dplyr)
library(readr)
library(janitor)
library(data.table)
library(tidyr)
library(hms)
```

## Preparing the Data

12 months of Cyclistic's historical trip data was downloaded in separate .csv files, dating from July 2021 to June 2022.  The data we are working on is of first-party type — data collected and used by Cyclistic. For the purposes of this capstone project, the data was provided under a license from divvybikes.com, a real bike-share company operating in Chicago. Riders' personal information has been scrubbed.

#### Loading .csv files

```{r message=FALSE, warning=FALSE}
jun22_df <- read_csv("C:/Users/lk/Downloads/divvy-tripdata/202206-divvy-tripdata.csv")
may22_df <- read_csv("C:/Users/lk/Downloads/divvy-tripdata/202205-divvy-tripdata.csv")
apr22_df <- read_csv("C:/Users/lk/Downloads/divvy-tripdata/202204-divvy-tripdata.csv")
mar22_df <- read_csv("C:/Users/lk/Downloads/divvy-tripdata/202203-divvy-tripdata.csv")
feb22_df <- read_csv("C:/Users/lk/Downloads/divvy-tripdata/202202-divvy-tripdata.csv")
jan22_df <- read_csv("C:/Users/lk/Downloads/divvy-tripdata/202201-divvy-tripdata.csv")
dec21_df <- read_csv("C:/Users/lk/Downloads/divvy-tripdata/202112-divvy-tripdata.csv")
nov21_df <- read_csv("C:/Users/lk/Downloads/divvy-tripdata/202111-divvy-tripdata.csv")
oct21_df <- read_csv("C:/Users/lk/Downloads/divvy-tripdata/202110-divvy-tripdata.csv")
sep21_df <- read_csv("C:/Users/lk/Downloads/divvy-tripdata/202109-divvy-tripdata.csv")
aug21_df <- read_csv("C:/Users/lk/Downloads/divvy-tripdata/202108-divvy-tripdata.csv")
jul21_df <- read_csv("C:/Users/lk/Downloads/divvy-tripdata/202107-divvy-tripdata.csv")

```

#### ROCCC approach is used to determine the credibility of the data

**Reliable** – It is complete and accurate and it represents all bike rides taken in the city of Chicago for the selected duration of our analysis.

**Original** - The data is made available by Motivate International Inc. which operates the city of Chicago’s Divvy bicycle sharing service which is powered by Lyft.

**Comprehensive** - the data includes all information about ride details including starting time, ending time, station name, station ID, type of membership and many more.

**Current** – It is up-to-date as it includes data until end of June 2022.

**Cited** - The data is cited and is available under the Data License Agreement.


## Processing Data

#### Random Sampling to determine confidence level and margin of error 

```{r}
df <-read_csv("C:/Users/lk/Downloads/divvy-tripdata/202107-divvy-tripdata.csv",col_types=cols(start_station_id=col_character(),end_station_id = col_character()))
sample_df <- sample_n(df, 767554, replace=F)
write_csv(sample_df, "sample_dataset.csv")
print(sample_df)
```


Sample size is calculated as follow:

Population size: 5,900,385

Confidence level: 99.99%

Margin of Error: 0.2

Sample size: 767,050 (13%)


#### *Data Limitations

Checking data for completeness shows that “start station name and ID” and “end station name and ID” for some rides are missing. Further observations suggest that the most missing data about “start station name” belongs to “electric bikes” as 201,975 out of 888,490 electric ride shares have missing data and it accounts for 22% of total electric-bike ride shares.

Additionally, because of data privacy issues, riders are not personally identified and, although each ride ID was checked and confirmed to be distinct, there is no way to tell if the same rider has purchased multiple single passes, or if they live in the Cyclistic service area.

This limitation could affect our analysis for electric bike use, and for profiling casual riders beyond our hypothesis.


## Cleaning

Combined individual month datasets into one data frame

```{r message=FALSE, warning=FALSE}
cyclistic_df <- rbind(jun22_df, may22_df, apr22_df, mar22_df, feb22_df, jan22_df, dec21_df, nov21_df, oct21_df, sep21_df, aug21_df, jul21_df)
```

Create new dataframe with columns that will be analyzed only

```{r message=FALSE, warning=FALSE}
trimmed_cyclistic_df <- select(cyclistic_df, "rideable_type", "started_at", "ended_at", "member_casual")

```
Create new data frame to contain new columns

```{r}
cyclistic_new_df <- trimmed_cyclistic_df

```
Create ride_length column
```{r}
cyclistic_new_df$ride_length <- difftime(cyclistic_new_df$ended_at, cyclistic_new_df$started_at, units = "mins")
cyclistic_new_df$ride_length <- round(cyclistic_new_df$ride_length, digits = 1)
```
Create columns for all date calculations

```{r message=FALSE, warning=FALSE}
cyclistic_new_df$date <- as.Date(cyclistic_new_df$started_at) 
cyclistic_new_df$day_of_week <- weekdays(cyclistic_new_df$started_at) #day of week calculation
cyclistic_new_df$day_of_week <- format(as.Date(cyclistic_new_df$date),"%A") #day of week column
cyclistic_new_df$month <- format(as.Date(cyclistic_new_df$date), "%m") #month column
cyclistic_new_df$day <- format(as.Date(cyclistic_new_df$date), "%d") #day column
cyclistic_new_df$year <- format(as.Date(cyclistic_new_df$date), "%Y") #year column
cyclistic_new_df$time <- format(as.Date(cyclistic_new_df$date), "%H:%M:%S") #time formatted HH:MM:SS
cyclistic_new_df$time <- as_hms(cyclistic_new_df$started_at) #time column
cyclistic_new_df$hour <- hour(cyclistic_new_df$time) #create new column for hour
```

Remove where ride_length is 0 or negative
```{r}
cyclistic_new_df <- na.omit(cyclistic_new_df) #remove rows with NA
cyclistic_new_df <- distinct(cyclistic_new_df) #remove duplicate rows
cyclistic_new_df <- cyclistic_new_df[!(cyclistic_new_df$ride_length <=0),] 
```

## Analyzing

#### Calculated Quick Stats for Dashboard

Total number of rides = 5,900,385

Average Ride Length = 20.28 minutes

Busiest Time = 5pm

Busiest Weekday = Saturday

Busiest Month = July

Busiest Season = Summer

Most popular bike type = Classic

Most rides = Members  


#### Calculated stats by customer type to analyze differences

* Total Rides          
  + **Member** 3,335,388      
  + **Casual** 2,554,813

* Average Ride Length  
  + **Member** 13.01 minutes  
  + **Casual** 29.86 minutes

* Busiest Time         
  + **Member** 5pm            
  + **Casual** 5pm

* Busiest Weekday      
  + **Member** Thursday       
  + **Casual** Saturday

* Busiest Month       
  + **Member** September      
  + **Casual** July

* Most Popular Bike   
  + **Member** Classic        
  + **Casual** Classic
  
* Ride Length by Weekday
  + **Member** Consistent duration across the week
  + **Casual** Duration peaks on weekends
   

## Visualizing

### Total Trips by Customer Type (Members **57%** vs. Casual Riders **43%**)

```{r}
view(cyclistic_new_df)
cyclistic_new_df %>%
  group_by(member_casual)%>%
  summarize(number_of_rides = n())%>%
  arrange(member_casual)%>%
  ggplot(aes(x = member_casual, y = number_of_rides, fill = member_casual)) +
  labs(title = "Total Trips By Customer Type") +
  geom_col(width = 0.5, position = position_dodge(width = 0.5)) +
  scale_y_continuous(labels = function(x) format(x, scientific = FALSE))

```  

### Average Ride Length by Customer Type (Members **13 min** vs. Casual Riders **30 min**) 

```{r message=FALSE, warning=FALSE}
cyclistic_new_df %>%
  group_by(member_casual)%>%
  summarize(average_ride_length = mean(ride_length))%>%
  ggplot(aes(x = member_casual, y = average_ride_length, fill = member_casual)) +
  labs(title = "Average Ride Length") +
  geom_col(width = 0.5, position = position_dodge(width = 0.5))

```  

### Busiest Times by Customer Type (Members **5pm** vs. Casual Riders **5pm**)

```{r message=FALSE, warning=FALSE}
cyclistic_new_df %>%
  group_by(member_casual, hour)%>%
  summarize(number_of_trips = n())%>%
  ggplot(aes(x = hour, y = number_of_trips, color = member_casual, group = member_casual)) +
  geom_line() +
  labs(title = "Bike Demand by Hour", x = "Time of Day") +
  theme(axis.text.x = element_text(angle = 90)) +
  scale_y_continuous(labels = function(x) format(x, scientific = FALSE))
```  

### Busiest Weekday by Customer Type (Members **Thursday** vs. Casual Riders **Saturday**)

```{r message=FALSE, warning=FALSE}
cyclistic_new_df %>%
  group_by(member_casual, day_of_week)%>%
  summarize(number_of_rides = n())%>%
  arrange(member_casual, day_of_week)%>%
  ggplot(aes(x = day_of_week, y = number_of_rides, fill = member_casual)) +
  labs(title = "Total Rides by Weekday") +
  geom_col(width = 0.5, position = position_dodge(width = 0.5)) +
  scale_y_continuous(labels = function(x) format(x, scientific = FALSE))
```  

### Busiest Season by Customer Type (Members **May-Sept** vs. Casual Riders **June-Sept**)

```{r message=FALSE, warning=FALSE}
cyclistic_new_df %>%
  group_by(member_casual, month)%>%
  summarize(number_of_rides = n())%>%
  arrange(member_casual, month)%>%
  ggplot(aes(x = month, y = number_of_rides, fill = member_casual)) +
  labs(title = "Total Rides by Month") +
  theme(axis.text.x = element_text(angle = 30)) +
  geom_col(width = 0.5, position = position_dodge(width = 0.5)) +
  scale_y_continuous(labels = function(x) format(x, scientific = FALSE))
```  

### Most Popular Bike by Customer Type (Members **Classic** vs. Casual Riders **Classic**)

```{r message=FALSE, warning=FALSE}
cyclistic_new_df %>%
  group_by(rideable_type, member_casual)%>%
  summarize(number_of_trips = n())%>%
  ggplot(aes(x = rideable_type, y = number_of_trips, fill = member_casual)) +
  geom_bar(stat = 'identity') +
  labs(title = "Total Rides by Bike Type") +
  scale_y_continuous(labels = function(x) format(x, scientific = FALSE))
```  

### Ride Length by Weekday (Members **Consistent all days** vs. Casual Riders **Longer Weekend**)

```{r message=FALSE, warning=FALSE}
cyclistic_new_df %>%
  group_by(member_casual, day_of_week)%>%
  summarize(average_ride_length = mean(ride_length))%>%
  ggplot(aes(x = day_of_week, y = average_ride_length, fill = member_casual)) +
  labs(title = "Average Ride Length by Weekday") +
  geom_col(width = 0.5, position = position_dodge(width = 0.5)) 

```

## Act

### Key Findings

-- Members took the most rides at 57% of the total trips compared to 43% for casual riders.

-- However, casual riders averaged 30 minutes per bike ride, while members trip durations were less than half of that, at an average of 13 minutes per ride. The shorter rides of members suggest a task driven purpose, while the longer rides for casual riders suggests a pleasure trip.

-- The busiest time of day for both members and casual riders was after lunch, peaking at 5pm for both.

-- The busiest weekdays for members were Monday through Friday. The numbers fell off during the weekend, suggesting that these are local commuters.

-- The busiest weekdays for casual riders were Saturday and Sunday, suggesting that these customers are using the bikeshare for recreation.

-- The busiest season for both types of customer was the summer, although member numbers remained high through October. This would follow that if the members are regularly using the bikeshare for commuting, October would be the last month of consistent bikeable weather in Chicago. Casual riders, on the other hand, were high in June, July and August only, further pointing to sightseeing and vacationers.

-- The most popular bike for both types of customer was the classic. 
-- If the ride was by electric bike, more members than casual riders chose this mode.

-- Casual customers use bikeshare services more during weekends, while members use them consistently over the entire week. Again, this suggests the conclusion that members are local commuters and casual riders are using the bikeshare for weekend recreation.


### Recommendations

-- As was predicted in the hypothesis, Cyclistic annual members use the bikeshare for commuting and tasks while casual riders are using the bikeshare mostly on the weekend for longer rides and recreation. Each group's use patterns validate that. 

-- This may make it difficult to convert casual riders into annual members. 

-- Focusing on casual riders who are mostly using the bikeshare during the summers months, I would recommend a seasonal membership program to prompt more frequent riding during the warmer months.

-- For digital media promotion, influencers on Instagram, YouTube, Snapchat, Twitch, and TikTok could be paid to document their bikeshare trips around Chicago, including some of their personal favorite locations, events, views of the city or activities.

-- Local influencers can help combat the "touristy" image of Cyclistic and show that it's for city people, too.

-- Another angle of marketing can be that Cyclistic is a good solution for locals that only want a bike in the short "good weather" period in Chicago and don't want to store, maintain and worry about theft of their own bikes.

-- Cyclistic classic bikes are heavy and slow compared to most personally owned bikes. Perhaps the digital media campaign can have a branch focusing on how to safely ride electric bikes and enjoy their speed and the lower physical demand they offer for cyclists. This could be a real area of growth for Cyclistic since very few individuals own their own electric bike.

### Additional data to expand scope of analysis

-- Because personally identifying information was scrubbed for the rider IDs, there was no way to know the percentage of casual riders who lived locally and the percentage of those who are visitors from out of town. This would be an important metric for marketing to local users and encouraging them to become members.

-- A number of start and end station ids were missing from the dataset. A full understanding of the most popular starting and ending locations would be helpful in marketing, as well as identifying possible "deserts" where bikes can be added.

# Follow me:
* Linkedin: https://www.linkedin.com/in/kant-ai/
* Portfolio: https://kantrixai.netlify.app/
