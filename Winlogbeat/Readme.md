# Winlogbeat Implementation Guide

## Overview

Winlogbeat is a lightweight log forwarding agent that collects Windows Event Logs and sends them to Elasticsearch for centralized storage and analysis. This guide documents the installation and configuration of Winlogbeat on a Windows 11 endpoint named `CYBERDEMI`.

## Purpose

Winlogbeat enables:

- Centralized collection of Windows Event Logs
- Real-time log forwarding to Elasticsearch
- Secure log transmission using TLS/SSL
- Integration with Kibana for log analysis and investigation
- Compliance and security auditing through log centralization

## Log Channels Collected

Winlogbeat was configured to collect the following Windows Event Log channels:

- **Application** - Application-level events and errors
- **System** - System-level events, driver errors, and hardware issues
- **Security** - Security events, login attempts, and audit logs

These logs provide useful information for:
- Troubleshooting system and application issues
- Monitoring endpoint health
- Security event auditing
- Incident investigation

## Prerequisites

### System Requirements

- Windows 11 endpoint (hostname: `[ENDPOINT_HOSTNAME]`)
- Administrator access (required for service installation)
- PowerShell (preferably version 5.0 or later)
- Network connectivity to Elasticsearch (`[SERVER_IP]:9200`)
- Network connectivity to Kibana (`[SERVER_IP]:5601`)

### Required Files and Information

- Winlogbeat 8.12.0 for Windows x86_64
- CA certificate file (`ca.crt`) from your Linux server
- Elasticsearch credentials (username and password)
- Kibana setup credentials
- Elasticsearch server IP address: `[SERVER_IP]`
- Linux server username: `[LINUX_USERNAME]`
- Elasticsearch internal user: `[ES_INTERNAL_USER]`
- Kibana setup user: `[KIBANA_SETUP_USER]`

## Placeholder Variables Reference

This guide uses placeholder variables that you must replace with your actual values. Below is a reference for all placeholders used:

| Placeholder | Description | Example |
|---|---|---|
| `[SERVER_IP]` | IP address of your Elasticsearch/Kibana server | `192.168.20.125` |
| `[ENDPOINT_HOSTNAME]` | Windows endpoint hostname | `CYBERDEMI` |
| `[LINUX_USERNAME]` | Username on Linux server | `Percy` |
| `[ES_INTERNAL_USER]` | Elasticsearch internal user for Winlogbeat | `winlogbeat_internal` |
| `[ES_INTERNAL_PASSWORD]` | Password for Elasticsearch internal user | (your actual password) |
| `[KIBANA_SETUP_USER]` | Kibana user for dashboard setup | `winlogbeat_setup` |
| `[KIBANA_SETUP_PASSWORD]` | Password for Kibana setup user | (your actual password) |
| `[ELASTICSEARCH_USERNAME]` | Elasticsearch admin/elastic user | `elastic` |
| `[ELASTICSEARCH_PASSWORD]` | Password for Elasticsearch admin user | (your actual password) |

**Important:** Replace all instances of these placeholders with your actual values before executing any commands or configurations.

## Step-by-Step Installation

### Step 1: Download Winlogbeat

Download Winlogbeat 8.12.0 for Windows x86_64 from the Elastic artifacts repository:

```text
https://artifacts.elastic.co/downloads/beats/winlogbeat/winlogbeat-8.12.0-windows-x86_64.zip
```

### Step 2: Extract Winlogbeat

Extract the downloaded zip file to the Program Files directory:

```powershell
Expand-Archive -Path "C:\Users\[YourUsername]\Downloads\winlogbeat-8.12.0-windows-x86_64.zip" `
  -DestinationPath "C:\Program Files\"
