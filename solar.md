#  Solar Performance & Weather Analytics Pipeline

##  Project Overview
This project investigates the impact of environmental factors (temperature, cloud cover, and radiation) on solar panel efficiency. By merging telemetry data from solar inverters with atmospheric weather data, we identify energy loss patterns caused by thermal stress.

## Stakeholders & Business Value

* **Operations Managers:**  monthly production target monitoring
* **Maintenance Teams:** Identifying heat-induced performance degradation.
* **Investors:** Quantifying the ROI of potential hardware cooling upgrades.

---

## The Dataset
The analysis is built upon two primary data sources stored in the `solar_db` database:

* **`telemetry` table:** Containes daily total energy yield in watts(Kaggle)
* **`weather` table:** From Open-Meteo Historical Weather API in JSON format

---
## Architecture Components
* **AWS Lambda** - Geathers data from a weather API and delivers raw json file to S3
* **S3** - Data lake for raw and processed data
* **AWS Glue Crawler** – Automatically scans the data in S3 to infer the schema and populate the Data Catalog.
* **Amazon Athena** – A serverless query service used to execute complex SQL transformations
---
## Methodology

* S3 Bucket Structure
* **raw/**
    * **telemetry/**
    * **weather/**
* **results/**
<img width="1059" height="198" alt="Screenshot 2026-01-13 at 11 04 59" src="https://github.com/user-attachments/assets/0ccc3b35-21f3-47c7-8671-82a540bb23e3" />

<img width="1601" height="213" alt="Screenshot 2026-01-13 at 11 05 28" src="https://github.com/user-attachments/assets/780ad8a6-6a48-4ce9-a520-8fa01cd8dddc" />

<img width="1599" height="224" alt="Screenshot 2026-01-13 at 11 06 11" src="https://github.com/user-attachments/assets/ecd86c48-bb9f-49ba-b335-e1ce2030d4bc" />
Everything is first stored in the S3 bucket, one file by uploading and one by an API call that is done by a Lambda function
<img width="587" height="531" alt="Screenshot 2026-01-13 at 11 24 19" src="https://github.com/user-attachments/assets/018c1334-1090-4f66-bcfe-13e4fedc2a21" />

In the weather api's specifications, I requested the 2023 data to match my telemetry data and specified the coordinates to match the exact location of the solar installation, ensuring the environmental context was as accurate as possible for the analysis

Once the data had settled in the S3 bucket, I utilized AWS Glue Crawlers to automatically scan the raw/ folders. The crawler identified the CSV structure of the telemetry data and the weather data, populating the AWS Glue Data Catalog. This step transformed flat files into queryable tables, allowing me to treat my S3 bucket as a high-performance relational database.
<img width="1740" height="610" alt="Screenshot 2026-01-13 at 11 41 28" src="https://github.com/user-attachments/assets/ec256f07-b0fb-45d6-bb78-17aaef4895a3" />

Then I used Amazon Athena to run SQL queries and get business insights. This was harder than anticipated since i recived the dat from the weather api in a nested json format.

My first step was to correlate the daily energy production with the environmental conditions at the precise moment of peak sun exposure. This query performs a relational join between the telemetry logs and the weather data, utilizing the UNNEST function to extract hourly snapshots from the raw weather arrays.
<img width="1638" height="963" alt="Screenshot 2026-01-13 at 11 44 04" src="https://github.com/user-attachments/assets/0caf8735-cc64-4700-be0a-6152e17c772b" />

---
## KPIs
*KPI1: Daily conversion efficiency
*This KPI tracks the daily conversion rate of sunlight into electricity, providing a granular view of how effectively the panels are performing under varying environmental conditions. Normalizing output against total daily radiation allows us to pinpoint exactly how much energy is being captured relative to the total sunlight available each day.

<img width="1791" height="992" alt="Screenshot 2026-01-13 at 12 19 03" src="https://github.com/user-attachments/assets/7e7437d1-b2fc-4cb2-b458-a78a85fe92bd" />
 
*KPI2: Weather-Based Performance Benchmarking
*This KPI categorizes performance based on environmental conditions, providing a direct comparison of how different weather types affect both raw output and conversion efficiency.
<img width="1942" height="881" alt="Screenshot 2026-01-13 at 12 20 59" src="https://github.com/user-attachments/assets/64120c0d-1f01-42d1-aadc-0c2c811999db" />


* KPI3: monthly performance health
* The Health Score is calculated by taking the Actual Yield and dividing it by the Expected Yield (Total Radiation × 0.137 baseline). The 0.137 (13.7%) coefficient represents the system's "gold standard" efficiency observed during cool, overcast days; by comparing all months to this benchmark, we can mathematically prove how much performance is being lost to heat stress during the clearer, hotter months.


<img width="1768" height="1169" alt="Screenshot 2026-01-13 at 12 11 53" src="https://github.com/user-attachments/assets/a60ca1ad-f6e8-4c8c-b204-72351ed7c67a" />

* KPI4:Peak Heat Impact
* This KPI provides a high-resolution look at the relationship between peak sunlight hours and total daily output. By isolating the atmospheric conditions at "Solar Noon" (12:00 PM)
<img width="1810" height="1047" alt="Screenshot 2026-01-13 at 12 25 06" src="https://github.com/user-attachments/assets/dd4107e7-babf-48aa-a9bc-e7a3886dd2e1" />

## Overview
* Project Overview
This project focuses on identifying and quantifying performance bottlenecks in a solar power installation located in Kerala, India. In tropical climates, solar production faces a unique challenge: while sunlight is abundant, the extreme heat often causes photovoltaic cells to lose efficiency, leading to a phenomenon known as the "Efficiency Paradox."

