

```python
import pandas as pd

# A csv file of Chicago crashes was downloaded on September 24, 2019 from 
# https://data.cityofchicago.org/Transportation/Traffic-Crashes-Crashes/85ca-t3if

# Read the file and save the data as crashes dataframe
crashes=pd.read_csv('DataCleaning/Traffic_Crashes.csv')
```


```python
# First remove irrelevant or mostly-empty columns

crashes=crashes.drop(columns=['CRASH_DATE_EST_I','LANE_CNT','ROAD_DEFECT','REPORT_TYPE', 'INTERSECTION_RELATED_I',
                              'NOT_RIGHT_OF_WAY_I','HIT_AND_RUN_I','DATE_POLICE_NOTIFIED','SEC_CONTRIBUTORY_CAUSE',
                              'PHOTOS_TAKEN_I','STATEMENTS_TAKEN_I','DOORING_I','WORK_ZONE_I','WORK_ZONE_TYPE', 
                              'WORKERS_PRESENT_I','LOCATION'])
```


```python
# Posted speed limit
# This column contains a lot of inconsistencies, which we try to fix here

# check if the location of the crash is a parking lot, alley or driveway, then the speed limit should be 15 mph
crashes.loc[crashes.TRAFFICWAY_TYPE.isin(['PARKING LOT','ALLEY','DRIVEWAY']),
            'POSTED_SPEED_LIMIT']=15

# this is a list of of substrings that are contained in the names of Chicago streets where the speed limit is at least 55 mph
list_roads=['SKYWAY','SHORE','95TH','EXPY','XPY','DAN RYAN','XPRS','CICERO','ROOSEVELT','INDIANAPOLIS','IRVING','DAN RYAN',
            '98TH PL','FETRIDGE','MANNHEIM','WENTWORTH']            

# some posted speed limits are 0, 99 and less than 10 mph which are unreasonable values 
# find where the speed is unreasonable
is_spd_unreasonable=(crashes.POSTED_SPEED_LIMIT<10) | (crashes.POSTED_SPEED_LIMIT==99) 

# check which street is possibly a highway 
is_highwy=crashes.STREET_NAME.str.contains(list_roads[0])
for road in list_roads[1:]:
    is_highwy = is_highwy |(crashes.STREET_NAME.str.contains(road))
    
# adjust the unreasonable speed to 55mph if it is a highway    
crashes.loc[(is_spd_unreasonable & is_highwy),'POSTED_SPEED_LIMIT']=55

# otherwise, assume the street as normal one with 30 mph as speed limit
crashes.loc[crashes.POSTED_SPEED_LIMIT<10,'POSTED_SPEED_LIMIT']=30
crashes.loc[crashes.POSTED_SPEED_LIMIT==99,'POSTED_SPEED_LIMIT']=30

# some speed limits were high for normal streets, adjust them here
crashes.loc[((~is_highwy) & (crashes.POSTED_SPEED_LIMIT>=50)),'POSTED_SPEED_LIMIT']=30           

# regroup the speeds into 15,20,25,30,35,40,45,50,55,60,65 to round up any in between value
crashes.loc[(crashes.POSTED_SPEED_LIMIT>10) & (crashes.POSTED_SPEED_LIMIT<15),'POSTED_SPEED_LIMIT']=15
crashes.loc[(crashes.POSTED_SPEED_LIMIT>15) & (crashes.POSTED_SPEED_LIMIT<20),'POSTED_SPEED_LIMIT']=20
crashes.loc[(crashes.POSTED_SPEED_LIMIT>20) & (crashes.POSTED_SPEED_LIMIT<25),'POSTED_SPEED_LIMIT']=25
crashes.loc[(crashes.POSTED_SPEED_LIMIT>25) & (crashes.POSTED_SPEED_LIMIT<30),'POSTED_SPEED_LIMIT']=30
crashes.loc[(crashes.POSTED_SPEED_LIMIT>30) & (crashes.POSTED_SPEED_LIMIT<35),'POSTED_SPEED_LIMIT']=35
crashes.loc[(crashes.POSTED_SPEED_LIMIT>35) & (crashes.POSTED_SPEED_LIMIT<40),'POSTED_SPEED_LIMIT']=40
crashes.loc[(crashes.POSTED_SPEED_LIMIT>40) & (crashes.POSTED_SPEED_LIMIT<45),'POSTED_SPEED_LIMIT']=45
crashes.loc[(crashes.POSTED_SPEED_LIMIT>45) & (crashes.POSTED_SPEED_LIMIT<50),'POSTED_SPEED_LIMIT']=50
crashes.loc[(crashes.POSTED_SPEED_LIMIT>50) & (crashes.POSTED_SPEED_LIMIT<55),'POSTED_SPEED_LIMIT']=55
crashes.loc[(crashes.POSTED_SPEED_LIMIT>55) & (crashes.POSTED_SPEED_LIMIT<60),'POSTED_SPEED_LIMIT']=60
crashes.loc[(crashes.POSTED_SPEED_LIMIT>60) & (crashes.POSTED_SPEED_LIMIT<65),'POSTED_SPEED_LIMIT']=65
```


