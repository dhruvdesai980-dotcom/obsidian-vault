SELECT
  COUNT(*) AS fact_rows,
  COUNT(DISTINCT f.trade_key) AS distinct_trades,
  COUNT(DISTINCT d.counterparty_key) AS counterparties,
  COUNT_IF(d.counterparty_key IS NULL) AS unmatched_rows
FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt507_fact_trade f
LEFT JOIN `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt507_dim_counterparty d
  ON f.fact_counterparty_key = d.counterparty_key;