For **MSTRT-487**, the objective is to test whether a **Consumer-only user** can open a dashboard backed by a Mosaic model in **Live mode**, while the same user can open an equivalent Import-mode dashboard.

A second Consumer test account is required. Testing as the model owner or administrator will not reproduce a permission-specific problem.

## 1. Create a net-new test table

```sql
CREATE OR REPLACE TABLE
  `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt487_sales
USING DELTA AS
SELECT *
FROM VALUES
  ('S001', 'Canada', 'Technology', 1200.00),
  ('S002', 'Canada', 'Furniture',   600.00),
  ('S003', 'USA',    'Technology', 1300.00),
  ('S004', 'USA',    'Furniture',   300.00),
  ('S005', 'UK',     'Technology',  450.00)
AS t(sale_id, country, category, sale_amount);
```

## 2. Create the Live model

Create:

```text
MSTRT-487 Live Model
```

Add `mstrt487_sales` and configure its data-access mode as **Live**.

Create:

- Sale from `sale_id`
    
- Country from `country`
    
- Category from `category`
    
- Sale Amount as `SUM(sale_amount)`
    

Validate and publish.

## 3. Create the Live dashboard

Create:

```text
MSTRT-487 Live Dashboard
```

Add:

- Grid: Country, Category and Sale Amount
    
- KPI: Sum of Sale Amount
    
- Bar chart: Country by Sale Amount
    

As the author, confirm:

- Five underlying rows are represented.
    
- Total Sale Amount is **3,850**.
    
- Canada = **1,800**
    
- USA = **1,600**
    
- UK = **450**
    

## 4. Create the Import-mode control

Create a second model using the same table:

```text
MSTRT-487 Import Model
```

Set this one to **Import/In-memory mode**, publish it, and build:

```text
MSTRT-487 Import Dashboard
```

Use the same three visualizations and verify the same results.

## 5. Configure the Consumer user

Give the Consumer test user equivalent viewing access to:

- Both dashboards
    
- Both Mosaic models/datasets
    
- The folder containing them
    
- Any required data-source or connection objects
    

Do not give the user Author, Architect, or Administrator privileges.

Merely proving that the user can query the Databricks table is not enough. Live dashboards may also require access to the Strategy connection, model and execution objects.

## 6. Test with a clean Consumer session

Use a private/incognito browser and sign in as the Consumer user.

Test in this order:

1. Open `MSTRT-487 Live Dashboard`.
    
2. Record whether it loads or displays **Application Error**.
    
3. Open `MSTRT-487 Import Dashboard`.
    
4. Record whether it loads successfully.
    
5. Refresh and repeat once to rule out a temporary failure.
    

Capture:

- The Consumer user’s application role
    
- Both dashboard permissions
    
- Both model permissions
    
- The complete Live-mode error
    
- Whether the Import dashboard works
    

## Interpretation

|Live dashboard|Import dashboard|Conclusion|
|---|---|---|
|Fails|Works|MSTRT-487 reproduced|
|Works|Works|Not reproduced|
|Fails|Fails|General sharing/permission problem|
|Works|Fails|Import dataset access or refresh problem|

If Live fails, temporarily grant only the missing connection/model privilege and retest. If that resolves it, document it as a permission dependency rather than a general inability of Consumers to use Live-mode dashboards.