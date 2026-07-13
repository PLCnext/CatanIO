## Running compose files with Podman

Podman provides built-in support for Docker Compose–style files using the `podman compose` command.

### Prerequisites
- Podman installed (Podman rootless is available in the firmware)
- A valid Compose file (`**-compose.yml` or `compose.yml`)

### Permissions
Configure container user permissions as follows:

```yaml
userns_mode: keep-id
user: 1002:1002
```

- The `admin` user has UID `1002`
- The `plcnext` group has GID `1002`
- `keep-id` ensures correct mapping between host and container users

**Important for PLCnext apps:**
- The `app_user` must use UID `1001`
- The group (`plcnext`) remains GID `1002`

### Manage Services
Run, stop, and manage all services defined in the Compose file:

```bash
podman compose up -d      # start services in background
podman compose down       # stop and remove all services
podman compose logs -f    # follow logs of all services
```

### Notes
- Podman Compose is mostly compatible with Docker Compose files.
- Volumes, networks, and environment variables are handled automatically.
- Ensure required host directories (e.g. bind mounts) exist before starting containers.

### Firewall
- If the system firewall is enabled, required ports for the containers must be explicitly allowed.
- Ensure that all exposed or required service ports (e.g. via `ports` or `network_mode: host`) are permitted in the firewall configuration.
- Otherwise, services inside the containers may not be reachable from outside the host system.