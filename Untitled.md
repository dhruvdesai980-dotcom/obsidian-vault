Yes. For the controlled **MSTRT-497** dataset, the dashboard uses:

```text
Rating → Counterparty → Agreement → Margin Type → Status → Action
```

Metric:

```sql
SUM(margin_amount)
```

Table:

```sql
`d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt497_multilevel_totals
```

No joins are required for this controlled test.

## 1. Validate the dashboard grand total

```sql
SELECT
    SUM(margin_amount) AS total_margin_amount
FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt497_multilevel_totals;
```

Expected result:

|total_margin_amount|
|--:|
|3,600|

This should match the top **Total = 3,600** shown in the dashboard.

---

## 2. Validate totals by rating

```sql
SELECT
    rating,
    SUM(margin_amount) AS total_margin_amount
FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt497_multilevel_totals
GROUP BY rating
ORDER BY rating;
```

Expected result:

|rating|total_margin_amount|
|---|--:|
|AA|2,600|
|AAA|1,000|

---

## 3. Validate totals by counterparty

```sql
SELECT
    rating,
    counterparty,
    SUM(margin_amount) AS total_margin_amount
FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt497_multilevel_totals
GROUP BY
    rating,
    counterparty
ORDER BY
    rating,
    counterparty;
```

Expected result:

|Rating|Counterparty|Total|
|---|---|--:|
|AA|Crest Bank|1,100|
|AA|Delta Bank|1,500|
|AAA|Apex Bank|300|
|AAA|Beacon Bank|700|

---

## 4. Validate totals by agreement

```sql
SELECT
    rating,
    counterparty,
    agreement,
    SUM(margin_amount) AS total_margin_amount
FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt497_multilevel_totals
GROUP BY
    rating,
    counterparty,
    agreement
ORDER BY
    rating,
    counterparty,
    agreement;
```

Expected result:

|Counterparty|Agreement|Total|
|---|---|--:|
|Crest Bank|AG003|1,100|
|Delta Bank|AG004|1,500|
|Apex Bank|AG001|300|
|Beacon Bank|AG002|700|

---

## 5. Validate margin-type totals

```sql
SELECT
    rating,
    counterparty,
    agreement,
    margin_type,
    SUM(margin_amount) AS total_margin_amount
FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt497_multilevel_totals
GROUP BY
    rating,
    counterparty,
    agreement,
    margin_type
ORDER BY
    rating,
    counterparty,
    agreement,
    margin_type;
```

Expected result:

|Counterparty|Agreement|Margin type|Total|
|---|---|---|--:|
|Crest Bank|AG003|INITIAL|500|
|Crest Bank|AG003|VARIATION|600|
|Delta Bank|AG004|INITIAL|700|
|Delta Bank|AG004|VARIATION|800|
|Apex Bank|AG001|INITIAL|100|
|Apex Bank|AG001|VARIATION|200|
|Beacon Bank|AG002|INITIAL|300|
|Beacon Bank|AG002|VARIATION|400|

---

## 6. Validate the lowest dashboard level

This query reproduces the values at the complete dashboard grain:

```sql
SELECT
    rating,
    counterparty,
    agreement,
    margin_type,
    status,
    action,
    SUM(margin_amount) AS margin_amount
FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt497_multilevel_totals
GROUP BY
    rating,
    counterparty,
    agreement,
    margin_type,
    status,
    action
ORDER BY
    rating,
    counterparty,
    agreement,
    margin_type,
    status,
    action;
```

If every combination represents one source row, this should return the eight leaf-level records used by the dashboard.

---

## 7. Reproduce all dashboard subtotals in one query

`ROLLUP` calculates totals at every level of the hierarchy:

```sql
SELECT
    rating,
    counterparty,
    agreement,
    margin_type,
    status,
    action,
    SUM(margin_amount) AS margin_amount
FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt497_multilevel_totals
GROUP BY ROLLUP (
    rating,
    counterparty,
    agreement,
    margin_type,
    status,
    action
)
ORDER BY
    rating NULLS FIRST,
    counterparty NULLS FIRST,
    agreement NULLS FIRST,
    margin_type NULLS FIRST,
    status NULLS FIRST,
    action NULLS FIRST;
```

In this result:

- All hierarchy columns `NULL` = grand total
    
- Only `rating` populated = rating subtotal
    
- `rating` and `counterparty` populated = counterparty subtotal
    
- More populated columns = increasingly detailed subtotals
    
- All columns populated = leaf-level value
    

---

## 8. Check whether duplicate rows are inflating the total

```sql
SELECT
    rating,
    counterparty,
    agreement,
    margin_type,
    status,
    action,
    COUNT(*) AS source_row_count,
    SUM(margin_amount) AS margin_amount
FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt497_multilevel_totals
GROUP BY
    rating,
    counterparty,
    agreement,
    margin_type,
    status,
    action
HAVING COUNT(*) > 1
ORDER BY source_row_count DESC;
```

Expected result for the controlled dataset:

```text
No rows
```

If this returns records, multiple physical rows exist at the same dashboard grain. That could explain unexpectedly high totals.