```python
# Check the entries of speed limit
print(crashes.POSTED_SPEED_LIMIT.value_counts().sort_index())
```

    10      1079
    15     35662
    20     11028
    25     19284
    30    246607
    35     22446
    40      3078
    45      2013
    50        44
    55       657
    60        13
    65         5
    70         1
    Name: POSTED_SPEED_LIMIT, dtype: int64
    


```python
# Weather Conditions

crashes.CRASH_DATE=pd.to_datetime(crashes.CRASH_DATE)
crashes=crashes.set_index('RD_NO')

# find wich entries has unknown weather conditions and which entries has known weather conditions
is_unknown=(crashes.WEATHER_CONDITION=='UNKNOWN')|(crashes.WEATHER_CONDITION=='OTHER')
subset=crashes.loc[is_unknown,['CRASH_DATE','WEATHER_CONDITION']]
subset2=crashes.loc[~is_unknown,['CRASH_DATE','WEATHER_CONDITION']]

# normalize the date to remove hours
subset['DAY']=subset.CRASH_DATE.dt.normalize()
subset2['DAY']=subset2.CRASH_DATE.dt.normalize()

# build a table with days and the dominant weather condition for each day from the data with known weather conditions
d=subset2.groupby('DAY')['WEATHER_CONDITION'].value_counts()
d=d.unstack()
table=d.idxmax(axis=1)

# fill the unknwon weather conditions usign the table that we just created
for index,row in subset.iterrows():
       if (row['DAY'] in table.keys()):
            crashes.at[index,'WEATHER_CONDITION']=table[row['DAY']]
```


```python
# three entries were left unknow, we remove their corresponding rows
still_unknown=(crashes.WEATHER_CONDITION=='UNKNOWN')
crashes=crashes.drop(index=crashes.index[still_unknown])

print(crashes.WEATHER_CONDITION.value_counts())
```

    CLEAR                     285456
    RAIN                       32222
    SNOW                       12563
    CLOUDY/OVERCAST            10336
    FOG/SMOKE/HAZE               640
    SLEET/HAIL                   556
    SEVERE CROSS WIND GATE        75
    FREEZING RAIN/DRIZZLE         64
    BLOWING SNOW                   2
    Name: WEATHER_CONDITION, dtype: int64
    


```python
# Lighting Conditions

# define a function that takes the hour and the month of a day and return: dusk, dawn, daylight or darkness accordingly
def mapToLight(hour,month):
    if ((hour==5 & month in ([8,7,4]))| (hour==4 & month in ([5,6]))|(hour==6 & month in ([1,2,3,9,10,11,12]))):
        return 'DAWN'
    if ((hour<19 & month in ([8,7,4,3,2,9]))| (hour<20 & month in ([5,6]))
        |(hour<18 & month==10)|(hour<16 & month in ([1,11,12]))):
        return 'DAYLIGHT'
    if ((hour==19 & month in ([8,7,4,3,2,9]))| (hour==20 & month in ([5,6])) 
        |(hour==16 & month in ([1,12,11]))|(hour==18 & month==10)):
        return 'DUSK'
    else:
        return 'DARKNESS'

# fill the unkown entries for ligting condition using the defined function mapToLight
noLight= crashes.loc[(crashes.LIGHTING_CONDITION=='UNKNOWN'),['CRASH_HOUR','CRASH_MONTH']]

for index,row in noLight.iterrows():
    crashes.at[index,'LIGHTING_CONDITION']=mapToLight(row.CRASH_HOUR,row.CRASH_MONTH) 
```


```python
print(crashes.LIGHTING_CONDITION.value_counts())
```

    DAYLIGHT                  227955
    DARKNESS, LIGHTED ROAD     68927
    DARKNESS                   28650
    DUSK                       10471
    DAWN                        5911
    Name: LIGHTING_CONDITION, dtype: int64
    


