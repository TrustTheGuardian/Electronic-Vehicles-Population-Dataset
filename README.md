# Electric Vehicle Population Data
## Documentation

DATA ANALYTICS
BSCS – C301
Group 1: 
1. Gomez, Jason
2. Lopez, Justine
3. Manansala, Joshua
4. Tongol, Katriel Cire
5. Turla, Lance Amiel

Dataset: https://www.kaggle.com/datasets/nilesh2042/electric-vehicle-population-data

## 1. Data Preprocessing
### Data Processing and Normalization

The dataset was cleaned and normalized following the **CLEAN** framework:

* **C — Conceptualize:** Identifies the core entities within the dataset, specifically distinguishing between geographic attributes (postal codes, cities, counties, states) and vehicle-related attributes (VIN numbers, vehicle models, manufacturers).
* **L — Locate:** Pinpoints instances of data redundancy and inconsistency within the raw dataset that need to be resolved.
* **E — Evaluate:** Assesses the complex relationships among these various geographic and vehicle attributes to determine the optimal database architecture.
* **A — Augment:** Enhances the dataset by structuring it into a Snowflake schema design, systematically linking fact tables and sub-dimensions to efficiently manage related details.
* **N — Note and Document:** Confirms that the normalization process successfully reduced data redundancy, improved overall consistency, and strengthened referential integrity across the database.
  
CONCEPTUALIZE THE DATA
The raw dataset contains over 270,000 rows and 16 columns.
- Understanding first what every row represents: Each row in the CSV dataset represents an electric vehicle (EV) registered in a state within the United States of America (USA). A vehicle with a DOL ID indicates that it has been officially purchased, registered, and approved for road use.
- Understanding what each code means: Each record contains a unique Department of Licensing (DOL) ID, geographic information such as postal code, city, county, and state, as well as vehicle identification details including VIN, model year, manufacturer or brand, vehicle model, electric vehicle type, driving range, and eligibility as an alternative fuel vehicle.
- Identify whether things are qualitative or categorical values: The dataset contains geographic dimensions, vehicle specifications, and categorical eligibility types.


### Locate Solvable Issues

Issues found:

1. **Inconsistent Formatting:** Some postal codes were inconsistently formatted, where identical postal codes appeared both with and without leading zeros.
   * **Resolution:** Postal code values were standardized by converting them into consistent numerical representations to simplify processing and comparison.
   
   ![Issue 1](IMAGES/Issue1.png)

2. **Invalid Data Entries:** The electric driving range column contained values of 0, which may indicate missing or invalid data.
   * **Resolution:** Invalid driving range values of 0 were replaced with NULL to properly represent missing or unavailable data.
   
   ![Issue 2](IMAGES/Issue2.png)

3. **Data Redundancy:** The dataset contained redundant and repeated information across multiple rows.
   * **Resolution:** The dataset was normalized into fact tables and sub-dimensions to eliminate redundancy, improve consistency, and ensure that each attribute is stored in only one location.
   
   ![Issue 3](IMAGES/Issue3.png)


### Evaluate Unsolvable Issues

1. **Missing Values:** The dataset contained missing values.
   * **Resolution:** Rather than deleting, these were kept in the sub-dimensions to maintain accurate “total count” KPIs but filtered out of map visuals to prevent errors.
   
   ![Unsolved Issue 1](IMAGES/UnsolvedIssue1.png)
   
   ![Unsolved Issue 2](IMAGES/UnsolvedIssue2.png)

AUGMENT THE DATA
*	During dataset normalization, unique identifiers were assigned to each sub-dimension to properly handle complex relationships and avoid ambiguity. This includes cases where a single postal code belongs to multiple cities, a city spans multiple counties, counties share the same name across different states, or multiple counties exist within a single state.
*	Lookup operations were performed to retrieve and integrate related data from other normalized tables, improving data consistency, relationship mapping, and overall referential integrity.

## Data Dictionary

