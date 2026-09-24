# Filebeat Implementation Guide

## Overview

Filebeat is a lightweight log shipping agent that monitors and forwards custom log files to Elasticsearch for centralized storage and analysis. This guide documents the installation and configuration of Filebeat on a Windows 11 endpoint to collect custom file-based logs alongside Winlogbeat's Windows Event Log collection.

## Purpose

Filebeat enables:

- Collection of custom text-based log files from local directories
- Monitoring of multiple log sources simultaneously
- File-based log ingestion in addition to Windows Event Logs
- Recursive folder monitoring for logs in subdirectories
- Custom field assignment to identify log sources
- Integration with Kibana for log analysis and investigation

## Log Sources Monitored

Filebeat was configured to monitor the following directories:

- **LogsToCloud** - `C:\Program Files\Filebeat\LogsToCloud\`
- **LogsTrial** - `C:\Users\[USERNAME]\Downloads\LogsTrial\`

### Log Types Supported

Filebeat can collect various log file formats:

- Plain text logs (`.log`)
- JSON logs (`.json`)
- PCAP files (`.pcap`)
- Comma-separated values (`.csv`)
- Any custom text-based log format

## Placeholder Variables Reference

This guide uses placeholder variables that you must replace with your actual values. Below is a reference for all placeholders used:

| Placeholder | Description | Example |
|---|---|---|
| `[SERVER_IP]` | IP address of your Elasticsearch/Kibana server | `192.168.20.125` |
| `[ENDPOINT_HOSTNAME]` | Windows endpoint hostname | `CYBERDEMI` |
| `[LINUX_USERNAME]` | Username on Linux server | `percy` |
| `[ES_INTERNAL_USER]` | Elasticsearch internal user for Filebeat | `filebeat_internal` |
| `[ES_INTERNAL_PASSWORD]` | Password for Elasticsearch internal user | (your actual password) |
| `[KIBANA_SETUP_USER]` | Kibana user for dashboard setup | `filebeat_setup` |
| `[KIBANA_SETUP_PASSWORD]` | Password for Kibana setup user | (your actual password) |
| `[ELASTICSEARCH_USERNAME]` | Elasticsearch admin/elastic user | `elastic` |
| `[ELASTICSEARCH_PASSWORD]` | Password for Elasticsearch admin user | (your actual password) |
| `[LOCAL_USERNAME]` | Windows username for log directories | `OFFICIAL USE` |

**Important:** Replace all instances of these placeholders with your actual values before executing any commands or configurations.

## Prerequisites

### System Requirements

- Windows 11 endpoint (hostname: `[ENDPOINT_HOSTNAME]`)
- Administrator access (required for service installation)
- PowerShell (preferably version 5.0 or later)
- Network connectivity to Elasticsearch (`[SERVER_IP]:9200`)
- Network connectivity to Kibana (`[SERVER_IP]:5601`)
- Disk space for log files (depends on log volume)

### Required Files and Information

- Filebeat 8.12.0 for Windows x86_64
- CA certificate file (`ca.crt`) - can be reused from Winlogbeat installation
- Elasticsearch credentials (username and password)
- Kibana setup credentials
- Custom log files ready for ingestion

## Step-by-Step Installation

### Step 1: Download Filebeat

Download Filebeat 8.12.0 for Windows x86_64 from the Elastic artifacts repository:

```text
https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.12.0-windows-x86_64.zip
```

### Step 2: Extract Filebeat

Extract the downloaded zip file to the Program Files directory:

```powershell
Expand-Archive -Path "C:\Users\[LOCAL_USERNAME]\Downloads\filebeat-8.12.0-windows-x86_64.zip" `
  -DestinationPath "C:\Program Files\"
```

After extraction, verify the directory structure:

```text
C:\Program Files\Filebeat\
├── filebeat.exe
├── filebeat.yml
├── install-service-filebeat.ps1
├── LICENSE.txt
└── NOTICE.txt
```

### Step 3: Verify Binary File Size

Verify that the Filebeat executable is valid (not 0 bytes):

```powershell
ls "C:\Program Files\Filebeat\filebeat.exe"
```

**Expected output:**
```text
-a---- [Date] [Time] 64000000 filebeat.exe
```

**Important:** The file size should be 60+ MB. If it shows 0 bytes, the extraction failed. Re-download and extract.

### Step 4: Open PowerShell as Administrator

