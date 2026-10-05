Both folders are empty:

- Original SSIS incoming folder: `E:\loaddata\Collateral\FIST_CALYPSO\incoming`
- Airflow incoming folder: `~/airflow/data/incoming`

This is normal if SSIS moves files to an archive after processing or if today’s WinSCP download has not occurred.

Search the likely SSIS directories in PowerShell:

```
Get-ChildItem "E:\loaddata\Collateral\FIST_CALYPSO" -Recurse -File -ErrorAction SilentlyContinue | Where-Object { $_.Name -match "internal.*counterparty|repo.*client.*mapping" } | Select-Object FullName,LastWriteTime,Length
```

Also search the FTP landing folders referenced by the package:

```
Get-ChildItem "E:\ftproot\ftpsite\Collateral" -Recurse -File -ErrorAction SilentlyContinue | Where-Object { $_.Name -match "internal.*counterparty|repo.*client.*mapping" } | Select-Object FullName,LastWriteTime,Length
```

If files are found in an archive folder, copy an approved historical set into the open WSL `incoming` window for the POC.

If nothing is found, the files only exist on the remote SFTP server and are downloaded by WinSCP during execution. In that case, the next migration step is to reproduce the WinSCP download using an Airflow SFTP connection. Do not run the production SSIS package merely to obtain files without confirming that it is safe, because it may also load or modify Oracle tables.