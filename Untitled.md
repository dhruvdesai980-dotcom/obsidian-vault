This is good progress: the first CSV completed successfully.

From the task states:

- `load_l_internal_counterparty` — succeeded
- `archive_internal_counterparty` — succeeded
- `delete_s_internal_counterparty` — succeeded
- `load_s_internal_counterparty` — succeeded
- `truncate_l_internal_counterparty_amend` — succeeded
- `load_l_internal_counterparty_amend` — failed

The new error is:

```
ORA-00904: "MAPPED": invalid identifier
```

The second CSV contains:

```
SYSTEM_NAME,CP_CODE,RESPONSIBILITY_CENTER,COMPANY,TD_SITE,TD_ENTITY,MAPPED
```

Airflow generated an Oracle insert containing `MAPPED`, but `COL.L_INTERNAL_COUNTERPARTY_AMEND` apparently does not have a column with that name.

Check the actual Oracle columns before changing the code:

```
cd ~/airflow-lab
source .venv/bin/activate
export AIRFLOW_HOME="$HOME/airflow"

python - <<'PY'
from airflow.providers.oracle.hooks.oracle import OracleHook

hook = OracleHook(oracle_conn_id="oracle_apr_conn")

rows = hook.get_records("""
    SELECT column_id, column_name, data_type
    FROM all_tab_columns
    WHERE owner = 'COL'
      AND table_name = 'L_INTERNAL_COUNTERPARTY_AMEND'
    ORDER BY column_id
""")

for column_id, column_name, data_type in rows:
    print(column_id, column_name, data_type)
PY
```

Compare that output with the CSV header. Most likely, one of these is true:

- `MAPPED` is not loaded by the original SSIS package.
- `MAPPED` maps to a differently named Oracle column.
- The Oracle table definition changed after the CSV was produced.
- The SSIS data flow calculated or discarded that field.

Do not simply remove `MAPPED` until we confirm the original SSIS column mapping.

Also, do not start an entirely new run right now:

- The first CSV has already been moved into the archive directory.
- The second loading table has already been truncated.
- A new full run may fail its file check because the first CSV is no longer in `incoming`.

After correcting the mapping, rerun/clear only:

```
load_l_internal_counterparty_amend
```

and its downstream tasks. Send the Oracle column output next, and we can make the exact source-to-target mapping.