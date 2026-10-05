The SSIS package confirms the files are date-stamped and handled through WinSCP. The relevant patterns are:

```
repo_client_mapping*.csv
internal_counterparty*.csv
```

The Windows folder used for processing is:

```
E:\loaddata\Collateral\FIST_CALYPSO\incoming
```

Your `Copy-Item` failed because the `\\wsl$\...` destination was unavailable—not necessarily because the CSV files were missing.

### Use WSL directly instead

Open the WSL terminal and run:

```
mkdir -p "$HOME/airflow/data/incoming"
mkdir -p "$HOME/airflow/data/archive"
mkdir -p "$HOME/airflow/data/failure"

ls -lh /mnt/e/loaddata/Collateral/FIST_CALYPSO/incoming
```

Search for the relevant files:

```
find /mnt/e/loaddata/Collateral/FIST_CALYPSO/incoming \
  -maxdepth 1 -type f \
  \( -iname "internal_counterparty*.csv" -o -iname "repo_client_mapping*.csv" \)
```

If they appear, copy them:

```
cp /mnt/e/loaddata/Collateral/FIST_CALYPSO/incoming/*.csv \
   "$HOME/airflow/data/incoming/"
```

Verify:

```
ls -lh "$HOME/airflow/data/incoming"
```

### Important discrepancy in the converted DAG

The SSIS package appears to use date-stamped filenames, such as:

```
internal_counterparty20251001.csv
repo_client_mapping20251001.csv
```

But the Airflow DAG currently expects fixed filenames:

```
internal_counterparty.csv
internal_counterparty_amend.csv
repo_client_mapping.csv
```

Also, the screenshot does not yet confirm that `internal_counterparty_amend.csv` exists in the original SSIS package.

Therefore, don’t trigger the DAG yet. First run:

```
Select-String `
  -Path "D:\Apps\SSIS\packages\COLLATERAL\APR_LATE_LOAD.dtsx" `
  -Pattern "amend|internal_counterparty|repo_client_mapping" `
  -Context 2,2
```

We need to modify the DAG to select the correct date-stamped files and remove `internal_counterparty_amend.csv` if that input is not actually part of `APR_LATE_LOAD.dtsx`.