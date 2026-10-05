Yes—the CSV files are now successfully copied into Airflow’s incoming folder.

A consistent dated set appears to be:

```
internal_counterparty20251021.csv
repo_client_mapping20251021.csv
inter_affiliate_xref20251021.csv
```

The smaller `20230418` files are likely old test or historical files.

Before mapping the third file, confirm that `inter_affiliate_xref` corresponds to the staging table currently called `I_INTERNAL_COUNTERPARTY_AMEND`:

```
Select-String -Path "D:\Apps\SSIS\packages\COLLATERAL\APR_LATE_LOAD.dtsx" -Pattern "inter_affiliate_xref|I_INTERNAL_COUNTERPARTY_AMEND" -Context 4,4
```

If confirmed, update the DAG parameters to:

```
"internal_counterparty_file": (    "/home/wsl2user/airflow/data/incoming/"    "internal_counterparty20251021.csv"),"internal_counterparty_amend_file": (    "/home/wsl2user/airflow/data/incoming/"    "inter_affiliate_xref20251021.csv"),"repo_client_mapping_file": (    "/home/wsl2user/airflow/data/incoming/"    "repo_client_mapping20251021.csv"),
```

Also replace the empty `download_command` with a validation command:

```
"download_command": (    "test -f /home/wsl2user/airflow/data/incoming/internal_counterparty20251021.csv && "    "test -f /home/wsl2user/airflow/data/incoming/inter_affiliate_xref20251021.csv && "    "test -f /home/wsl2user/airflow/data/incoming/repo_client_mapping20251021.csv"),
```

Then save the DAG and validate the files:

```
ls -lh "$HOME/airflow/data/incoming"
```

Do not trigger it yet unless `oracle_apr_conn` points to an approved development/test schema. The DAG contains truncate, bulk-load, merge and commit operations, so running it can modify Oracle tables. The XML mapping for `inter_affiliate_xref` should also be confirmed first.