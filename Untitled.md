Let’s create **two tables dedicated to MSTRT-505** in the same Databricks catalog and `raw` schema. Run each statement in a separate SQL cell.

**1. Agreement dimension — 12 agreements**

```sql
CREATE OR REPLACE TABLE `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt505_dim_agreement
USING DELTA
AS
SELECT *
FROM VALUES
  ('AG001', 'Agreement 01'),
  ('AG002', 'Agreement 02'),
  ('AG003', 'Agreement 03'),
  ('AG004', 'Agreement 04'),
  ('AG005', 'Agreement 05'),
  ('AG006', 'Agreement 06'),
  ('AG007', 'Agreement 07'),
  ('AG008', 'Agreement 08'),
  ('AG009', 'Agreement 09'),
  ('AG010', 'Agreement 10'),
  ('AG011', 'Agreement 11'),
  ('AG012', 'Agreement 12')
AS v(agreement_key, agreement_name);
```

**2. Trade fact — trades for only five agreements**

```sql
CREATE OR REPLACE TABLE `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt505_fact_trade
USING DELTA
AS
SELECT *
FROM VALUES
  ('TR001', 'AG001', DATE '2026-01-05'),
  ('TR002', 'AG002', DATE '2026-01-06'),
  ('TR003', 'AG003', DATE '2026-01-07'),
  ('TR004', 'AG004', DATE '2026-01-08'),
  ('TR005', 'AG005', DATE '2026-01-09')
AS v(trade_key, agreement_key, trade_date);
```

The model should relate the tables through `agreement_key`. Before building it, verify the test data:

```sql
SELECT
  (SELECT COUNT(*) FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt505_dim_agreement)
    AS all_agreements,
  (SELECT COUNT(DISTINCT agreement_key)
   FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt505_fact_trade)
    AS agreements_with_trades;
```

Expected result: **12** and **5**. When you add them to Mosaic, use these two `mstrt505_` tables only.