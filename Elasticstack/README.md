# Elastic Stack Deployment Guide

## Overview

This guide documents the deployment of a three-node Elasticsearch cluster with Kibana using Docker Compose on a Linux server. The deployment includes TLS/SSL certificate generation, security configuration, and integration preparation for Winlogbeat and Filebeat log ingestion.

## Prerequisites

### System Requirements

- Linux server with Docker installed
- Docker Compose installed
- Sufficient disk space for Elasticsearch data volumes
- Network access to port 9200 (Elasticsearch) and 5601 (Kibana)
- Certificate files and configuration files ready (instances.yml, create-certs.yml, elastic-docker-tls.yml, .env)

### Verify Docker Installation

Before beginning, confirm that Docker and Docker Compose are installed:

```bash
docker --version
docker compose version
```

## Project Structure

Create the following directory structure for your Elastic Stack deployment:

```bash
mkdir -p ~/Elastic-Stack/ElasticMN
cd ~/Elastic-Stack/ElasticMN
```

Your project directory should contain:

```text
~/Elastic-Stack/ElasticMN/
├── instances.yml              # Certificate Subject Alternative Names configuration
├── create-certs.yml           # Certificate generation Docker Compose file
├── elastic-docker-tls.yml     # Main Elasticsearch cluster Docker Compose file
├── .env                       # Environment variables (Elasticsearch version, passwords)
└── exported-certs/            # Directory for CA certificates (created later)
```

## Step-by-Step Installation

### Step 1: Prepare Configuration Files

Copy the following files from your elasticstack folder to your project directory:

- `instances.yml`
- `create-certs.yml`
- `elastic-docker-tls.yml`
- `.env`

Navigate to the project directory:

```bash
cd ~/Elastic-Stack/ElasticMN
```

### Step 2: Create instances.yml (Certificate Subject Alternative Names)

The `instances.yml` file defines the certificate Subject Alternative Names (SANs) for all Elasticsearch and Kibana nodes.

This file should be in your project directory and include entries for:
- All three Elasticsearch nodes (es01, es02, es03)
- Kibana (kib01)
- Your server IP address 

### Step 3: Create create-certs.yml (Certificate Generation)

The `create-certs.yml` file defines a Docker Compose service that generates the required certificates for the Elastic Stack.

This file should be in your project directory and configure:
- The `create_certs` service
- Volume mounting for the `es_certs` Docker volume
- Certificate generation using Elasticsearch's built-in certificate tool

### Step 4: Run Certificate Generation

Generate TLS certificates using the `create-certs.yml` file:

```bash
docker compose -f create-certs.yml run --rm create_certs
```

Wait for the operation to complete. The certificates will be stored in the `es_certs` Docker volume.

**Output:** You should see the generated certificates without errors.

### Step 5: Verify Certificate Subject Alternative Names (IMPORTANT)

Before starting the cluster, verify that your server IP address is included in the certificates. This is critical for TLS connectivity.

#### Verify Elasticsearch Certificate SANs

```bash
docker run --rm -v es_certs:/certs:ro \
  docker.elastic.co/elasticsearch/elasticsearch:8.12.0 \
  bash -c "openssl x509 -in /certs/es01/es01.crt -noout -ext subjectAltName"
```

#### Verify Kibana Certificate SANs

```bash
docker run --rm -v es_certs:/certs:ro \
  docker.elastic.co/elasticsearch/elasticsearch:8.12.0 \
  bash -c "openssl x509 -in /certs/kib01/kib01.crt -noout -ext subjectAltName"
```

#### Expected Output

Both commands should include your server IP address:

```text
X509v3 Subject Alternative Name:
    DNS:es01, DNS:es02, DNS:es03, DNS:kib01, IP Address:
```

**Important:** If `your ip a` is not present, regenerate the certificates and update your `instances.yml` file to include the correct IP address.