```

After extraction, verify the directory structure:

```text
C:\Program Files\Winlogbeat\
├── winlogbeat.exe
├── winlogbeat.yml
├── install-service-winlogbeat.ps1
├── LICENSE.txt
└── NOTICE.txt
```

### Step 3: Open PowerShell as Administrator

Press `Win + X` and select **Windows PowerShell (Admin)**, or:

1. Right-click on PowerShell
2. Select **Run as Administrator**
3. Click **Yes** when prompted for administrator access

Navigate to the Winlogbeat directory:

```powershell
cd "C:\Program Files\Winlogbeat"
```

### Step 4: Copy CA Certificate from Linux Server

Copy the CA certificate file (`ca.crt`) from your Linux server to the Winlogbeat directory. This certificate is required for secure TLS connections to Elasticsearch.

Using `scp` (Secure Copy Protocol):

```powershell
scp [LINUX_USERNAME]@[SERVER_IP]:/home/[LINUX_USERNAME]/Elastic-Stack/ElasticMN/exported-certs/ca.crt `
  "C:\Program Files\Winlogbeat\ca.crt"
```

When prompted, enter the password for the `[LINUX_USERNAME]` user on your Linux server.

**Expected output:**
```text
ca.crt                                        100% 1234     1.2KB/s   00:00
```

Verify the file was copied successfully:

```powershell
ls "C:\Program Files\Winlogbeat\ca.crt"
```

### Step 5: Configure winlogbeat.yml

The `winlogbeat.yml` file contains the configuration for log collection, Elasticsearch output, and security settings.

Open the configuration file in Notepad:

```powershell
notepad winlogbeat.yml
```

Configure the following sections:

#### 5.1 Input Section - Event Log Channels

Configure Winlogbeat to collect Windows Event Logs:

```yaml
winlogbeat.event_logs:
  - name: Application
  - name: System
  - name: Security
```

#### 5.2 Output Section - Elasticsearch Configuration

Configure Elasticsearch as the output destination:

```yaml
output.elasticsearch:
  hosts: ["[SERVER_IP]:9200"]
  username: "[ES_INTERNAL_USER]"
  password: "[ES_INTERNAL_PASSWORD]"
  ssl:
    enabled: true
    certificate_authorities: "C:\\Program Files\\Winlogbeat\\ca.crt"
    verification_mode: certificate
```

**Replace:**
- `[SERVER_IP]` with your Elasticsearch server IP address
- `[ES_INTERNAL_USER]` with the Elasticsearch internal user (e.g., `winlogbeat_internal`)
- `[ES_INTERNAL_PASSWORD]` with the actual password for the internal user

#### 5.3 Kibana Section - Dashboard Setup

Configure Kibana for dashboard and index pattern setup:

```yaml
setup.kibana:
  host: "[SERVER_IP]:5601"
  username: "[KIBANA_SETUP_USER]"
  password: "[KIBANA_SETUP_PASSWORD]"
  ssl:
    enabled: true
    certificate_authorities: "C:\\Program Files\\Winlogbeat\\ca.crt"
    verification_mode: certificate
```

**Replace:**
- `[SERVER_IP]` with your Kibana server IP address
- `[KIBANA_SETUP_USER]` with the Kibana setup user (e.g., `winlogbeat_setup`)
- `[KIBANA_SETUP_PASSWORD]` with the actual password for the setup user

#### 5.4 Index Lifecycle Management (ILM)

Configure index lifecycle policies:

```yaml
setup.ilm.enabled: true
setup.ilm.policy_name: "winlogbeat"
setup.ilm.overwrite: false
```

Save the file when finished editing (Ctrl + S, then close).

### Step 6: Test the Configuration

Verify that the Winlogbeat configuration is syntactically correct:

```powershell
.\winlogbeat.exe test config -e
```

**Expected output:**
```text
Config OK
```

If you see errors, review the configuration file and correct any issues.

### Step 7: Test Output Connection

Test that Winlogbeat can connect to Elasticsearch:

```powershell
.\winlogbeat.exe test output -e
```