```python
# Roadway surface

# fill the unkown road surface according to the weather conditions
crashes.loc[crashes.ROADWAY_SURFACE_COND.isin(['UNKNOWN','OTHER']) & crashes.WEATHER_CONDITION.isin(['RAIN']),
            'ROADWAY_SURFACE_COND']='WET'
crashes.loc[crashes.ROADWAY_SURFACE_COND.isin(['UNKNOWN','OTHER']) & crashes.WEATHER_CONDITION.isin(['SNOW']),
            'ROADWAY_SURFACE_COND']='SNOW OR SLUSH'
crashes.loc[crashes.ROADWAY_SURFACE_COND.isin(['UNKNOWN','OTHER']) & crashes.WEATHER_CONDITION.isin(['FREEZING RAIN/DRIZZLE']),
            'ROADWAY_SURFACE_COND']='ICE'
crashes.loc[crashes.ROADWAY_SURFACE_COND.isin(['UNKNOWN','OTHER']) & crashes.WEATHER_CONDITION.isin(['SLEET/HAIL']),
            'ROADWAY_SURFACE_COND']='SNOW OR SLUSH'
crashes.loc[crashes.ROADWAY_SURFACE_COND.isin(['UNKNOWN','OTHER']),
            'ROADWAY_SURFACE_COND']='DRY'
```


```python
print(crashes.ROADWAY_SURFACE_COND.value_counts())
```

    DRY                278571
    WET                 48263
    SNOW OR SLUSH       12457
    ICE                  2448
    SAND, MUD, DIRT       175
    Name: ROADWAY_SURFACE_COND, dtype: int64
    


```python
#Traffic Control Device

# combine the different labels Railroad signs into one category
crashes.loc[crashes.TRAFFIC_CONTROL_DEVICE.isin(['RAILROAD CROSSING GATE','OTHER RAILROAD CROSSING','RR CROSSING SIGN']),
            'TRAFFIC_CONTROL_DEVICE']='RAILROAD CROSSING SIGN'            

# make the columns 'traffic control device' and 'device condition consistent
# fill the unknown entries of 'traffic control device' from the column 'device condition'
crashes.loc[(crashes.TRAFFIC_CONTROL_DEVICE=='UNKNOWN') & (crashes.DEVICE_CONDITION=='NO CONTROLS'),
            'TRAFFIC_CONTROL_DEVICE']='NO CONTROLS'
crashes.loc[crashes.TRAFFIC_CONTROL_DEVICE=='NO CONTROLS','DEVICE_CONDITION']='NO CONTROLS'
crashes.loc[(crashes.DEVICE_CONDITION=='NO CONTROLS') & (crashes.TRAFFIC_CONTROL_DEVICE!='NO CONTROLS'),
            'DEVICE_CONDITION']='FUNCTIONING PROPERLY'
crashes.loc[crashes.DEVICE_CONDITION.isin(['UNKNOWN','MISSING']),'DEVICE_CONDITION']='UNKNOWN'
```


```python
print(crashes.TRAFFIC_CONTROL_DEVICE.value_counts())
```

    NO CONTROLS                 197823
    TRAFFIC SIGNAL               95425
    STOP SIGN/FLASHER            32972
    UNKNOWN                      10608
    OTHER                         1994
    LANE USE MARKING              1224
    YIELD                          475
    RAILROAD CROSSING SIGN         329
    OTHER REG. SIGN                306
    OTHER WARNING SIGN             289
    SCHOOL ZONE                    138
    POLICE/FLAGMAN                 136
    DELINEATORS                     82
    PEDESTRIAN CROSSING SIGN        64
    FLASHING CONTROL SIGNAL         31
    NO PASSING                      10
    BICYCLE CROSSING SIGN            8
    Name: TRAFFIC_CONTROL_DEVICE, dtype: int64
    


```python
print(crashes.DEVICE_CONDITION.value_counts())
```

    NO CONTROLS                 197823
    FUNCTIONING PROPERLY        123296
    UNKNOWN                      16122
    OTHER                         2043
    FUNCTIONING IMPROPERLY        1861
    NOT FUNCTIONING                611
    WORN REFLECTIVE MATERIAL       158
    Name: DEVICE_CONDITION, dtype: int64
    


```python
# All remaining columns with unknown entries are left as they are

# The last step is to make the column "m injuryost severe injury" consistent with "crash type" (injury or no inujry)
# The entires that are empty for the column "most severe" are filled with "no inidcation of injury" when the crash type is
# no injury
crashes.loc[(crashes.MOST_SEVERE_INJURY.isnull()) & (crashes.CRASH_TYPE=='NO INJURY / DRIVE AWAY'),
            'MOST_SEVERE_INJURY']='NO INDICATION OF INJURY'
```
