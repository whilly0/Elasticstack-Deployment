# Elasticstack-Deployment

# Centralized Windows Log Monitoring with Elastic Stack

This project documents the installation and deployment of a centralized log collection and monitoring environment using Docker, Elasticsearch, Kibana, Winlogbeat, and Filebeat.

The project began with installing Docker on a Linux server. Docker was then used to deploy a three-node Elasticsearch cluster and a Kibana instance. After verifying that the Elastic Stack was healthy and reachable, Winlogbeat and Filebeat were installed on a Windows 11 endpoint.

Winlogbeat was used to collect Windows Application, System, and Security Event Logs. Filebeat was used to collect custom logs from selected local folders. Both Beats were configured to send logs to Elasticsearch, where they could be searched, filtered, and analyzed in Kibana.

> Configuration files were included in this repository. They sensitive information such as passwords, usernames, certificate paths, and environment-specific values have been replaced with safe placeholders so you edit that part when making use of the configurations.

## Project Objectives

The objectives of this project were to:

- Install Docker on a Linux server.
- Deploy Elasticsearch and Kibana using Docker containers.
- Create a multi-node Elasticsearch cluster.
- Verify Elasticsearch cluster health and master node election.
- Install Winlogbeat on a Windows endpoint.
- Collect Windows Application, System, and Security Event Logs.
- Install Filebeat on the Windows endpoint.
- Collect custom logs from local Windows folders.
- Send logs securely to Elasticsearch.
- Use Kibana to search, filter, and analyze centralized logs.
- Troubleshoot connectivity and service issues.

## Architecture

```text
Windows Endpoint: CYBERDEMI
│
├── Winlogbeat
│   ├── Application Event Logs
│   ├── System Event Logs
│   └── Security Event Logs
│
├── Filebeat
│   ├── LogsToCloud custom log folder
│   └── LogsTrial custom log folder
│
└────────────── Secure connection ──────────────┐
                                                 │
                                                 v
Linux Elastic Server: 192.168.10.140
│
├── Docker
│   ├── Elasticsearch Cluster
│   │   ├── es01
│   │   ├── es02
│   │   └── es03
│   │
│   └── Kibana
│       └── kib01
│
└── Kibana Web Interface
    └── Port 5601
```

## Technologies Used

| Technology | Purpose |
|---|---|
| Linux | Operating system used to host Docker and the Elastic Stack |
| Docker | Used to run Elasticsearch and Kibana in containers |
| Docker Compose | Used to define and manage the Elastic Stack containers |
| Elasticsearch | Stores, indexes, and searches logs |
| Kibana | Used to visualize, search, filter, and analyze logs |
| Winlogbeat | Collects Windows Event Logs |
| Filebeat | Collects custom text-based log files |
| Windows 11 | Operating system of the monitored endpoint |
| PowerShell | Used to configure, test, manage, and troubleshoot Beats |

## Environment Details

| Component | Details |
|---|---|
| Linux Elastic Server IP | `YOUR_ELASTICSEARCH_HOST` |
| Windows endpoint hostname | `CYBERDEMI` |
| Windows operating system | Windows 11 Pro |
| Elasticsearch port | `YOUR_ELASTICSEARCH_HOST_PORT` |
| Kibana port | `YOUR_KIBANA_PORT` |
| Winlogbeat version | `8.12.0` |
| Elasticsearch nodes | `es01`, `es02`, `es03` |
| Kibana container | `kib01` |

# Docker Installation

Docker was installed on the Linux server before deploying the Elastic Stack.

Docker was chosen because it allows applications to run inside isolated containers. This made it easier to deploy Elasticsearch and Kibana without installing them directly on the Linux operating system.

The following components were installed:

- Docker Engine
- Docker Compose plugin or Docker Compose
- Elasticsearch Docker images
- Kibana Docker image

After installation, Docker was enabled and started as a system service.

Docker installation was verified using:

```bash
docker --version
```

Docker Compose availability was verified using:

```bash
docker compose version
```

The Docker service status was checked using:

```bash
sudo systemctl status docker
```

Docker needed to be active before Elasticsearch and Kibana containers could be started.

# Elastic Stack Deployment

After Docker installation, the Elastic Stack was deployed using Docker containers.

The Elastic Stack deployment included:

- A three-node Elasticsearch cluster.
- One Kibana instance.
- Docker networking for communication between containers.
- Persistent storage for Elasticsearch data.
- Security settings for Elasticsearch and Kibana.
- TLS/SSL certificates for secure communication.

The Elastic Stack project directory was located at:

```text
~/Elastic-Stack/ElasticMN
```

The Elasticsearch cluster contained three nodes:

```text
es01
es02
es03
```

Kibana was deployed in a container named:

```text
kib01
```

A multi-node Elasticsearch cluster was used to simulate a more realistic production environment. The nodes communicate with one another, maintain cluster state, and elect one node as the active master node.

# Docker Compose Deployment

A Docker Compose file was used to define and manage the Elastic Stack services.