### Step 6: Create elastic-docker-tls.yml (Main Cluster Configuration)

The `elastic-docker-tls.yml` file is your main Docker Compose configuration that defines:

- Three Elasticsearch container services (es01, es02, es03)
- One Kibana container service (kib01)
- Docker networks for internal communication
- Persistent volumes for data
- TLS/SSL certificate mounting
- Environment variables and security settings
- Port mappings for external access

This file should be in your project directory.

### Step 7: Create .env (Environment Configuration)

The `.env` file defines environment variables used by the Docker Compose deployment:

Key variables include:
- `VERSION` - Elasticsearch/Kibana version (e.g., 8.12.0)
- `COMPOSE_PROJECT_NAME` - 
- `CERTS_DIR` 


This file should be in your project directory.

### Step 8: Start the Elasticsearch Cluster

Start all services using Docker Compose:

```bash
docker compose -f elastic-docker-tls.yml up -d
```

The `-d` option starts containers in detached mode, allowing them to run in the background.

#### Check Cluster Status

Verify that all containers are running:

```bash
docker compose -f elastic-docker-tls.yml ps
```

Expected output:

```text
NAME      IMAGE                                    COMMAND                  STATUS
es01      docker.elastic.co/elasticsearch/elasticsearch:8.12.0   "/bin/tini -e 143 /usr/local/bin/docker-entrypoint.sh ..."   Up (healthy)
es02      docker.elastic.co/elasticsearch/elasticsearch:8.12.0   "/bin/tini -e 143 /usr/local/bin/docker-entrypoint.sh ..."   Up (healthy)
es03      docker.elastic.co/elasticsearch/elasticsearch:8.12.0   "/bin/tini -e 143 /usr/local/bin/docker-entrypoint.sh ..."   Up (healthy)
kib01     docker.elastic.co/kibana/kibana:8.12.0                "/bin/tini -e 143 /usr/local/bin/docker-entrypoint.sh ..."   Up
```

### Step 9: Verify Elasticsearch Connectivity

Verify that Elasticsearch is running (you will receive a 401 Unauthorized response, which is expected):

```bash
curl -vk https://localhost:9200
```

Expected response headers:

```text
< HTTP/1.1 401 Unauthorized
< www-authenticate: Basic realm="security" charset="UTF-8"
```

### Step 10: Verify Kibana Status

Check that Kibana is running and accessible:

```bash
curl -k https://localhost:5601/api/status
```

Expected response:

```json
{
  "status": {
    "overall": {
      "level": "available"
    }
  }
}
```

### Step 11: Generate Random Passwords for Built-in Users

Generate automatic passwords for all built-in Elasticsearch users:

```bash
docker compose exec es01 \
  bin/elasticsearch-setup-passwords auto
```

Alternatively, you can use `docker exec`:

```bash
docker exec -it es01 \
  /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
```

**Important:** Save the generated passwords. You will need them to log in to Kibana and Elasticsearch.

Expected output includes passwords for:
- elastic
- apm_system
- kibana_system
- kibana
- logstash_system
- beats_system

### Step 12: Export CA Certificate for Windows

Export the CA certificate to use with Winlogbeat and Filebeat on your Windows endpoint:

```bash
mkdir -p exported-certs
docker run --rm -v es_certs:/certs:ro \
  -v "$PWD/exported-certs:/export" alpine \
  cp /certs/ca/ca.crt /export/ca.crt
```

This creates an `exported-certs/ca.crt` file that can be transferred to your Windows server.

### Step 13: Set Up Users and Passwords

#### Initial Access

After generating passwords in Step 11, you can access Kibana:

- **URL:** https:YOUR_ELASTIC_SEVER_IP:5601
- **Username:** elastic
- **Password:** (from Step 11 output)

#### Create Dedicated Users

Create the following users for log ingestion:

1. **winlogbeat_internal** - For Winlogbeat log sending (Windows endpoint)
2. **winlogbeat_setup** - For Winlogbeat dashboard setup
3. **filebeat_internal** - For Filebeat log sending
4. **filebeat_setup** - For Filebeat dashboard setup

Use Kibana's Stack Management interface to create these users:

1. Navigate to **Stack Management > Users**
2. Click **Create user**
3. Set username and password
4. Assign appropriate roles based on user purpose:
   - For `*_internal` users: Assign `beats_admin` or `beats_system` role
   - For `*_setup` users: Assign `admin` or `beats_admin` role

Alternatively, use the Elasticsearch API to create users.

## Accessing the Elastic Stack

### Kibana

```text
URL:      https://YOUR_ELASTIC_SERVER_IP:5601
Username: elastic
Password: (generated in Step 11)
```

### Elasticsearch

```text
URL:      https://YOUR_ELASTIC_SERVER_IP:9200
Username: elastic
Password: (generated in Step 11)
```

## Managing the Deployment

### View Running Containers

```bash
docker compose -f elastic-docker-tls.yml ps
```

### View All Containers (including stopped)

```bash
docker ps -a
```

### Start the Elastic Stack

```bash
docker compose -f elastic-docker-tls.yml up -d
```

### Stop the Elastic Stack

```bash
docker compose -f elastic-docker-tls.yml down
```

### View Kibana Logs (last 100 lines)

```bash
docker logs kib01 --tail 100
```

### Monitor Kibana Logs Continuously

```bash
docker logs -f kib01
```

### View Elasticsearch Logs (es01)

```bash
docker logs es01 --tail 100
```

### Monitor Elasticsearch Logs Continuously

```bash
docker logs -f es01
```

### Check Individual Container Status

```bash
docker inspect --format='{{.State.Status}}' es01
docker inspect --format='{{.State.Status}}' kib01
```

### Execute Commands Inside a Container

Access Elasticsearch shell:

```bash
docker exec -it es01 /bin/bash
```

## Troubleshooting

### Certificate Issues

**Problem:** Certificate verification fails or IP address mismatch errors appear.

**Solution:**
1. Verify the `instances.yml` file includes your server IP (192.168.10.140)
2. Regenerate certificates:
   ```bash
   docker compose -f create-certs.yml run --rm create_certs
   ```
3. Verify the new certificates include the IP address (Step 5)
4. Restart the cluster:
   ```bash
   docker compose -f elastic-docker-tls.yml down
   docker compose -f elastic-docker-tls.yml up -d
   ```

### Elasticsearch Cluster Not Healthy

**Problem:** Containers are running, but the cluster is not healthy.

**Solution:**
1. Check container logs:
   ```bash
   docker logs es01
   docker logs es02
   docker logs es03
   ```
2. Verify all three nodes are connected:
   ```bash
   curl -k -u elastic:password https://localhost:9200/_cat/nodes?v
   ```
3. Check cluster status:
   ```bash
   curl -k -u elastic:password https://localhost:9200/_cluster/health?pretty
   ```

### Kibana Cannot Connect to Elasticsearch

**Problem:** Kibana logs show connection errors to Elasticsearch.

**Solution:**
1. Verify Elasticsearch is running:
   ```bash
   docker ps | grep es01
   ```
2. Check Kibana logs:
   ```bash
   docker logs -f kib01
   ```
3. Verify network connectivity:
   ```bash
   docker exec kib01 curl -k https://es01:9200
   ```
4. Restart Kibana:
   ```bash
   docker compose -f elastic-docker-tls.yml restart kib01
   ```

### High Memory Usage

**Problem:** Containers are using excessive memory.

**Solution:**
1. Adjust `MEM_LIMIT` in the `.env` file
2. Restart containers:
   ```bash
   docker compose -f elastic-docker-tls.yml restart
   ```

## Next Steps

After successful deployment:

1. **Install Winlogbeat** on your Windows endpoint(Winlogbeat)


