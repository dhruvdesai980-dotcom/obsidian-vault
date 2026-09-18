Let’s test **MSTRT-499** with one small table and **one Counterparty attribute that has multiple forms**. The important distinction is that `counterparty_code`, `counterparty_name`, and `legal_entity_id` must be forms of **the same attribute**, rather than three separate attributes on the grid.

### 1. Create the test table in Databricks

Run this in a SQL notebook, using the same catalog and `raw` schema as your earlier tests:

```sql
CREATE OR REPLACE TABLE
  `d4001-centralus-tdvip-tdsbi_mstrt_catalog`.raw.mstrt499_counterparty
USING DELTA
AS
SELECT *
FROM VALUES
  ('CP001', 'Apex Bank',       'LEI-APEX-001'),
  ('CP002', 'Beacon Bank',     'LEI-BEACON-002'),
  ('CP003', 'Crest Insurance', 'LEI-CREST-003')
AS t(counterparty_code, counterparty_name, legal_entity_id);
```

Each code has exactly one name and one legal entity ID, making the forms easy to check.

### 2. Create a separate Mosaic model

Add **only** `mstrt499_counterparty` to a new model named **MSTRT-499 Multi-Form Test**. In **Prep and Model → Tables**, create a **Counterparty** attribute using `counterparty_code` as its ID. In that attribute’s setup, add `counterparty_name` as its description form and `legal_entity_id` as another form.

Before moving on, check that the model lists **one Counterparty attribute** with all three forms. If it lists three independent attributes, the test is not set up yet.

### 3. Create the dashboard

Publish the model and create a dashboard named **MSTRT-499 Attribute Forms Test**. Add a grid, then put **Counterparty** in **Rows** once.

Record whether the grid shows the code, name, and legal entity ID as separate subcolumns. Strategy documents controls for selecting which forms display in an individual visualization. [Strategy: Select Which Attribute Forms to Display](https://www2.strategy.com/producthelp/current/MSTRWeb/webhelp/lang_1033/content/Selecting_which_attribute_forms_to_display_in_a_vi.htm)

### 4. Test the display control

In the grid’s **Editor** panel, right-click **Counterparty** in Rows and select **Display Attribute Forms**. Clear the checkbox for `legal_entity_id`, apply, and confirm that its subcolumn disappears. Save and reopen the dashboard to check that the choice persists. Then add Counterparty to a second grid and see which forms it shows initially. [Strategy’s documented steps](https://www2.strategy.com/producthelp/current/MSTRWeb/webhelp/lang_1033/content/Selecting_which_attribute_forms_to_display_in_a_vi.htm)

**First checkpoint:** create the table and show me the Mosaic attribute setup screen. The exact form-setting controls depend on what your editor presents, so we can configure that part from your screen.