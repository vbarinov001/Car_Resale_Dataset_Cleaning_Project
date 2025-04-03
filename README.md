# Car_Resale_Dataset_Cleaning_Project

## Table of Contents
- [Data Wrangling Step-By-Step](#data-wrangling)
- [To Complete Cleaning Query](#complete-cleaned-query)
- [Exploratory Analysis](#exploratory-data-analysis)

## 📚 About Data

This dataset contains listings of 1000 used cars for sale in India during 2023, capturing key details such as make, model, year, price, mileage, fuel type, transmission, and location. The goal of this project was to practice my data cleaning skills using SQL, 
normalizing the data and converting it to a US-based format. The dataset was obtained from Kaggle, created by the user, Rahul Menon 1758. (Link: https://www.kaggle.com/datasets/rahulmenon1758/car-resale-prices)

## ✏️ Data Wrangling

- Data Cleaning: Removed unwanted characters (e.g., currency symbols, commas), replaced missing values with "Not Available," and reformatted inconsistent text entries (e.g., insurance types, ownership types).
- Unit Conversions: Converted resale prices from lakh INR to USD, engine capacity from cc to liters, mileage from KMPL to MPG, max power from BHP to HP, and distance driven from kilometers to miles.
- Column Renaming & Formatting:  Renamed columns for clarity (e.g., city to car_location_city, kms_driven to miles_driven), ensured proper spacing in text outputs, and optimized numerical formatting by rounding values to two decimals.

## Step by Step Process

### Cutting up full_name column
```sql
SELECT
    SUBSTR(full_name, 1, 4) AS Year,
    TRIM(SUBSTR(full_name, 6, INSTR(SUBSTR(full_name, 6), ' ') - 1)) AS Make,
    TRIM(SUBSTR(full_name, 
        INSTR(full_name, ' ') + INSTR(SUBSTR(full_name, INSTR(full_name, ' ') + 1), ' ') + 1,
        LENGTH(full_name) - INSTR(full_name, ' ') - INSTR(SUBSTR(full_name, INSTR(full_name, ' ') + 1), ' ') - INSTR(SUBSTR(full_name, INSTR(full_name, ' ') + 1), ' ') 
    )) AS Model,
    TRIM(SUBSTR(full_name, 
        LENGTH(full_name) - 
        INSTR(SUBSTR(full_name, LENGTH(full_name) - 10), ' ') 
        + 1)) AS Trim
FROM car_sales_india;
```

### Resale price adjustments
```sql
SELECT
    -- Converting the Lakh price to USD and round to 2 decimal places (Jan 1 2023 unit conversion rate)
    ROUND(CAST(REPLACE(SUBSTR(resale_price, 
    INSTR(resale_price, ' ') + 1), ' Lakh', '') 
    AS DECIMAL(10,2)) * 100000 * 120.85, 2) 
    AS resale_price_USD
FROM
    car_sales_india;
```

### Engine capacity Unit Conversions
```sql
SELECT
    
    ROUND(CAST(REPLACE(engine_capacity, ' cc', '') AS FLOAT) 
    / 1000, 2) AS Engine_Capacity_Liters
FROM
    car_sales_india;
```

### Insurance adjustments
```sql
SELECT
    CASE
        WHEN insurance LIKE '%Third Party%' THEN 'Liability Insurance'
        WHEN insurance LIKE '%Comprehensive%' THEN 'Full Coverage'
        WHEN insurance LIKE '%Zero Dep%' THEN 'Gap Insurance / Extended Coverage'
        WHEN insurance IS NULL OR TRIM(insurance) = '' THEN 'Not Available'  
        ELSE insurance 
    END AS Insurance_Type
FROM
    car_sales_india;
```

### Mileage conversions
```sql
SELECT
    ROUND(CAST(REPLACE(kms_driven, ',', '') 
    AS DECIMAL(12,2)) * 0.621371, 2) 
    AS Mileage
FROM
    car_sales_india;
```

### Owner Type Adjustments
```sql
SELECT
    CASE
        WHEN owner_type = 'First Owner' THEN 1
        WHEN owner_type = 'Second Owner' THEN 2
        WHEN owner_type = 'Third Owner' THEN 3
        WHEN owner_type = 'Fourth Owner' THEN 4
        WHEN owner_type = 'Fifth Owner' THEN 5
        WHEN owner_type = 'Sixth Owner' THEN 6
        WHEN owner_type = 'Seventh Owner' THEN 7
        WHEN owner_type = 'Eighth Owner' THEN 8
        WHEN owner_type = 'Ninth Owner' THEN 9
        WHEN owner_type = 'Tenth Owner' THEN 10
        ELSE 'More than 10'
    END AS Number_of_Owners
FROM
    car_sales_india;
```

### Fuel Type Adjustments
```sql

--fuel_type
SELECT
    CASE
        WHEN fuel_type = 'Petrol' THEN 'Gasoline'
        WHEN fuel_type = 'Diesel' THEN 'Diesel'
        WHEN fuel_type = 'CNG' THEN 'CNG'  
        ELSE 'Other' 
    END AS fuel_type
FROM
    car_sales_india;
```

### Max Power Adjustments
```sql
SELECT
    REPLACE(max_power, 'bhp', ' hp') AS Power
FROM
    car_sales_india;
```

### MPG conversions
```sql
SELECT
    ROUND(CAST(REPLACE(mileage, ' kmpl', '') 
    AS DECIMAL(12,2)) * 2.352, 2) 
    AS MPG
FROM
    car_sales_india;
```

### Table Column Name Alterations
```sql
ALTER TABLE car_sales_india RENAME COLUMN transmission_type TO Transmission;
ALTER TABLE car_sales_india RENAME COLUMN city TO Car_Location_City;
ALTER TABLE car_sales_india RENAME COLUMN registered_year TO Registered_Year;
ALTER TABLE car_sales_india RENAME COLUMN seats TO Seats;
ALTER TABLE car_sales_india RENAME COLUMN body_type TO Body_Type;
```

## Complete Cleaning Query
```sql
ALTER TABLE car_sales_india RENAME COLUMN transmission_type TO Transmission;
ALTER TABLE car_sales_india RENAME COLUMN city TO Car_Location_City;
ALTER TABLE car_sales_india RENAME COLUMN registered_year TO Registered_Year;
ALTER TABLE car_sales_india RENAME COLUMN seats TO Seats;
ALTER TABLE car_sales_india RENAME COLUMN body_type TO Body_Type;


SELECT
    Transmission, 
    Car_Location_City,
    Registered_Year,
    Seats,
    Body_Type,
    SUBSTR(full_name, 1, 4) AS Year,
    TRIM(SUBSTR(full_name, 6, INSTR(SUBSTR(full_name, 6), ' ') - 1)) AS Make,
    TRIM(SUBSTR(full_name, 
        INSTR(full_name, ' ') + INSTR(SUBSTR(full_name, INSTR(full_name, ' ') + 1), ' ') + 1,
        LENGTH(full_name) - INSTR(full_name, ' ') - INSTR(SUBSTR(full_name, INSTR(full_name, ' ') + 1), ' ') - INSTR(SUBSTR(full_name, INSTR(full_name, ' ') + 1), ' ') 
    )) AS Model,
    TRIM(SUBSTR(full_name, 
        LENGTH(full_name) - 
        INSTR(SUBSTR(full_name, LENGTH(full_name) - 10), ' ') 
        + 1)) AS Trim,
    ROUND(
    CAST(REPLACE(SUBSTR(resale_price, INSTR(resale_price, ' ') + 1), ' Lakh', '') AS DECIMAL(10,2)) 
    * 100000 
    * 0.01209, 2) AS resale_price_USD,
    ROUND(CAST(REPLACE(engine_capacity, ' cc', '') AS FLOAT) 
    / 1000, 2) AS Engine_Capacity_Liters,
    CASE
        WHEN insurance LIKE '%Third Party%' THEN 'Liability Insurance'
        WHEN insurance LIKE '%Comprehensive%' THEN 'Full Coverage'
        WHEN insurance LIKE '%Zero Dep%' THEN 'Gap Insurance / Extended Coverage'
        WHEN insurance IS NULL OR TRIM(insurance) = '' THEN 'Not Available'  
        ELSE insurance 
    END AS Insurance_Type,
    ROUND(CAST(REPLACE(kms_driven, ',', '') 
    AS DECIMAL(12,2)) * 0.621371, 2) 
    AS Mileage,
    CASE
        WHEN owner_type = 'First Owner' THEN 1
        WHEN owner_type = 'Second Owner' THEN 2
        WHEN owner_type = 'Third Owner' THEN 3
        WHEN owner_type = 'Fourth Owner' THEN 4
        WHEN owner_type = 'Fifth Owner' THEN 5
        WHEN owner_type = 'Sixth Owner' THEN 6
        WHEN owner_type = 'Seventh Owner' THEN 7
        WHEN owner_type = 'Eighth Owner' THEN 8
        WHEN owner_type = 'Ninth Owner' THEN 9
        WHEN owner_type = 'Tenth Owner' THEN 10
        ELSE 'More than 10'
    END AS Number_of_Owners,
    CASE
        WHEN fuel_type = 'Petrol' THEN 'Gasoline'
        WHEN fuel_type = 'Diesel' THEN 'Diesel'
        WHEN fuel_type = 'CNG' THEN 'CNG'  
        ELSE 'Other' 
    END AS fuel_type,
    REPLACE(max_power, 'bhp', ' hp') AS Power,
    ROUND(CAST(REPLACE(mileage, ' kmpl', '') 
    AS DECIMAL(12,2)) * 2.352, 2) 
    AS MPG
FROM
    car_sales_india;
```
![image](https://github.com/user-attachments/assets/bde769b0-4c7f-48ef-b846-58e2149a43a5)


## Exploratory Data Analysis

###YOU MUST USE the With Statement before all EDA QUERIES
```sql
ALTER TABLE car_sales_india RENAME COLUMN transmission_type TO Transmission;
    ALTER TABLE car_sales_india RENAME COLUMN city TO Car_Location_City;
    ALTER TABLE car_sales_india RENAME COLUMN registered_year TO Registered_Year;
    ALTER TABLE car_sales_india RENAME COLUMN seats TO Seats;
    ALTER TABLE car_sales_india RENAME COLUMN body_type TO Body_Type;

WITH cleaned_data as (
    SELECT
        Transmission, 
        Car_Location_City,
        Registered_Year,
        Seats,
        Body_Type,
        SUBSTR(full_name, 1, 4) AS Year,
        TRIM(SUBSTR(full_name, 6, INSTR(SUBSTR(full_name, 6), ' ') - 1)) AS Make,
        TRIM(SUBSTR(full_name, 
            INSTR(full_name, ' ') + INSTR(SUBSTR(full_name, INSTR(full_name, ' ') + 1), ' ') + 1,
            LENGTH(full_name) - INSTR(full_name, ' ') - INSTR(SUBSTR(full_name, INSTR(full_name, ' ') + 1), ' ') -   INSTR(SUBSTR(full_name, INSTR(full_name, ' ') + 1), ' ') 
        )) AS Model,
        TRIM(SUBSTR(full_name, 
            LENGTH(full_name) - 
            INSTR(SUBSTR(full_name, LENGTH(full_name) - 10), ' ') 
            + 1)) AS Trim,
        ROUND(
        CAST(REPLACE(SUBSTR(resale_price, INSTR(resale_price, ' ') + 1), ' Lakh', '') AS DECIMAL(10,2)) 
        * 100000 
        * 0.01209, 2) AS resale_price_USD,
        ROUND(CAST(REPLACE(engine_capacity, ' cc', '') AS FLOAT) 
        / 1000, 2) AS Engine_Capacity_Liters,
        CASE
            WHEN insurance LIKE '%Third Party%' THEN 'Liability Insurance'
            WHEN insurance LIKE '%Comprehensive%' THEN 'Full Coverage'
            WHEN insurance LIKE '%Zero Dep%' THEN 'Gap Insurance / Extended Coverage'
            WHEN insurance IS NULL OR TRIM(insurance) = '' THEN 'Not Available'  
            ELSE insurance 
        END AS Insurance_Type,
        ROUND(CAST(REPLACE(kms_driven, ',', '') 
        AS DECIMAL(12,2)) * 0.621371, 2) 
        AS Mileage,
        CASE
            WHEN owner_type = 'First Owner' THEN 1
            WHEN owner_type = 'Second Owner' THEN 2
            WHEN owner_type = 'Third Owner' THEN 3
            WHEN owner_type = 'Fourth Owner' THEN 4
            WHEN owner_type = 'Fifth Owner' THEN 5
            WHEN owner_type = 'Sixth Owner' THEN 6
            WHEN owner_type = 'Seventh Owner' THEN 7
            WHEN owner_type = 'Eighth Owner' THEN 8
            WHEN owner_type = 'Ninth Owner' THEN 9
            WHEN owner_type = 'Tenth Owner' THEN 10
            ELSE 'More than 10'
        END AS Number_of_Owners,
        CASE
            WHEN fuel_type = 'Petrol' THEN 'Gasoline'
            WHEN fuel_type = 'Diesel' THEN 'Diesel'
            WHEN fuel_type = 'CNG' THEN 'CNG'  
            ELSE 'Other' 
        END AS fuel_type,
        REPLACE(max_power, 'bhp', ' hp') AS Power,
        ROUND(CAST(REPLACE(mileage, ' kmpl', '') 
        AS DECIMAL(12,2)) * 2.352, 2) 
        AS MPG
    FROM
        car_sales_india
)
```


-- 1. Total cars analyzed
```sql
SELECT COUNT(*) AS Total_Cars FROM cleaned_data;
```
![image](https://github.com/user-attachments/assets/f109cf77-45e8-4517-99b3-a522875f3160)

-- 2. Most common car brands
```sql
SELECT Make, COUNT(*) AS Count FROM cleaned_data 
GROUP BY Make ORDER BY Count DESC LIMIT 10;
```
![image](https://github.com/user-attachments/assets/6cbf5f40-347f-4bfb-8362-94f939dc31e7)

-- 3. Market segmentation (Body type distribution)
```sql
SELECT Body_Type, COUNT(*) AS Count FROM cleaned_data 
GROUP BY Body_Type ORDER BY Count DESC;
```
![image](https://github.com/user-attachments/assets/25b22930-b28d-4749-b6f5-9677d8a2334c)

-- 4. Fuel type segmentation
```sql
SELECT fuel_type, COUNT(*) AS Count FROM cleaned_data 
GROUP BY fuel_type ORDER BY Count DESC;
```
![image](https://github.com/user-attachments/assets/99aed5a3-022f-4e04-85f2-5937012be95f)

-- 5. Engine power segmentation
```sql
SELECT Power, COUNT(*) AS Count FROM cleaned_data 
GROUP BY Power ORDER BY Count DESC;
```
![image](https://github.com/user-attachments/assets/90b48abe-c651-4c8e-af65-049c9a92e24c)

-- 6. Average mileage
```sql
SELECT ROUND(AVG(Mileage), 2) AS Avg_Mileage FROM cleaned_data;
```
![image](https://github.com/user-attachments/assets/008cdaa4-49ac-4b63-94be-afea2feed5a9)

-- 7. Average MPG
```sql
SELECT ROUND(AVG(MPG), 2) AS Avg_MPG FROM cleaned_data;
```
![image](https://github.com/user-attachments/assets/ab0b1109-7ec3-4ab0-8e8a-406de353ac50)

-- 8. Top 5 most expensive cars
```sql
SELECT Make, Model, resale_price_USD FROM cleaned_data 
ORDER BY resale_price_USD DESC LIMIT 5;
```
![image](https://github.com/user-attachments/assets/8f4e4c4b-f759-4d71-87ba-65910cef18b5)

--9 Most Common Brands
```sql
SELECT Make, 
COUNT(*) AS count
FROM cleaned_data
GROUP BY Make
ORDER BY count DESC
LIMIT 10;
```
![image](https://github.com/user-attachments/assets/aa4c8f63-3dd5-4b90-9eeb-f49d3bfbe8de)

--10. Car Type Distribution
```sql
SELECT Body_Type, COUNT(*) AS total
FROM cleaned_data
GROUP BY Body_type
ORDER BY total DESC;
```
![image](https://github.com/user-attachments/assets/fa90ee9f-ef5e-4a41-8bb3-3713c55a39c6)

--11. Average Price
```sql
SELECT 
    AVG(resale_price_USD) AS avg_price
FROM cleaned_data;
```
![image](https://github.com/user-attachments/assets/9908b6b7-44aa-427a-81f8-e2b9c8ef5e46)

--12. Engine Power Segmentation
```sql
SELECT 
    CASE 
        WHEN Power < 100 THEN 'Low Power (<100 hp)'
        WHEN Power BETWEEN 100 AND 200 THEN 'Medium Power (100-200 hp)'
        ELSE 'High Power (>200 hp)'
    END AS power_category,
    COUNT(*) AS total
FROM cleaned_data
GROUP BY power_category
ORDER BY total DESC;
```

--13. Owner Number Distribution
```sql
SELECT Number_of_Owners, COUNT(*) AS total
FROM cleaned_data
GROUP BY Number_of_Owners
ORDER BY total DESC;
```
![image](https://github.com/user-attachments/assets/63b4a3f5-d937-4a45-ab8e-6011d8d94fcf)

--14. Impact of Ownership on Resale Value
```sql
SELECT Number_of_Owners, AVG(resale_price_USD) AS avg_resale_price
FROM cleaned_data
GROUP BY Number_of_Owners
ORDER BY Number_of_Owners;
```
![image](https://github.com/user-attachments/assets/90a00ac5-509a-4881-8968-e179ab5f0433)

--15. City Wise Car Popularity
```sql
SELECT Car_Location_City, COUNT(*) AS total
FROM cleaned_data
GROUP BY Car_Location_City
ORDER BY total DESC;
```
![image](https://github.com/user-attachments/assets/8073a1bc-d712-4b2b-91b4-40eceb1cb88d)
