# Winlogbeat Implementation

I installed Winlogbeat on my Windows 11 endpoint named `CYBERDEMI`.

Winlogbeat was used to collect Windows Event Logs and send them to Elasticsearch for centralized storage and analysis.

## Log Channels Collected

Winlogbeat was configured to collect the following Windows Event Log channels:

- Application
- System
- Security

These logs provide useful information for troubleshooting, monitoring, auditing, and security investigations.

## Implementation Process

I completed the following tasks:

1. Installed Winlogbeat on the Windows endpoint
2. Configured Windows Event Log collection
3. Configured Elasticsearch as the log output
4. Configured Kibana connectivity for setup tasks
5. Tested the Winlogbeat configuration
6. Tested the output connection
7. Started the Winlogbeat service
8. Checked Winlogbeat diagnostic logs
9. Verified that the endpoint could connect to Kibana
10. Used Kibana to investigate collected Windows logs

## Validation

I verified that Winlogbeat registered the following event log inputs:

```text
Application
System
Security
```

I also checked that the Winlogbeat service was running.

## Useful Commands

### Check service status

```powershell
Get-Service winlogbeat
```

### Restart Winlogbeat

```powershell
Restart-Service winlogbeat
```

### Test configuration

```powershell
cd "C:\Program Files\Winlogbeat"
.\winlogbeat.exe test config -e
```

### Test output

```powershell
.\winlogbeat.exe test output -e
```

### Run in the foreground

```powershell
.\winlogbeat.exe -e
```

## Kibana Investigation

In Kibana, Windows Event Logs can be filtered by:

- Host name: `CYBERDEMI`
- Event channel: Application, System, or Security
- Event ID
- User name
- Log level
- Timestamp

## Security Note

The Winlogbeat configuration file is not included yet because it may contain sensitive details such as:

- Elasticsearch credentials
- Kibana credentials
- Certificate paths
- Certificate authority files
- Internal server details