### EV_Population_Fact
| Field Name | Data Type | Description | Example |
|---|---|---|---|
| `VIN (1-10)` | String | Foreign Key. First 10 characters of the Vehicle Identification Number | `JN1AZ0CP5C` |
| `Postal Code` | Float | Foreign Key. Geographic postal/ZIP code where the vehicle is registered | `99114.0` |
| `DOL Vehicle ID` | Integer | Primary Key. Unique identifier assigned to each vehicle by the Department of Licensing | `153331706` |

### EV_Dim
| Field Name | Data Type | Description | Example |
|---|---|---|---|
| `VIN (1-10)` | String | Primary Key. Partial Vehicle Identification Number | `JN1AZ0CP5C` |
| `Model Year` | Integer | Manufacturing year of the vehicle model | `2012` |
| `Make` | String | Brand or manufacturer of the vehicle | `NISSAN` |
| `Model` | String | Specific model of the vehicle | `LEAF` |
| `EVType_Code` | String | Foreign Key. Code representing the electric vehicle type | `BEV` |
| `Eligibility_Code` | Integer | Foreign Key. Code indicating alternative fuel eligibility status | `1` |
| `Electric Range` | Float | Maximum distance the vehicle can travel on a single battery charge (in miles) | `73.0` |

### EV-Type_Dim
| Field Name | Data Type | Description | Example |
|---|---|---|---|
| `EVType_Code` | String | Primary Key. Abbreviation for the EV type | `BEV` |
| `Electric Vehicle Type` | String | Full text description of the electric vehicle type | `Battery Electric Vehicle` |

### Eligibility_Dim
| Field Name | Data Type | Description | Example |
|---|---|---|---|
| `Eligibility_Code` | Integer | Primary Key. Numeric code for eligibility status | `1` |
| `Clean Alternative Fuel Vehicle Eligibility` | String | Full text description explaining whether the vehicle qualifies for clean alternative fuel vehicle incentives | `Clean Alternative Fuel Vehicle Eligible` |

### State_Dim
| Field Name | Data Type | Description | Example |
|---|---|---|---|
| `State_Code` | String | Primary Key. The 2-letter state abbreviation | `WA` |
| `State` | String | The full name of the state | `Washington` |

### County_Dim
| Field Name | Data Type | Description | Example |
|---|---|---|---|
| `County_Code` | String | Primary Key. Internal code identifying the county | `CC1`|
| `County` | String | The name of the county | `Ada` |
| `State` | String | Foreign Key. Links the county to its state | `ID` |

### City_Dim
| Field Name | Data Type | Description | Example |
|---|---|---|---|
| `City_Code` | String | Primary Key. Internal code identifying the city | `CTY1` |
| `City` | String | The name of the city | `Aberdeen` |
| `County_Code` | String | Foreign Key. Links the city to its respective county | `CC100` |

### Postal_Dim
| Field Name | Data Type | Description | Example |
|---|---|---|---|
| `Postal_Code` | Float | Primary Key. The ZIP/Postal code | `99114.0` |
| `City_Code` | String | Foreign Key. Links the postal code to its specific city | `CTY164` |

---

# 2. Exploratory Data Analysis (EDA)

