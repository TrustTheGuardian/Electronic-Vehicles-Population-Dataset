# Electric Vehicle Population Data
DATA ANALYTICS
BSCS – C301
Group 1: 
1. Gomez, Jason
2. Lopez, Justine
3. Manansala, Joshua
4. Tongol, Katriel Cire
5. Turla, Lance Amiel

DOCUMENTATION
Dataset: 
## 1. Data Preprocessing
CONCEPTUALIZE THE DATA
The raw dataset contains over 270,000 rows and 16 columns.
- Understanding first what every row represents: Each row in the CSV dataset represents an electric vehicle (EV) registered in a state within the United States of America (USA). A vehicle with a DOL ID indicates that it has been officially purchased, registered, and approved for road use.
- Understanding what each code means: Each record contains a unique Department of Licensing (DOL) ID, geographic information such as postal code, city, county, and state, as well as vehicle identification details including VIN, model year, manufacturer or brand, vehicle model, electric vehicle type, driving range, and eligibility as an alternative fuel vehicle.
- Identify whether things are qualitative or categorical values: The dataset contains geographic dimensions, vehicle specifications, and categorical eligibility types.

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

### Locate Solvable Issues

### Locate Solvable Issues

Issues found:

1. **Inconsistent Formatting:** Some postal codes were inconsistently formatted, where identical postal codes appeared both with and without leading zeros.
   * **Resolution:** Postal code values were standardized by converting them into consistent numerical representations to simplify processing and comparison.
   
   ![Issue 1](Issue1.png)

2. **Invalid Data Entries:** The electric driving range column contained values of 0, which may indicate missing or invalid data.
   * **Resolution:** Invalid driving range values of 0 were replaced with NULL to properly represent missing or unavailable data.
   
   ![Issue 2](Issue2.png)

3. **Data Redundancy:** The dataset contained redundant and repeated information across multiple rows.
   * **Resolution:** The dataset was normalized into fact tables and sub-dimensions to eliminate redundancy, improve consistency, and ensure that each attribute is stored in only one location.
   
   ![Issue 3](Issue3.png)

---

### Evaluate Unsolvable Issues

1. **Missing Values:** The dataset contained missing values.
   * **Resolution:** Rather than deleting, these were kept in the sub-dimensions to maintain accurate “total count” KPIs but filtered out of map visuals to prevent errors.
   
   ![Unsolved Issue 1](UnsolvedIssue1.png)
   
   ![Unsolved Issue 2](UnsolvedIssue2.png)

AUGMENT THE DATA
•	During dataset normalization, unique identifiers were assigned to each sub-dimension to properly handle complex relationships and avoid ambiguity. This includes cases where a single postal code belongs to multiple cities, a city spans multiple counties, counties share the same name across different states, or multiple 
counties exist within a single state. 
•	Lookup operations were performed to retrieve and integrate related data from other normalized tables, improving data consistency, relationship mapping, and overall referential integrity.

NOTE AND DOCUMENT
The dataset was cleaned and normalized using the CLEAN framework and structured following a Snowflake schema design. This approach was used to efficiently manage complex relationships among geographic attributes such as postal codes, cities, counties, and states, as well as vehicle-related attributes including VIN numbers, vehicle models, manufacturers, and other related details. The normalization process reduced data redundancy, improved consistency, and strengthened referential integrity across the database.

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

# 3. Data Modeling Analytics
The dataset was cleaned and normalized using the CLEAN framework and structured following a Snowflake schema design.  
<image src="DataModel.png">
Snowflake Schema ImplementationThis approach was used to efficiently manage complex relationships among geographic attributes such as postal codes, cities, counties, and states, as well as vehicle-related attributes including VIN numbers, vehicle models, manufacturers, and other related details. The normalization process reduced data redundancy, improved consistency, and strengthened referential integrity across the database.  
By separating entities into EV_Population_Fact, EV_Dim, and hierarchical location tables (Postal_Dim -> City_Dim -> County_Dim -> State_Dim), the model optimizes query performance and allows for accurate aggregation without counting duplicate text values.  