Press `Win + X` and select **Windows PowerShell (Admin)**, or:

1. Right-click on PowerShell
2. Select **Run as Administrator**
3. Click **Yes** when prompted for administrator access

Navigate to the Filebeat directory:

```powershell
cd "C:\Program Files\Filebeat"
```

### Step 5: Create Data Input Directory

Create the primary log collection directory:

```powershell
mkdir "C:\Program Files\Filebeat\LogsToCloud"
```

This directory will be monitored by Filebeat for log files to ingest.

### Step 6: Copy CA Certificate (if not already present)

If you have already installed Winlogbeat, you can reuse the CA certificate. If not, copy the CA certificate from your Linux server:

```powershell
scp [LINUX_USERNAME]@[SERVER_IP]:/home/[LINUX_USERNAME]/Elastic-Stack/ElasticMN/exported-certs/ca.crt `
  "C:\Program Files\Filebeat\ca.crt"
```

Alternatively, if the certificate is already in the Winlogbeat directory, you can reference it from there in your Filebeat configuration.

Verify the certificate file exists:

```powershell
ls "C:\Program Files\Filebeat\ca.crt"
```

### Step 7: Configure filebeat.yml

The `filebeat.yml` file contains the configuration for log collection, Elasticsearch output, and security settings.

Open the configuration file in Notepad:

```powershell
notepad filebeat.yml
```

Configure the following sections:

#### 7.1 Input Section - Filestream Input for Custom Logs

Configure Filebeat to monitor custom log directories:

```yaml
filebeat.inputs:
- type: filestream
  id: cloud_logs_input
  enabled: true
  paths:
    - "C:\\Program Files\\Filebeat\\LogsToCloud\\*"
  fields:
    source: "cloud_logs"
    endpoint: "[ENDPOINT_HOSTNAME]"

- type: filestream
  id: trial_logs_input
  enabled: true
  paths:
    - "C:\\Users\\[LOCAL_USERNAME]\\Downloads\\LogsTrial\\*"
  fields:
    source: "Logs_Trial"
    endpoint: "[ENDPOINT_HOSTNAME]"
```

**Configuration Details:**

- `type: filestream` - Read logs from files (recommended over `log` input)
- `id` - Unique identifier for each input (required with filestream)
- `enabled: true` - Enable this input
- `paths` - Path patterns to monitor (use `*` for all files in directory)
- `fields` - Custom fields added to every log entry for filtering

**Replace:**
- `[ENDPOINT_HOSTNAME]` with your Windows hostname (e.g., `CYBERDEMI`)
- `[LOCAL_USERNAME]` with your Windows username (e.g., `OFFICIAL USE`)

#### 7.2 Output Section - Elasticsearch Configuration

Configure Elasticsearch as the output destination:

```yaml
output.elasticsearch:
  hosts: ["[SERVER_IP]:9200"]
  username: "[ES_INTERNAL_USER]"
  password: "[ES_INTERNAL_PASSWORD]"
  ssl:
    enabled: true
    certificate_authorities: "C:\\Program Files\\Filebeat\\ca.crt"
    verification_mode: certificate
```

**Replace:**
- `[SERVER_IP]` with your Elasticsearch server IP address
- `[ES_INTERNAL_USER]` with the Elasticsearch internal user (e.g., `filebeat_internal`)
- `[ES_INTERNAL_PASSWORD]` with the actual password for the internal user

#### 7.3 Kibana Section - Dashboard Setup

Configure Kibana for dashboard and index pattern setup:

```yaml
setup.kibana:
  host: "[SERVER_IP]:5601"
  username: "[KIBANA_SETUP_USER]"
  password: "[KIBANA_SETUP_PASSWORD]"
  ssl:
    enabled: true
    certificate_authorities: "C:\\Program Files\\Filebeat\\ca.crt"
    verification_mode: certificate
