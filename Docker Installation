
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

## Removing Old Docker Packages

I removed older Docker-related packages to avoid package conflicts with the Docker Engine version installed from Docker’s official repository.

```bash
sudo apt remove -y \
  docker.io \
  docker-doc \
  docker-compose \
  docker-compose-v2 \
  podman-docker \
  containerd \
  runc
```

## Installing Dependencies

I installed the dependencies required to download Docker packages securely.

```bash
sudo apt install -y \
  ca-certificates \
  curl
```

The `ca-certificates` package allows the server to validate secure HTTPS connections, while `curl` was used to download Docker’s official repository key.

## Adding Docker’s Official Repository

I added Docker’s official GPG key so that the Linux server could verify Docker packages before installation.

```bash
sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

I then added Docker’s official APT repository:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

After adding the repository, I updated the package list again:

```bash
sudo apt update
```

## Installing Docker Engine

I installed Docker Engine and the supporting Docker tools.

```bash
sudo apt install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

The installation included:

- Docker Engine, which runs containers
- Docker CLI, which is used to manage Docker from the terminal
- containerd, which manages container execution
- Docker Buildx, used for building Docker images
- Docker Compose plugin, used to deploy multiple containers together

Docker’s official installation method includes the Docker Engine, CLI, container runtime, Buildx plugin, and Compose plugin. [docs.docker](https://docs.docker.com/engine/install/ubuntu/)

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

## Configuring Docker Permissions

By default, Docker commands require `sudo`.

To allow my Linux user account to run Docker commands without using `sudo`, I added my account to the Docker group:

```bash
sudo usermod -aG docker $USER
```

After running this command, I logged out and logged back in for the group membership to take effect.

I could also activate the new group in the current terminal session with:

```bash
newgrp docker
```

I then tested Docker without `sudo`:

```bash
docker run hello-world
```

Adding a user to the Docker group allows that user to manage Docker containers without `sudo`. However, Docker group access is highly privileged and should only be granted to trusted users. [docs.docker](https://docs.docker.com/engine/install/linux-postinstall/)

## Docker Installation Validation

After completing the installation, I confirmed that:

- Docker Engine was installed successfully
- Docker Compose was installed successfully
- The Docker service was running
- Docker was enabled to start automatically after a server reboot
- My user account could run Docker commands
- Docker could successfully download and run containers
- The Linux server was ready for the Elastic Stack deployment
