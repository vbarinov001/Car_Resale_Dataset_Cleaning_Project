# Car_Resale_Dataset_Cleaning_Project

- [To Complete Query](#complete-query)

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

## Complete Query
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
    ROUND(CAST(REPLACE(SUBSTR(resale_price, 
    INSTR(resale_price, ' ') + 1), ' Lakh', '') 
    AS DECIMAL(10,2)) * 100000 * 120.85, 2) 
    AS resale_price_USD,
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
![image](https://github.com/user-attachments/assets/18fa0ac6-3858-428b-b41a-e15a22401a6f)