The Docker Compose deployment handled:

- Creating Elasticsearch containers.
- Creating the Kibana container.
- Creating internal Docker networks.
- Publishing Elasticsearch port `9200`.
- Publishing Kibana port `5601`.
- Applying environment variables.
- Mounting persistent storage.
- Configuring service startup and health checks.
- Configuring Elasticsearch and Kibana communication.

The containers were started from the project directory:

```bash
cd ~/Elastic-Stack/ElasticMN
```

The Elastic Stack was started in detached mode using:

```bash
docker compose up -d
```

Running Docker Compose in detached mode allowed Elasticsearch and Kibana to continue running in the background.

# Verifying Docker Containers

After starting the Elastic Stack, Docker containers were checked using:

```bash
docker ps
```

A formatted Docker command was also used to show container names, health, and exposed ports:

```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

The expected containers were:

```text
kib01
es01
es02
es03
```

The Docker output confirmed that:

- Kibana was running.
- Elasticsearch nodes were running.
- The primary Elasticsearch node was healthy.
- Elasticsearch was exposed on port `YOUR_ELASTICSEARCH_HOST_PORT`.
- Kibana was exposed on port `YOUR_KIBANA_HOST_PORT`.

The services were accessible through:

```text
Elasticsearch: [https://YOUR_ELASTICSEARCH_HOST](https://YOUR_ELASTICSEARCH_HOST)
Kibana:        [https://YOUR_KIBANA_HOST](https://YOUR_KIBANA_HOST)
```

# Checking Elasticsearch Cluster Health

After the Elasticsearch containers were started, the cluster was checked to confirm that all nodes were available and communicating correctly.

The cluster health check was used to verify:

- Elasticsearch availability.
- Cluster node status.
- Node roles.
- Cluster health.
- Master-node election.
- Communication between Elasticsearch nodes.

The active master node was checked using the Elasticsearch CAT API.

The master node is responsible for managing the Elasticsearch cluster state. Only one node is elected as the active master at a time.

When checking all Elasticsearch nodes, the node marked with an asterisk (`*`) in the master column represented the currently elected master node.

# Kibana Deployment and Validation

Kibana was deployed in the Docker container named:

```text
kib01
```

Kibana was exposed through port:

```text
5601
```

The Kibana API status endpoint was checked locally from the Linux server to verify that Kibana was healthy and available.

The response confirmed that Kibana was available:

```json
{
  "status": {
    "overall": {
      "level": "available"
    }
  }
}
```

The Kibana container state was also checked to confirm that it was running.

Kibana was then used as the web interface for:

- Searching collected logs.
- Creating data views.
- Filtering log events.
- Viewing Windows Event Logs.
- Viewing custom Filebeat log events.
- Investigating errors and warnings.

# Network Connectivity Verification

The Linux Elastic server used the following IP address:

```text
YOUR__HOST_IP
```

The server IP address was checked using:

```bash
hostname -I
```

The Windows endpoint was required to reach:

```text
YOUR_ELASTICSEARCH_HOST
```

for Elasticsearch communication and:

```text
YOUR_KIBANA_HOST
```

for Kibana communication.

Connectivity from Windows to Kibana was tested using PowerShell.

The test confirmed that:

- The Windows endpoint could reach the Linux server.
- Port `5601` was reachable.
- Docker was correctly publishing Kibana port `5601`.
- Kibana was available from the Windows machine.
- There was no network firewall issue preventing access.

# Winlogbeat Implementation

Winlogbeat was installed on the Windows 11 endpoint named:

```text
CYBERDEMI
```

Winlogbeat was used to collect Windows Event Logs and forward them to Elasticsearch.

The following Windows Event Log channels were selected:

- Application
- System
- Security

These channels were selected because they contain important system, security, application, and operational information.

## Winlogbeat Tasks Performed

The following tasks were carried out during the Winlogbeat implementation:

1. Installed Winlogbeat on the Windows endpoint.
2. Configured Winlogbeat to collect Application Event Logs.
3. Configured Winlogbeat to collect System Event Logs.
4. Configured Winlogbeat to collect Security Event Logs.
5. Configured secure communication with Elasticsearch.
6. Configured Kibana connectivity for setup-related tasks.
7. Tested the Winlogbeat configuration before running it.
8. Tested Winlogbeat output connectivity.
9. Started and restarted the Winlogbeat service.
10. Reviewed Winlogbeat logs for errors.
11. Confirmed that the Windows endpoint could reach Kibana.
12. Verified that Winlogbeat was registering the configured event log channels.

# Checking Winlogbeat Logs

Winlogbeat logs were reviewed to troubleshoot startup and connectivity issues.

The logs provided information about:

- Winlogbeat version.
- Host information.
- Windows operating system details.
- Elasticsearch URL.
- Kibana URL.
- Configuration path.
- Data path.
- Log path.
- Registered Windows Event Log channels.
- Service startup events.
- Service stop events.
- Connection issues.
- Certificate warnings.
- Publishing and indexing status.

The Winlogbeat logs confirmed that the following event log inputs were registered:

```text
Application
System
Security
```

The logs were also used to identify and investigate a Kibana connection issue.

# Kibana Connection Troubleshooting

Winlogbeat initially reported a connection issue when attempting to communicate with Kibana.

The following troubleshooting steps were carried out:

1. Checked the Elastic server IP address.
2. Checked that the Kibana Docker container was running.
3. Verified that Docker exposed port `YOUR_KIBANA_HOST_PORT`.
4. Checked the Kibana API status locally on the Linux server.
5. Tested connectivity from the Windows endpoint to port `YOUR_KIBANA_HOST_PORT`.
6. Accessed the Kibana status API from the Windows endpoint.
7. Confirmed that Kibana was available.

The final tests confirmed that:

- Kibana was running.
- The Kibana API returned an available status.
- The Windows endpoint could access Kibana.
- Port `YOUR_KIBANA_HOST_PORT` was reachable.
- The Linux server IP address was correct.

# Filebeat Implementation

Filebeat was installed on the same Windows endpoint.

Filebeat was used to collect custom text-based log files from local Windows directories and forward those logs to Elasticsearch.

The following folders were configured as custom log sources:

```text
C:\Program Files\Filebeat\LogsToCloud
```

```text
C:\Users\OFFICIAL USE\Downloads\LogsTrial
```

Filebeat used the `filestream` input type to monitor and collect files from these locations.

## Filebeat Tasks Performed

The following tasks were carried out during the Filebeat implementation:

1. Installed Filebeat on the Windows endpoint.
2. Created Filebeat log inputs for custom directories.
3. Used the `filestream` input type for file monitoring.
4. Configured unique input identifiers.
5. Configured Filebeat to monitor files in subdirectories.
6. Added custom fields to identify each log source.
7. Tested the Filebeat configuration syntax.
8. Tested Filebeat output connectivity.
9. Ran Filebeat in the foreground for troubleshooting.
10. Created test events in monitored files.
11. Confirmed that custom logs could be identified in Kibana.

# Log Source Identification

Custom fields were added to Filebeat events to identify the origin of each log.

This allowed logs from different folders to be distinguished in Kibana.

The custom log sources included:

- `cloud_logs` for logs collected from the `LogsToCloud` folder.
- `Logs_Trial` for logs collected from the `LogsTrial` folder.

This approach makes it possible to send multiple kinds of logs to Elasticsearch while still filtering and investigating each source independently.

# Kibana Data Views

Kibana data views were used to access and analyze the collected logs.

Data views can be used for:

- Winlogbeat Windows Event Logs.
- Filebeat custom log files.
- Logs from the Windows host `CYBERDEMI`.
- Windows Security events.
- Windows System events.
- Windows Application events.
- Logs from the `LogsToCloud` directory.
- Logs from the `LogsTrial` directory.

Useful Kibana investigations include:

- Checking Windows security activity.
- Reviewing failed sign-in events.
- Viewing application errors.
- Monitoring system warnings.
- Investigating service failures.
- Filtering events by hostname.
- Filtering Filebeat events by source.
- Reviewing recent events by timestamp.

# Validation Results

The following results were confirmed during the implementation:

- Docker was installed and running on the Linux server.
- Docker Compose was available for service deployment.
- Elasticsearch and Kibana were deployed successfully with Docker.
- The Elasticsearch cluster contained three nodes.
- The primary Elasticsearch node was healthy.
- Kibana was running and accessible on port `5601`.
- The Linux server was reachable at `192.168.10.140`.
- The Windows endpoint could access Kibana successfully.
- Winlogbeat was installed and configured on the Windows endpoint.
- Winlogbeat registered Application, System, and Security Event Logs.
- Filebeat was installed and configured for custom log collection.
- Filebeat monitored the two required custom log folders.
- Winlogbeat and Filebeat configurations could be tested before service startup.
- Logs could be centrally searched and analyzed in Kibana.



# Repository Structure

The repository will later use a structure similar to this:

```text
.
├── README.md
├── docs/
│   ├── architecture.md
│   ├── installation.md
│   ├── troubleshooting.md
│   └── screenshots/
├── docker/
│   └── README.md
├── winlogbeat/
│   └── README.md
├── filebeat/
│   └── README.md
└── .gitignore
```

Configuration files will be added after all sensitive information has been removed.

# Security Notice



```text
YOUR_ELASTICSEARCH_HOST
YOUR_KIBANA_HOST
YOUR_USERNAME
YOUR_PASSWORD
PATH_TO_CA_CERTIFICATE
```

# Author

This project was completed as a hands-on implementation of centralized Windows log collection and monitoring

## Documentation

- [Docker Installation](docker-installation.md)
- [Elasticsearch and Kibana Deployment](elasticsearch-installation.md)
- [Winlogbeat Setup](winlogbeat-setup.md)
- [Filebeat Setup](filebeat-setup.md)
