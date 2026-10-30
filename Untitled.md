```sql
CREATE SCHEMA IF NOT EXISTS cdo_mosaic_poc;

-- Team A: Equity positions
CREATE OR REPLACE TABLE cdo_mosaic_poc.eq_positional (
    posID INT,
    rptDate DATE,
    pnl DECIMAL(18,2),
    mtm DECIMAL(18,2),
    cash DECIMAL(18,2),
    instrumentID INT
) USING DELTA;

INSERT INTO cdo_mosaic_poc.eq_positional VALUES
(101, DATE '2026-10-30', 100.00, 110.00, 15.00, 10),
(102, DATE '2026-10-30',  50.00,  55.00, 10.00, 20);


-- Team B: Instrument master
CREATE OR REPLACE TABLE cdo_mosaic_poc.instrument (
    code INT,
    cusip STRING,
    isin STRING,
    securityType STRING
) USING DELTA;

INSERT INTO cdo_mosaic_poc.instrument VALUES
(10, '459200101', 'US4592001014', 'Equity'),
(20, '594918104', 'US5949181045', 'Equity'),
(30, '037833100', 'US0378331005', 'Bond');


-- Team C: Fixed income positions
CREATE OR REPLACE TABLE cdo_mosaic_poc.fi_positional (
    posIdentity INT,
    rptDate DATE,
    profitloss DECIMAL(18,2),
    settledCash DECIMAL(18,2),
    insCode INT
) USING DELTA;

INSERT INTO cdo_mosaic_poc.fi_positional VALUES
(201, DATE '2026-10-30', 200.00, 20.00, 10),
(202, DATE '2026-10-30',  75.00,  5.00, 30);
```