KPI's and Measures:
1.	Clean EV% - Measure to show the percentage of clean Electric vehicles used in combination with slicers for manufacturers and models to see how much of the EVs produced by them are clean fuel alternatives
```dax
Clean EV % = DIVIDE(CALCULATE(COUNTROWS('EV_Fact'), Vehicle_Dim[Eligibility_ID]= 1), COUNTROWS('EV_Fact'))
```
2.	Total EVs – Measure to count the total amount of registered EVs, used to see how many EVs there are and combined with slicer to more specifically see how much of a certain model or a certain manufacturers amount of EVs.
```dax
Total EVs = CALCULATE(COUNTROWS('EV_Fact')) 
```
3.	Average Electric Range – Measure for getting the average electric range of the EVs, used mainly to show the current general average range of EVs but with slicers it can used to show the average range of certain models or certain manufacturers’ vehicles
```dax
Average Electric Range = CALCULATE(AVERAGE(Vehicle_Dim[Electric Range]))
```
4.	Current EV Models – shows the total number of EV models, can be used with the manufacturer slicer to see the amount of models from a manufacturer.
```dax
Current EV Models = CALCULATE(DISTINCTCOUNT(Vehicle_Dim[Model]))
```
5.	Distinct Model per Year -shows the amount of models made/released in a year, in this case its used to show how if a model is being made or being released in that year since usually its just 1 of the model per year
```dax
Distinct Models per Year = DISTINCTCOUNT(Vehicle_Dim[Model Year])
```
6.	Electrical Range Spread – shows the difference between the vehicle with the highest electrical range and the vehicle with the lowest electrical range, used with slicers to specify the spread of range either per model or base on all the models of a manufacturer.
```dax
Electrical Range Spread = MAX(Vehicle_Dim[Electric Range]) - MIN(Vehicle_Dim[Electric Range])
```
7.	EV Manufacturers- used to count how many EV manufacturers there are, used with a year slicer to see how many manufacturers there are per year.
```dax
EV Manufacturers = DISTINCTCOUNT(Vehicle_Dim[Make])
```
8.	Growth % - used to show how much more or less EVs there are compared to the last year, can be used again with slicers
```dax
Growth % = DIVIDE([Model Growth], [Previous Year Models])
```
9.	Model Growth – used by the above measure to calculate the difference between the current and last year’s amount of EVs
```dax
Model Growth = [Total EVs] - [Previous Year Models]
```
10.	Previous Year Model – used to calculate the models of the last year, used mainly in the Growth % measure.
```dax
Previous Year Models = CALCULATE([Total EVs], FILTER(ALL('Vehicle_Dim'),'Vehicle_Dim'[Model Year] = MAX('Vehicle_Dim'[Model Year]) - 1))
```
11.	Max Electric Range – used to show the highest range a model has.
```dax
Max Electric Range = MAX(Vehicle_Dim[Electric Range])
```
12.	Min Electric Range – used to show the lowest range a model has.
```dax
Min Electric Range = MIN(Vehicle_Dim[Electric Range])
```
---

# 3. Data Modeling Analytics
The dataset was cleaned and normalized using the CLEAN framework and structured following a Snowflake schema design.  
<image src="IMAGES/DataModel.png">

## Snowflake Schema Implementation
This approach was used to efficiently manage complex relationships among geographic attributes such as postal codes, cities, counties, and states, as well as vehicle-related attributes including VIN numbers, vehicle models, manufacturers, and other related details. The normalization process reduced data redundancy, improved consistency, and strengthened referential integrity across the database.  
By separating entities into EV_Population_Fact, EV_Dim, and hierarchical location tables (Postal_Dim -> City_Dim -> County_Dim -> State_Dim), the model optimizes query performance and allows for accurate aggregation without counting duplicate text values.  

## Descriptive and Predictive Analytics
The study utilized a combination of descriptive and predictive analytics to provide a comprehensive view of the electric vehicle (EV) market. The objective was not only to understand historical adoption patterns but also to forecast future growth.

**Descriptive Analytics** was used to analyze, summarize, and present the existing EV population data.

* **Focus:** Identifying trends, distributions, frequencies, and relationships among variables such as geographic location (State, County, Postal Code), vehicle manufacturer (Make/Model), electric range, and EV type (BEV vs. PHEV).
* **Questions Answered:** "What happened?", "What is the current state of the EV market?", and "Where are the existing EV adoption hotspots?"
* **Application:** The team used statistical summaries, Power BI visualizations, KPIs, and measures such as Total EVs Registered, Average Electric Range, and BEV vs. PHEV Proportion based on historical Department of Licensing (DOL) records.

