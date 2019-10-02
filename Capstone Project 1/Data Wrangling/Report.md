
 ## Overview
 
This report describes the steps done to clean the dataset of [Chicago Traffic Crashes](https://data.cityofchicago.org/Transportation/Traffic-Crashes-Crashes/85ca-t3if). The main issue with the data is that it includes some missing values and outliers due to human errors. In this document, we explain how we handled this issue and made the data ready for crash severity analysis.

## First Step: Removing redundant columns

We first removed all irrelevant and mostly empty columns. The obtained data consists of the following columns:

1. Identification number of the crash;
2. Location information: street number, direction and name, beat of occurrence, latitude and longitude of the crash location;
3. Time information: the date, month, hour, and day of the week of the crash; 
4. Crash description and conditions: posted speed limit, traffic control device and its condition, weather condition, lighting condition, first collision type, traffic-way type, roadway surface condition, alignment of the road, primary contributory causes;
5. Crash damage and injuries: crash type (1- no injury/drive away or 2- injury or/and tow due to crash), damage cost in dollars, type of the most severe injury.

The first three types of information do not have any outliers or missing values. The main focus was for the fourth type: crash description and conditions, which are mostly categorical columns. These columns contain some missing or unknown entries and some outliers. The strategies we followed to deal with missing entries are either deducing their values from the available data or leaving them as an 'unknown/other' category. The latter strategy was especially followed to avoid removing the corresponding rows of the missing entries and thus to avoid losing any possible information from other columns. We next describe what entries we were able to fill and what entries we left as an additional category.

## Second Step: Filling the missing entries

We describe how we filled some of the missing entries.

- **Posted Speed Limit**: This column indicates the posted speed limit (in miles per hour (MPH)) for the crash location. By checking the values of this column, we notice that some values are 0 or 99, which are considered as unreasonable numbers for the speed limit. To replace these values with reasonable ones, we first checked the column of the traffic-way type, which clarifies the location type of the crash (parking lot, driveway, alley, divided or undivided highway, ...). If the location of the crash is a parking lot, driveway or alley, we adjusted the speed limit to 15 MPH. On the other hand, by checking the name of the street, if the street is a highway or expressway, we adjusted the speed limit to 55 MPH. The remaining streets are assumed to have 30 MPH as speed limit (30 MPH is the speed limit of the most city streets of Chicago and it represents the maximum speed limit when there is no posted speed limit). We then regrouped the speeds into the following values: 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60, 65 and 70. This is because there were some values in between these numbers (18, 23, 36, ...) which are not considered as common posted speed limits and which we believe are caused because of human errors.

- **Weather Condition**: The possible weather conditions reported at the time of the crash include: rain, clear, ice, snow, wind, ... The unknown entries of this column were deduced from the date and time of the crash. More specifically, by checking the reported weather conditions of other crashes that happened during the same day, the unknown weather condition of the same day crash was filled accordingly. If more than weather conditions were reported for the same day, we took the most common weather condition. Only three crashes with unknown weather conditions remained unknown, as no other crashes happened during the same day. Since the total number of crashes is around 340k, 3 crashes represent a tiny percentage of the whole data, therefore we removed the three rows with unknown weather conditions.

- **Lighting Condition**: The possible lighting conditions are: daylight, darkness, dusk, dawn. The unknown entries for this column were deduced from the time and month of the crash.

- **Roadway Surface Condition**: The possible road conditions include: dry, wet, snow or ice, etc... The unknown entries for this column were deduced from the weather condition of the crash. More specifically, if the weather condition is 'rain', the road condition is set to 'wet'. If the weather condition is 'snow' or 'sleet/hail', the road condition is set to 'snow or slush'. If the weather condition is 'freezing rain', the road condition is set to 'ice'. Otherwise, the road condition is set to 'dry'.

## Third Step: Dealing with remaining columns

We describe next the remaining columns with unknown entries which were not possible to be filled and which we preferred keeping as an additional category.

- **Traffic-way type**: This column describes the type of the street location where the crash happened: divided, not divided, one way, parking lot, alley, driveway, ramp, intersection, roundabout,.... However around 3.8k entries are reported as 'unknown' and 34 as 'not reported'. There is no available data for this type of information for the city streets of Chicago. For now, we leave them as an additional category for future deep analysis.

- **Traffic control device**: This column clarifies whether there was a traffic control device (traffic signal, stop sign, yield, railroad crossing, ...) or not at the crash location. However 10k entries are reported as 'unknown', which we also leave as an additional category. Moreover, we grouped the following categories: 'RAILROAD CROSSING GATE','OTHER RAILROAD CROSSING', 'RR CROSSING SIGN' as one category labeled as 'railroad crossing sign'.

- **Device condition**: This column indicates whether the traffic control device was working or not (no control, functioning properly or improperly or not functioning). We first made sure that this column and the previous one (traffic control device) are consistent, so that they both show similar entries when there was no control device. In this column, 16k entries were reported as 'unknown'. We also leave these entries for future deep analysis.

- **Primary contributory causes**: This column includes a lot of possible entries that describe the primary driver's behavior that led to the crash (improper backing, following too closely, improper turning, reckless driving,...). However, around one third of the entries (110K) are reported as 'unable to determine'. We also leave them for future analysis and to investigate if there is any type of driver's behavior that can lead to a severe crash.

The remaining two columns: **alignment of the street** (straight, curve,...) and **first crash type** (angle, sideswipe, rear to rear, rear end,....) have no missing entries.
