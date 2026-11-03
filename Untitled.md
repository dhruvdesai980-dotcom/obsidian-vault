```sql

-- Team A: Equity Positions
SELECT
    CONCAT('EQ-', CAST(posID AS STRING)) AS cdo_position_key,
    'team_a' AS source_team,
    CAST(posID AS BIGINT) AS position_id,
    CAST(rptDate AS DATE) AS report_date,
    CAST(pnl AS DECIMAL(18,2)) AS profit_loss,
    CAST(cash AS DECIMAL(18,2)) AS settled_cash,
    CAST(NULL AS DECIMAL(18,2)) AS unsettled_cash,
    CAST(instrumentID AS BIGINT) AS instrument_id

FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.`raw`.`eq_positional`

UNION ALL

-- Team C: Fixed Income Positions
SELECT
    CONCAT('FI-', CAST(posIdentity AS STRING)) AS cdo_position_key,
    'team_c' AS source_team,
    CAST(posIdentity AS BIGINT) AS position_id,
    CAST(rptDate AS DATE) AS report_date,
    CAST(profitloss AS DECIMAL(18,2)) AS profit_loss,
    CAST(settledCash AS DECIMAL(18,2)) AS settled_cash,
    CAST(NULL AS DECIMAL(18,2)) AS unsettled_cash,
    CAST(insCode AS BIGINT) AS instrument_id

FROM `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.`raw`.`fi_positional`

```