**Predictive Analytics** was applied to project future market behavior and anticipate infrastructure needs based on historical data trends.

* **Focus:** Forecasting the future volume of electric vehicles entering the market and evaluating potential market acceleration or saturation.
* **Questions Answered:** "What will happen next?" and "At what rate will EV adoption continue to grow?"
* **Application:** The study involved forecasting techniques—specifically utilizing Power BI's built-in exponential smoothing algorithms. This generated predictive models like the Primary Volume Predictor, which uses trend lines and 95% statistical confidence intervals to illustrate best-case (rapid adoption) and worst-case (market disruption) future scenarios.

---

# 4. Visualization and Dashboard

The dashboard design follows the **DASH** framework:

* **D — Decision:** Enables policymakers, infrastructure planners, and auto manufacturers to see where EV adoption is highest, determining where to build charging stations or target market incentives.
* **A — Audience:** Targeted at **Government Transportation Officials** and **Infrastructure Planners** who need both high-level geographic breakdowns and granular vehicle performance metrics.
* **S — Signal:** Emphasizes key signals like Total Registered EVs, BEV vs. PHEV distributions, and the Average Electric Range, immediately highlighting the current state of EV technology adoption.
* **H — Hierarchy:** The dashboard flows from high-level geographic adoption maps and total counts, drilling down into manufacturer dominance (Make/Model), and finally evaluating clean-energy eligibility and electric range limits.

### Descriptive Analytics Dashboard

![Descriptive Dashboard](IMAGES/DescDash.png)

### Predictive Analytics Dashboard

![Predictive Dashboard](IMAGES/PredDash.png)

---

# 5. Insights and Recommendations

## Executive Summary

This report presents findings, insights, and actionable recommendations from the Exploratory Data Analysis (EDA) of the Electric Vehicle (EV) Population Dataset sourced from the Department of Licensing (DOL) via Kaggle. 

The dataset covers Battery Electric Vehicles (BEVs) and Plug-in Hybrid Electric Vehicles (PHEVs) registered across the USA, spanning vehicle, geographic, and technical attributes from model years 1999 to 2027. The data was cleaned and preprocessed using the **CLEAN** framework and structured under a Snowflake schema. Analysis was conducted in Power BI using custom DAX measures and an interactive three-page dashboard (Overview and Forecast).

### High-Level KPIs
| Total EVs Registered | Current EV Models | Avg. Electric Range | Clean EV % |
| :--- | :--- | :--- | :--- |
| 280K | 187 | 70.48 mi | 27.70% |

---

## Key Insights

The following insights are drawn directly from the Power BI dashboard KPIs, DAX measures, and visualizations built by Group 1. Each insight is grounded in actual data outputs and interpreted within a real-world context.

### 1. Tesla Dominates the EV Market Across All Manufacturers
The "Total EVs by each Manufacturer" chart on the Overview dashboard shows Tesla leading by an extraordinary margin over all 47 registered manufacturers. Chevrolet, Nissan, Ford, Kia, Toyota, BMW, and Hyundai follow as distant competitors. Out of 280,000 total registered EVs, Tesla's volume is more than double the nearest rival. 

This dominance reflects the brand's early market entry, aggressive pricing on its most popular models (which also rank among the highest in Average Electric Range), and its proprietary charging network. Within a competitive field of 47 manufacturers, this level of concentration introduces a structural dependency risk for the state's EV ecosystem as a whole.

### 2. BEV Adoption Outpaces PHEV at 80.02% of Total Registrations
The "Total EVs by Electric Vehicle Type" donut chart on the Overview page shows Battery Electric Vehicles accounting for 224K registrations (80.02%), while Plug-in Hybrid Electric Vehicles represent 56K (19.98%). 

This 4:1 ratio reflects a clear consumer and policy shift away from transitional hybrid technology. The Forecast page further supports this trend, with BEVs maintaining a consistently higher registration trajectory across all model years. This aligns with Zero Emission Vehicle (ZEV) standards and hydroelectric-heavy power grids, which make full BEV ownership a significantly lower-carbon option compared to most other U.S. states.

