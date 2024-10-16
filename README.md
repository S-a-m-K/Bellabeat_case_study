<h1>Bellabeat Case Study</h1> 

**Author: Samuel Kleger**  
**Date: 2023-08-08**

---

#### Table of Content
- [Introduction](#Introduction)
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

<div style="margin-bottom: 40px;">

</div>

# **Introduction**

Bellabeat is a successful small business focused on manufacturing high-tech health and wellness products for women. They have the potential to become a major player in the global smart device market. Urska Srsen, Bellabeat's co-founder and chief creative officer, believes that analyzing fitness data from smart devices could help uncover new growth opportunities for the company.

### **Buisness Task**

Analyzing non-Bellabeat smart device usage data to gain insight into how smart devices are being used. Based on these findings, a general recommendation is desired as to how these trends could influence Bellabeat's marketing strategy.

### **Stakeholder** 


**Urška Sršen**: Co-founder and Chief Creative Officer of Bellabeat

**Sando Mur**: Mathematician and co-founder of Bellabeat; key member of Bellabeat's leadership team

**Marketing-Analytics-Team**: A team of data analytics professionals who collect, analyze and report on data to support Bellabeat's marketing strategy

<div style="margin-bottom: 40px;">

</div>

# **Prepare**



### **Data Description**

The data comes from Bellabeats, a healthcare company that focuses on health tracking systems. These log various data such as: User id, date and time, activity, calories burned and much more. The datasets are suitable for the purpose of this case study and allow answering the company-related questions.

**Source**: 33 participants FitBit Fitness Tracker data from Mobius: https://www.kaggle.com/arashnic/fitbit

**License**: CC0: Public Domain

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

* **License**: Creative Commons Attribution 4.0 International
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

### **Cleaning data set daily_activity**

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

Since we only see null values at first glance, I want to check if the columns are identical
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

### **Cleaning data set sleep_day**

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

### **Cleaning data set weight_info**

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
# We don't know what unit of measure the distances are based on. I omit these columns for this analysis
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

Here we can see a positive correlation between total_steps and calories. The more active we are, the more calories we will burn.

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

If we look at the patterns of each chart, it is clear to see that activity level correlates with calorie expenditure.

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

Illustrating the average steps per weekday, we note that Monday, Tuesday and Saturday are the most active days. Sunday is comparatively low. We also see that the 10,000 steps per day recommended by the WHO are not reached on average.

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

Even when illustrating the average steps taken by users per day, the recommended value is not reached for most of them. In fact, 79% users average below this recommended level.


<div style="margin-bottom: 40px;">

</div>

### **Percentage distribution of activity minutes**

The WHO recommends 150 minutes of moderate aerobic physical activity per week, or 75 minutes of aerobic physical activity. Or a combination of both spread over several days. Unfortunately, we cannot use the data to infer exactly which activity is moderately aerobic and which is aerobic. Assuming that moderate and heavy activity corresponds to fairly active and very active activity, we can see what proportion of users meet these criteria.

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

Even when it comes to the percentage of minutes worked, most of them do not reach the specified target.

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

If we look at the percentage breakdown of the various activities, it becomes clear that we spend most of our waking hours in a seated position.

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

Unfortunately, the data are not sufficient for a clear determination. Still, I wanted to use the data to show how exercise and sleep quality have something in common. Of course, there are many other factors that influence sleep quality. Nonetheless, a pattern can be discerned. namely that those who have more trouble falling asleep are the ones who burn the fewest calories, but are not necessarily the ones who exercise the least.

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

The two users who had tracked their weight data for a month were not very successful. Here, too, we need significantly more data in order to make clear decisions.

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

Despite the lack of data, a clear pattern can be seen here. Those who exercise more also have a lower BMI.

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

When comparing the BMI to the active minutes, we cannot draw any clear conclusions. Those who have a lower BMI are not necessarily those who spend their time very actively. Only one user with a high BMI does not seem to be particularly active.

<div style="margin-bottom: 60px;">

</div>

## **Recommendation based on analysis**

* App reminder how many steps you have taken or how many are still to do to reach your goal. 
  For example, based on weight or movement goals.

* Reminders with ideas for Sunday activities to increase steps or calories on sedentary days.

* Make recommendations via app to reduce sedentary work. 
  For example, going to work by bike or walking instead of taking the car or bus.

* Sedentary users at work spend their breaks standing or walking.

* Encourage users who sleep less well to move more to improve their sleep quality.

* Good wearing comfort of the "Time" wristwatch to promote sleep tracking.

* Reminder to track weight daily. For example, build a routine to get on the scale at the same time every morning.

* Maybe develop a scale that sends weight and body fat percentage directly to the app. 
  Thus, one might have more data for a subsequent analysis.

* Together with a challenge, which takes place among friends, who loses how much weight in what time.

<div style="margin-bottom: 40px;">

</div>


## **General recommendations**

* General push notification of the progress of daily and weekly goals. 
  For example, if he's still behind on his calorie count, suggest a walk of appropriate distance.
  
* Notification of users who have been inactive for a longer period of time.

* overall health tracking through social interactions, allowing users to share their activity and participate in daily/weekly challenges together.

* Allow certain features only with a subscription-based membership. Send push notifications to show subscription benefits.

* Provide incentives for constant tracking. For example in connection with a competition with a reward system. This could be in the form of an electronic identifier like a trophy once the daily or weekly goal is met. Such measures could help such applications become more legitimate and thus increase the likelihood that users will achieve their goals. At the same time, data would be created that could possibly be used for another analysis.

<div style="margin-bottom: 40px;">

</div>

### **Thank you**

<div style="margin-bottom: 40px;">

</div>
