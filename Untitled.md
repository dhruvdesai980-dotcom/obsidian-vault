The query did not run because this terminal is using the system Python, where Airflow is not installed:

```
ModuleNotFoundError: No module named 'airflow.providers'
```

Notice that `(.venv)` is missing from your prompt.

Run these commands first:

```
cd ~/airflow-lab
source .venv/bin/activate

export AIRFLOW_HOME="$HOME/airflow"
export LD_LIBRARY_PATH="$HOME/oracle/instantclient_23_26:$LD_LIBRARY_PATH"

which python
python -c "import airflow; print(airflow.__version__)"
```

You should see:

```
/home/wsl2user/airflow-lab/.venv/bin/python
3.3.2
```

Then run the column query again:

```
python - <<'PY'
from airflow.providers.oracle.hooks.oracle import OracleHook

hook = OracleHook(oracle_conn_id="oracle_apr_conn")

rows = hook.get_records(
    """
    SELECT column_id, column_name, data_type
    FROM all_tab_columns
    WHERE owner = 'COL'
      AND table_name = 'L_INTERNAL_COUNTERPARTY_AMEND'
    ORDER BY column_id
    """
)

for column_id, column_name, data_type in rows:
    print(column_id, column_name, data_type)
PY
```

The key issue was simply that you opened a new terminal and the Airflow virtual environment was no longer active. Virtual-environment activation applies separately to each terminal window.