```

**Replace:**
- `[SERVER_IP]` with your Kibana server IP address
- `[KIBANA_SETUP_USER]` with the Kibana setup user (e.g., `filebeat_setup`)
- `[KIBANA_SETUP_PASSWORD]` with the actual password for the setup user

#### 7.4 Index Lifecycle Management (ILM)

Configure index lifecycle policies:

```yaml
setup.ilm.enabled: true
setup.ilm.policy_name: "filebeat"
setup.ilm.overwrite: false
```

Save the file when finished editing (Ctrl + S, then close).

### Step 8: Test the Configuration

Verify that the Filebeat configuration is syntactically correct:

```powershell
.\filebeat.exe test config -e
```

**Expected output:**
```text
Config OK
```

If you see errors, review the configuration file and correct any issues.

### Step 9: Test Output Connection

Test that Filebeat can connect to Elasticsearch:

```powershell
.\filebeat.exe test output -e
```

**Expected output:**
```text
elasticsearch: [SERVER_IP]:9200...
  parse url... OK
  connection...
    TLS...
      Verify mode... certificate
      Trusted CA file... OK
    Connection attempt... OK
  Talking to server... OK
```

If the connection test fails, verify:
- Elasticsearch is running on the Linux server
- The IP address `[SERVER_IP]` is correct
- Firewall allows outbound connections to port 9200
- The CA certificate file is in the correct location
- Credentials are correct

### Step 10: Place Log Files to Ingest

Copy your custom log files to the monitored directories:

```powershell
# Copy files to LogsToCloud directory
Copy-Item -Path "C:\Users\[LOCAL_USERNAME]\Downloads\*.log" `
  -Destination "C:\Program Files\Filebeat\LogsToCloud\" -Force

Copy-Item -Path "C:\Users\[LOCAL_USERNAME]\Downloads\PCAP.json" `
  -Destination "C:\Program Files\Filebeat\LogsToCloud\" -Force
```

Supported file types:
- `.log` - Plain text logs
- `.json` - JSON formatted logs
- `.pcap` - PCAP network capture files
- `.csv` - Comma-separated values
- Any custom text-based file format

Verify files were copied:

```powershell
ls "C:\Program Files\Filebeat\LogsToCloud\"
```

### Step 11: Load Assets and Install Service

Load Filebeat dashboards, visualizations, and index templates into Kibana:

```powershell
.\filebeat.exe setup -e
```

Wait for the setup to complete. You should see:

```text
Index template loaded successfully
Dashboards loaded successfully
```

Install Filebeat as a Windows service:

```powershell
PowerShell.exe -ExecutionPolicy Bypass -File .\install-service-filebeat.ps1
```

**Expected output:**
```text
Service 'filebeat' installed successfully
```

### Step 12: Start the Filebeat Service

Start the Filebeat service:

```powershell
Start-Service filebeat
```

### Step 13: Verify Service Status

Check that the Filebeat service is running:

```powershell
Get-Service filebeat
```

**Expected output:**
```text
Status   Name            DisplayName
------   ----            -----------
Running  filebeat        filebeat
```

If the status shows `Stopped`, check the logs for errors:

```powershell
.\filebeat.exe -e
```

### Step 14: Wait for Data Ingestion

**Important:** Data ingestion time depends on file size:

- Small files (< 10 MB): 30-60 seconds
- Medium files (10-50 MB): 2-5 minutes
- Large files (50+ MB): 5-15 minutes

Monitor ingestion progress in the Filebeat logs:

```powershell
ls "C:\Program Files\Filebeat\Logs\"
```

View real-time logs during ingestion:

```powershell
.\filebeat.exe -e
```

Look for lines indicating file reading:

```text
Harvester started for file
Processing line
```

Check for errors:

```powershell
Get-Content "C:\Program Files\Filebeat\Logs\filebeat*.ndjson" | Select-String "error|failed"
```

## Verification in Kibana

### Accessing Kibana

1. Open a web browser on a computer with network access to your Linux server
2. Navigate to: `https://[SERVER_IP]:5601`
3. Log in with your Elasticsearch credentials (username: `elastic` or `[ELASTICSEARCH_USERNAME]`)

### Viewing Collected Data

1. In Kibana, navigate to **Discover** (or **Analytics > Discover**)
2. In the data view selector (top left), look for: `filebeat-*`
3. If the data view does not exist, click **Create data view** and enter `filebeat-*`
4. Wait 30-60 seconds (or up to 15 minutes for large files) for the first events to appear
5. If data appears with old timestamps (e.g., 2017):
   - Click the time range selector (top right)
   - Set to a custom range that covers your log dates
   - Example: "Jan 1, 2017 - Dec 31, 2017" for legacy logs
6. Click the **Refresh** button if no data is visible

### Filtering and Searching

In Kibana's Discover interface, you can filter custom Filebeat logs by:

- **Log source** - `cloud_logs` or `Logs_Trial`
- **File path** - Full path to the log file
- **Host name** - `[ENDPOINT_HOSTNAME]`
- **Endpoint** - Custom field identifying the source endpoint
- **Message content** - Search within log messages
- **Timestamp** - Date and time range
- **File name** - Specific log file

### Example Queries

**Filter by log source (LogsToCloud):**
```text
source: "cloud_logs"
```

**Filter by log source (LogsTrial):**
```text
source: "Logs_Trial"
```

**Filter by hostname:**
```text
endpoint: "[ENDPOINT_HOSTNAME]"
```

**Filter by both hostname and source:**
```text
endpoint: "[ENDPOINT_HOSTNAME]" AND source: "cloud_logs"
```

**Search for specific keywords:**
```text
message: "error" OR message: "failed"
```

### Importing Dashboards (Optional)

If Filebeat dashboards are missing from Kibana after setup, manually import them:

```powershell
.\filebeat.exe setup --dashboards -e
```

Wait for completion (30+ seconds). You should see:

```text
Loaded dashboards successfully
```

## Useful Commands

### Check Filebeat Service Status

```powershell
Get-Service filebeat
```

### Restart Filebeat Service

```powershell
Restart-Service filebeat
```

### Stop Filebeat Service

```powershell
Stop-Service filebeat
```

### Test Configuration

```powershell
cd "C:\Program Files\Filebeat"
.\filebeat.exe test config -e
```

### Test Output Connection

```powershell
.\filebeat.exe test output -e
```

### Run Filebeat in Foreground (for debugging)

```powershell
.\filebeat.exe -e
```

This will display logs in real-time and help identify connectivity or configuration issues.

### Create a Test Log Entry

Test that Filebeat is monitoring files by creating a test log entry:

```powershell
Add-Content `
  "C:\Program Files\Filebeat\LogsToCloud\test.log" `
  "Filebeat test event $(Get-Date)"
```

Check that the entry appears in Kibana within 30-60 seconds.

### View Filebeat Service Logs

```powershell
Get-EventLog -LogName Application -Source filebeat -Newest 10
```

### View Filebeat Application Logs

```powershell
Get-Content "C:\Program Files\Filebeat\Logs\filebeat*.ndjson" -Tail 20
```

### Uninstall Service (if needed)

```powershell
PowerShell.exe -ExecutionPolicy Bypass -File .\uninstall-service-filebeat.ps1
```

## Troubleshooting

### Filebeat Service Will Not Start

**Problem:** The Filebeat service fails to start or stops immediately after starting.

**Solution:**
1. Run Filebeat in foreground mode to see error messages:
   ```powershell
   cd "C:\Program Files\Filebeat"
   .\filebeat.exe -e
   ```
2. Check the output for error messages related to:
   - Certificate not found
   - Configuration syntax error
   - Connection refused
   - Authentication failure
3. Address the specific error and restart the service:
   ```powershell
   Start-Service filebeat
   ```

### Connection Errors to Elasticsearch

**Problem:** Logs like "connection refused" or "certificate verification failed"

**Solutions:**
1. Verify Elasticsearch is running on the Linux server:
   ```bash
   docker compose -f elastic-docker-tls.yml ps
   ```
2. Verify the IP address is correct (`[SERVER_IP]`)
3. Verify the CA certificate is in the correct location:
   ```powershell
   ls "C:\Program Files\Filebeat\ca.crt"
   ```
4. Verify firewall allows outbound connections to port 9200:
   ```powershell
   Test-NetConnection [SERVER_IP] -Port 9200
   ```
5. Test the output connection:
   ```powershell
   .\filebeat.exe test output -e
   ```

### Authentication Failures

**Problem:** Errors like "invalid username or password" or "401 Unauthorized"

**Solutions:**
1. Verify the credentials in `filebeat.yml` are correct
2. Verify the `filebeat_internal` user exists in Elasticsearch
3. Verify the user has the correct role assignment
4. Regenerate the password and update the configuration file
5. Test the output connection:
   ```powershell
   .\filebeat.exe test output -e
   ```

### No Data Appearing in Kibana

**Problem:** Filebeat is running but no data appears in Kibana after 60 seconds

