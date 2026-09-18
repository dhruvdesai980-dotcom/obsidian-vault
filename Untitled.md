Let’s run a stronger MSTRT-510 test. We can reuse your `mstrt510_dim_counterparty` and `mstrt510_fact_trade` tables, but create an **Agreement table without `counterparty_code`** and a separate bridge. This removes the alternate path that kept Page 1 working.

Run these in Databricks:

```sql
CREATE OR REPLACE TABLE `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt510b_dim_agreement
USING DELTA AS
SELECT agreement_key, agreement_name
FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt510_dim_agreement;
```

```sql
CREATE OR REPLACE TABLE `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt510b_agreement_counterparty
USING DELTA AS
SELECT agreement_key, counterparty_code
FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt510_dim_agreement;
```

**Version A**

1. Create a **new Mosaic model** using those two `mstrt510b_` tables plus the existing `mstrt510_dim_counterparty` and `mstrt510_fact_trade`. Leave the current model and dashboard as evidence of our first test.
    
2. Check the model’s table mappings: Counterparty connects to the bridge by `counterparty_code`; the bridge connects to Agreement by `agreement_key`; Agreement connects to Trade by `agreement_key`.
    
3. Publish it. Create a dashboard grid with **Counterparty + Agreement** and confirm it shows six agreements, including **AG006 under Apex Bank**. Save the dashboard and its Query Details.
    

**Version B**

4. In the model, remove **only the bridge connection between Counterparty and Agreement**, then publish. Check the saved grid after a fresh reopen. We need an actual error, missing row, or changed pairing before proceeding.
    
5. Create an **Agreement + Trade Date + Trade Amount** page and confirm it works.
    
6. Use Revert to restore Version A’s bridge connection, **publish the restored model**, and reopen both pages.
    

The decisive observation is what happens at step 6: if the old grid remains broken despite the restored mapping and a fresh query, MSTRT-510 is reproduced. If it recovers, this controlled test does not reproduce the revert problem.