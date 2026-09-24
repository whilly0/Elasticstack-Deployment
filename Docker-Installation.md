# Docker Installation for Linux Server

## Quick Setup

### Update Package Manager

```bash
sudo apt update
sudo apt upgrade -y
```

### Install Docker

```bash
sudo apt install docker.io docker-compose -y
```

### Enable and Start Docker

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

### Verify Installation

```bash
docker --version
docker compose version
sudo docker run hello-world
```

## Setup Complete

The Linux server is ready to deploy the Elastic Stack with Docker.
