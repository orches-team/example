![orches logo](https://raw.githubusercontent.com/orches-team/common/main/orches-logo-text.png)

# orches-config-rootless: Rootless orches Template Repository

This repository provides a rootless configuration template for [orches](https://github.com/orches-team/orches), a simple git-ops tool for orchestrating [Podman](https://podman.io/) containers and systemd units on a single machine. It is designed for users who want to run orches and managed containers without root privileges, leveraging systemd user services and Podman rootless containers.

## Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Repository Structure](#repository-structure)
- [Customizing Your Deployment](#customizing-your-deployment)

## Overview

This repository contains sample unit files for running orches and a demo [Caddy](https://caddyserver.com/) webserver as rootless Podman containers. It is intended to be used as a starting point for your own orches deployments. You can fork this repository and add or modify unit files to manage your own containers and services.

## Quick Start

### Prerequisites
- Podman >= 4.4
- systemd (user mode)

Tested on Fedora 41, Ubuntu 24.04, and CentOS Stream 9/derivatives.

### 1. Enable systemd user lingering
```bash
loginctl enable-linger (whoami)
```

### 2. Create required directories
```bash
mkdir -p ~/.config/orches ~/.config/containers/systemd
```

### 3. Initialize orches with this repository
```bash
podman run --rm -it --userns=keep-id --pid=host --pull=newer \
  --mount \
    type=bind,source=/run/user/(id -u)/systemd,destination=/run/user/(id -u)/systemd \
  -v ~/.config/orches:/var/lib/orches \
  -v ~/.config/containers/systemd:/etc/containers/systemd  \
  --env XDG_RUNTIME_DIR=/run/user/(id -u) \
  ghcr.io/orches-team/orches init \
  https://github.com/orches-team/orches-config-rootless.git
```

### 4. Verify orches and caddy are running
```bash
systemctl --user status orches
systemctl --user status caddy
podman exec systemd-orches orches status
curl localhost:8080
```

## Repository Structure

- `orches.container`: Unit file for running orches as a rootless Podman container
- `caddy.container`: Example unit file for running a Caddy webserver

You can add more `.container` or `.service` files to manage additional containers or systemd services.

## Customizing Your Deployment

To add a new application (e.g., [Jellyfin](https://jellyfin.org/)):
1. Fork this repository and clone it locally.
2. Add a new unit file, e.g. `jellyfin.service`:

```ini
[Container]
Image=docker.io/jellyfin/jellyfin
Volume=config:/config:Z
Volume=cache:/cache:Z
Volume=media:/media:Z
PublishPort=8096:8096

[Install]
WantedBy=multi-user.target default.target
```

3. Commit and push your changes.
4. Switch orches to your fork:
```bash
podman exec systemd-orches orches switch <YOUR_FORK_URL>
```
