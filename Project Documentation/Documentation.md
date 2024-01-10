# 🧬 Adventure Works Project Phase-wise Implementation
--Brief Intro--

---

## Table of Contents
- [Phase 1: Connecting & Shaping Data using PBI Power Query Editor](#phase-1:-connecting-&-shaping-data-using-pbi-power-query-editor)


---

## Phase 1: Connecting & Shaping Data using PBI Power Query Editor

**Step 1:** Query Territory Lookup table to the Power Query Editor (PQE). Rename table. Check all data types.

**Step 2:** Query Product Lookup table to the PQE. Rename table. Check all data types.
- Change ProductCost & ProductPrice data type to Currency (i.e. Fixed decimal number).
- Add new column ‘SKU Type’ based on all characters before 2nd ‘-‘ character in Product SKU column.
- Replace all '0' in Product Style column with ‘NA’.

**Step 3:** Query Product Category Lookup table to the PQE. Rename table. Check all data types.

**Step 4:** Query Product Subcategory Lookup table to the PQE. Rename table. Check all data types.

**Step 5:** Utilize Data QA & Column Profiling tools to improve quality of data:
- Update Column Profiling to be based on entire dataset.
- Remove Error & Empty rows from Primary Key columns like CustomerKey from the Customer Lookup table.

**Step 6:** Implement Text-specific table transformations:
- Proper Case Prefix & Name columns in Customer Lookup table.
- Add new FullName column using Merge Columns tool.

**Step 7:** Implement Number-specific table transformations:
- Inspect Max, Min, Avg Product prices in Product Lookup table.
- Add DiscountPrice column by multiplying ProductPrice column by 0.9 (10% Discount) using Standard tool in Number-specific transformation tab.

**Step 8:** Implement Date-specific table transformations:
- Add new columns – Dayname, MonthName, Year, StartofWeek, StartofMonth, StartofQuarter & StartofYear.
- Modify StartofWeek step custom column formula to set Monday as first day of week instead of default Sunday using `Date.StartOfWeek([Date],Day.Monday)`.

**Step 9:** Add Conditional Columns based on OrderQuantity column:
- If the column value is 1 then add “Single Item”, if column value is greater than 1 then add “Multiple Items” else add “Other” in the Sales Data tables.

**Step 10:** Append all the Sales data together using Append from Folder functionality:
- Move all 3 Sales Data tables for years 2020 to 2022 to a folder.
- Implement an Append Query to combine all the tables using the folder as a data source into a combined Sales Data 2020-2022 table.
- Transform data by Combine Files option from the Content Column Settings.
- Remove the Source.Name column that mentions the file source as it’s not required.

**Step 11:** For all Lookup tables disable the ‘Include in Report Refresh’ option from the PQE Query list since they contain static data. This will leave only the Sales Data to be refreshed. This will optimize query refresh times.

---

## Phase 2: Creating the Data Model using the PBI Model View

**Step 1:** Identify Primary Keys & Foreign Keys for all the tables to form relationships. In the Model view, in Table properties set the Key Column value as the column that is to act as the Primary Key for the respective Lookup Table.

**Step 2:** Setup Table Relationships by dragging Primary Keys from Lookup tables to Foreign Keys in other tables to setup the Filter Flow. This can also be done using the Manage Relationships setting.

|Primary Key|Foreign Key|
|-|-|
|Territory Lookup (SalesTerritoryKey)|Sales Data (TerritoryKey)|
|Customer Lookup (CustomerKey)|Sales Data (CustomerKey)|
|Calendar Lookup (Date)|Sales Data (OrderDate)|
|Calendar Lookup (Date)|Sales Data (StockDate) (Inactive Relationship)|
|Product Lookup (ProductKey)|Sales Data (ProductKey)|
|Product Category Lookup (ProductCategoryKey)|Product Subcategory Lookup (ProductCategoryKey)|
|Product Subcategory Lookup (ProductSubcategoryKey|Product Lookup (ProductSubcategoryKey)|
|Product Lookup (ProductKey)|Returns Data (ProductKey)|
|Calendar Lookup (Date)|Returns Data (ReturnDate)|
|Territory Lookup (SalesTerritoryKey)|Returns Data (TerritoryKey)|

**Step 3:** Setup Model Schemas for the tables:
- Calender Lookup, Territory Lookup, Customer Lookup & Product Lookup -> Sales Data will form Star Schema.
- Product Category Lookup -> Product Subcategory Lookup -> Product Lookup will form Snowflake Schema.

**Step 4:** Hide all the FK from Fact tables to force filter context to be based on PK in lookup tables:
- Sales Data -> CustomerKey, OrderDate, Stockdate, ProductKey, TerritoryKey
- Returns Data -> ProductKey, TerritoryKey, ReturnDate
- Product Lookup -> ProductSubcategoryKey
- Product Subcategories Lookup -> ProductCategoryKey

**Step 5:** Setup any Data Format changes required Column tools tab in Data view:
- Set the Data format for dates to be the Short date in date related columns in Calender, Sales & Return tables for easier interpretation during visualization.
- Set Data format for price related columns as Currency and decimal places as 2 in the Product Lookup table.

**Step 6:** Setup any Data Category changes required Column tools tab in Data view:
- Set the Data category as Country and Continent for the Country and Continent columns respectively in Territory Lookup table.

**Step 7:** Implement Hierarchies:
- Territory hierarchy: Continent -> Country -> Region
- Date hierarchy: StartofYear -> StartofMonth -> StartofWeek -> Date

---

## Phase 3: Creating Calculated Fields with DAX

**Step 1:** Create Calculated Columns:
1. ‘QuantityType’ based on Order Quantity column in Sales Data. If the Qty > 1 then insert “Multiple Items” else insert “Single Item”.

   DAX: `Quantity Type = IF('AW Sales Data'[OrderQuantity] > 1, "Multiple Items", "Single Item")`

2. ‘Parent’ based on TotalChildren column in Customer Lookup. If the value > 0 then insert “Yes” else insert “No”.

   DAX: `Parent = IF('AW Customer Lookup'[TotalChildren] > 0, "Yes", "No")`

3. ‘PricePoint’ based on ProductPrice column in Product Lookup. This will be a SWITCH function based implementation using > operator based on TRUE logic.

   DAX: `PricePoint = SWITCH( TRUE( ), 'AW Product Lookup'[ProductPrice] > 500, "High", 'AW Product Lookup'[ProductPrice] > 100, "Mid", "Low" )`

4. ‘CustomerPriority’ based on Parent & AnnualIncome columns in Customer Lookup. If Parent and Annual Income is more than 100000 then insert “Priority” else insert             “Standard”.

   DAX: `CustomerPriority = IF('AW Customer Lookup'[Parent] = "Yes" && 'AW Customer Lookup'[AnnualIncome] > 100000, "Priority", "Standard")`

5. ‘IncomeLevel’ based on AnnualIncome column in Customer Lookup. This will be a SWITCH function based implementation using >= operator based on TRUE logic.

   DAX: `IncomeLevel = SWITCH(TRUE(), 'AW Customer Lookup'[AnnualIncome] >= 150000, "Very High", 'AW Customer Lookup'[AnnualIncome] >= 100000, "High", 'AW Customer             Lookup'[AnnualIncome] >= 50000, "Average", "Low")`

6. ‘EducationCategory’ based on EducationLevel column in Customer Lookup. This will be a SWITCH function based implementation.

   DAX: `EducationCategory = SWITCH('AW Customer Lookup'[EducationLevel], "High School", "High School", "Partial High School", "High School", "Bachelors", "Undergrad",         "Partial College", "Undergrad", "Graduate Degree", "Graduate")`

7. ‘MonthShort’ based on MonthName column in Calendar Lookup. This will be 3 character capitalized month names.

   DAX: `MonthShort = UPPER(LEFT('AW Calendar Lookup'[MonthName], 3))`

8. ‘SKUCategory’ based on ProductSKU column in Product Lookup. This will be any characters before the first hyphen in ProductSKU column.

   DAX: `SKUCategory = LEFT('AW Product Lookup'[ProductSKU], SEARCH("-", 'AW Product Lookup'[ProductSKU]) - 1)`

9. ‘DayofWeek’ based on Date column in Calendar Lookup. This configured to ReturnType set as 2 so Monday: 1 to Sunday: 7.

   DAX: `DayofWeek = WEEKDAY('AW Calendar Lookup'[Date], 2)`

10. ‘Weekend’ based on DayofWeek column in Calendar Lookup. If DayofWeek is 6 or 7 then insert “Weekend” else “Weekday”.

    DAX: `Weekend = IF('AW Calendar Lookup'[DayofWeek] IN {6, 7}, "Weekend", "Weekday")`

11. ‘BirthYear’ based on BirthDate column in Customer Lookup. Extract only the year data.

    DAX: `BirthYear = YEAR('AW Customer Lookup'[BirthDate])`

12. ‘RetailPrice’ based on ProductPrice column in Product Lookup. Use RELATED function to get the column.

    DAX: `RetailPrice = RELATED('AW Product Lookup'[ProductPrice])`

13. ‘Revenue’ based on RetailPrice & OrderQuantity column in Sales Data.

    DAX: `Revenue = 'AW Sales Data'[RetailPrice] * 'AW Sales Data'[OrderQuantity]`

14.

**Step 2:** Configure a Measure Table to organize and store all the measures using the Enter Data functionality in the Data View. Move all the created Measures to this table.









