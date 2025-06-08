![orches logo](https://raw.githubusercontent.com/orches-team/common/main/orches-logo-text.png)

# [orches](https://github.com/orches-team/orches) example deployment

Feel free to [fork this repository](https://github.com/orches-team/example/fork), or use the [minimal rootless template](https://github.com/orches-team/orches-config-rootless), or the [rootful template](https://github.com/orches-team/orches-config-rootful) to get started quickly.

## Quick Start

To run this example deployment with orches, execute the following commands:

```bash
loginctl enable-linger $(whoami)

mkdir -p ~/.config/orches ~/.config/containers/systemd

podman run --rm -it --userns=keep-id --pid=host --pull=newer \
  --mount \
    type=bind,source=/run/user/$(id -u)/systemd,destination=/run/user/$(id -u)/systemd \
  -v ~/.config/orches:/var/lib/orches \
  -v ~/.config/containers/systemd:/etc/containers/systemd  \
  --env XDG_RUNTIME_DIR=/run/user/$(id -u) \
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

## Enabling Extra Services

Some services are included but disabled by default. To enable them:

1. **Fork this repository** to your own GitHub account.
2. **Clone your fork** locally and make the desired changes (such as adding or modifying configuration files for the extra services).
3. **Push your changes** to your forked repository.
4. On your orches host, **switch orches to use your fork** by running:

  ```bash
  podman exec systemd-orches orches switch <YOUR_FORK_URL>
  ```

5. orches will automatically apply the changes and start the newly enabled services.

For more details, see the [orches documentation](https://github.com/orches-team/orches#readme).

### Disabled Services

- **vaultwarden** – Self-hosted password manager
- **pi-hole** – Network-wide ad blocker

#### Enabling vaultwarden
To enable **vaultwarden** with HTTPS using Caddy, enable the required unit files:

```bash
mv caddy.container.disabled caddy.container
mv main.network.disabled main.network
mv vaultwarden.container.disabled vaultwarden.container
```

2. **Edit the `Caddyfile`** and replace `{YOUR_IP_ADDRESS}` with your node's actual IP address.
  > **Note:** Using a domain name is recommended for production, but this quick start uses your IP for simplicity.

  Example diff for `Caddyfile`:

  ```diff
  {
  -    default_sni {YOUR_IP_ADDRESS}
  +    default_sni 192.168.1.42
  }

  -https://{YOUR_IP_ADDRESS} {
  +https://192.168.1.42 {
      reverse_proxy systemd-vaultwarden:80
  }
  ```

3. **Sync orches** to apply the changes:

  ```bash
  podman exec systemd-orches orches sync
  ```

You can now access vaultwarden at `https://<YOUR_IP_ADDRESS>:4443/`.

#### Enabling pi-hole

To enable **pi-hole** (which requires binding to port 53):

1. **Allow unprivileged users to bind to port 53** (required for rootless Podman):
Allow non-root users to bind to port 53 and above by running:

```bash
echo "net.ipv4.ip_unprivileged_port_start=53" | sudo tee /etc/sysctl.d/50-unprivileged-ports.conf
sudo sysctl --system
```

2. **Remove the `.disabled` suffix** from the following file:
  - `pihole.container.disabled` → `pihole.container`

3. **Sync orches** to apply the changes:

  ```bash
  podman exec systemd-orches orches sync
  ```

Pi-hole should now be running and accessible on your server. The dashboard is available at http://<YOUR_IP_ADDRESS>:8082/.
