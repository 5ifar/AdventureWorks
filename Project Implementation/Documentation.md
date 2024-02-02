# 🧬 Adventure Works Project Phase-wise Implementation
--Brief Intro--

---

## Table of Contents
- [Phase 1: Connecting & Shaping Data using PBI Power Query Editor](#Phase-1-Connecting--Shaping-Data-using-PBI-Power-Query-Editor)
- [Phase 2: Creating the Data Model using PBI Model View](#Phase-2-Creating-the-Data-Model-using-PBI-Model-View)
- [Phase 3: Creating Calculated Fields with DAX](#Phase-3-Creating-Calculated-Fields-with-DAX)

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

## Phase 2: Creating the Data Model using PBI Model View

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

   DAX: `IncomeLevel = SWITCH(TRUE(), 'AW Customer Lookup'[AnnualIncome] >= 150000, "Very High", 'AW Customer Lookup'[AnnualIncome] >= 100000, "High", 'AW Customer Lookup'[AnnualIncome] >= 50000, "Average", "Low")`

6. ‘EducationCategory’ based on EducationLevel column in Customer Lookup. This will be a SWITCH function based implementation.

   DAX: `EducationCategory = SWITCH('AW Customer Lookup'[EducationLevel], "High School", "High School", "Partial High School", "High School", "Bachelors", "Undergrad", "Partial College", "Undergrad", "Graduate Degree", "Graduate")`

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


**Step 2:** Configure a Measure Table to organize and store all the measures using the Enter Data functionality in the Data View. Move all the created Measures to this table.

**Step 3:** Create Explicit Measures in the Measures Table:
1. ‘Quantity Sold’: It is the sum of the Order Quantity column in Sales Data. Format as Whole number with commas.

   DAX: `Quantity Sold = SUM('AW Sales Data'[OrderQuantity])`

2. ‘Quantity Returned’: It is the sum of the Return Quantity column in Returns Data. Format as Whole number with commas.

   DAX: `Quantity Returned = SUM('AW Returns Data'[ReturnQuantity])`

3. ‘Average Retail Price’: It is the average of the Product Price column in Product Lookup. Format as Currency with commas.

   DAX: `Average Retail Price = AVERAGE('AW Product Lookup'[ProductPrice])`

4. ‘Total Returns’: It is the count of the Return Quantity column in Returns Data. This is different from Quantity Returned Measure since it shows the count of number of       times returns were made unlike Quantity Returned that shows the total number of products returned. Format as Whole number with commas.

   DAX: `Total Returns = COUNT('AW Returns Data'[ReturnQuantity])`

5. ‘Total Orders’: It is the distinct count of the Order Number column in Sales Data. This is done since we simply cannot use COUNT like in the case of Total Returns since     the Sales Data table contains multiple rows with same Order Number with different ProductKeys hence we’ll use DISTINCTCOUNT. Format as Whole number with commas.

   DAX: `Total Orders = DISTINCTCOUNT('AW Sales Data'[OrderNumber])`

6. ‘Total Customers’: It is the distinct count of the CustomerKey column in Sales Data. Format as Whole number with commas.

   DAX: `Total Customers = DISTINCTCOUNT('AW Sales Data'[CustomerKey])`

7. ‘Return Rate’: It is the division of the Quantity Returned Measure by the Quantity Sold Measure. Mark as “No Sales” if Quantity Sold is 0. Format as Percentage with 2       decimal places.

   DAX: `Return Rate = DIVIDE([Quantity Returned], [Quantity Sold], "No Sales")`

8. ‘Bulk Orders’: It is the value of Total Orders Measure when OrderQunatity is greater than 1. Implemented using the CALCULATE function. Format as Whole number with commas.

   DAX: `Bulk Orders = CALCULATE([Total Orders], 'AW Sales Data'[OrderQuantity] > 1)`

9. ‘Weekend Orders’: It is the value of Total Orders Measure when it’s the Weekend. Implemented using the CALCULATE function. Format as Whole number with commas.

   DAX: `Weekend Orders = CALCULATE([Total Orders], 'AW Calendar Lookup'[Weekend] = "Weekend")`

10. ‘Bike Sales’: It is the value of Quantity Sold Measure when the CategoryName is “Bike”. Format as Whole number with commas.

    DAX: `Bike Sales = CALCULATE([Quantity Sold], 'AW Product Categories Lookup'[CategoryName] = "Bikes")`

11. ‘Bike Returns’: It is the value of Quantity Returned Measure when the CategoryName is “Bike”. Format as Whole number with commas.

    DAX: `Bike Returns = CALCULATE([Quantity Returned], 'AW Product Categories Lookup'[CategoryName] = "Bikes")`

12. ‘Bike Return Rate’: It is the value of Return Rate Measure when the CategoryName is “Bike”. Format as Percentage with 2 decimal places.

    DAX: `Bike Return Rate = CALCULATE([Return Rate], 'AW Product Categories Lookup'[CategoryName] = "Bikes")`

13. ‘All Orders’: It is the value of Total Orders Measure when there is no filter context. Implemented using the ALL function inside CALCULATE function to remove any set        filter on the Sales Data table. Format as Whole number with commas.

    DAX: `All Orders = CALCULATE([Total Orders], ALL('AW Sales Data'))`

14. ‘% of All Orders’: It is the value of division of Total Orders Measure by All Orders Measure. Format as Percentage with 2 decimal places.

    DAX: `% of All Orders = DIVIDE([Total Orders], [All Orders])`

15. ‘Overall Average Price’: It is the value of Average Retail Price Measure when there is no filter context. Implemented using the ALL function inside CALCULATE function       to remove any set filter on the Product Lookup table. Format as Whole number with commas.

    DAX: `Overall Average Price = CALCULATE([Average Retail Price], ALL('AW Product Lookup'))`

16. ‘High Ticket Orders’: It is the value of Total Orders measure when ProductPrice is greater than Overall Average Price. Format as Whole number with commas.

    DAX: `High Ticket Orders = CALCULATE([Total Orders], FILTER('AW Product Lookup', 'AW Product Lookup'[ProductPrice] > [Overall Average Price]))`

17. ‘Total Revenue’: It is the sum of product iteration of OrderQuantity and ProductPrice. Format as Currency with commas and no decimal.

    DAX: `Total Revenue = SUMX('AW Sales Data', 'AW Sales Data'[OrderQuantity] * RELATED('AW Product Lookup'[ProductPrice]))`

18. ‘Average Revenue per Customer’: It is the product of Total Revenue Measure and Total Customers Measure. Format as Currency with commas and no decimal.

    DAX: `Average Revenue per Customer = DIVIDE([Total Revenue], [Total Customers])`

19. ‘Total Cost’: It is the sum of product iteration of OrderQuantity and ProductCost. Format as Currency with commas and no decimal.

    DAX: `Total Cost = SUMX('AW Sales Data', 'AW Sales Data'[OrderQuantity] * RELATED('AW Product Lookup'[ProductCost]))`

20. ‘Total Profit’: It is the difference between the Total Revenue measure and the Total Cost measure. Format as Currency with commas and no decimal.

    DAX: `Total Profit = [Total Revenue] - [Total Cost]`

21. ‘YTD Revenue’: It is the Year to Date value of the Total Revenue measure. Format as Currency with commas and no decimal.

    DAX: `YTD Revenue = CALCULATE([Total Revenue], DATESYTD('AW Calendar Lookup'[Date]))`

22. ‘Previous Month Revenue’: It is the previous month value of the Total Revenue measure with an interval of 1 month. Format as Currency with commas and no decimal.

    DAX: `Previous Month Revenue = CALCULATE([Total Revenue], DATEADD('AW Calendar Lookup'[Date], -1, MONTH))`

23. ‘Previous Month Returns’: It is the previous month value of the Total Returns measure with an interval of 1 month. Format as Whole number with commas.

    DAX: `Previous Month Returns = CALCULATE([Total Returns], DATEADD('AW Calendar Lookup'[Date], -1, MONTH))`

24. ‘Previous Month Orders’: It is the previous month value of the Total Orders measure with an interval of 1 month. Format as Whole number with commas.

    DAX: `Previous Month Orders = CALCULATE([Total Orders], DATEADD('AW Calendar Lookup'[Date], -1, MONTH))`

25. ‘Previous Month Profit’: It is the previous month value of the Total Profit measure with an interval of 1 month. Format as Currency with commas and no decimal.

    DAX: `Previous Month Profit = CALCULATE([Total Profit], DATEADD('AW Calendar Lookup'[Date], -1, MONTH))`

26. ‘Revenue Target’: It is a 10% more than the Previous Month Revenue measure. Format as Currency with commas and no decimal.

    DAX: `Revenue Target = [Previous Month Revenue] * 1.1`

27. ‘Profit Target’: It is a 10% more than the Previous Month Profit measure. Format as Currency with commas and no decimal.

    DAX: `Profit Target = [Previous Month Profit] * 1.1`

28. ‘Order Target’: It is a 10% more than the Previous Month Orders measure. Format as Whole number with commas.

    DAX: `Order Target = [Previous Month Orders] * 1.1`

29. ‘10 Day Rolling Revenue’: It is the value of the running total of 10 days of the Total Revenue measure. Format as Currency with commas and no decimal.

    DAX: `10 Day Rolling Revenue = CALCULATE([Total Revenue], DATESINPERIOD('AW Calendar Lookup'[Date], MAX('AW Calendar Lookup'[Date]), -10, DAY))`

30. ‘90 Day Rolling Profit’: It is the value of the running total of 90 days of the Total Profit measure. Format as Currency with commas and no decimal.

    DAX: `90 Day Rolling Profit = CALCULATE([Total Profit], DATESINPERIOD('AW Calendar Lookup'[Date], MAX('AW Calendar Lookup'[Date]), -90, DAY))`

---

## Phase 4: Visualizing Data with Reports

**Step 1:** Sketching the Dashboard Layout:

**I:** According to design framework guideline we would first define the objective of the dashboard. Since we have multiple objectives we would need a multi-page dashboard with each page serving a distinct deliberate purpose.
Our goals are:
1. Track KPIs: This would be a Executive targeted dashboard
2. Compare regional performance: This would require geospatial (map) analysis
3. Analyze product-level trends: This would need a product detail view
4. Identify high-value customers: This would need a customer detail view

**II:** 3 Key Questions:
1. Type of Data: Time-series (calendar data), Categorical (product category/subcategory data), Geospatial (territory data), Hierarchies
2. Communication Objective: Comparison & Composition
3. End User Type: Managers

**III:** Executive Dashboard Home View:

View layout design is often based on reading patterns: F & Z. The visuals are then arranged in the way these letters are drawn. The visual granularity increases as we move through the reading pattern. The top area is the most important in both layout designs since its the first place views are going to look. We start by adding the brand logo to the top left corner to set the context. The logo is followed by main high-level overall KPI cards in the remaining top area since this are important for executives and managers to see first.
The middle left area will contain a line chart trending visual with date hierarchy to allow granularity drilling, followed by KPI context cards below it like current month/previous month/month over month revenue change etc. Increasing the revenue granularity in the middle we can do a product category breakout showing comparison across categories. Followed by a little product level visual like Top 10 revenue driving products with some conditional formatting.
We can add an intuitive user friendly way for navigating between pages by creating a left navigation panel with some icons using the page navigation and bookmark functionalities.

**IV:** Map detail Page: We can add a slicer to the top of the map visual to dig into a specific country or continent and display corresponding data.

**V:** Product detail Page: We can filter focus into specific products on this page in the top right and show target revenue/profit/order stats using gauge charts to show product performance against those targets. We can also add some trending visuals like revenue/profit trending chart and column chart trending to show returns/return rate trends.

**VI:** Customer detail Page: We start on top right with high level metrics like total customers/customer revenue/orders per customer followed by compositional analysis of the customer demographics using donut charts. We can add some trending visuals like total customers and finally for the objective of identifying high-value customers we can add a table/matrix visual showing the top n customers. We can also add some info buttons to display some insight using bookmark to draw to a filtered view. 

**Step 2:** Add 4 Report Views – Exec Dashboard, Map, Product Detail, Customer Detail.

**In Exec Dashboard View:**

**Step 3:** Add AW Logo to the top left corner using Insert Image option. Add Rounded Rectangle Shapes as a background for the Main KPI cards. Change Rounded Corner Prop to 15%. Change Fill Colour to Black 20%. Remove Border. Height: 100 Width: 225. Make 3 more copies of the shape. Select all, Format Align – Distribute Horizontally & Align Top.

**Step 4:** Add Rectangle shape for the Navigation Bar to the View left edge from end to end. Change Fill Colour to 20% Black. Remove Border. Copy the bar to all the views. In the Selection pane, rename all added objects and group the KPI Card Backgrounds as KPI Backgrounds.

**Step 5:** Add Simple Cards: 1. Total Revenue as REVENUE 2. Total Profit as PROFIT 3. Total Orders as ORDERS 4. Return Rate as RETURN RATE
Rename Title by editing the field name in the visual. Remove Card Background. Change Callout Value Font to Segoe UI Bold, size 38. Format colour to #20E2D7 (Maven Blue). Add 1 Value decimal place. Change Category Label Font to Segoe UI, size 9. Format colour to White. Select all, Format Align – Distribute Horizontally & Align Top. Group all cards as KPI Values and then group it along with KPI Backgrounds as KPIs.

**In Customer Detail View:**

**Step 6:** Add Simple Cards: 1. Total Customers as UNIQUE CUSTOMERS 2. Average Revenue per Customer as REVENUE PER CUSTOMER. Follow same formatting as Cards on Exec Dashboard.

**In Exec Dashboard View:**

**Step 7:** Add Line Chart with Start of Month on X axis and Total Revenue on Y axis. Change Title to Monthly Revenue. Remove Axis Titles. Change Font to Segoe UI Size 10 Center Aligned. Add Zoom Slider for X axis. Line Stroke Width to 2px. Line Colour Black 20%.

Format Tooltip from Properties. Type Default. Text Label Colour White. Text value Colour Maven Blue. Drill Text Colour White. Background Colour Black 20% with 10% Transparency. Add Trend Line with 75% Transparency. Add Forecast with 2 points length with last 1 point ignored. Set 85% confidence.

**In Customer Detail View:**

**Step 8:** Add Line Chart with Start of Month on X axis and Total Customers on Y axis. Change Title to Monthly Customers. Add Trend line & Tooltips. Format similar to the line chart above.












