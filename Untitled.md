Your virtual environment is correct now, but the query failed for a different reason:

```
AirflowNotFoundException:
The conn_id `oracle_apr_conn` isn't defined
```

In this standalone Python execution, Airflow 3’s task SDK cannot retrieve the connection context available to running Airflow tasks. The connection still exists in the UI—your DAG already used it successfully.

For this diagnostic query, connect directly with `python-oracledb`. The password will be requested securely and will not appear on screen or in command history.

```
cd ~/airflow-lab
source .venv/bin/activate

export LD_LIBRARY_PATH="$HOME/oracle/instantclient_23_26:$LD_LIBRARY_PATH"

python - <<'PY'
import getpass
import os
import oracledb

oracledb.init_oracle_client(
    lib_dir=os.path.expanduser(
        "~/oracle/instantclient_23_26"
    )
)

password = getpass.getpass("Oracle password: ")

dsn = oracledb.makedsn(
    "baocacdbv3-scan.ocac2.dev.tdbank.ca",
    1521,
    service_name="bis_dev_tdsrv.bis_dev.tdbank.ca",
)

with oracledb.connect(
    user="col",
    password=password,
    dsn=dsn,
) as connection:
    with connection.cursor() as cursor:
        cursor.execute(
            """
            SELECT column_id, column_name, data_type
            FROM all_tab_columns
            WHERE owner = 'COL'
              AND table_name = 'L_INTERNAL_COUNTERPARTY_AMEND'
            ORDER BY column_id
            """
        )

        for column_id, column_name, data_type in cursor:
            print(column_id, column_name, data_type)
PY
```

Enter the same Oracle password stored in `oracle_apr_conn` when prompted.

Only share the resulting column names and data types—not the password. That output will tell us what the CSV field `MAPPED` should map to or whether it should be excluded.