Yes. We’ll create two clean Delta tables specifically for MSTRT-507, without PK/FK constraints or catalog relationships.

### 1. Create the controlled dimension table

Run this in Databricks:

```sql
CREATE OR REPLACE TABLE
  `d4001-centralus-tdvip-tdsbi_mstr_catalog`.raw.mstrt507_dim_counterparty
USING DELTA
AS
SELECT *
FROM VALUES
  ('CP001', 'Apex Bank',             'BANK',       'AAA', 'REGIME_A'),
  ('CP002', 'Beacon Bank',           'BANK',       'AAA', 'REGIME_A'),
  ('CP003', 'Crest Bank',            'BANK',       'AA',  'REGIME_A'),
  ('CP004', 'Delta Insurance',       'INSURANCE',  'AA',  'REGIME_A'),
  ('CP005', 'Evergreen Insurance',   'INSURANCE',  'A',   'REGIME_B'),
  ('CP006', 'Frontier Insurance',    'INSURANCE',  'A',   'REGIME_B'),
  ('CP007', 'Global Pension',        'PENSION',    'AAA', 'REGIME_B'),
  ('CP008', 'Horizon Pension',       'PENSION',    'AA',  'REGIME_B'),
  ('CP009', 'Infinity Pension',      'PENSION',    'A',   'REGIME_C'),
  ('CP010', 'Jupiter Broker',        'BROKER',     'BBB', 'REGIME_C'),
  ('CP011', 'Keystone Broker',       'BROKER',     'BBB', 'REGIME_C'),
  ('CP012', 'Liberty Broker',        'BROKER',     'A',   'REGIME_C')
AS t(
  counterparty_key,
  counterparty_name,
  counterparty_type,
  credit_rating,
  regulatory_regime
);
```

The intended functional dependencies are:

```text
counterparty_name → counterparty_type
counterparty_name → credit_rating
counterparty_name → regulatory_regime
```

Each parent value applies to multiple counterparties.

### 2. Create the controlled fact table

```sql
CREATE OR REPLACE TABLE
  `d4001-centralus-tdvip-tdsbi_mstr_catalog`.raw.mstrt507_fact_trade
USING DELTA
AS
SELECT *
FROM VALUES
  ('TR001', 'CP001', DATE '2026-01-01', 1000.00),
  ('TR002', 'CP001', DATE '2026-01-02', 1500.00),
  ('TR003', 'CP002', DATE '2026-01-03', 2000.00),
  ('TR004', 'CP002', DATE '2026-01-04', 2500.00),
  ('TR005', 'CP003', DATE '2026-01-05', 3000.00),
  ('TR006', 'CP003', DATE '2026-01-06', 3500.00),
  ('TR007', 'CP004', DATE '2026-01-07', 4000.00),
  ('TR008', 'CP004', DATE '2026-01-08', 4500.00),
  ('TR009', 'CP005', DATE '2026-01-09', 5000.00),
  ('TR010', 'CP005', DATE '2026-01-10', 5500.00),
  ('TR011', 'CP006', DATE '2026-01-11', 6000.00),
  ('TR012', 'CP006', DATE '2026-01-12', 6500.00),
  ('TR013', 'CP007', DATE '2026-01-13', 7000.00),
  ('TR014', 'CP007', DATE '2026-01-14', 7500.00),
  ('TR015', 'CP008', DATE '2026-01-15', 8000.00),
  ('TR016', 'CP008', DATE '2026-01-16', 8500.00),
  ('TR017', 'CP009', DATE '2026-01-17', 9000.00),
  ('TR018', 'CP009', DATE '2026-01-18', 9500.00),
  ('TR019', 'CP010', DATE '2026-01-19', 10000.00),
  ('TR020', 'CP010', DATE '2026-01-20', 10500.00),
  ('TR021', 'CP011', DATE '2026-01-21', 11000.00),
  ('TR022', 'CP011', DATE '2026-01-22', 11500.00),
  ('TR023', 'CP012', DATE '2026-01-23', 12000.00),
  ('TR024', 'CP012', DATE '2026-01-24', 12500.00)
AS t(
  trade_key,
  fact_counterparty_key,
  trade_date,
  trade_amount
);
```

Using different key-column names helps prevent an automatic same-name relationship from being inherited.

### 3. Validate the test data

```sql
SELECT
  COUNT(*) AS fact_rows,
  COUNT(DISTINCT f.trade_key) AS distinct_trades,
  COUNT(DISTINCT d.counterparty_key) AS counterparties,
  COUNT_IF(d.counterparty_key IS NULL) AS unmatched_rows
FROM `d4001-centralus-tdvip-tdsbi_mstr_catalog`.raw.mstrt507_fact_trade f
LEFT JOIN `d4001-centralus-tdvip-tdsbi_mstr_catalog`.raw.mstrt507_dim_counterparty d
  ON f.fact_counterparty_key = d.counterparty_key;
```

Expected:

|fact_rows|distinct_trades|counterparties|unmatched_rows|
|--:|--:|--:|--:|
|24|24|12|0|

### 4. Build a completely new Mosaic model

Import only these two tables. Do not reuse the previous model and do not copy existing attributes.

Create only the required fact-to-dimension table relationship:

```text
fact_trade.fact_counterparty_key
    =
dim_counterparty.counterparty_key
```

Do not manually create any relationships between:

- Counterparty Type and Counterparty
    
- Credit Rating and Counterparty
    
- Regulatory Regime and Counterparty
    

Also avoid accepting hierarchy suggestions before taking the initial screenshot.

Create these objects:

|Object|Source|
|---|---|
|Counterparty|`counterparty_name`|
|Counterparty Type|`counterparty_type`|
|Credit Rating|`credit_rating`|
|Regulatory Regime|`regulatory_regime`|
|Trade|`trade_key`|
|Trade Count|Distinct count of `trade_key`|

### 5. Build the dashboard test

Create a grid containing only:

- `Counterparty`
    
- `Trade Count`
    

Then create three selectors:

- Counterparty Type
    
- Credit Rating
    
- Regulatory Regime
    

Test each selector separately before combining them.

For example, selecting `BANK` should return:

|Counterparty|Trade Count|
|---|--:|
|Apex Bank|2|
|Beacon Bank|2|
|Crest Bank|2|

### Reproduction criteria

MSTRT-507 is reproduced if either occurs:

- Mosaic does not auto-create the parent-child relationships in the hierarchy view.
    
- A parent selector is accepted but does not filter the Counterparty grid correctly.
    
- Adding a parent attribute changes the row count or grain unexpectedly.
    
- Query Details lacks a valid relationship path from the selected parent to Counterparty.
    

Even if filtering works, failure to auto-declare the obvious hierarchies would still reproduce the issue’s **model-generation portion**, but not its dashboard failure mode.