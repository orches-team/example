![orches logo](https://raw.githubusercontent.com/orches-team/common/main/orches-logo-text.png)

# [orches](https://github.com/orches-team/orches) example deployment

Feel free to [fork this repository](https://github.com/orches-team/example/fork), or use the [minimal rootless template](https://github.com/orches-team/orches-config-rootless), or the [rootful template](https://github.com/orches-team/orches-config-rootful) to get started quickly.

## Quick Start

To run this example deployment with orches, execute the following commands:

```bash
loginctl enable-linger (whoami)

mkdir -p ~/.config/orches ~/.config/containers/systemd

podman run --rm -it --userns=keep-id --pid=host --pull=newer \
  --mount \
    type=bind,source=/run/user/(id -u)/systemd,destination=/run/user/(id -u)/systemd \
  -v ~/.config/orches:/var/lib/orches \
  -v ~/.config/containers/systemd:/etc/containers/systemd  \
  --env XDG_RUNTIME_DIR=/run/user/(id -u) \
  ghcr.io/orches-team/orches init \
  https://github.com/orches-team/example.git
```

This will initialize orches with the contents of this repository and start all defined services as user units.

## Included Services

The following services are installed and managed by default:

### [forgejo](https://forgejo.org/)
Self-hosted Git service

**Ports:** web: `8080`, ssh: `2222`

### [grafana](https://grafana.com/)
Analytics and monitoring dashboard

**Port:** `8081`

### [homarr](https://homarr.dev/)
Simple and customizable homepage for your server

**Port:** `7575`

### [homeassistant](https://www.home-assistant.io/)
Open-source home automation platform

**Port:** `8123`

### [jellyfin](https://jellyfin.org/)
Media server for streaming movies, TV, music, and more

**Port:** `8096`

### [syncthing](https://syncthing.net/)
Continuous file synchronization

**Ports:** web: `8384`, sync: `22000/tcp+udp`, discovery: `21027/udp`

### [uptime-kuma](https://github.com/louislam/uptime-kuma)
Self-hosted monitoring tool

**Port:** `3001`

You can customize this deployment by editing or adding unit files in this repository. For more information, see the [orches documentation](https://github.com/orches-team/orches#readme).
