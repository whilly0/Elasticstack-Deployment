
# Docker Installation

For this project, I installed Docker on my Linux server before deploying the Elastic Stack.

I used Docker because it allowed me to run Elasticsearch and Kibana inside isolated containers instead of installing them directly on the Linux operating system. This made the environment easier to manage, start, stop, troubleshoot, and recreate.

## Updating the Server

Before installing Docker, I updated the package list on the Linux server.

```bash
sudo apt update
sudo apt upgrade -y
```

This ensured that the server had the latest available package information and updates before Docker installation.



## Starting Docker

After installation, I enabled Docker so that it starts automatically whenever the Linux server reboots.

```bash
sudo systemctl enable docker
```

I started the Docker service:

```bash
sudo systemctl start docker
```

I checked the Docker service status:

```bash
sudo systemctl status docker
```

A successful installation should show that the Docker service is active and running.

## Verifying Docker Installation

I verified the Docker installation by checking the Docker version:

```bash
docker --version
```

I also checked the Docker Compose version:

```bash
docker compose version
```

To confirm that Docker could download and run a container, I ran the following test:

```bash
sudo docker run hello-world
```

The `hello-world` container confirmed that Docker was working correctly.


## Docker Installation Validation

After completing the installation, I confirmed that:

- Docker Engine was installed successfully
- Docker Compose was installed successfully
- The Docker service was running
- Docker was enabled to start automatically after a server reboot
- My user account could run Docker commands
- Docker could successfully download and run containers
- The Linux server was ready for the Elastic Stack deployment