### 3. Electric Range Disparity Reveals a 336-Mile Spread Across Registered Models
The Overview dashboard KPIs show that the Average Electric Range across all registered EVs is 70.48 miles. The "Average Electrical Range of each Model" bar chart shows top models such as the Porsche Macan EV, Tesla Model Y, Hyundai Kona EV, Tesla Model 3, and Chevrolet Bolt EV all exceeding 200 miles. 

The low overall average is pulled down by older, lower-range vehicles still actively registered, indicating an aging fleet segment that may benefit from targeted upgrade programs.

### 4. Plug-in Hybrid Electric Vehicle Range Stagnates Due to "Two-Engine" Engineering Limits
The forecasting data shows Plug-in Hybrid (PHEV) electric ranges flatlining because building a perfectly balanced "half-gas, half-electric" car is physically and economically impractical. Housing a large battery alongside a full internal combustion engine creates massive weight penalties that destroy efficiency, spatial constraints that compromise cargo room, and doubles both the manufacturing costs and points of mechanical failure. Consequently, automakers cap PHEV batteries, limiting them strictly to transitional, short-commute vehicles rather than long-term EV competitors.
---

## Recommendations

Based on the key insights derived from the Power BI dashboard analysis, the following actionable recommendations are proposed for policymakers, urban planners, EV industry stakeholders, and state government agencies.

### 1. Develop Targeted EV Adoption Programs Outside Major Urban Centers
Seattle and its surrounding areas account for a disproportionate share of the 280K registered EVs, while rural counties represent a minimal portion. States should implement geographically targeted incentive programs that include expanded Level 2 and DC Fast Charger deployment in mid-size cities (like Spokane, Tacoma, and Yakima) as well as rural corridors. State purchase subsidies should be prioritized for residents outside major urban hubs to directly address the urban-rural adoption gap visible in the `City_Dim` geographic distribution.

### 2. Introduce Manufacturer Diversity Incentives to Reduce Market Concentration Risk
Tesla's overwhelming lead represents a concentration risk for the EV fleet. A disruption from a single manufacturer due to supply chain issues, regulatory changes, or reputational events could significantly affect infrastructure reliability. State rebate programs should be restructured to offer enhanced incentives for non-Tesla BEV purchases from brands like Hyundai, Ford, Kia, and BMW. This approach broadens consumer choice across price points and supports more equitable access to EV adoption across different income groups.

### 3. Launch a Fleet Modernization Initiative to Raise the Average Electric Range
The Average Electric Range of 70.48 miles is far below the top-performing models that reach beyond 300 miles. This gap points to a significant portion of the registered fleet still comprising older, lower-range vehicles. A state-backed fleet modernization program offering trade-in incentives for pre-2018 EVs would encourage upgrades to newer, higher-range models. This would directly improve the Average Electric Range KPI, reduce range anxiety as a barrier to EV adoption, and accelerate the retirement of aging vehicles with lower battery efficiency and higher maintenance costs.

---

## Conclusion

The EDA conducted by Group 1 on the Electric Vehicle Population Dataset—built on a Snowflake schema with over 280,000 records and visualized through a three-page Power BI dashboard—reveals an EV market that is growing rapidly, geographically concentrated, and structurally dominated by a single manufacturer.

The three key insights (Tesla's market dominance, the 80.02% BEV adoption rate, and the 70.48-mile average range) provide a data-accurate picture of the USA's current EV landscape. Each finding is directly traceable to the dashboard KPIs and DAX measures developed by the team.

The three corresponding recommendations offer a clear, actionable roadmap: expand charging infrastructure in underserved areas, diversify manufacturer incentives, and modernize the aging EV fleet. Together, these measures support the USA's continued progress in sustainable transportation and its greenhouse gas reduction commitments.
