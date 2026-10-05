The screenshot confirms:

```
SYSTEM,BOOK
"ATLS","TD_AMS_CTNV"
```

Therefore:

- The file is comma-separated.
- Double quotes are used around values.
- The first row is a header.
- This file contains two columns: `SYSTEM` and `BOOK`.

Use the following `_bulk_load()` implementation. It reads the header and passes it to Oracle as the target-column list.

```
def _bulk_load(table: str, source: str) -> None:    source_path = Path(source)    if not source_path.exists():        raise AirflowException(            f"Input file does not exist: {source_path}"        )    hook = OracleHook(oracle_conn_id=ORACLE_CONN_ID)    with source_path.open(        mode="r",        encoding="utf-8-sig",        newline="",    ) as csv_file:        reader = csv.reader(            csv_file,            delimiter=",",            quotechar='"',        )        # The first row contains Oracle column names.        target_fields = next(reader, None)        if not target_fields:            raise AirflowException(                f"Input file is empty: {source_path}"            )        target_fields = [            column.strip() for column in target_fields        ]        def prepared_rows():            for line_number, row in enumerate(reader, start=2):                if not row:                    continue                if len(row) != len(target_fields):                    raise AirflowException(                        f"Column-count mismatch in {source_path} "                        f"at line {line_number}: expected "                        f"{len(target_fields)}, found {len(row)}"                    )                yield tuple(                    None if value == "" else value                    for value in row                )        hook.bulk_insert_rows(            table=table,            rows=prepared_rows(),            target_fields=target_fields,            commit_every=5000,        )
```

Also ensure this is included at the top:

```
import csv
```

Check the headers of the other two files:

```
head -n 1 ~/airflow/data/incoming/inter_affiliate_xref20251021.csv
head -n 1 ~/airflow/data/incoming/repo_client_mapping20251021.csv
```

The header names must exist as columns in their corresponding Oracle loading tables.

After saving:

```
cd ~/airflow-lab
source .venv/bin/activate
export AIRFLOW_HOME="$HOME/airflow"

python -m py_compile ~/airflow/dags/apr_late_load.py
airflow dags list-import-errors
```

If both commands finish without errors, confirm Airflow shows a new DAG version. Before triggering it, verify this is a safe staging environment because the first loading table was already truncated during the failed run.