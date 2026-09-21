Sure—create a completely separate model and dashboard for MSTRT-360.

### 1. Create a new Databricks table

```sql
CREATE OR REPLACE TABLE
  `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt360_sales
USING DELTA AS
SELECT *
FROM VALUES
  ('S001', DATE '2026-01-05', 'Retail',    100.00),
  ('S002', DATE '2026-01-10', 'Corporate', 200.00),
  ('S003', DATE '2026-02-05', 'Retail',    300.00),
  ('S004', DATE '2026-02-10', 'Corporate', 400.00),
  ('S005', DATE '2026-03-05', 'Retail',    500.00),
  ('S006', DATE '2026-03-10', 'Corporate', 600.00)
AS t(sale_id, sale_date, customer_type, sale_amount);
```

This produces six rows: two in each month from January through March.

### 2. Create a new Mosaic model

Name it:

```text
MSTRT-360 Filter Scope Model
```

Add only `mstrt360_sales`, then create:

- **Sale** attribute using `sale_id`
    
- **Sale Date** time attribute using `sale_date`
    
- **Customer Type** attribute using `customer_type`
    
- **Sale Amount** metric using `SUM(sale_amount)`
    
- **Sale Count** metric using `COUNT DISTINCT(sale_id)`
    

Validate and publish the model.

### 3. Create a new dashboard

Name it:

```text
MSTRT-360 Page vs Chapter Filter Test
```

Create this layout:

- Chapter 1
    
    - Page 1: Sale Count KPI and a grid with Sale Date, Sale and Sale Amount
        
    - Page 2: the same Sale Count KPI and grid
        
- Chapter 2
    
    - Page 1: the same Sale Count KPI and grid
        

Before adding filters, every page should show a Sale Count of **6**.

### 4. Add the filter from Chapter 1, Page 1

While Chapter 1 → Page 1 is open:

1. Add `Sale Date` to the dashboard/page filter area.
    
2. Select January 2026.
    
3. Confirm Page 1 changes from **6 to 2**.
    
4. Open Chapter 1 → Page 2 without adding a filter there.
    
5. Open Chapter 2 → Page 1.
    

### Expected comparison

|Location|If the reported behavior occurs|
|---|--:|
|Chapter 1 → Page 1|2|
|Chapter 1 → Page 2|2|
|Chapter 2 → Page 1|6|

If both Chapter 1 pages change to 2 while Chapter 2 stays at 6, MSTRT-360 is reproduced: the apparent page filter is actually chapter-scoped.

Also record whether Page 2 visibly indicates that the January filter is active. The lack of an inherited-filter indicator is part of the reported usability problem.