This is a new test: **MSTRT-496 — Filter widget does not warn when the model relationship is missing.**

The goal is to determine whether Strategy allows a filter from an unrelated table, even though that filter cannot affect the visualization.

### 1. Create two unrelated tables

Run this in Databricks:

```sql
CREATE OR REPLACE TABLE raw.mstrt496_counterparty
USING DELTA AS
SELECT *
FROM VALUES
  ('CP001', 'Apex Bank'),
  ('CP002', 'Beacon Bank'),
  ('CP003', 'Crest Insurance'),
  ('CP004', 'Delta Pension')
AS t(counterparty_code, counterparty_name);
```

```sql
CREATE OR REPLACE TABLE raw.mstrt496_td_entity
USING DELTA AS
SELECT *
FROM VALUES
  ('TD001', 'TD Securities Canada'),
  ('TD002', 'TD Securities USA'),
  ('TD003', 'TD Securities UK')
AS t(td_entity_code, td_entity_name);
```

These tables deliberately have **no common key and no relationship**.

### 2. Create the Mosaic model

Create a new model named:

```text
MSTRT-496 Missing Filter Relationship
```

Add both tables.

Create these attributes:

|Table|Attribute|Forms|
|---|---|---|
|`mstrt496_counterparty`|Counterparty|Code and Name|
|`mstrt496_td_entity`|TD Entity|Code and Name|

In the hierarchy/model view, leave them disconnected:

```text
Counterparty                 TD Entity
    ●                            ●
    No relationship between them
```

Do not create a bridge, relationship, or merged attribute. Validate and publish the model.

### 3. Create the dashboard

Create a dashboard using this model.

Add a grid visualization and place **Counterparty** in Rows. It should show four counterparties:

- Apex Bank
    
- Beacon Bank
    
- Crest Insurance
    
- Delta Pension
    

### 4. Add the unrelated filter

Add a filter widget using **TD Entity** or **TD Entity Name**.

Check whether the filter:

- Is created successfully
    
- Displays all three TD Entity values
    
- Allows you to select one value
    
- Shows no warning about the missing relationship
    

Select only:

```text
TD Securities Canada
```

### 5. Observe the grid

The important result is whether the grid remains unchanged and continues showing all four counterparties.

If that happens with no warning, the issue is reproduced:

- The filter looks valid.
    
- Users can select values.
    
- The selection has no effect on the target grid.
    
- The dashboard does not explain that TD Entity and Counterparty are disconnected.
    

Also open **Query Details** for the grid. The query should access the Counterparty table but not the TD Entity table.

### Expected versus actual

|Behaviour|Expected|Reported problem|
|---|---|---|
|Add unrelated filter|Warning or prevent assignment|Filter is accepted|
|Select filter value|Filter target grid or explain incompatibility|Grid remains unchanged|
|User feedback|Relationship/path warning|No warning|
|Query|Valid relationship path required|Filter attribute is absent from visualization query|

The issue can be marked **reproduced** only if the TD Entity selection does nothing and Strategy gives no clear warning.