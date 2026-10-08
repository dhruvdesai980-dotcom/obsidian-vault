Yes — the problem is clear from the screenshots. Your Airflow DAG itself is reaching Oracle correctly; the failure is caused by **how `TABLE_LOAD_CHECK` is being invoked**.

Your current code does this:

```
result = hook.get_first(    "select TABLE_LOAD_CHECK(:table_name, :date_key) from dual",    parameters={        "table_name": table_name,        "date_key": date_ctx["datekey"],    },)
```

The Airflow log shows:

```
ORA-14552: cannot perform a DDL, commit or rollback inside a query or DML
ORA-06512: at "COL.TABLE_LOAD_CHECK", line 8
```

### What's happening

`get_first()` turns your call into a normal SQL query:

```
SELECT COL.TABLE_LOAD_CHECK(...) FROM dual;
```

Oracle therefore treats `TABLE_LOAD_CHECK` as a function being executed **inside a SQL statement**.

But somewhere around **line 8 of `COL.TABLE_LOAD_CHECK`**, the function apparently performs something like:

```
COMMIT;
ROLLBACK;
-- or DDL
TRUNCATE ...
CREATE ...
ALTER ...
```

Oracle does not permit that when a function is invoked from a `SELECT`.

So this is not an Airflow connection problem and not a bind-variable problem. The function is actually being reached successfully. The execution context is the problem.

## Change `check_row_count`

Instead of calling the function through:

```
SELECT ... FROM dual
```

call the Oracle function directly through the cursor.

Replace your current task with:

```
@task(task_id="check_row_count")def check_row_count(table_names: list, date_ctx: Dict) -> None:    hook = _hook()    conn = hook.get_conn()    cursor = conn.cursor()    try:        for table_name in table_names:            result = cursor.callfunc(                "COL.TABLE_LOAD_CHECK",                str,                [                    table_name,                    date_ctx["datekey"],                ],            )            if result and str(result).strip().upper() == "N":                raise AirflowException(                    "TABLE_LOAD_CHECK failed for %s" % table_name                )    finally:        cursor.close()        conn.close()
```

The important difference is:

```
cursor.callfunc(...)
```

instead of:

```
hook.get_first("SELECT ... FROM dual")
```

Conceptually, you're changing this:

```
Airflow
   ↓
SELECT COL.TABLE_LOAD_CHECK(...) FROM dual
   ↓
Oracle SQL engine
   ↓
Function tries COMMIT
   ↓
❌ ORA-14552
```

to:

```
Airflow
   ↓
Oracle PL/SQL function call
   ↓
COL.TABLE_LOAD_CHECK(...)
   ↓
Function executes
   ↓
✅ COMMIT/transaction logic allowed
```

### Your updated section should therefore look like this

```
@task(task_id="create_table_name_array")def create_table_name_array() -> list:    return ROW_COUNT_TABLES@task(task_id="check_row_count")def check_row_count(table_names: list, date_ctx: Dict) -> None:    hook = _hook()    conn = hook.get_conn()    cursor = conn.cursor()    try:        for table_name in table_names:            result = cursor.callfunc(                "COL.TABLE_LOAD_CHECK",                str,                [                    table_name,                    date_ctx["datekey"],                ],            )            if result and str(result).strip().upper() == "N":                raise AirflowException(                    "TABLE_LOAD_CHECK failed for %s" % table_name                )    finally:        cursor.close()        conn.close()@task(task_id="delete_i_gstm_detail")def delete_i_gstm_detail(date_ctx: Dict) -> None:    _hook().run(        "delete from COL.I_GSTM_DETAIL "
```

One thing I would **not** do yet is change the Oracle `TABLE_LOAD_CHECK` function itself. Since this is an **SSIS → Airflow migration**, the safer approach is to preserve the existing database procedure/function behavior and make Airflow invoke it in the appropriate Oracle execution context.

After making this change, rerun only `check_row_count`. If it then throws a different Oracle error, send me that log — that will tell us whether the return datatype/signature of `TABLE_LOAD_CHECK` needs a small adjustment.