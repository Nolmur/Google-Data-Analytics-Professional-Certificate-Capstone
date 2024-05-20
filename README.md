# Google-Data-Analytics-Professional-Certificate-Capstone
my Capstone for the Google Data Analytics Professional Certificate

## Introduction
In this case study, I am working as a Junior Data Analyst on the marketing team at Cyclistic, a fictional bike-share company in Chicago. Since its launch in 2016, Cyclistic has gained recognition for the flexibility of its pricing models and the inclusivity of its bikes. The company offers traditional bikes, as well as reclining bikes, hand tricycles, and cargo bikes to accommodate people with disabilities and other specific needs.

Data Source
For this analysis, I used historical bike trip data from Cyclistic, which is publicly available and licensed appropriately. This data includes information on trips conducted over the past 12 months.

Case Study Objective
The primary objective of this case study is to compare the usage patterns between casual riders and annual members. These insights will help design a new marketing strategy aimed at converting more casual riders into annual members.

Key Questions:
To achieve this objective, my analysis focused on answering the following question:

How do annual members and casual riders use Cyclistic bikes differently?

Sure! Here is the text formatted for inclusion in your GitHub documentation:

---

### Data Processing

To prepare the data for analysis, I undertook the following steps:

#### 1. Data Cleaning

- **Removing Missing Values and Duplicates:**
  Ensured that the dataset was free from missing values and duplicates to maintain data integrity.

```r
# Load necessary libraries
library(tidyverse)
library(janitor)

# Combine all the datasets into one single dataframe
all_trips <- bind_rows(march2023, april2023, may2023, june2023, july2023, august2023, 
                       september2023, october2023, november2023, december2023, january2024, 
                       february2024, march2024)

# Remove rows with negative trip durations
all_trips <- all_trips[!(all_trips$trip_duration < 0),]

# Remove test rides
all_trips <- all_trips[!((all_trips$start_station_name %like% "TEST" | 
                          all_trips$start_station_name %like% "test")),]

# Check the dataframe
glimpse(all_trips)
```

#### 2. Data Transformation

- **Converting Data Types:**
  Converted relevant columns to appropriate data types for consistency and accurate analysis.

```r
# Rename columns for consistency
all_trips <- all_trips %>%
     rename(ride_type = rideable_type, 
            start_time = started_at,
            end_time = ended_at)

# Ensure station IDs are numeric
all_trips <- all_trips %>%
  mutate(start_station_id = as.double(start_station_id),
         end_station_id = as.double(end_station_id))
```

#### 3. Feature Engineering

- **Creating New Features:**
  Added new columns for the day of the week, month, time of day, and trip duration to enrich the dataset and provide more variables for analysis.

```r
# Add columns for day of the week, month, time, and trip duration
all_trips$day_of_the_week <- format(as.Date(all_trips$start_time),'%a')
all_trips$month <- format(as.Date(all_trips$start_time),'%b_%y')
all_trips$time <- format(all_trips$start_time, format = "%H:%M")
all_trips$time <- as.POSIXct(all_trips$time, format = "%H:%M")
all_trips$trip_duration <- (as.double(difftime(all_trips$end_time, all_trips$start_time)))/60

# Check the dataframe
glimpse(all_trips)
```

A particular challenge I encountered was dealing with the large size of the dataset, which sometimes exceeded the memory limits of my system. I overcame this by processing the data in smaller chunks and combining the results, ensuring efficient memory usage and preventing system crashes.

---

### Analysis

For the analysis, I used the following methods and techniques:

- **Descriptive Statistics:** To get an overview of the data.
- **Correlation:** To identify relationships between different variables.
- **Visualizations:** Created with R to better understand the data.

Here are some of the key charts and their interpretations:

#### Chart 1: Ride Type vs. Number of Trips

This chart shows the number of trips taken by casual riders and annual members for each ride type. It highlights that both customer types predominantly use classic bikes and electric bikes, with a negligible number of trips using docked bikes. Members tend to use classic bikes more, while casual riders have a higher proportion of electric bike usage.

```r
all_trips_v2 %>%
  group_by(ride_type, customer_type) %>%
  summarise(number_of_trips = n()) %>%  
  ggplot(aes(x= ride_type, y=number_of_trips, fill= customer_type)) +
  geom_bar(stat='identity') +
  scale_y_continuous(labels = function(x) format(x, scientific = FALSE)) +
  labs(title ="Ride type Vs. Number of trips")
```

