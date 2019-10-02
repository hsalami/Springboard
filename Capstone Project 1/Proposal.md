
### Proposal for Capstone Project 1 

**Title**: Predicting crash severity on city streets of Chicago

**Problem**: Traffic crashes are one of the leading causes of death in the US. Each year, tens of thousands of women and men are killed or injured in traffic accidents. In Chicago for example, more than 2,000 people were killed or seriously injured in 2016 traffic crashes, with an average of five people seriously injured each day and one person killed every three days. City departments are continually making efforts to improve roadway safety and to reduce fatalities and serious injuries. One of these efforts is based on data-driven models that aim to understand the factors and causes of traffic collisions, and reveal any pattern or trend from past accidents. In this project, we propose to use the datasets of Chicago traffic crashes to build a model that predicts crash severity.

**Who might care?** The city of Chicago can use the model to understand the leading factors of a severity of a crash and to identify any risky driving behaviors. In this way, the model can help them decide on whether to improve the infrastructure, to incorporate more safety features on the roads or to develop better traffic control policies.   

**Data**: The data will be acquired from [Chicago Data Portal - Traffic Crashes] (https://data.cityofchicago.org/Transportation/Traffic-Crashes-Crashes/85ca-t3if). The data contains information about each traffic crash on city streets of Chicago from 2015 to present, including the time and location of the crash, weather, road and lighting conditions, type of crash and its primary cause, injuries and damage, etc. The same data portal also has the following two datasets: [Traffic-Crashes-Vehicles](https://data.cityofchicago.org/Transportation/Traffic-Crashes-Vehicles/68nd-jvt3) and [Traffic-Crashes-People](https://data.cityofchicago.org/Transportation/Traffic-Crashes-People/u6pd-qa9d), which contain information about the vehicles and people involved in the crashes.

**Modeling Approach**: The crashes provided in the data are classified into two classes according to the crash type (a general severity classification for the crash): 1) No Injury / Drive Away or 2) Injury and/or Tow Due to Crash. In this project, we will consider the problem of building a predictive model for the crash type, which is a supervised classification problem. We will explore what features to extract from the data in order to incorporate them in the model. Some possible features include weather and road conditions, spatial and temporal information, vehicle information and driver’s behavior. Note that the data contains imbalanced classes where 22% of the data belong to the second class (Injury and/or Tow Due to Crash) and 78% belong to the first class (No Injury / Drive Away). We will address this issue while building the predictive model.

**Deliverables**: Here are the deliverables of the project:
1.	Codes detailing the steps done to analyze the data;
2.	Report on the capstone project explaining the problem, our approach, and our findings;
3.	Slide deck for the project.

