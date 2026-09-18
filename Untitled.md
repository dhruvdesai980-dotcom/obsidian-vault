For **MSTRT-497**, we are testing dashboard subtotal calculations—not model relationships.

## Problem in simple terms

Suppose the grid contains:

```text
Rating
  └── Counterparty
       └── Agreement
            └── Margin Type
                 └── Status
                      └── Action
```

When **Show Totals** is enabled, Strategy should calculate a subtotal using only the rows inside each group.

Example:

|Rating|Amounts|Correct subtotal|
|---|---|--:|
|AAA|100 + 200 + 300 + 400|1,000|
|AA|500 + 600 + 700 + 800|2,600|
|Overall|All rows|3,600|

The reported problem is that different rating groups may display the same number, potentially using the entire-grid total instead of their own rows.

## Create a controlled table

Run:

```sql
CREATE OR REPLACE TABLE
  `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt497_multilevel_totals
USING DELTA
AS
SELECT *
FROM VALUES
  ('AAA', 'Apex Bank',   'AG001', 'INITIAL',   'OPEN',   'CALL',   100.00),
  ('AAA', 'Apex Bank',   'AG001', 'VARIATION', 'OPEN',   'CALL',   200.00),
  ('AAA', 'Beacon Bank', 'AG002', 'INITIAL',   'CLOSED', 'RETURN', 300.00),
  ('AAA', 'Beacon Bank', 'AG002', 'VARIATION', 'CLOSED', 'RETURN', 400.00),

  ('AA',  'Crest Bank',  'AG003', 'INITIAL',   'OPEN',   'CALL',   500.00),
  ('AA',  'Crest Bank',  'AG003', 'VARIATION', 'OPEN',   'CALL',   600.00),
  ('AA',  'Delta Bank',  'AG004', 'INITIAL',   'CLOSED', 'RETURN', 700.00),
  ('AA',  'Delta Bank',  'AG004', 'VARIATION', 'CLOSED', 'RETURN', 800.00)
AS t(
  rating,
  counterparty,
  agreement,
  margin_type,
  status,
  action,
  margin_amount
);
```

## Confirm the expected totals

```sql
SELECT
  rating,
  SUM(margin_amount) AS expected_rating_total
FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt497_multilevel_totals
GROUP BY rating
ORDER BY rating;
```

Expected:

|Rating|Expected total|
|---|--:|
|AA|2,600|
|AAA|1,000|

Grand total:

```sql
SELECT SUM(margin_amount) AS expected_grand_total
FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt497_multilevel_totals;
```

Expected: `3,600`.

## Create the Mosaic model

Name:

**MSTRT-497 Multi-Level Totals Model**

Import only `mstrt497_multilevel_totals`.

Use:

- Rating
    
- Counterparty
    
- Agreement
    
- Margin Type
    
- Status
    
- Action
    
- Margin Amount as `SUM`
    

Publish the model without manually adding attribute relationships unless Mosaic requires them.

## Create the dashboard

Name:

**MSTRT-497 – Multi-Level Grid Totals Validation**

Create one grid with rows in this exact order:

1. Rating
    
2. Counterparty
    
3. Agreement
    
4. Margin Type
    
5. Status
    
6. Action
    

Add `Margin Amount` as the metric.

First capture the grid with totals disabled. Then enable **Show Totals** and display subtotals at every available level.

## Expected totals

|Group|Correct subtotal|
|---|--:|
|AAA|1,000|
|Apex Bank|300|
|Beacon Bank|700|
|AA|2,600|
|Crest Bank|1,100|
|Delta Bank|1,500|
|Grand total|3,600|

MSTRT-497 is reproduced if, for example:

- AAA and AA both show `3,600`.
    
- Different groups display identical incorrect subtotals.
    
- A rating subtotal includes rows belonging to another rating.
    
- Detail rows are correct but subtotals are wrong.
    

The intentionally different group totals make the defect easy to identify.