```sql
-- Run within your selected Databricks catalog
CREATE SCHEMA IF NOT EXISTS cdo_mosaic_poc;

USE SCHEMA cdo_mosaic_poc;

CREATE OR REPLACE TABLE team_a_positions (
    position_id STRING,
    report_date DATE,
    instrument_code STRING,
    profit_loss DECIMAL(18,2),

    -- Team A specific attributes
    trader_name STRING,
    desk_code STRING
)
USING DELTA;

INSERT INTO team_a_positions
(
    position_id,
    report_date,
    instrument_code,
    profit_loss,
    trader_name,
    desk_code
)
VALUES
(
    'A101',
    DATE '2026-10-30',
    'IBM',
    100.00,
    'John',
    'Equity'
),
(
    'A102',
    DATE '2026-10-30',
    'MSFT',
    50.00,
    'Sarah',
    'Technology'
);

CREATE OR REPLACE TABLE team_c_positions (
    position_ref STRING,
    business_date DATE,
    instrument_id STRING,
    net_pnl DECIMAL(18,2),

    -- Team C specific attributes
    portfolio_name STRING,
    risk_category STRING
)
USING DELTA;

INSERT INTO team_c_positions
(
    position_ref,
    business_date,
    instrument_id,
    net_pnl,
    portfolio_name,
    risk_category
)
VALUES
(
    'C201',
    DATE '2026-10-30',
    'IBM',
    200.00,
    'Portfolio Alpha',
    'Low'
),
(
    'C202',
    DATE '2026-10-30',
    'AAPL',
    75.00,
    'Portfolio Beta',
    'Medium'
);
```