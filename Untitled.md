Yes—this screenshot confirms the problem is exactly line 36:

```
OracleHook(    oracle_conn_id=ORACLE_CONN_ID).bulk_load(table, str(source_path))
```

Replace lines 32–36 with the following.

First, add this near the other imports:

```
import csv
```

Then use:

```
def _bulk_load(table: str, source: str) -> None:    source_path = Path(source)    if not source_path.exists():        raise AirflowException(            f"Input file does not exist: {source_path}"        )    hook = OracleHook(oracle_conn_id=ORACLE_CONN_ID)    with source_path.open(        mode="r",        encoding="utf-8-sig",        newline="",    ) as csv_file:        reader = csv.reader(            csv_file,            delimiter=",",            quotechar='"',        )        # Uncomment only if the CSV has a header row:        # next(reader, None)        rows = (            tuple(None if value == "" else value for value in row)            for row in reader        )        hook.bulk_insert_rows(            table=table,            rows=rows,            commit_every=5000,        )
```

This changes only the unsupported loading mechanism:

```
bulk_load()          → unsupported for OracleHook
bulk_insert_rows()   → supported Oracle batch insertion
```

Before running it, confirm:

- The files are comma-separated.
- They use UTF-8 encoding.
- Whether the first line is a header.
- CSV column order matches the Oracle table column order.

To inspect only the beginning of the first file:

```
head -n 2 ~/airflow/data/incoming/internal_counterparty20251021.csv
```

Do not share those lines if they contain sensitive business data. You only need to determine whether the first line contains column names.

After saving the DAG:

```
cd ~/airflow-lab
source .venv/bin/activate
export AIRFLOW_HOME="$HOME/airflow"

python -m py_compile ~/airflow/dags/apr_late_load.py
airflow dags list-import-errors
```

Airflow should detect the saved file automatically. Confirm that no import errors appear and that the UI shows a newer DAG version.

Do not rerun it yet if this is a production database: the earlier successful `TRUNCATE` has likely already emptied `COL.L_INTERNAL_COUNTERPARTY`. First verify whether this is an expendable loading/staging table and confirm the CSV structure.