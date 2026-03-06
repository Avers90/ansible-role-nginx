# ansible-role-nginx

Installs and configures vanilla Nginx web server on Debian/Ubuntu.

## Requirements

- Debian 11+ or Ubuntu 20.04+
- Ansible 2.14+

## Role Variables

| Variable | Default | Description |
|---|---|---|
| `nginx_worker_processes` | `"auto"` | Worker processes (auto = CPU cores) |
| `nginx_worker_connections` | `1024` | Max connections per worker |
| `nginx_server_tokens` | `"off"` | Hide nginx version |
| `nginx_gzip_enabled` | `true` | Enable gzip compression |
| `nginx_default_vhost_enabled` | `true` | Deploy default virtual host |
| `nginx_default_vhost_root` | `/var/www/html` | Document root |
| `nginx_default_vhost_port` | `80` | Listen port |
| `nginx_default_vhost_server_name` | `"_"` | Server name (`default_server` if `_`) |
| `nginx_config_deploy` | `true` | Deploy nginx.conf from template |
| `nginx_client_max_body_size` | `"10m"` | Max request body size |
| `nginx_keepalive_timeout` | `65` | Keepalive timeout (seconds) |
| `nginx_service_name` | `nginx` | Service name |
| `nginx_service_enabled` | `true` | Enable at boot |
| `nginx_service_state` | `started` | Service state |

## Example Playbook

```yaml
- hosts: webservers
  roles:
    - role: ansible-role-nginx
```

### With custom settings

```yaml
- hosts: webservers
  roles:
    - role: ansible-role-nginx
      vars:
        nginx_worker_processes: "2"
        nginx_default_vhost_port: 8080
        nginx_default_vhost_root: /var/www/mysite
```

### Disable default vhost (when used as dependency)

```yaml
- hosts: vpn_servers
  roles:
    - role: ansible-role-nginx
      vars:
        nginx_default_vhost_enabled: false
```

## Notes

- Designed to work standalone or as a dependency for other roles (e.g. `ansible-role-marzban`)
- When used with `ansible-role-marzban`, the default vhost serves the decoy site on `127.0.0.1:8080`
- `/var/www/html/index.html` is created only on fresh install (`force: false`) — safe to run repeatedly