**Solutions:**
1. Wait appropriate time based on file size (5-15 minutes for large files)
2. Verify the data view `filebeat-*` exists in Kibana
3. Check if log files contain valid content:
   ```powershell
   Get-Content "C:\Program Files\Filebeat\LogsToCloud\*.log" -Head 10
   ```
4. Verify log files are in the correct directory:
   ```powershell
   ls "C:\Program Files\Filebeat\LogsToCloud\"
   ls "C:\Users\[LOCAL_USERNAME]\Downloads\LogsTrial\"
   ```
5. Check Filebeat is reading the files:
   ```powershell
   .\filebeat.exe -e
   ```
   Look for lines like:
   ```text
   Harvester started for file
   ```
6. Verify Elasticsearch is receiving data:
   ```text
   curl -k -u [ELASTICSEARCH_USERNAME]:[ELASTICSEARCH_PASSWORD] https://[SERVER_IP]:9200/_cat/indices?v
   ```
   Look for indices like `filebeat-*`

### Old Log Timestamps (e.g., 2017)

**Problem:** Logs appear in Kibana but with dates from years ago

**Cause:** Legacy log files contain old timestamps

**Solution:**
1. In Kibana, open the time range selector (top right)
2. Select **Custom** and set the date range to match your log dates
3. Example: Set to "Jan 1, 2017 - Dec 31, 2017" for 2017 logs
4. Click **Refresh**
5. Your logs should now be visible

### Service Status Issues

**Problem:** Service shows as "Running" but no data is being collected

**Solutions:**
1. Restart the service:
   ```powershell
   Restart-Service filebeat
   ```
2. Check service logs:
   ```powershell
   Get-EventLog -LogName Application -Source filebeat -Newest 10
   ```
3. Run in foreground to identify specific errors:
   ```powershell
   Stop-Service filebeat
   .\filebeat.exe -e
   ```

### Certificate Errors

**Problem:** "certificate verification failed" or "certificate not found"

**Solutions:**
1. Verify the CA certificate file exists:
   ```powershell
   Test-Path "C:\Program Files\Filebeat\ca.crt"
   ```
2. Re-copy the certificate from the Linux server:
   ```powershell
   scp [LINUX_USERNAME]@[SERVER_IP]:/home/[LINUX_USERNAME]/Elastic-Stack/ElasticMN/exported-certs/ca.crt `
     "C:\Program Files\Filebeat\ca.crt"
   ```
3. Verify certificate has correct permissions (should be readable)
4. Verify the path in `filebeat.yml` uses double backslashes: `C:\\Program Files\\Filebeat\\ca.crt`
5. Restart the service:
   ```powershell
   Restart-Service filebeat
   ```

### Large File Ingestion Delays

**Problem:** Files larger than 50 MB are taking a long time to ingest

**Expected behavior:** Large file ingestion is normal and can take 5-15 minutes

**Solutions:**
1. Monitor ingestion progress:
   ```powershell
   .\filebeat.exe -e
   ```
2. Check if the process is still running:
   ```powershell
   Get-Process filebeat
   ```
3. Wait for ingestion to complete before restarting Filebeat
4. For very large files (100+ MB), consider splitting them before ingestion

### Configuration Syntax Errors

**Problem:** "Config OK" fails or shows syntax errors

**Solutions:**
1. Verify YAML syntax (indentation is critical):
   ```powershell
   .\filebeat.exe test config -e
   ```
2. Check for common issues:
   - Spaces vs. tabs (use spaces only)
   - Incorrect indentation of nested fields
   - Missing colons in key-value pairs
   - Quotes around values with special characters
3. Compare your configuration with the examples in this guide
4. Test again after fixes:
   ```powershell
   .\filebeat.exe test config -e
   ```

## Configuration File Security Note

The `filebeat.yml` configuration file contains sensitive information and should be protected:

- Elasticsearch credentials (username and password)
- Kibana credentials
- Certificate authority file path
- Server IP addresses and port numbers

**Protect the configuration file:**
1. Restrict file permissions (only administrators should read it)
2. Do not commit to version control systems
3. Do not share via email or messaging
4. Rotate credentials periodically
5. Use group passwords or service accounts when possible

## Next Steps

After successful Filebeat deployment:

1. Monitor custom logs in Kibana
2. Create custom dashboards for log analysis
3. Set up alerts for critical log entries
4. Configure log retention policies
5. Review and analyze collected log data
6. Integrate Filebeat logs with Winlogbeat data for comprehensive endpoint monitoring
