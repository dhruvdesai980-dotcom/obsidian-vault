I can see your `i_gstm_load.py` code. The problem is in the `load_i_gstm_detail` task, specifically this SQL:

```
_hook().run(    """    insert into COL.I_GSTM_DETAIL    select *    from COL.MASTER_DATA_I_GSTM_VW    where bus_process_date = :bus_process_date    """,    parameters={"bus_process_date": date_ctx["bus_process_date"]},)
```

Your earlier error was:

```
ORA-00947: not enough values
```

This means Oracle expects more values for the destination table than the `SELECT *` is supplying.

### What we need to fix

Before changing the Python code, let's establish the actual source-to-target mapping. Since this is an SSIS-to-Airflow migration, we should preserve the original SSIS transformation logic rather than guess column mappings.

Run this query in Oracle to compare the columns in both objects:

```
SELECT
    CASE
        WHEN table_name = 'I_GSTM_DETAIL' THEN 'TARGET'
        ELSE 'SOURCE'
    END AS object_type,
    table_name,
    column_id,
    column_name,
    data_type
FROM all_tab_columns
WHERE owner = 'COL'
  AND table_name IN (
      'I_GSTM_DETAIL',
      'MASTER_DATA_I_GSTM_VW'
  )
ORDER BY object_type, column_id;
```

This will show whether the source view has fewer columns, whether the column order differs, or whether additional destination columns need values.

### One additional observation

Your two tasks:

```
update_sql_bbg_src()update_sql_master_data_i_bbg_vw()
```

currently return SQL strings using `.replace()`. They do not execute those SQL statements. That is fine if their purpose is simply to generate SQL for downstream tasks, but we should verify that the generated SQL is actually consumed later in the DAG.

This is important because `load_i_gstm_detail` reads directly from `COL.MASTER_DATA_I_GSTM_VW`.

Next step: Share the output of the column-comparison query. I'll then give you the exact replacement for lines 183–193, including the required column mappings, without changing the original SSIS business logic.