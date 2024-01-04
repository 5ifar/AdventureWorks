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

**Step 5:** Utilize Data QA & Column Profiling tools to improve quality of dat:
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




