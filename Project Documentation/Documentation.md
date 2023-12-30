# 🧬 Adventure Works Project Phase-wise Implementation
--Brief Intro--

---

## Table of Contents
- [Phase 1: Connecting & Shaping Data using Power Query Editor](#Phase-1:-Connecting-&-Shaping-Data-using-Power-Query-Editor)


---

## Phase 1: Connecting & Shaping Data using Power Query Editor

**Step 1:** Query Territory Lookup table to the Power Query Editor (PQE). Rename table. Check all data types.

**Step 2:** Query Product Lookup table to the PQE. Rename table. Check all data types.
- Change ProductCost & ProductPrice data type to Currency (i.e. Fixed decimal number).
- Add new column ‘SKU Type’ based on all characters before 2nd ‘-‘ character in Product SKU column.
- Replace all 0 in Product Style column with ‘NA’.

**Step 3:** Query Product Category Lookup table to the PQE. Rename table. Check all data types.

**Step 4:** Query Product Subcategory Lookup table to the PQE. Rename table. Check all data types.

**Step 5:** Utilize Data QA & Column Profiling tools to improve quality of data. Update Column Profiling to be based on entire dataset. Remove Error & Empty rows from Primary Key columns like CustomerKey from the Customer Lookup table.

**Step 6:** Implement Text-specific table transformations:
- Proper Case Prefix & Name columns in Customer Lookup table.
- Add new FullName column using Merge Columns tool.

**Step 7:** Implement Number-specific table transformations:
- Inspect Max, Min, Avg Product prices in Product Lookup table.
- Add DiscountPrice column by multiplying ProductPrice column by 0.9 (10% Discount) using Standard tool in Number-specific transformation tab.
