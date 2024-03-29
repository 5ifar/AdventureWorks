# 🧬 Adventure Works Project Phase-wise Implementation
--Brief Intro--

---

## Table of Contents
- [Phase 1: Connecting & Shaping Data using PBI Power Query Editor](#Phase-1-Connecting--Shaping-Data-using-PBI-Power-Query-Editor)
- [Phase 2: Creating the Data Model using PBI Model View](#Phase-2-Creating-the-Data-Model-using-PBI-Model-View)
- [Phase 3: Creating Calculated Fields with DAX](#Phase-3-Creating-Calculated-Fields-with-DAX)
- [Phase 4: Visualizing Data with Reports](#Phase-4-Visualizing-Data-with-Reports)

---

## Phase 1: Connecting & Shaping Data using PBI Power Query Editor

`Step 1:` Query Territory Lookup table to the Power Query Editor (PQE). Rename table. Check all data types.

`Step 2:` Query Product Lookup table to the PQE. Rename table. Check all data types.
- Change ProductCost & ProductPrice data type to Currency (i.e. Fixed decimal number).
- Add new column ‘SKU Type’ based on all characters before 2nd ‘-‘ character in Product SKU column.
- Replace all '0' in Product Style column with ‘NA’.

`Step 3:` Query Product Category Lookup table to the PQE. Rename table. Check all data types.

`Step 4:` Query Product Subcategory Lookup table to the PQE. Rename table. Check all data types.

`Step 5: Utilize Data QA & Column Profiling tools to improve quality of data:`
- Update Column Profiling to be based on entire dataset.
- Remove Error & Empty rows from Primary Key columns like CustomerKey from the Customer Lookup table.

`Step 6: Implement Text-specific table transformations:`
- Proper Case Prefix & Name columns in Customer Lookup table.
- Add new FullName column using Merge Columns tool.

`Step 7: Implement Number-specific table transformations:`
- Inspect Max, Min, Avg Product prices in Product Lookup table.
- Add DiscountPrice column by multiplying ProductPrice column by 0.9 (10% Discount) using Standard tool in Number-specific transformation tab.

`Step 8: Implement Date-specific table transformations:`
- Add new columns – Dayname, MonthName, Year, StartofWeek, StartofMonth, StartofQuarter & StartofYear.
- Modify StartofWeek step custom column formula to set Monday as first day of week instead of default Sunday using `Date.StartOfWeek([Date],Day.Monday)`.

`Step 9: Add Conditional Columns based on OrderQuantity column:`
- If the column value is 1 then add “Single Item”, if column value is greater than 1 then add “Multiple Items” else add “Other” in the Sales Data tables.

`Step 10: Append all the Sales data together using Append from Folder functionality:`
- Move all 3 Sales Data tables for years 2020 to 2022 to a folder.
- Implement an Append Query to combine all the tables using the folder as a data source into a combined Sales Data 2020-2022 table.
- Transform data by Combine Files option from the Content Column Settings.
- Remove the Source.Name column that mentions the file source as it’s not required.

`Step 11:` For all Lookup tables disable the ‘Include in Report Refresh’ option from the PQE Query list since they contain static data. This will leave only the Sales Data to be refreshed. This will optimize query refresh times.

---

## Phase 2: Creating the Data Model using PBI Model View

`Step 1:` Identify Primary Keys & Foreign Keys for all the tables to form relationships. In the Model view, in Table properties set the Key Column value as the column that is to act as the Primary Key for the respective Lookup Table.

`Step 2:` Setup Table Relationships by dragging Primary Keys from Lookup tables to Foreign Keys in other tables to setup the Filter Flow. This can also be done using the Manage Relationships setting.

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

`Step 3: Setup Model Schemas for the tables:`
- Calender Lookup, Territory Lookup, Customer Lookup & Product Lookup -> Sales Data will form Star Schema.
- Product Category Lookup -> Product Subcategory Lookup -> Product Lookup will form Snowflake Schema.

`Step 4:` Hide all the FK from Fact tables to force filter context to be based on PK in lookup tables:
- Sales Data -> CustomerKey, OrderDate, Stockdate, ProductKey, TerritoryKey
- Returns Data -> ProductKey, TerritoryKey, ReturnDate
- Product Lookup -> ProductSubcategoryKey
- Product Subcategories Lookup -> ProductCategoryKey

`Step 5: Setup any Data Format changes required Column tools tab in Data view:`
- Set the Data format for dates to be the Short date in date related columns in Calender, Sales & Return tables for easier interpretation during visualization.
- Set Data format for price related columns as Currency and decimal places as 2 in the Product Lookup table.

`Step 6: Setup any Data Category changes required Column tools tab in Data view:`
- Set the Data category as Country and Continent for the Country and Continent columns respectively in Territory Lookup table.

`Step 7: Implement Hierarchies:`
- Territory hierarchy: Continent -> Country -> Region
- Date hierarchy: StartofYear -> StartofMonth -> StartofWeek -> Date

---

## Phase 3: Creating Calculated Fields with DAX

`Step 1: Create Calculated Columns:`
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


`Step 2:` Configure a Measure Table to organize and store all the measures using the Enter Data functionality in the Data View. Move all the created Measures to this table.

`Step 3: Create Explicit Measures in the Measures Table:`
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

`Step 1: Sketching the Dashboard Layout:`

**I:** According to design framework guideline we would first define the objective of the dashboard. Since we have multiple objectives we would need a multi-page dashboard with each page serving a distinct deliberate purpose.
Our goals are:
1. Track KPIs: This would be a Executive targeted dashboard
2. Compare regional performance: This would require geospatial (map) analysis
3. Analyze product-level trends: This would need a product detail page
4. Identify high-value customers: This would need a customer detail page

**II:** 3 Key Questions:
1. Type of Data: Time-series (calendar data), Categorical (product category/subcategory data), Geospatial (territory data), Hierarchies
2. Communication Objective: Comparison & Composition
3. End User Type: Managers

**III:** Executive Dashboard Home Page:

Page layout design is often based on reading patterns: F & Z. The visuals are then arranged in the way these letters are drawn. The visual granularity increases as we move through the reading pattern. The top area is the most important in both layout designs since its the first place viewers are going to look. We start by adding the brand logo to the top left corner to set the context. The logo is followed by main high-level overall KPI cards in the remaining top area since this are important for executives and managers to see first.
The middle left area will contain a line chart trending visual with date hierarchy to allow granularity drilling, followed by KPI context cards below it like current month/previous month/month over month revenue change etc. Increasing the revenue granularity in the middle we can do a product category breakout showing comparison across categories. Followed by a little product level visual like Top 10 revenue driving products with some conditional formatting.
We can add an intuitive user friendly way for navigating between pages by creating a left navigation panel with some icons using the page navigation and bookmark functionalities.

**IV:** Map detail Page: We can add a slicer to the top of the map visual to dig into a specific country or continent and display corresponding data.

**V:** Product detail Page: We can filter focus into specific products on this page in the top right and show target revenue/profit/order stats using gauge charts to show product performance against those targets. We can also add some trending visuals like revenue/profit trending chart and column chart trending to show returns/return rate trends.

**VI:** Customer detail Page: We start on top right with high level metrics like total customers/customer revenue/orders per customer followed by compositional analysis of the customer demographics using donut charts. We can add some trending visuals like total customers and finally for the objective of identifying high-value customers we can add a table/matrix visual showing the top n customers. We can also add some info buttons to display some insight using bookmark to draw to a filtered view. 

`Step 2:` Add 4 Report Pages – Exec Dashboard, Map, Product Detail, Customer Detail.

**In Exec Dashboard Page:**

`Step 3:` Add AW Logo to the top left corner using Insert Image option. Add Rounded Rectangle Shapes as a background for the Main KPI cards. Change Rounded Corner Prop to 15%. Change Fill Colour to Black 20%. Remove Border. Height: 100 Width: 225. Make 3 more copies of the shape. Select all, Format Align – Distribute Horizontally & Align Top.

`Step 4:` Add Rectangle shape for the Navigation Bar to the Page left edge from end to end. Change Fill Colour to 20% Black. Remove Border. Copy the bar to all the Pages. In the Selection pane, rename all added objects and group the KPI Card Backgrounds as KPI Backgrounds.

`Step 5: Add Simple Cards:` 1. Total Revenue as REVENUE 2. Total Profit as PROFIT 3. Total Orders as ORDERS 4. Return Rate as RETURN RATE
Rename Title by editing the field name in the visual. Remove Card Background. Change Callout Value Font to Segoe UI Bold, size 38. Format colour to #20E2D7 (Maven Blue). Add 1 Value decimal place. Change Category Label Font to Segoe UI, size 9. Format colour to White. Select all, Format Align – Distribute Horizontally & Align Top. Group all cards as KPI Values and then group it along with KPI Backgrounds as KPIs.

**In Customer Detail Page:**

`Step 6: Add Simple Cards:` 1. Total Customers as UNIQUE CUSTOMERS 2. Average Revenue per Customer as REVENUE PER CUSTOMER. Follow same formatting as Cards on Exec Dashboard.

**In Exec Dashboard Page:**

`Step 7:` Add Line Chart with Start of Month on X axis and Total Revenue on Y axis. Change Title to Monthly Revenue. Remove Axis Titles. Change Font to Segoe UI Size 10 Center Aligned. Add Zoom Slider for X axis. Line Stroke Width to 2px. Line Colour Black 20%.

Format Tooltip from Properties. Type Default. Text Label Colour White. Text value Colour Maven Blue. Drill Text Colour White. Background Colour Black 20% with 10% Transparency. Add Trend Line with 75% Transparency. Add Forecast with 2 points length with last 1 point ignored. Set 85% confidence.

**In Customer Detail Page:**

`Step 8:` Add Line Chart with Start of Month on X axis and Total Customers on Y axis. Change Title to Monthly Customers. Add Trend line & Tooltips. Format similar to the line chart above.

**In Exec Dashboard Page:**

`Step 9:` Add KPI card with Total Revenue measure value with Trend axis as Start of Month and target values as Previous Month Revenue measure. If target is met the KPI card will show green colour else, it will be in red colour. Change Title to Monthly Revenue. Change Title Font to Segoe UI Size 10 Center Aligned. Change Callout Value Font to Segoe UI Bold Size 30 Center Aligned with Display units as Millions. Drop Trend axis transparency to 10% and set colours and Direction according to the measure plotted. Change Target Lable Font to Segoe UI Size 9 Center Aligned and Label as Prev Month.

Replicate the KPI card for Total Orders and Total Returns measures with targets as Previous Month Orders and Previous Month Returns. Change Titles. For both KPIs set the display units as None since we want to see whole figures. For Monthly Returns KPI set the direction as Low is good.

`Step 10:` Add Bar chart to see Product wise categorical breakdown with Y axis as CategoryName and X axis as Total Orders values. Remove Axis Titles. Remove X axis data value range and add data labels instead in Outside end position. Change Title to Orders by Category. Change Title Font to Segoe UI Size 10 Center Aligned. Change Bar colour to 20% Black. Format Tooltip from Properties. Type Default. Text Label Colour White. Text value Colour Maven Blue. Drill Text Colour White. Background Colour Black 20% with 10% Transparency.

**In Customer Detail Page:**

`Step 11:` Add Donut chart of Total Orders Measure by Income level. Change Title to Orders by Income Level. Change Title Font to Segoe UI Size 10 Center Aligned. Turn off the legend and instead show data labels of category and value with font size 8 and 1 decimal place. Update the colour of slices. Add visual-level filter to exclude customers with “Very High” income level.

Replicate above donut chart for Total Orders measure by Occupation. Change Title to Orders by Occupation. Change Title Font to Segoe UI Size 10 Center Aligned. Add visual-level filter to display the 3 occupations with the most orders using the Top N filtering type with filter data values as Total Orders.

**In Exec Dashboard Page:**

`Step 12:`  Add Matrix visual with Product Name as row and Total Orders, Total Revenue & Return Rate measures as values. Rename columns accordingly. Set Style Preset as None. Disable Column and Row Subtotals.

`Step 13:` Configure Conditional formatting for the Matrix visual. In Properties go to Cell elements:
1. For Orders column, Set Data bars. For Negative Bar set colour as White and for Positive Bar set colour as 20% Dark White.
2. For Return % column, Set Background colour. For Max value set colour as Red and for Min value set colour as White.  Format style is Gradient.

`Step 14:` Configure Top 10 Visual level filter on the Matrix visual by the Total Orders measure value. Change column name to Top 10 Products.

**In Customer Detail Page:**

`Step 15:` Add Table visual with Customer Key, Full Name, Total Orders and Total Revenue as columns. Add Conditional formatting: For Orders column add White to Grey colour Data bars & for Revenue add White to Maven Blue colour scale. Configure Top 100 Visual level filter by the Total Orders measure value. Change table name to Top 100 Customers. Sort descending by Orders column.

**In Exec Dashboard Page:**

`Step 16:` Add Simple card to show Top Product Subcategory based on Total Orders measure. Configure Top 1 Visual level filter by the Total Orders measure value. Set background colour as 20% Black. Set callout value colour as Maven Blue with Segoe UI bold font and 14 size. Add a Text box to add Card title as Most Ordered product Type.

Add Simple card to show Top Product Subcategory based on Return Rate measure. Configure Top 1 Visual level filter by the Return Rate measure value. Set background colour as 20% Black. Set callout value colour as Maven Blue with Segoe UI bold font and 14 size. Add a Text box to add Card title as Most Returned Product Type.

**In Customer Detail Page:**

`Step 17:` Add Simple card to show Top Customer based on Total Revenue measure. Configure Top 1 Visual level filter by the Total Revenue measure value on Customer key parameter. Set background colour as 20% Black. Set callout value colour as Maven Blue with Segoe UI bold font and 26 size. Add a Text box to add Card title as Top Customer (by Revenue). Copy the above Simple card to create 2 more cards showing the Total Orders and the Total revenue measure for the top customer. The visual level filters should also be kept consistent.

**In Map Page:**

`Step 18:` Add Map visual for Location as Country field and bubble size based on Total Orders measure. Set Map Style as Dark. Change Bubble colour to Maven Blue. Enable Category Labels. Remove Title.

Format Tooltip from Properties. Type Default. Text Label Colour White. Text value Colour Maven Blue. Drill Text Colour White. Background Colour Black 20% with 10% Transparency.

Add Continent field slicer. Hide Slicer headers. Change Slicer Style to Tile. Change Value font to Segoe UI Bold Size 10. Add White border with 3-line width. Add Grey Background. In Slicer settings add Select All option.

**In Customer Detail Page:**

`Step 19:` Add Year field slicer. Hide Slicer headers. Change Slicer Style to Between. Change Value font to Segoe UI Bold Size 10. Add visual level filter to exclude blanks.

`Step 20:` For year 2020, we have multiple customers with the same highest revenue. To display this what Power BI does is it aggregates the sum of all values which is not the correct approach. To fix this we’ll use the HASONEVALUE operator to create new measures to be used instead. Swap in these new measures for the 3 Top Customer card visuals.

New DAX Measures:
1. Total Orders (Customer Detail) = IF(HASONEVALUE('AW Customer Lookup'[CustomerKey]), [Total Orders], "-")\
2. Total Revenue (Customer Detail) = IF(HASONEVALUE('AW Customer Lookup'[CustomerKey]), [Total Revenue], "-")
3. Full Name (Customer Detail) = IF(HASONEVALUE('AW Customer Lookup'[CustomerKey]), MAX('AW Customer Lookup'[FullName]), "Multiple Customers")

**In Product Detail Page:**

`Step 21:` Add Gauge chart visual on Total Order measure value. Add Top 1 visual level filter based on Latest Start of Month value. Set the Max value as the Order Target measure. Set Title as Monthly Orders Vs Target. Set Fill colour as 20% Black. Replicate the above Gauge chart for Total Revenue and Total Profit measures. Set the visual title accordingly.

`Step 22:` Add Conditional formatting for the above 3 Gauge Charts by creating new measures to base the formatting on.

DAX: Order Target Gap = [Total Orders] - [Order Target]
DAX: Revenue Target Gap = [Total Revenue] - [Revenue Target]
DAX: Profit Target Gap = [Total Profit] - [Profit Target]

Set Callout value and Gauge colour Conditional formatting Format Style as Rules. Set Order Target Gap as the field to be based on. Set 2 rules:
1. If value >= Min and < 0 then colour as Red
2. If value >= 0 and <= Max then colour as 20% Black.

Apply same formatting for other Gauges based on Revenue Target Gap and Profit Target Gap measures.

`Step 23:` Add Area Chart with Total Profit on Y Axis and Start of Month on X Axis. Set Title as Monthly Profit. Change Line colour to 20% Black and Shade Area with 80% transparency. Add another Area Chart with Total Returns on Y Axis and Start of Month on X Axis. Set Title as Monthly Returns. Change Line colour to Red and Shade Area with 80% transparency.

`Step 24:` Add Date Hierarchy to the Monthly Profit and Monthly Returns Area charts. Rename Titles to Profit Trending and Return Trending respectively.

**In Exec Dashboard Page:**

`Step 25:` Create visual drill down hierarchy for X Axis of Monthly Revenue line chart as Year -> Start of Quarter -> Start of Month. This helps us create a sort of visual level hierarchy for top level executives. Change Title to Revenue Trending.

**In Customer Detail Page:**

`Step 26:` Add Date Hierarchy to the Monthly Customers line chart. Rename Title to Customer Trending.

**In Product Detail Page:**

`Step 27:` Configure Product Drill through functionality: In the Product Detail Page change the Page Type to Drill through and Drill through from Product Name field. Delete Back Button if automatically created, this would be configured later. Remove Product Name slicer added earlier. In Exec Dashboard Product table, right click on any product and in drill through option select Product Detail Page. Add a Simple card on the page to display Product Name being drilled through. Add Text Box above this card with Selected Product text.

`Step 28: Setup Report Interactions:`

*In Exec Dashboard Page:* 1. Set the Orders by Category bar graph to Filter interaction based on Revenue Trending line chart filter since its more intuitive. 2. Set the 3 Monthly KPIs to None interaction based on Revenue Trending line chart filter since they represent current month metrics and hence shouldn’t change. 3. Set the Most Ordered/Returned Product Type Card to None interaction based on product selected filter in the Top 10 Products Matrix since then they will not show overall type but the type of the selected product which is not correct behaviour. 4. Set the Orders by category Bar chart to None interaction based on product selected filter in the Top 10 Products Matrix.

*In Customer Detail Page:* 1. Set the Orders by Income/Occupation Donuts to Filter interaction based on Customer Trending line chart filter. 2. Set the Customer Trending and Orders by Income/Occupation Donuts to None interaction based on Top 100 Customer Table filter. Set Year slicer to Filter interaction to all visuals.

`Step 29: Adding Bookmarks:`
In Exec Dashboard Page: Clear all applied filters. Navigate to the Bookmark pane and select Add Bookmark. Rename to Clear Exec Filters. Now we need something like a button to trigger the bookmark. Add a Reset Button. In Button formatting panel enable Actions. Set Action type to Bookmark and select our bookmark. Add tooltip if required. Add button formatting so the icon line colour is White in default state and Maven Blue in hover and pressed states.

`Step 30: Setting up Custom Navigation Buttons:`
Copy the left Nav Bar to a new page where we’ll build the navigation setup. This needs to be done since we’ll be configuring Nav button actions to navigate to each of the pages and if we build this bar on anyone of those pages we won’t have the option to configure the button to navigate to same page it was built on. So building it on a new page gives us the option to configure navigation to all the other pages and then we can copy the bar to all the pages.
Insert a Blank Button, in button formatting change state to Default and set Icon Type as Custom and Browse to select custom icon. Setup similarly custom icons for hover and click state. Enable Button Action as Type: Page Navigation and set the Destination as Required Page. Group all icons in the selection pane as Navigation Buttons.

`Step 31: Adding Custom Slicer Panel:`
Add Filter button in the left Nav bar and configure style with all states and Filter custom icons as required. Add a rectangle shape preferable the height of the page to configure the slicer. Format the shape as required with 20% Black background. Add the Year and Continent slicers inside the shape. Format the slicers as required.
We need another button to go back to the hidden state. Add Left Arrow/Back button and format as required. Group all of these slicers, back button and background shape as Slicer Panel.
Now we need to create 2 bookmarks: 1. Hide Slicer Panel 2. Show Slicer Panel. For 2, make the bookmark as it is. For 1, from the selection pane hide the Slicer Panel group and then create the bookmark. Uncheck Data option from both bookmark settings to ensure show/hide bookmark does not affect the existing applied filters. Configure both Filter & Back button action type as Bookmarks and select corresponding Show/Hide Panel Bookmarks.

`Step 32: Configuring Parameters:`

**In Product Detail Page:**

1. Adding Numeric Parameter:
Create a new Parameter named Price Adjustment % with numeric range variable adjustment. Set data type as decimal, with minimum value as -1 and maximum value of 1 with increment of 0.1 and default value of 0. Add this slicer to the page.
Parameter Table: Price Adjustment (%) = GENERATESERIES(-1, 1, 0.1)
Parameter Measure: Price Adjustment (%) Value = SELECTEDVALUE('Price Adjustment (%)'[Price Adjustment (%)], 0)
Format the slicer as required with the slicer style as single value.
Now we need to create new measures: Adjusted Revenue, Adjusted Profit & Adjusted Price which will take into account these parameter changes.
AdjustedPrice = [AverageRetailPrice] * (1 + 'Price Adjustment (%)'[Price Adjustment (%) Value])
AdjustedRevenue = SUMX('AW Sales Data', 'AW Sales Data'[OrderQuantity] * [AdjustedPrice])
AdjustedProfit = [AdjustedRevenue] - [TotalCost]
Change Profit trending chart into a line chart and in the Y axis add the Adjusted Profit measure. Remove the Profit Trending Title since the Legend is now available. Change Adjusted Profit line colour to Maven blue for better visual interpretation. Add small data markers if required.
2. Adding Fields Parameter:
Create a new Parameter named Product Metric Selection with fields selection as Total Order, Total Revenue, Total Profit, Total Returns and Return Rate measures. Format the slicer as required with the slicer style as single select.
Parameter Table: Product Metric Selection = {
    ("Orders", NAMEOF('AW Measure Table'[TotalOrders]), 0),
    ("Revenue", NAMEOF('AW Measure Table'[TotalRevenue]), 1),
    ("Profit", NAMEOF('AW Measure Table'[TotalProfit]), 2),
    ("Returns", NAMEOF('AW Measure Table'[TotalReturns]), 3),
    ("Return %", NAMEOF('AW Measure Table'[ReturnRate]), 4)
}
Change Return trending chart Y axis to the Product Metric Selection parameter. Remove the Return Trending Title. Format the Return and Return Rate metric line colour to Red while the rest as 20% Black to indicate high value is not favourable.

`Step 33: Create new Category Tooltip page:`
Set the page type as Tooltip. Set custom tooltip size to 225px height and 425px width. Set background to 20% Black with no transparency.
1. Add a multi row card. Add fields as Total Revenue, Total Profit, Total Orders, Total Returns & Return Rate measures. Set the formatting as required with white font and no background.
2. Add an area chart to display order trending by category. Set Start of week as X axis and Total Orders measure as Y axis. Remove background and set the Title as Weekly Orders and line colour as Maven blue.
To connect this to our visual in the Exec Dashboard, select the visual and in the visual properties -> Tooltip settings select the Type as Report Page and the page as Category Tooltip.
Hide the tooltip page from the report to prevent accidental modifications by end users.

`Step 34: Implement User Roles:`
Although Roles (Row level security rules) are being added here in PBI Desktop, they will be applied in PBI Service.
In the Modelling tab click on Manage Roles and Create Roles:
1. Europe: Territory Lookup Table -> Rule: Continent equals Europe
2. North America: Territory Lookup Table -> Rule: Continent equals North America
3. Pacific: Territory Lookup Table -> Rule: Continent equals Pacific