![Ridetype_vs_num_trips](https://github.com/Nolmur/Google-Data-Analytics-Professional-Certificate-Capstone/assets/159724617/347ff3b6-455a-4006-820e-bfdea35f7ce8)


#### Chart 2: Total Trips by Customer Type vs. Day of the Week

This chart displays the total number of trips taken by casual riders and annual members on each day of the week. It shows that members consistently have higher trip numbers throughout the week, with a noticeable increase in trips on weekdays compared to casual riders who have more balanced usage across the week.

```r
all_trips_v2 %>%  
  group_by(customer_type, day_of_the_week) %>% 
  summarise(number_of_rides = n()) %>% 
  arrange(customer_type, day_of_the_week)  %>% 
  ggplot(aes(x = day_of_the_week, y = number_of_rides, fill = customer_type)) +
  labs(title = "Total trips by customer type Vs. Day of the week") +
  geom_col(width = 0.5, position = position_dodge(width = 0.5)) +
  scale_y_continuous(labels = function(x) format(x, scientific = FALSE))
```

![Totaltrips_by_customer_type](https://github.com/Nolmur/Google-Data-Analytics-Professional-Certificate-Capstone/assets/159724617/ff5413d3-d187-4b9c-8d01-15537ca95cd4)


#### Chart 3: Average Trip Duration by Customer Type vs. Day of the Week

This chart shows the average trip duration for casual riders and annual members on each day of the week. Casual riders tend to have longer trip durations compared to annual members. This is particularly evident on weekends where casual riders' trips are significantly longer.

```r
all_trips_v2 %>%  
  group_by(customer_type, day_of_the_week) %>% 
  summarise(average_trip_duration = mean(trip_duration)) %>%
  ggplot(aes(x = day_of_the_week, y = average_trip_duration, fill = customer_type)) +
  geom_col(width = 0.5, position = position_dodge(width = 0.5)) + 
  labs(title = "Average trip duration by customer type Vs. Day of the week")
```

![Aver_trip_duratiom_by_customer](https://github.com/Nolmur/Google-Data-Analytics-Professional-Certificate-Capstone/assets/159724617/ead53c05-4189-459b-9634-047fe9f38b08)


#### Chart 4: Demand over 24 Hours of a Day

This chart illustrates the number of trips taken by casual riders and annual members throughout the day. It reveals peak usage times, with members having distinct morning and evening peaks likely corresponding to commute times. Casual riders have a more spread-out usage pattern with a slight peak around midday.

```r
all_trips_v2 %>%  
  group_by(customer_type, time) %>% 
  summarise(number_of_trips = n()) %>%
  ggplot(aes(x = time, y = number_of_trips, color = customer_type, group = customer_type)) +
  geom_line() +
  scale_x_datetime(date_breaks = "1 hour", minor_breaks = NULL,
                   date_labels = "%H:%M", expand = c(0,0)) +
  theme(axis.text.x = element_text(angle = 90)) +
  labs(title = "Demand over 24 hours of a day", x = "Time of the day")
```
![Deman_over_24h_a_day](https://github.com/Nolmur/Google-Data-Analytics-Professional-Certificate-Capstone/assets/159724617/b0823349-786c-41ea-9a9b-5c3dc454d658)

## Results and Insights

The analysis of bike-sharing usage patterns based on ride type, day of the week, and customer type has yielded several key insights:

#### Insight 1: Preference for Ride Type by Customer Type
- **Description and Implication**: The chart "Ride Type vs. Number of Trips" shows that classic bikes are the most popular among both customer types. However, members use electric bikes proportionally more than casual riders, suggesting that members might be more receptive to using electric bikes for their efficiency and ease of use. This could imply that increasing the availability and visibility of electric bikes could enhance membership attractiveness.

#### Insight 2: Usage Patterns Throughout the Week
- **Description and Implication**: From "Total Trips by Customer Type vs. Day of the Week," it is evident that members use the service more consistently throughout the week with peaks on weekdays, which likely aligns with commuting patterns. In contrast, casual riders show increased usage during weekends, indicative of recreational or non-commuting uses. This suggests tailored marketing strategies targeting weekend promotions for casual riders and commuter plans for members.

#### Insight 3: Trip Duration Variances
- **Description and Implication**: The "Average Trip Duration by Customer Type vs. Day of the Week" graph highlights that casual riders generally have longer trip durations, especially on weekends. This could indicate that casual riders are using the service for leisure activities, which might be longer trips compared to members who might be using the services for practical and commuting purposes. Tailoring services and marketing to these usage habits can potentially convert casual riders into members by offering membership benefits aligned with these trends.

#### Insight 4: Demand Fluctuations Throughout the Day
- **Description and Implication**: The "Demand over 24 Hours of a Day" chart shows distinct peaks during morning and evening hours for members, aligning with typical work commute times. Casual riders' usage peaks around midday and is generally lower than members during peak hours. This pattern can inform the deployment strategies for bikes and docking stations to meet peak demand times and optimize bike availability.

### Recommendations Based on Analysis

Based on these insights, I recommend the following strategies to enhance bike-share usage and increase membership conversion:
- **Increase Availability of Electric Bikes**: Promote electric bike usage among casual riders by highlighting their benefits through targeted campaigns, potentially increasing membership conversions by appealing to the convenience and efficiency of electric bikes.
- **Weekend and Leisure Promotions**: Develop promotions and targeted marketing campaigns for weekends to attract more casual riders by offering discounts or special rates, which could also include family or group ride options.
- **Optimize Bike and Dock Availability During Peak Times**: Adjust the deployment of bikes and availability of docking stations to meet the clear demand peaks observed in the data, ensuring availability during morning and evening commutes for members and midday availability for casual riders.
- **Tailored Membership Plans**: Offer customized membership plans that cater to the predominant usage patterns, such as commuter plans with benefits during peak hours and leisure plans with benefits for weekend use.

In summary, this case study has shown that there are distinct usage patterns between casual riders and annual members. Casual riders tend to use the service more for leisure activities, especially on weekends, and have longer trip durations, while annual members use the service more consistently throughout the week with noticeable peaks during commuting hours. These insights can help Cyclistic tailor their marketing strategies to convert casual riders into members by addressing their specific needs and usage habits.

Future research could focus on:

Investigating seasonal variations: Analyzing how bike usage patterns change across different seasons could provide further insights into optimizing bike availability and marketing efforts throughout the year.
Exploring demographic factors: Understanding how demographics such as age, gender, and location influence bike usage could help in creating more personalized marketing campaigns.
Evaluating the impact of promotions: Assessing the effectiveness of different promotional strategies and their impact on converting casual riders to members could provide valuable information for future marketing initiatives.
By addressing these areas, Cyclistic can continue to improve their service, increase user satisfaction, and drive membership growth.


# Thank you for taking the time to read this case study! 📚✌

