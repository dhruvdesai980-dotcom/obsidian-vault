For **MSTRT-536**, we need test whether relationships inferred from a **Free-form SQL custom dataset** are stable or change when reopening the Mosaic **Hierarchy** tab.

The reported concern has two parts:

1. Relationships may need to be manually rebuilt after using custom SQL.
    
2. AI-inferred relationships may change across repeated visits to Hierarchy.
    

## 1. Create net-new source tables

Run these in Databricks:

```sql
CREATE OR REPLACE TABLE
  `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt536_customer
USING DELTA AS
SELECT *
FROM VALUES
  ('C001', 'Apex Corporation',  'Canada'),
  ('C002', 'Beacon Industries', 'USA'),
  ('C003', 'Crest Limited',     'UK')
AS t(customer_id, customer_name, country);
```

```sql
CREATE OR REPLACE TABLE
  `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt536_order
USING DELTA AS
SELECT *
FROM VALUES
  ('O001', 'C001', DATE '2026-01-05', 1000.00),
  ('O002', 'C001', DATE '2026-01-10', 1500.00),
  ('O003', 'C002', DATE '2026-02-05', 2000.00),
  ('O004', 'C003', DATE '2026-03-05', 2500.00)
AS t(order_id, customer_id, order_date, order_amount);
```

```sql
CREATE OR REPLACE TABLE
  `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt536_payment
USING DELTA AS
SELECT *
FROM VALUES
  ('P001', 'O001', DATE '2026-01-06', 1000.00),
  ('P002', 'O002', DATE '2026-01-11', 1500.00),
  ('P003', 'O003', DATE '2026-02-06', 2000.00),
  ('P004', 'O004', DATE '2026-03-06', 2500.00)
AS t(payment_id, order_id, payment_date, payment_amount);
```

The intended relationships are:

|Parent|Child|Key|
|---|---|---|
|Customer|Order|`customer_id`|
|Order|Payment|`order_id`|

## 2. Create a Free-form SQL dataset

In Strategy, create a new dataset using **Free-form SQL**, not by directly importing the three tables.

Use:

```sql
SELECT
    c.customer_id,
    c.customer_name,
    c.country,
    o.order_id,
    o.order_date,
    o.order_amount,
    p.payment_id,
    p.payment_date,
    p.payment_amount
FROM
    `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt536_customer c
LEFT JOIN
    `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt536_order o
    ON c.customer_id = o.customer_id
LEFT JOIN
    `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt536_payment p
    ON o.order_id = p.order_id
```

Name the custom dataset:

```text
MSTRT-536 Custom SQL Dataset
```

Import it into a new Mosaic model named:

```text
MSTRT-536 Relationship Stability Test
```

## 3. Record the initial inferred model

Before changing anything manually:

1. Open **Prep and Model → Hierarchy**.
    
2. Record the attributes and relationships Strategy generated.
    
3. Capture a screenshot.
    
4. Check whether it produced this logical path:
    

```text
Customer → Order → Payment
```

Also record whether Strategy:

- Created all three attributes
    
- Left them disconnected
    
- Created a different hierarchy
    
- Created duplicate attributes or forms
    

Do not correct the relationships yet.

## 4. Test stability

Repeat this exact sequence at least five times:

1. Leave the **Hierarchy** tab and open **Tables**.
    
2. Return to **Hierarchy**.
    
3. Close the model.
    
4. Reopen the same model.
    
5. Return to **Hierarchy**.
    
6. Refresh the browser and check again.
    

Capture screenshots after the first, third, and fifth checks.

Do not publish or edit the relationships between checks. Otherwise, we cannot tell whether Strategy changed the inference on its own.

## 5. Test after publishing

If the initial hierarchy is correct:

1. Validate and publish the model.
    
2. Close it completely.
    
3. Reopen it.
    
4. Check Hierarchy again.
    
5. Create a grid with:
    
    - Customer
        
    - Order
        
    - Payment
        
    - Order Amount
        
    - Payment Amount
        
6. Confirm four correct order/payment combinations appear.
    
7. Reopen Hierarchy once more and compare it with the original screenshot.
    

## Conclusion criteria

|Observation|Conclusion|
|---|---|
|Hierarchy changes without any model edit|MSTRT-536 reproduced|
|Relationships disappear after reopening|MSTRT-536 reproduced|
|Incorrect inference is consistent every time|Manual modeling required, but instability not reproduced|
|Relationships remain identical through all checks|MSTRT-536 not reproduced|
|Free-form SQL produces one flattened table without relationships|Test is inconclusive for relationship stability; the custom SQL design does not expose multiple tables for Mosaic to relate|

One important limitation: because the SQL above returns a **single flattened dataset**, Strategy may correctly treat it as one logical table. If that happens, it tests attribute/hierarchy inference but not physical relationships between separate custom datasets. To test the exact claim more strongly, create **three separate Free-form SQL datasets**—Customer, Order, and Payment—then let Mosaic infer relationships among them.