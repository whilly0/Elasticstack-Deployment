# Filebeat Implementation

I installed Filebeat on my Windows 11 endpoint to collect custom file-based logs and send them to Elasticsearch.

Filebeat was used alongside Winlogbeat.

While Winlogbeat collected Windows Event Logs, Filebeat collected custom text logs from selected local directories.

## Custom Log Sources

Filebeat monitored the following folders:

```text
C:\Program Files\Filebeat\LogsToCloud
```

```text
C:\Users\OFFICIAL USE\Downloads\LogsTrial
```

## Input Type

I used the Filebeat `filestream` input type.

The `filestream` input was used to read log files from the configured folders, including subfolders where required.

## Implementation Process

I completed the following tasks:

1. Installed Filebeat on the Windows endpoint
2. Created separate inputs for each log source
3. Used unique IDs for each Filebeat input
4. Configured Filebeat to monitor custom folders
5. Configured recursive folder monitoring where required
6. Added custom source fields to identify logs
7. Tested the Filebeat configuration
8. Tested the Elasticsearch output
9. Created test log entries
10. Used Kibana to confirm and filter the collected events

## Log Identification

Custom fields were added to distinguish the different log sources.

This made it possible to filter logs in Kibana according to their origin.

The custom sources included:

- `cloud_logs` for logs from the LogsToCloud folder
- `Logs_Trial` for logs from the LogsTrial folder

## Useful Commands

### Test Filebeat configuration

```powershell
cd "C:\Program Files\Filebeat"
.\filebeat.exe test config -e
```

### Test Filebeat output

```powershell
.\filebeat.exe test output -e
```

### Run Filebeat in the foreground

```powershell
.\filebeat.exe -e
```

### Create a test event

```powershell
Add-Content `
  "C:\Users\OFFICIAL USE\Downloads\LogsTrial\test.log" `
  "Filebeat test event $(Get-Date)"
```

## Kibana Investigation

In Kibana, I can filter custom Filebeat logs by:

- Log source
- File path
- Host name
- Timestamp
- Message content
- Log level

## Security Note

The Filebeat configuration file is not included yet because it may contain sensitive information such as:

- Elasticsearch credentials
- Certificate paths
- Certificate authority files
- Internal server addresses