**Expected output:**
```text
elasticsearch: SEVER-IP:9200...
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
- The IP address (SERVER-IP) is correct
- Firewall allows outbound connections to port 9200
- The CA certificate file is in the correct location
- Credentials are correct

### Step 8: Load Assets and Install Service

Load Winlogbeat dashboards, visualizations, and index templates into Kibana:

```powershell
.\winlogbeat.exe setup -e
```

Wait for the setup to complete. You should see:

```text
Index template loaded successfully
Dashboards loaded successfully
```

Install Winlogbeat as a Windows service:

```powershell
PowerShell.exe -ExecutionPolicy Bypass -File .\install-service-winlogbeat.ps1
```

**Expected output:**
```text
Service 'winlogbeat' installed successfully
```

### Step 9: Start the Winlogbeat Service

Start the Winlogbeat service:

```powershell
Start-Service winlogbeat
```

### Step 10: Verify Service Status

Check that the Winlogbeat service is running:

```powershell
Get-Service winlogbeat
```

**Expected output:**
```text
Status   Name            DisplayName
------   ----            -----------
Running  winlogbeat      winlogbeat
```

If the status shows `Stopped`, check the logs for errors:

```powershell
.\winlogbeat.exe -e
```

## Verification in Kibana

### Accessing Kibana

1. Open a web browser on a computer with network access to your Linux server
2. Navigate to: `https://[SERVER_IP]:5601`
3. Log in with your Elasticsearch credentials (username: `elastic` or `[ELASTICSEARCH_USERNAME]`)

### Viewing Collected Data

1. In Kibana, navigate to **Discover** (or **Analytics > Discover**)
2. In the data view selector (top left), look for: `winlogbeat-*`
3. If the data view does not exist, click **Create data view** and enter `winlogbeat-*`
4. Wait 30-60 seconds for the first events to appear from your Windows endpoint
5. Click the **Refresh** button if no data is visible
6. Verify that the timestamp is correct

### Filtering and Searching

In Kibana's Discover interface, you can filter Windows Event Logs by:

- **Host name:** `[ENDPOINT_HOSTNAME]`
- **Event channel:** Application, System, or Security
- **Event ID:** Specific Windows event numbers
- **User name:** Windows user accounts
- **Log level:** Warning, Error, Critical
- **Timestamp:** Date and time range

### Example Queries

**Filter by hostname:**
```text
host.name: "[ENDPOINT_HOSTNAME]"
```

**Filter by Security events:**
```text
winlogbeat.event_data.Channel: "Security"
```

**Filter by event ID (e.g., logon events):**
```text
winlogbeat.event_data.EventID: 4624
```

**Filter by error level:**
```text
event.severity: "critical"
```

## Useful Commands

### Check Winlogbeat Service Status

```powershell
Get-Service winlogbeat
```

### Restart Winlogbeat Service

```powershell
Restart-Service winlogbeat
```

### Stop Winlogbeat Service

```powershell
Stop-Service winlogbeat
```

### Test Configuration

```powershell
cd "C:\Program Files\Winlogbeat"
.\winlogbeat.exe test config -e
```

### Test Output Connection

```powershell
.\winlogbeat.exe test output -e
```

### Run Winlogbeat in Foreground (for debugging)

```powershell
.\winlogbeat.exe -e
```

This will display logs in real-time and help identify connectivity or configuration issues.

### View Registered Event Log Inputs

After running `.\winlogbeat.exe setup -e`, the following event logs are registered for collection:

```text
Application
System
Security
```

### Uninstall Service (if needed)

```powershell
PowerShell.exe -ExecutionPolicy Bypass -File .\uninstall-service-winlogbeat.ps1
```

## Troubleshooting

### Winlogbeat Service Will Not Start

**Problem:** The Winlogbeat service fails to start or stops immediately after starting.

**Solution:**
1. Run Winlogbeat in foreground mode to see error messages:
   ```powershell
   cd "C:\Program Files\Winlogbeat"
   .\winlogbeat.exe -e
   ```
