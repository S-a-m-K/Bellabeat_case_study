![Banner](https://github.com/S-a-m-K/bellabeat_case_study/blob/main/Bellabeat.png)

<h1>Google Data Analytics Bellabeat Case Study</h1> 

**Author: Samuel Kleger**  
**Date: 2023-08-08**


# **Introduction**

Bellabeat is a successful small business specializing in the production of cutting-edge health and wellness products for women. With the potential to become a major player in the global smart device market, the company is on the cusp of an exciting growth phase. Urska Srsen, Bellabeat's co-founder and Chief Creative Officer, believes that analyzing fitness data from smart devices could provide valuable insights for uncovering new growth opportunities.

---

#### Table of Content
- [Introduction](#Introduction)
- [Ask](#Ask)
  - [Buisness Task](#Buisness-Task)
  - [Stakeholder](#Stakeholder)
- [Prepare](#Prepare)
  - [Data Description](#Data-Description)
  - [Credibility of the Data](#Credibility-of-the-Data)
  - [Confirmation of the ROCCC-Process](#Confirmation-of-the-ROCCC-Process)
- [Process](#Process)
- [Analyze](#Analyze)
- [Visualize](#Visualize)
- [Recommendation](#Recommendation-based-on-analysis)

---

# **Ask**

<div style="margin-bottom: 40px;">

</div>

### **Buisness Task**

Analyzing data from non-Bellabeat smart devices to gain insights into how these devices are being used. Based on the findings, a general recommendation is sought on how these trends could shape Bellabeat's marketing strategy.

### **Stakeholder** 


**Urška Sršen**: Co-founder and Chief Creative Officer of Bellabeat

**Sando Mur**: Mathematician and co-founder of Bellabeat; key member of Bellabeat's leadership team

**Marketing-Analytics-Team**: A team of data analytics professionals who collect, analyze and report on data to support Bellabeat's marketing strategy

<div style="margin-bottom: 40px;">

</div>

# **Prepare**



### **Data Description**

FitBit Fitness Tracker Data (CC0: Public Domain, a dataset provided by Mobius): This Kaggle dataset includes personal fitness trackers from thirty Fitbit users. These users have agreed to share their personal tracker data, including minute-by-minute physical activity, heart rate, and sleep data. The dataset includes data on daily activity, steps, and heart rate. This data can be used to study users' habits.

**Source**: 33 participants FitBit Fitness Tracker data from [kaggle](https://www.kaggle.com/datasets/arashnic/fitbit).

**Type**: CSV

**Format**: Long Data

**Duration**: 03.12.2016-05.12.2016

<div style="margin-bottom: 40px;">

</div>

### **Credibility of the Data**

This dataset was created from participants in a distributed survey on Amazon Mechanical Turk. Thirty-three eligible Fitbit users consented to submit personal tracker data. Variations in results reflect use of different types of Fitbit trackers and individual tracking behaviors/preferences.

### **Confirmation of the ROCCC-Process**

* **Reliable**: The dataset contains 33 user data from daily activity. It thus exceeds the minimum sample size requirement of 30. However, some users have not fully recorded their data and may not be comprehensive enough to allow for detailed analysis.

* **Original**: No, the original data set is generated by a third party, Amazon Mechanical Turk

* **Comprehensive**: The data is closely related to the sleep and activity functions of the Bellabeat Leaf product. Unfortunately, important information such as age, height and location of users, which would be beneficial for marketing purposes to target customers, is missing.

* **Current**: The data set is from March 2016 and is therefore not up-to-date. User habits might be different now.

* **Cited**: The data was collected anonymously by a third party.

* **License**: [CC0: Public Domain](https://creativecommons.org/publicdomain/zero/1.0/):
Apart from the ID number, no personal data are included in the collected data. So there are no privacy concerns to consider. The participants remain anonymous.

* **Conclusion**: Overall, this is not a quality dataset that can be used for actual business recommendations.

<div style="margin-bottom: 40px;">

</div>

# **Process**

<div style="margin-bottom: 20px;">

</div>

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
# import library
library(tidyverse)
library(lubridate)
library(skimr)
library(gridExtra)
library(ggplot2)
```

```{r}
# Import Dataset
daily_activity <- read_csv("Bellabeats/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")
sleep_day <- read_csv("Bellabeats/Fitabase Data 4.12.16-5.12.16/sleepDay_merged.csv")
weight_info <- read_csv("Bellabeats/Fitabase Data 4.12.16-5.12.16/weightLogInfo_merged.csv")
```

```{r}
# overview and first impression
head(daily_activity)
head(sleep_day)
head(weight_info)
str(daily_activity)
str(sleep_day)
str(weight_info)
```

<div style="margin-bottom: 40px;">

</div>

### **Cleansing data set daily_activity**

<div style="margin-bottom: 20px;">

</div>

```{r}
# change data type as character
daily_activity$Id <- as.character(daily_activity$Id)

# change data type as date
daily_activity$ActivityDate <- as.Date(daily_activity$ActivityDate, format = "%m/%d/%Y") 
```

<div style="margin-bottom: 40px;">

</div>

Since the columns appear to contain only null values at first glance, I want to verify if they are actually identical.
```{r}
# check if TotalDistance and TrackerDistance have the same values
daily_activity$distance_diff <- daily_activity$TotalDistance - daily_activity$TrackerDistance
distance_diff <- daily_activity[daily_activity$distance_diff != 0,]
view(distance_diff)
```

```{r}
# change column names to "snake_case"
daily_activity <- daily_activity %>% 
  rename(
    id = Id,
    date = ActivityDate,
    total_steps = TotalSteps, 
    total_distance = TotalDistance,
    tracker_distance = TrackerDistance,
    very_active_distance = VeryActiveDistance,
    logged_distance = LoggedActivitiesDistance,
    moderately_active_distance = ModeratelyActiveDistance,
    light_active_distance = LightActiveDistance,
    sedentary_distance = SedentaryActiveDistance,
    very_active_minutes = VeryActiveMinutes,
    fairly_active_minutes = FairlyActiveMinutes,
    lightly_active_minutes = LightlyActiveMinutes,
    sedentary_minutes = SedentaryMinutes,
    calories = Calories)
colnames(daily_activity)
```

```{r}
# check for whitespace
skim_without_charts(daily_activity)
```

```{r}
# check for NA
anyNA(daily_activity)
```

```{r}
# check whether there are any duplicates and how many
daily_activity %>% 
  duplicated() %>% 
  sum()
```

<div style="margin-bottom: 40px;">

</div>

### **Cleansing data set sleep_day**

<div style="margin-bottom: 20px;">

</div>

```{r}
# change data type as character
sleep_day$Id <- as.character(sleep_day$Id)

# change data type as Date
sleep_day$SleepDay <- as.POSIXct(sleep_day$SleepDay, format = "%m/%d/%Y %H:%M:%S")
```

```{r}
# change column names to "snake_case"
sleep_day <- sleep_day %>% 
  rename(id = Id, date = SleepDay,
         num_of_records = TotalSleepRecords, 
         total_sleep = TotalMinutesAsleep, 
         bed_time = TotalTimeInBed)
```

```{r}
# check for whitespace
skim_without_charts(sleep_day)
```

```{r}
# check for NA
anyNA(sleep_day)
```

```{r}
# check whether ther are any duplicates and how many 
sleep_day %>% 
  duplicated() %>% 
  sum()
```

```{r}
# check which rows are duplicates
duplicates <- sleep_day[duplicated(sleep_day[, c("id", "date", "num_of_records", "total_sleep", "bed_time")]), ]
print(duplicates)
```

```{r}
# delete the duplicates from the dataset
sleep_day <- distinct(sleep_day, id, date, num_of_records, total_sleep, bed_time)
```

```{r}
# remove time in the column date
sleep_day$date <- as.POSIXct(sleep_day$date, format = "%Y-%m-%d %H:%M:%S")

sleep_day$date <- as.Date(sleep_day$date)
```

<div style="margin-bottom: 40px;">

</div>

### **Cleansing data set weight_info**

<div style="margin-bottom: 20px;">

</div>

```{r}
# change data type as character
weight_info$Id <- as.character(weight_info$Id)

# change data type as date
weight_info$Date <- as.POSIXct(weight_info$Date, format = "%m/%d/%Y %I:%M:%S %p")
```

```{r}
# change column names to "snake_case"
weight_info <- weight_info %>% 
  rename(id = Id, date = Date, weight_kg = WeightKg, weight_gbp = WeightPounds,
         fat = Fat, bmi = BMI, manual_rep = IsManualReport, log_id = LogId)
```

```{r}
# check for white spaces
skim_without_charts(weight_info)
```

```{r}
# check whether ther are any duplicates and how many 
weight_info %>% 
  duplicated() %>% 
  sum()
```

<div style="margin-bottom: 40px;">

</div>

# **Analyze**

### **Analysis data set daily_activity**

<div style="margin-bottom: 20px;">

</div>

```{r}
# add a column with days of the week
daily_activity$day_of_week <- weekdays(as.Date(daily_activity$date))

# add a column numbering the days of the week
wochentage <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday")
daily_activity$n_of_weekday <- match(daily_activity$day_of_week, wochentage)
```

```{r}
# change the order of columns and delete unused columns
# Since the unit of measurement for the distances is unknown, I will exclude these columns from the analysis
daily_activity <- daily_activity %>%
  select(id, date, day_of_week, n_of_weekday, total_steps, total_distance,
         # tracker_distance, logged_distance, very_active_distance,
         # moderately_active_distance, light_active_distance, sedentary_distance,
         very_active_minutes, fairly_active_minutes,
         lightly_active_minutes, sedentary_minutes, calories)

head(daily_activity)
```

```{r}
# create a new frame with id and average steps
id_activity_lv <- daily_activity %>%
  group_by(id) %>%
  mutate(id_avg_steps = mean(total_steps)) %>%
  ungroup() %>%
  select(id, id_avg_steps) %>%
  distinct()
```

```{r}
# add new column "activity_level" based on steps per id
id_activity_lv <- id_activity_lv %>%
  mutate(activity_level = case_when(
    id_avg_steps < 6000 ~ "sedentary",
    id_avg_steps >= 6000 & id_avg_steps <= 11999 ~ "active",
    id_avg_steps >= 12000 ~ "very active",
# In case there are unexpected values
    TRUE ~ NA_character_
  ))
head(id_activity_lv)
```

```{r}
# add column activity_level to dataset daily_activity based on activity level of each id
daily_activity$activity_level <- id_activity_lv$activity_level[match(daily_activity$id, id_activity_lv$id)]

head(daily_activity)
```

```{r}
# count the number of unique combinations in a set of one/more vectors
n_distinct(daily_activity$id)
```

```{r}
# group by id and count entries for each identity
id_count_entries <- daily_activity %>%
  group_by(id) %>%
  summarise(count_entries = n())
head(id_count_entries)
```

```{r}
# get a summary of the dataset
summary(daily_activity)
```

<div style="margin-bottom: 40px;">

</div>

### **Analysis data set sleep_day**

<div style="margin-bottom: 20px;">

</div>

```{r}
# Create a new frame with columns from data set daily_activity and sleep_day
combined_data <- inner_join(
  select(sleep_day, id, date, total_sleep, bed_time),
  select(daily_activity, id, date, total_steps, calories),
  by = c("id", "date")
)

combined_data <- combined_data %>% 
  mutate(fall_asleep = bed_time - total_sleep)

head(combined_data)
```

<div style="margin-bottom: 40px;">

</div>

### **Analysis data set weight_info**

<div style="margin-bottom: 20px;">

</div>

```{r}
# remove columns not needed for this analysis
weight_info <- weight_info %>% 
  select(id, date, weight_kg, bmi)
```

```{r}
# separate columns for date and time
weight_info <- weight_info %>%
  mutate(date = as.POSIXct(date, format = "%Y-%m-%d %H:%M:%S"),
         separate_date = as.Date(date),
         separate_time = format(date, format = "%H:%M:%S")) %>%
  select(-date) %>% 
  rename(date = separate_date, time = separate_time)

head(weight_info)
str(weight_info)
colnames(weight_info)
```

```{r}
# create new frame with id, avg_bmi and avg_steps
bmi_cor <- weight_info %>%
  group_by(id) %>%
  mutate(avg_bmi = mean(bmi)) %>%
  ungroup() %>%
  select(id, avg_bmi) %>%
  distinct() %>%
  left_join(daily_activity %>%
              group_by(id) %>%
              summarise(avg_steps = mean(total_steps),
                        avg_very_active = mean(very_active_minutes)),
            by = "id")

head(bmi_cor)
```

<div style="margin-bottom: 40px;">

</div>

# **Visualize**

<div style="margin-bottom: 20px;">

</div>

### **Correlation between steps and calories burned**
```{r}
# create visualization
ggplot(daily_activity) +
  geom_point(mapping = aes(x = total_steps, y = calories, colour = activity_level)) +
  scale_color_manual(values = c("very active" = "lightseagreen", "active" = "khaki3", "sedentary" = "darkorange"),
                     breaks = c("very active", "active", "sedentary")) +
  labs(title = "Correlation between steps and calories burned")
```

<img src="https://github.com/S-a-m-K/Bellabeatcasestudy/assets/143487615/e69f66b4-49a0-4368-abb0-0fd14f2fd6db" alt="Beschreibung des Bildes" width="700"/>

A positive correlation between total steps and calories is evident: the more active one is, the more calories are burned.

<div style="margin-bottom: 40px;">

</div>

### **Correlation between activity level in minutes and calories**
```{r}
# create individual subcharts
very_active_plot <- ggplot(daily_activity) +
  geom_point(aes(x = calories, y = very_active_minutes, color = activity_level)) +
  scale_color_manual(values = c("very active" = "lightseagreen", "active" = "khaki3", "sedentary" = "darkorange"),
                     breaks = c("very active", "active", "sedentary")) +
  labs(title = "Very Active Activity") +
  theme_minimal()

fairly_active_plot <- ggplot(daily_activity) +
  geom_point(aes(x = calories, y = fairly_active_minutes, color = activity_level)) +
  scale_color_manual(values = c("very active" = "lightseagreen", "active" = "khaki3", "sedentary" = "darkorange"),
                     breaks = c("very active", "active", "sedentary")) +
  labs(title = "Fairly Active Activity") +
  theme_minimal()

lightly_active_plot <- ggplot(daily_activity) +
  geom_point(aes(x = calories, y = lightly_active_minutes, color = activity_level)) +
  scale_color_manual(values = c("very active" = "lightseagreen", "active" = "khaki3", "sedentary" = "darkorange"),
                     breaks = c("very active", "active", "sedentary")) +
  labs(title = "Lightly Active Activity") +
  theme_minimal()

sedentary_plot <- ggplot(daily_activity) +
  geom_point(aes(x = calories, y = sedentary_minutes, color = activity_level)) +
  scale_color_manual(values = c("very active" = "lightseagreen", "active" = "khaki3", "sedentary" = "darkorange"),
                     breaks = c("very active", "active", "sedentary")) +
  labs(title = "Sedentary Activity") +
  theme_minimal()

# combined representation
combined_plot <- grid.arrange(very_active_plot, fairly_active_plot, lightly_active_plot, sedentary_plot, ncol = 2)
```
<img src="https://github.com/S-a-m-K/Bellabeatcasestudy/assets/143487615/bcfb7cbc-570c-4f98-a58d-ab6661cca147" alt="Beschreibung des Bildes" style="width:700px;"/>

The charts for the individual activity levels clearly show that the activity level correlates with calorie consumption.

<div style="margin-bottom: 40px;">

</div>

### **Average steps per day**
```{r}
# create new frame with average steps based on days of the week
average_steps <- daily_activity %>%
  group_by(day_of_week) %>%
  summarise(average_steps = mean(total_steps))

# define the desired order of the days of the week
order_day_of_week <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday")

# update the day_of_week column as a factor with the new order
average_steps$day_of_week <- factor(average_steps$day_of_week, levels = order_day_of_week)

# sort the dataset by the order of the new day_of_week column
average_steps <- average_steps[order(average_steps$day_of_week), ]

# individual colors for the days of the week
day_colors <- c("Monday" = "khaki",
                "Tuesday" = "khaki",
                "Wednesday" = "khaki3",
                "Thursday" = "khaki3",
                "Friday" = "khaki3",
                "Saturday" = "khaki",
                "Sunday" = "khaki4")

# calculate average steps for average line visualization
average_total_steps <- mean(average_steps$average_steps)

# create visualization
ggplot(average_steps, aes(x = day_of_week, y = average_steps, fill = day_of_week)) +
  geom_bar(stat = "identity") +
  scale_fill_manual(values = day_colors) +
  geom_hline(yintercept = 10000, color = "red", linetype = "dashed") +
  geom_text(aes(label = round(average_steps), y = average_steps), color = "black", size = 3.5, position = position_stack(vjust = 0.9)) +
  geom_text(aes(label = "Recommended steps per day"), 
            x = 4, y = 10250, color = "red", fontface = "italic", hjust = 0.5) +
  labs(title = "Average steps per day", x = "Day of week", y = "Average steps") +
  theme_minimal()
```

<img src="https://github.com/S-a-m-K/Bellabeatcasestudy/assets/143487615/edee96df-d459-4c42-8752-b98ac83854c3" alt="Beschreibung des Bildes" width="700"/>

Illustrating the average steps taken per weekday, we observe that Monday, Tuesday, and Saturday are the most active days, while Sunday shows comparatively lower activity. Additionally, we note that the average daily steps fall short of the 10,000-step recommendation set by the WHO.

<div style="margin-bottom: 40px;">

</div>

### **Average daily steps per id**
```{r}
# create a frame with average steps and average active minutes per id
id_movement <- daily_activity %>% 
  group_by(id) %>% 
  mutate(avg_steps = ceiling(mean(total_steps))) %>% 
  mutate(avg_fairly_active = ceiling(mean(fairly_active_minutes))) %>% 
  mutate(avg_very_active = ceiling(mean(very_active_minutes))) %>% 
  mutate(sum_active_min = avg_fairly_active + avg_very_active) %>% 
  ungroup() %>% 
  select(id, avg_steps, avg_fairly_active, avg_very_active, sum_active_min) %>% 
  distinct()

head(id_movement)

ggplot(id_movement, aes(x = id, y = avg_steps, fill = id)) +
  geom_bar(stat = "identity", fill = "khaki2") +
  geom_hline(yintercept = 10000, color = "red", linetype = "dashed") +
  geom_text(aes(label = "Recommended daily steps"), 
            x = 16, y = 10500, color = "red", fontface = "italic") +
  labs(title = "Average daily steps per id", x = "Users", y = "Average steps") +
  guides(fill = "none") +
  scale_x_discrete(labels = c(1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33)) + 
  theme_minimal()
```

<img src="https://github.com/S-a-m-K/Bellabeatcasestudy/assets/143487615/f5fddd8e-8b60-459f-bb3a-6bd5167cd72e" alt="Beschreibung des Bildes" width="700"/>

Even when examining the average daily steps taken by users, most do not meet the recommended value. In fact, 79% of users average below this guideline.

<div style="margin-bottom: 40px;">

</div>

### **Percentage distribution of activity minutes**

The WHO recommends 150 minutes of moderate aerobic physical activity per week or 75 minutes of vigorous aerobic activity, or a combination of both spread over several days. Unfortunately, the data does not allow us to determine which activities are classified as moderate or vigorous. However, by assuming that moderate and vigorous activity corresponds to fairly active and very active levels, we can analyze the proportion of users who meet these criteria.

```{r}
# create visualization
ggplot(id_movement, aes(x = id, y = avg_fairly_active, fill = id)) +
  geom_bar(stat = "identity", fill = "khaki2") +
  geom_hline(yintercept = 150, color = "red", linetype = "dashed") +
  geom_text(aes(label = "Recommended fairly active minutes"), 
            x = 16, y = 155, color = "red", fontface = "italic") +
  labs(title = "Fairly active minutes per id", x = "Users", y = "Fairly active minutes") +
  guides(fill = "none") +
  scale_x_discrete(labels = c(1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33)) + 
  theme_minimal()
```

<img src="https://github.com/S-a-m-K/Bellabeatcasestudy/assets/143487615/52c539bc-2144-4fd8-89e8-1361c63670dd" alt="Beschreibung des Bildes" width="700"/>

```{r}

ggplot(id_movement, aes(x = id, y = avg_very_active, fill = id)) +
  geom_bar(stat = "identity", fill = "khaki2") +
  geom_hline(yintercept = 75, color = "red", linetype = "dashed") +
  geom_text(aes(label = "Recommended very active minutes"), 
            x = 16, y = 78, color = "red", fontface = "italic") +
  labs(title = "Very active minutes per id", x = "Users", y = "Very active minutes") +
  guides(fill = "none") +
  scale_x_discrete(labels = c(1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33)) + 
  theme_minimal()
```

<img src="https://github.com/S-a-m-K/Bellabeatcasestudy/assets/143487615/9156b1e3-33bb-4d56-a9cc-44051928bd49" alt="Beischreibung des Bildes" width="700"/>

```{r}
ggplot(id_movement, aes(x = id, y = sum_active_min, fill = id)) +
  geom_bar(stat = "identity", fill = "khaki2") +
  geom_hline(yintercept = 75, color = "red", linetype = "dashed") +
  geom_text(aes(label = "Recommended very active minutes"), 
            x = 16, y = 80, color = "red", fontface = "italic") +
  geom_hline(yintercept = 150, color = "red", linetype = "dashed") +
  geom_text(aes(label = "Recommended fairly active minutes"), 
            x = 16, y = 155, color = "red", fontface = "italic") +
  labs(title = "Sum of active minutes per id", x = "Users", y = "Activity in minutes") +
  guides(fill = "none") +
  scale_x_discrete(labels = c(1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33)) + 
  theme_minimal()

```

<img src="https://github.com/S-a-m-K/Bellabeatcasestudy/assets/143487615/2825e145-4f45-4193-acb1-4a475fa01a75" alt="Beischreibung des Bildes" width="700"/>

Even regarding the percentage of minutes exercised, most users do not meet the specified target.

<div style="margin-bottom: 40px;">

</div>

### **Percentage distribution of activity minutes**
```{r}
# create a frame with the sum of the active minutes columns
data <- data.frame(
  "category" = c('very_active', 'fairly_active', 'lightly_active', 'sedentary'),
  "amount" = c(
    sum(daily_activity$very_active_minutes),
    sum(daily_activity$fairly_active_minutes),
    sum(daily_activity$lightly_active_minutes),
    sum(daily_activity$sedentary_minutes)
  )
)

head(data)

# determine order
category_order <- c('very_active', 'fairly_active', 'lightly_active', 'sedentary')

data$category <- factor(data$category, levels = rev(category_order))

# create visualization
ggplot(data, aes(x = factor(category, levels = category_order), y = amount, fill = category)) +
  geom_bar(stat = "identity") +
  labs(title = "Distribution of Activity Categories", x = "Activity Category", y = "Duration of activity") +
  scale_fill_manual(values = c('very_active' = 'lightseagreen', 'fairly_active' = 'lightblue2', 
                               'lightly_active' = 'khaki3', 'sedentary' = 'orange')) +
  theme_minimal() +
  geom_text(aes(label = scales::percent(amount/sum(amount))),
            position = position_stack(vjust = 0.8))
```

<img src="https://github.com/S-a-m-K/Bellabeatcasestudy/assets/143487615/dbea5454-a28b-4212-a8da-6c71ae6870a8" alt="Beischreibung des Bildes" width="700"/>

When examining the percentage breakdown of various activities, it becomes evident that we spend the majority of our waking hours in a seated position.

<div style="margin-bottom: 40px;">

</div>

### **Correlation between steps, calories and sleep quality**
```{r}
ggplot(combined_data, aes(x = total_steps, y = calories, size = fall_asleep, color = fall_asleep)) +
  geom_point() +
  scale_color_gradient(low = "lightseagreen", high = "orange") +
  labs(x = "Steps", y = "Calories", title = "Correlation between steps, calories and sleep quality",
       size = "Time to fall asleep", color = "Time to fall asleep") +
  theme_minimal()
```

<img src="https://github.com/S-a-m-K/Bellabeatcasestudy/assets/143487615/9172a230-6710-4940-b2e1-7ef63e687d42" alt="Beischreibung des Bildes" width="700"/>

Unfortunately, the data are insufficient for a definitive conclusion. Nevertheless, I wanted to illustrate how exercise and sleep quality are related. While many other factors influence sleep quality, a discernible pattern emerges: those who struggle more with falling asleep tend to burn the fewest calories, although they are not necessarily the ones who exercise the least.

<div style="margin-bottom: 40px;">

</div>

### **Weight loss during this period**
```{r}
# Filter data for the specified IDs
filtered_data <- weight_info %>%
  filter(id %in% c("8877689391", "6962181067")) %>%
  mutate(user_label = ifelse(id == "8877689391", "User1", "User2"))

# Weight Loss During this Period for the specified IDs
ggplot(filtered_data, aes(x = date, y = weight_kg, color = date)) +
  geom_point(size = 3) +
  scale_color_gradient(low = "orange", high = "lightseagreen") +
  labs(x = "Period", y = "Weight in Kg",
       title = "Weight Loss During this Period",
       subtitle = "Users: {user_label}") +
  facet_wrap(~ user_label, ncol = 1) +  # Separate plots for each user label
  theme_minimal()
```

<img src="https://github.com/S-a-m-K/Bellabeatcasestudy/assets/143487615/7fd9e78a-cf48-4466-8b11-50b1ff16ed2b" alt="Beischreibung des Bildes" width="700"/>

The two users who tracked their weight data for a month did not achieve significant success. Once again, we require substantially more data to make informed conclusions.

<div style="margin-bottom: 40px;">

</div>

### **Correlation between steps and BMI**
```{r}
ggplot(bmi_cor, aes(x = avg_steps, y = avg_bmi)) +
  geom_point(aes(color = "Users"), size = 4) +
  labs(x = "Steps", y = "BMI", title = "Correlation between Steps and BMI",
       color = "Users") +
  scale_color_manual(values = "lightseagreen", guide = "legend") +
  theme_minimal()
```

<img src="https://github.com/S-a-m-K/Bellabeatcasestudy/assets/143487615/72a4f9d8-947a-4cd3-a837-ef4f6d27b299" alt="Beischreibung des Bildes" width="700"/>

Despite the limited data, a clear pattern emerges: those who exercise more tend to have a lower BMI.

<div style="margin-bottom: 40px;">

</div>

### **Correlation between active minutes and BMI**
```{r}
ggplot(bmi_cor, aes(x = avg_very_active, y = avg_bmi)) +
  geom_point(aes(color = "Users"), size = 4) +
  labs(x = "Active Minutes", y = "BMI", title = "Correlation Between Active Minutes and BMI",
       color = "Users", size = "Users", color = "Users") +
  scale_color_manual(values = "lightseagreen", guide = "legend") +
  theme_minimal()
```

<img src="https://github.com/S-a-m-K/Bellabeatcasestudy/assets/143487615/741c9100-26b8-4fec-bec7-e527ded7f5bd" alt="Beischreibung des Bildes" width="700"/>

When comparing BMI to active minutes, we cannot draw any definitive conclusions. Individuals with a lower BMI are not necessarily those who engage in high levels of activity. Only one user with a high BMI appears to be particularly inactive.

<div style="margin-bottom: 60px;">

</div>

## **Recommendations based on analysis**

* **Step Reminders:** Implement app reminders to notify users of their current step count and how many more steps they need to reach their goals, based on weight or exercise targets.

* **Activity Suggestions:** Provide reminders with ideas for Sunday activities to encourage users to increase calorie expenditure.

* **Sedentary Activity Reduction:** Offer general recommendations through the app to reduce sedentary behavior, such as biking or walking to work instead of driving or taking public transport, and suggesting standing or walking during work breaks.

* **Encouragement for Better Sleep:** Motivate users who experience poor sleep quality to increase their physical activity to enhance their sleep.

* **Comfortable Wear for Tracking:** Design the “Time” watch to be comfortable to wear, encouraging users to track their sleep patterns.

* **Daily Weight Tracking Reminders:** Send reminders to users to weigh themselves daily, suggesting they establish a routine to step on the scale at the same time each morning.

* **Smart Scale Development:** Create a scale that directly transmits weight and body fat percentage to the app, providing additional data for further analysis.

* **Friend Challenges:** Offer challenges that users can engage in with friends, where they can earn rewards based on metrics such as higher activity levels or greater weight loss over a weekly or monthly basis.

<div style="margin-bottom: 40px;">

</div>


## **General recommendations**

* **Progress Notifications:** Send push notifications to update users on their progress toward daily and weekly goals. If a user is behind on their calorie goal, suggest specific activities, such as a walk of an appropriate distance, to help them catch up.

* **Inactivity Alerts:** Notify users who have been inactive for an extended period, encouraging them to get moving again.

* **Health Tracking through Social Interaction:** Enable general health tracking via social features that allow users to share their activity and participate in daily or weekly challenges together.

* **Subscription-Based Features:** Offer certain features exclusively with a subscription-based membership, and send push notifications to highlight the benefits of subscribing.

* **Incentives for Continuous Tracking:** Provide incentives for consistent tracking, such as rewards in a competition-based system. For example, users could earn electronic badges or trophies when they reach daily or weekly goals. Such measures could help legitimize the app and increase the likelihood of users achieving their goals while also generating more data for further analysis.

<div style="margin-bottom: 40px;">

</div>
