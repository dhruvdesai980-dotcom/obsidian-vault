For **MSTRT-537**, test whether one source visualization can simultaneously:

1. Filter another visualization using **Target Visualization**, and
    
2. Open a destination using a **Contextual Link**.
    

We’ll create a separate setup.

## 1. Create a net-new table

```sql
CREATE OR REPLACE TABLE
  `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt537_sales
USING DELTA AS
SELECT *
FROM VALUES
  ('S001', 'Canada', 'Technology', 'Laptop',  1200.00),
  ('S002', 'Canada', 'Technology', 'Monitor',  400.00),
  ('S003', 'Canada', 'Furniture',  'Desk',     600.00),
  ('S004', 'USA',    'Technology', 'Laptop',  1300.00),
  ('S005', 'USA',    'Furniture',  'Chair',    300.00),
  ('S006', 'UK',     'Technology', 'Monitor',  450.00)
AS t(sale_id, country, category, product, sale_amount);
```

## 2. Create the model

Create a model named:

```text
MSTRT-537 Contextual Linking Model
```

Add only `mstrt537_sales` and create:

- Sale using `sale_id`
    
- Country using `country`
    
- Category using `category`
    
- Product using `product`
    
- Sale Amount using `SUM(sale_amount)`
    

Validate and publish.

## 3. Build the dashboard

Create:

```text
MSTRT-537 Contextual Link Test
```

On **Page 1**, add:

- **Visualization A — Source:** bar chart with Country and Sale Amount
    
- **Visualization B — Target:** grid with Country, Category, Product and Sale Amount
    

Create **Page 2 — Country Details** containing:

- Country
    
- Product
    
- Sale Amount
    

Save the dashboard.

## 4. Test Target Visualization first

Select Visualization A and configure it to target Visualization B.

Click **Canada** in the bar chart.

Expected result:

- Visualization B displays only Canadian rows.
    
- The chart-to-grid targeting works.
    

Take a screenshot as the baseline.

## 5. Add the Contextual Link

On the same source, Visualization A:

1. Open its menu or configuration panel.
    
2. Add a **Contextual Link**.
    
3. Set the destination to **Page 2 — Country Details**.
    
4. Pass the selected Country as context.
    
5. Apply the configuration.
    

Watch carefully for what happens to the existing Target Visualization setting.

## 6. Test both behaviours

Click or right-click Canada in Visualization A.

Check whether:

- Visualization B still filters to Canada.
    
- The contextual link remains available.
    
- Opening the link takes you to Page 2 filtered to Canada.
    
- Configuring the contextual link removed or disabled Target Visualization.
    
- Re-enabling Target Visualization removes the contextual link.
    

## Conclusion criteria

|Observation|Conclusion|
|---|---|
|Both functions remain configured and work|Not reproduced|
|Adding Contextual Link removes/disables Target Visualization|MSTRT-537 reproduced|
|Adding Target Visualization removes/disables Contextual Link|MSTRT-537 reproduced|
|Both appear configured but only one works|MSTRT-537 reproduced|
|Feature is unsupported only for the chosen visualization type|Retest with a grid before concluding|

The exact evidence we want is a before-and-after screenshot of Visualization A’s settings. First configure **Target Visualization**, then add **Contextual Link** and capture whether the target setting disappears. The reported workaround—putting the link in a panel—should be tested only after confirming the limitation.