1. Total EVs by Model Year (Top Right)
Role: The Primary Volume Predictor
This line chart represents the core of the forecasting analysis. It utilizes Power BI's built-in
exponential smoothing algorithms to project the future volume of electric vehicles entering
the market.
The Trend Line: The solid blue line tracks historical adoption. The shift from a flat
trajectory to an exponential curve indicates market acceleration.
The Grey Cone (Confidence Interval): This shaded area is crucial for presenting forecasts.
It represents the 95% statistical confidence interval. The upper bound shows the "best-case
scenario" (rapid, continued adoption), while the lower bound shows the "worst-case
scenario" (market saturation or disruption).
Presentation Note for the Team: When explaining this chart, point out the sharp drop at
the very end of the blue line. You must explain that this is a data artifact representing the
current, incomplete year. To achieve an accurate forecast during the live presentation, you
will demonstrate applying the "Ignore last 1 point" parameter, which will immediately
correct the forecast line to reflect true upward market momentum.
•
•
•
2. Scatter Chart: Range vs. Time (Bottom Left)
Role: Technological Trajectory & Linear Regression
While the line chart forecasts how many cars will exist, this scatter chart forecasts how capable
those cars will be. It shifts the analysis from market volume to technological evolution.
Linear Regression (Trend Line): The dashed black line across the chart calculates the line
of best fit for battery efficiency. By extending this line mentally, we can predict the baseline
range expectations for vehicles released in 2030 and beyond.
Cluster Analysis: The chart plots historical models by type (light blue for Battery EVs, dark
blue for Plug-in Hybrids). The visible separation of these clusters proves that pure electric
technology is advancing rapidly, while hybrid technology is statically anchored by its
reliance on combustion engines.
Presentation Note for the Team: Address the fact that the trend line appears relatively
flat. Explain that this is due to the heavy volume of low-range hybrids pulling the overall
mathematical average down. This transitions perfectly into explaining why the slicers
(filters) are required to run accurate, isolated forecasts for pure BEVs.
3. Growth % by Model Year (Bottom Middle)
Role: Velocity and Momentum Validation
Forecasting requires an understanding of momentum. It is dangerous to forecast purely on
absolute numbers without checking the underlying growth rate.
Validating the Curve: The massive spike around 2011 represents the initial explosion of
modern EV adoption. The subsequent columns show lower, but stabilized, year-over-year
percentage growth.
Predictive Value: As long as the growth percentage remains positive, the total volume
curve (Chart 1) will continue to compound. If this chart began showing negative columns, it
would act as a leading indicator that the exponential growth forecasted in the primary
chart is at risk of failing.
•
•
•
•
•
4. Average Electric Range Area Chart (Bottom Right)
Role: Market Stabilization Indicators
This visual serves as a historical volatility check, which is necessary to defend the reliability of
the forecasting algorithms.
Volatility vs. Stability: The massive, jagged spikes in the early 2000s indicate a highly
volatile, immature market where one or two niche vehicles skewed the entire average. The
smoother, more consistent flow in recent years indicates market stabilization.
Why it matters: Forecasting algorithms require a stable baseline to be accurate. By
showing this chart, you prove to the audience that the "wild west" era of EV data is over,
and the modern data feeding the predictive models in Charts 1 and 2 is robust and reliable.
5. The Control Panel: Slicers (Left Sidebar)
Role: Scenario Testing
Without these slicers, the dashboard is merely a static report. The slicers transform the page
into an interactive forecasting engine.
Micro-Forecasting: By interacting with the "Make" and "Electric Vehicle Type" tiles, the
team can dynamically recalculate the statistical models in real-time.
Application: You can isolate a specific manufacturer (e.g., Tesla) to project their specific
future market share and technological ceiling, comparing it against a legacy automaker
(e.g., Ford) pivoting to electric. This allows the team to hypothesize and test specific market
scenarios on the fly.

