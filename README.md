# Docker-in-Docker for Cursor Cloud Agents

This repository demonstrates how to configure Docker-in-Docker (DinD) for use with Cursor Cloud Agents, enabling agents to build and run Docker containers within their own containerized environment.

To use: Fork this repo, and ask a new cloud agent to run `docker ps`.

> [!IMPORTANT]
> This was created in December 2025. Cursor cloud agents are evolving quickly, so this may become outdated.

## Overview

Cursor Cloud Agents run in isolated containers. This example shows how to set up Docker inside the agent container so that agents can:

- Build Docker images
- Run Docker containers
- Use Docker Compose
- Execute any Docker commands needed for development workflows

## Architecture

The setup consists of three key components:

### 1. Dockerfile (`.cursor/Dockerfile`)

The Dockerfile creates an Ubuntu-based image with:

- **Docker CE** - The Docker engine installed from official Docker repositories
- **Docker Compose Plugin** - For multi-container applications
- **Development tools** - Build essentials, git, python, and other common utilities
- **User permissions** - The `ubuntu` user is added to the `docker` group and granted sudo access

### 2. Install Script (`.cursor/install.sh`)

The install script configures Docker to work properly in a containerized environment:

- **Disables iptables** - Prevents conflicts with the host's network stack
- **Uses VFS storage driver** - More compatible for nested container scenarios
- **Handles Docker daemon lifecycle** - Properly stops and starts Docker service
- **Fixes group permissions** - Ensures Docker commands work without requiring explicit sudo

The script is idempotent, meaning it can be run multiple times safely. Cursor will snapshot the container after running this script.

### 3. Environment Configuration (`.cursor/environment.json`)

Defines the Cursor Cloud Agent environment:

- **Build context** - Points to the `.cursor` directory
- **Install script** - References the `install.sh` script
- **Terminal commands** - Pre-configured terminal for running `docker compose up`

## How It Works

1. **Container Build**: Cursor builds the Docker image using the Dockerfile in `.cursor/`
2. **Installation**: The `install.sh` script runs inside the container to configure Docker
3. **Snapshot**: Cursor snapshots the configured container for reuse
4. **Agent Execution**: Agents can now use Docker commands directly

## Usage

### Basic Docker Commands

Once the environment is set up, agents can use Docker commands normally:

```bash
docker ps
docker build -t myapp .
docker run myapp
```

### Docker Compose

The environment includes a pre-configured terminal for Docker Compose:

```bash
docker compose up
```

Or use compose commands directly:

```bash
docker compose build
docker compose down
```

## Key Configuration Details

### Docker Daemon Settings

The install script configures Docker with these critical settings:

```json
{
  "iptables": false,
  "storage-driver": "vfs"
}
```

- **`iptables: false`** - Required because the container doesn't have full network control
- **`storage-driver: vfs`** - More compatible for nested containers, though slower than overlay2

### Group Permissions

The Dockerfile adds the `ubuntu` user to the `docker` group. However, since group membership changes don't take effect in running shells, the install script adds a wrapper function to `~/.bashrc`:

```bash
docker() { sudo -g docker docker "$@"; }
```

This ensures Docker commands work correctly without requiring explicit `sudo docker`.

## Limitations

- **Performance**: The VFS storage driver is slower than overlay2 but necessary for compatibility
- **Network isolation**: Docker-in-Docker containers share the host's network namespace
- **Resource limits**: Subject to the Cursor Cloud Agent container's resource constraints

## Troubleshooting

### Docker daemon not starting

If Docker fails to start, check:

- The install script has run successfully
- No conflicting Docker processes are running
- The daemon configuration file exists at `/etc/docker/daemon.json`

### Permission denied errors

If you see permission errors:

- Ensure the install script has run (it sets up the docker wrapper function)
- Try running `sudo docker` instead

## Example Use Cases

This setup enables agents to:

1. **Build and test Docker images** during development
2. **Run multi-container applications** with Docker Compose
3. **Execute CI/CD-like workflows** that require Docker
4. **Test containerized applications** without leaving the agent environment
5. **Manage containerized dependencies** for development

## See Also

- [Cursor Cloud Agents Documentation](https://cursor.sh/docs)
- [Docker-in-Docker Documentation](https://hub.docker.com/_/docker)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
