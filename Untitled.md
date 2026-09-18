Use these **three dedicated MSTRT-510 tables**. They give you a Counterparty → Agreement relationship to change in the model, plus a trade table for a dashboard metric. Run each statement in a separate Databricks SQL cell.

```sql
CREATE OR REPLACE TABLE `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt510_dim_counterparty
USING DELTA AS
SELECT *
FROM VALUES
  ('CP001', 'Apex Bank',       'BANK'),
  ('CP002', 'Beacon Bank',     'BANK'),
  ('CP003', 'Crest Insurance', 'INSURANCE'),
  ('CP004', 'Delta Pension',   'PENSION')
AS v(counterparty_code, counterparty_name, counterparty_type);
```

```sql
CREATE OR REPLACE TABLE `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt510_dim_agreement
USING DELTA AS
SELECT *
FROM VALUES
  ('AG001', 'Apex Loan Agreement',       'CP001'),
  ('AG002', 'Apex Derivative Agreement', 'CP001'),
  ('AG003', 'Beacon Credit Agreement',   'CP002'),
  ('AG004', 'Crest Insurance Agreement', 'CP003'),
  ('AG005', 'Delta Pension Agreement',   'CP004'),
  ('AG006', 'Apex Future Agreement',     'CP001')
AS v(agreement_key, agreement_name, counterparty_code);
```

```sql
CREATE OR REPLACE TABLE `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt510_fact_trade
USING DELTA AS
SELECT *
FROM VALUES
  ('TR001', 'AG001', DATE '2026-01-05', 1000.00),
  ('TR002', 'AG001', DATE '2026-01-06', 1500.00),
  ('TR003', 'AG002', DATE '2026-01-07', 2000.00),
  ('TR004', 'AG003', DATE '2026-01-08', 2500.00),
  ('TR005', 'AG004', DATE '2026-01-09', 3000.00),
  ('TR006', 'AG005', DATE '2026-01-10', 3500.00)
AS v(trade_key, agreement_key, trade_date, trade_amount);
```

For **Version A**, the intended links are:

|Parent table|Child table|Matching columns|
|---|---|---|
|`mstrt510_dim_counterparty`|`mstrt510_dim_agreement`|`counterparty_code`|
|`mstrt510_dim_agreement`|`mstrt510_fact_trade`|`agreement_key`|

**AG006 belongs to Apex Bank but has no trade.** It gives us an easy row to watch when a relationship changes: a Counterparty + Agreement grid should show all **6 agreements** when it uses the direct dimension relationship.

The tables make the test controlled, but they do not guarantee that a particular model edit will break an existing dashboard. First publish Version A and save the working results; then we can choose one relationship change for Version B and measure what Revert actually restores.