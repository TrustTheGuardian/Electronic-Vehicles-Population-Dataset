# Electronic-Vehicles-Population-Dataset

DATA ANALYTICS
BSCS – C301
Group 1: 
1. Gomez, Jason
2. Lopez, Justine
3. Manansala, Joshua
4. Tongol, Katriel Cire
5. Turla, Lance Amiel

DOCUMENTATION
Dataset: https://www.kaggle.com/datasets/nilesh2042/electric-vehicle-population-data

CONCEPTUALIZE THE DATA
Each row in the CSV dataset represents an electric vehicle (EV) registered in a state within the United States of America (USA). Each record contains a unique Department of Licensing (DOL) ID, geographic information such as postal code, city, county, and state, as well as vehicle identification details including VIN, model year, manufacturer or brand, vehicle model, electric vehicle type, driving range, and eligibility as an alternative fuel vehicle. A vehicle with a DOL ID indicates that it has been officially purchased, registered, and approved for road use.
LOCATE SOLVABLE ISSUES
Issues found: 
1.	Some postal codes were inconsistently formatted, where identical postal codes appeared both with and without leading zeros.
2.	The electric driving range column contained values of 0, which may indicate missing or invalid data.
3.	The dataset contained redundant and repeated information across multiple rows.
Resolutions:
1.	Postal code values were standardized by converting them into consistent numerical representations to simplify processing and comparison.
2.	Invalid driving range values of 0 were replaced with NULL to properly represent missing or unavailable data.
3.	The dataset was normalized into fact tables and sub-dimensions to eliminate redundancy, improve consistency, and ensure that each attribute is stored in only one location.
EVALUATE UNSOLVABLE ISSUES
Issues found: 
1.	Missing values.
Resolutions:
1.	Rather than deleting, these were kept in the sub-dimensions to maintain accurate “total count” KPIs but filtered out of map visuals to prevent errors.
AUGMENT THE DATA
•	During dataset normalization, unique identifiers were assigned to each sub-dimension to properly handle complex relationships and avoid ambiguity. This includes cases where a single postal code belongs to multiple cities, a city spans multiple counties, counties share the same name across different states, or multiple counties exist within a single state. 
•	Lookup operations were performed to retrieve and integrate related data from other normalized tables, improving data consistency, relationship mapping, and overall referential integrity.
NOTE AND DOCUMENT
The dataset was cleaned and normalized using the CLEAN framework and structured following a Snowflake schema design. This approach was used to efficiently manage complex relationships among geographic attributes such as postal codes, cities, counties, and states, as well as vehicle-related attributes including VIN numbers, vehicle models, manufacturers, and other related details. The normalization process reduced data redundancy, improved consistency, and strengthened referential integrity across the database.
