We found the actual Windows landing directory in the SSIS package:

```
E:\loaddata\Collateral\FIST_CALYPSO\incoming
```

The screenshot specifically confirms this file:

```
E:\loaddata\Collateral\FIST_CALYPSO\incoming\internal_counterparty.csv
```

So the earlier search returned nothing because we searched `D:`, while the data is expected under `E:`.

Run:

```
Get-ChildItem "E:\loaddata\Collateral\FIST_CALYPSO\incoming" -Force
```

To search the complete application directory:

```
Get-ChildItem "E:\loaddata\Collateral\FIST_CALYPSO" -Recurse -File -ErrorAction SilentlyContinue |
Where-Object { $_.Name -match "counterparty|repo.*client" } |
Select-Object FullName,LastWriteTime,Length
```

The package also references:

```
User::FileName
User::winscp_result
WinSCP_Results_File
```

This confirms that the SSIS package uses WinSCP and dynamically determines at least some filenames. Therefore, the files may only appear in `incoming` after the WinSCP transfer runs.

To show every CSV reference from the package:

```
Select-String -Path "D:\Apps\SSIS\packages\COLLATERAL\APR_LATE_LOAD.dtsx" -Pattern "\.csv" -Context 2,2
```

For the Airflow POC, after confirming the files, copy them from Windows into WSL:

```
Copy-Item "E:\loaddata\Collateral\FIST_CALYPSO\incoming\*.csv" "\\wsl$\rhel8_TAP3507\home\wsl2user\airflow\data\incoming\"
```

Then verify inside WSL:

```
ls -lh "$HOME/airflow/data/incoming"
```

At this point, `internal_counterparty.csv` is confirmed. We still need to verify the exact names for the amended counterparty and repo-client files before triggering Airflow. Use only approved/non-sensitive data for the POC.