2. Check the output for error messages related to:
   - Certificate not found
   - Connection refused
   - Authentication failure
3. Address the specific error and restart the service:
   ```powershell
   Start-Service winlogbeat
   ```

### Connection Errors to Elasticsearch

**Problem:** Logs like "connection refused" or "certificate verification failed"

**Solutions:**
1. Verify Elasticsearch is running on the Linux server:
   ```bash
   docker compose -f elastic-docker-tls.yml ps
   ```
2. Verify the IP address is correct (SEVER-IP)
3. Verify the CA certificate is in the correct location:
   ```powershell
   ls "C:\Program Files\Winlogbeat\ca.crt"
   ```
4. Verify firewall allows outbound connections to port 9200:
   ```powershell
   Test-NetConnection [SERVER_IP] -Port 9200
   ```
5. Test the output connection:
   ```powershell
   .\winlogbeat.exe test output -e
   ```

### Authentication Failures

**Problem:** Errors like "invalid username or password" or "401 Unauthorized"

**Solutions:**
1. Verify the credentials in `winlogbeat.yml` are correct
2. Verify the `winlogbeat_internal` user exists in Elasticsearch
3. Verify the user has the correct role assignment
4. Regenerate the password and update the configuration file
5. Test the output connection:
   ```powershell
   .\winlogbeat.exe test output -e
   ```

### No Data Appearing in Kibana

**Problem:** Winlogbeat is running but no data appears in Kibana after 60 seconds

**Solutions:**
1. Wait 60-120 seconds for initial indexing
2. Verify the data view `winlogbeat-*` exists in Kibana
3. Check Winlogbeat is collecting logs:
   ```powershell
   .\winlogbeat.exe -e
   ```
   Look for lines like:
   ```text
   2024-01-15T10:30:45Z INFO Harvester started for file
   ```
4. Verify Elasticsearch is receiving data:
   ```text
   curl -k -u [ELASTICSEARCH_USERNAME]:[ELASTICSEARCH_PASSWORD] https://[SERVER_IP]:9200/_cat/indices?v
   ```
   Look for indices like `winlogbeat-*`

### Service Status Issues

**Problem:** Service shows as "Running" but logs are not being sent

**Solutions:**
1. Restart the service:
   ```powershell
   Restart-Service winlogbeat
   ```
2. Check service logs:
   ```powershell
   Get-EventLog -LogName Application -Source winlogbeat -Newest 10
   ```
3. Run in foreground to identify specific errors:
   ```powershell
   Stop-Service winlogbeat
   .\winlogbeat.exe -e
   ```

### Certificate Errors

**Problem:** "certificate verification failed" or "certificate not found"

**Solutions:**
1. Verify the CA certificate file exists:
   ```powershell
   Test-Path "C:\Program Files\Winlogbeat\ca.crt"
   ```
2. Re-copy the certificate from the Linux server:
   ```powershell
   scp [LINUX_USERNAME]@[SERVER_IP]:/home/[LINUX_USERNAME]/Elastic-Stack/ElasticMN/exported-certs/ca.crt `
     "C:\Program Files\Winlogbeat\ca.crt"
   ```
3. Verify certificate has correct permissions (should be readable)
4. Verify the path in `winlogbeat.yml` uses double backslashes: `C:\\Program Files\\Winlogbeat\\ca.crt`
5. Restart the service:
   ```powershell
   Restart-Service winlogbeat
   ```

## Configuration File Security Note

The `winlogbeat.yml` configuration file contains sensitive information and should be protected:

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

After successful Winlogbeat deployment:

1. Monitor Windows Event Logs in Kibana
2. Create custom dashboards for security events
3. Set up alerts for critical events
4. Configure dashboards for Application events
5. Review System event logs for errors and warnings
6. Deploy Filebeat on additional endpoints for additional log sources
