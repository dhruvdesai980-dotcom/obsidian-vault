Yes. This will produce a clearer business impact: a valid agreement will disappear because Strategy uses the fact table as the bridge.

## 1. Add an agreement with no trade

Run this in Databricks:

```sql
MERGE INTO
  `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt506_dim_agreement AS target
USING (
  SELECT
    'AG006' AS agreement_key,
    'Apex Future Agreement' AS agreement_name,
    'CP001' AS counterparty_code,
    'FUTURE' AS agreement_type
) AS source
ON target.agreement_key = source.agreement_key

WHEN NOT MATCHED THEN
  INSERT (
    agreement_key,
    agreement_name,
    counterparty_code,
    agreement_type
  )
  VALUES (
    source.agreement_key,
    source.agreement_name,
    source.counterparty_code,
    source.agreement_type
  );
```

Do not add `AG006` to the fact table.

## 2. Verify the source data

Run the direct dimension join:

```sql
SELECT
  c.counterparty_code,
  c.counterparty_name,
  a.agreement_key,
  a.agreement_name
FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt506_dim_counterparty c
JOIN `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt506_dim_agreement a
  ON c.counterparty_code = a.counterparty_code
ORDER BY c.counterparty_code, a.agreement_key;
```

The correct result should now contain six agreements, including:

|Counterparty|Agreement|
|---|---|
|Apex Bank|Apex Future Agreement|

Verify that the fact table has no `AG006`:

```sql
SELECT *
FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt506_fact_trade
WHERE fact_agreement_key = 'AG006';
```

Expected result: zero rows.

## 3. Refresh Strategy

1. Refresh the Mosaic model’s source data or schema.
    
2. Republish the model if required.
    
3. Refresh the dashboard dataset.
    
4. Do not manually create the direct relationship yet.
    

## 4. Check Visualization 1

The left grid still contains only:

- Counterparty
    
- Agreement
    
- No metric
    

There are two possible outcomes:

### Expected with a direct relationship

The grid displays six agreements, including:

```text
Apex Bank | AG006 | Apex Future Agreement
```

### Expected with the current fact-mediated relationship

The grid displays only the original five agreements.

`AG006` will be missing because Strategy searches for a Trade Key connecting Apex Bank to AG006, but no such trade exists.

## 5. Capture Query Details again

If `AG006` is absent and Query Details still shows:

```text
REL_COUNTERPARTY_TRADE_KEY
REL_AGREEMENT_TRADE_KEY
```

then we have reproduced a visible failure:

> A valid Counterparty–Agreement relationship is silently excluded because Mosaic uses the Trade fact as the relationship bridge instead of the direct dimension key.

That would strengthen the status from “core behavior reproduced” to **“missing-row impact reproduced.”**

## 6. Prove the manual fix

After capturing the failure:

1. Manually declare `Counterparty → Agreement` as one-to-many.
    
2. Publish and refresh.
    
3. Reopen Visualization 1.
    
4. Confirm that `Apex Future Agreement` appears.
    
5. Check that Query Details no longer relies on Trade Key to connect the two attributes.
    

That gives a clean before-and-after demonstration:

|Model behavior|Agreements displayed|
|---|--:|
|Fact-mediated path|5|
|Direct dimension relationship|6|