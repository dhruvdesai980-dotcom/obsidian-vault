For **MSTRT-506**, we should create another small controlled model. This issue differs from MSTRT-507.

## Problem statement in simple terms

Imagine two dimension tables:

### Counterparty

|Counterparty Code|Counterparty Name|
|---|---|
|CP001|Apex Bank|
|CP002|Beacon Bank|

### Agreement

|Agreement ID|Agreement Name|Counterparty Code|
|---|---|---|
|AG001|Loan Agreement|CP001|
|AG002|Derivative Agreement|CP001|
|AG003|Credit Agreement|CP002|

These tables can join directly using `Counterparty Code`.

```text
Counterparty ──Counterparty Code── Agreement
```

However, Mosaic may connect both dimensions only through a fact table:

```text
Counterparty → Fact ← Agreement
```

That works when a fact metric is included. But if the dashboard grid contains only:

- Counterparty Name
    
- Agreement Name
    

Mosaic may not know how to join them directly and may report a Cartesian-product warning.

## Controlled tables required

We need three tables:

1. `mstrt506_dim_counterparty`
    
2. `mstrt506_dim_agreement`
    
3. `mstrt506_fact_trade`
    

### 1. Counterparty dimension

```sql
CREATE OR REPLACE TABLE
  `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt506_dim_counterparty
USING DELTA
AS
SELECT *
FROM VALUES
  ('CP001', 'Apex Bank',       'BANK'),
  ('CP002', 'Beacon Bank',     'BANK'),
  ('CP003', 'Crest Insurance', 'INSURANCE'),
  ('CP004', 'Delta Pension',   'PENSION')
AS t(
  counterparty_code,
  counterparty_name,
  counterparty_type
);
```

### 2. Agreement dimension

Notice that this table also contains `counterparty_code`, allowing a direct dimension-to-dimension join.

```sql
CREATE OR REPLACE TABLE
  `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt506_dim_agreement
USING DELTA
AS
SELECT *
FROM VALUES
  ('AG001', 'Apex Loan Agreement',       'CP001', 'LOAN'),
  ('AG002', 'Apex Derivative Agreement', 'CP001', 'DERIVATIVE'),
  ('AG003', 'Beacon Credit Agreement',   'CP002', 'CREDIT'),
  ('AG004', 'Crest Insurance Agreement', 'CP003', 'INSURANCE'),
  ('AG005', 'Delta Pension Agreement',   'CP004', 'PENSION')
AS t(
  agreement_key,
  agreement_name,
  counterparty_code,
  agreement_type
);
```

### 3. Fact table

```sql
CREATE OR REPLACE TABLE
  `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt506_fact_trade
USING DELTA
AS
SELECT *
FROM VALUES
  ('TR001', 'CP001', 'AG001', 1000.00),
  ('TR002', 'CP001', 'AG001', 1500.00),
  ('TR003', 'CP001', 'AG002', 2000.00),
  ('TR004', 'CP002', 'AG003', 2500.00),
  ('TR005', 'CP003', 'AG004', 3000.00),
  ('TR006', 'CP004', 'AG005', 3500.00)
AS t(
  trade_key,
  fact_counterparty_code,
  fact_agreement_key,
  trade_amount
);
```

## Validate the direct dimension join

```sql
SELECT
  c.counterparty_name,
  a.agreement_name
FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt506_dim_counterparty c
JOIN `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt506_dim_agreement a
  ON c.counterparty_code = a.counterparty_code
ORDER BY c.counterparty_name, a.agreement_name;
```

Expected result: five valid counterparty-agreement combinations.

## Mosaic test

Create a new Mosaic model named:

**MSTRT-506 – Dimension Relationship Validation**

Import only these three tables and let Mosaic generate its initial relationships.

Before accepting suggestions, inspect whether Mosaic creates:

```text
Counterparty → Agreement
```

or only:

```text
Counterparty → Trade ← Agreement
```

Then create a dashboard named:

**MSTRT-506 – Counterparty Agreement Validation**

Create two grids:

### Grid 1: No fact object

- Counterparty Name
    
- Agreement Name
    

This is the critical test. If Mosaic produces a Cartesian-product warning, MSTRT-506 is reproduced.

### Grid 2: With fact metric

- Counterparty Name
    
- Agreement Name
    
- Distinct Count of Trade Key
    

If this grid works while Grid 1 fails, it confirms that Mosaic only understands the relationship through the fact table.

Do not manually create the direct Counterparty-to-Agreement relationship until both results and Query Details have been captured.