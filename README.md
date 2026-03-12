# ansible-role-nginx

Installs and configures vanilla Nginx web server on Debian/Ubuntu.

## Requirements

- Debian 11+ or Ubuntu 20.04+
- Ansible 2.14+

## Role Variables

### Worker

| Variable | Default | Description |
|---|---|---|
| `nginx_worker_processes` | `"auto"` | Worker processes (`auto` = CPU cores) |
| `nginx_worker_connections` | `1024` | Max simultaneous connections per worker |
| `nginx_multi_accept` | `"on"` | Accept all new connections at once |
| `nginx_use_method` | `epoll` | Connection processing method (optimal for Linux) |

### HTTP

| Variable | Default | Description |
|---|---|---|
| `nginx_server_tokens` | `"off"` | Hide nginx version in headers and error pages |
| `nginx_client_max_body_size` | `"10m"` | Max allowed client request body size |
| `nginx_keepalive_timeout` | `65` | Keepalive timeout (seconds) |
| `nginx_keepalive_requests` | `1000` | Max requests per keepalive connection |
| `nginx_server_names_hash_bucket_size` | `64` | Hash bucket size (increase for long domain names) |

### SSL

| Variable | Default | Description |
|---|---|---|
| `nginx_ssl_protocols` | `"TLSv1.2 TLSv1.3"` | Allowed TLS versions |
| `nginx_ssl_prefer_server_ciphers` | `"on"` | Prefer server cipher order |

### Logging

| Variable | Default | Description |
|---|---|---|
| `nginx_access_log` | `"/var/log/nginx/access.log"` | Access log path |
| `nginx_error_log` | `"/var/log/nginx/error.log"` | Error log path |
| `nginx_error_log_level` | `"warn"` | Error log level (`debug\|info\|notice\|warn\|error\|crit`) |
| `nginx_access_log_format` | `"main"` | Log format name (`main\|json\|""` for combined) |
| `nginx_logrotate_rotate` | `7` | Number of rotations to keep |
| `nginx_logrotate_maxsize` | `"500M"` | Rotate when log reaches this size |

### Gzip

| Variable | Default | Description |
|---|---|---|
| `nginx_gzip` | `"on"` | Enable gzip compression |
| `nginx_gzip_comp_level` | `6` | Compression level (1–9) |
| `nginx_gzip_types` | `[text/plain, text/css, ...]` | MIME types to compress |

### Default virtual host

| Variable | Default | Description |
|---|---|---|
| `nginx_config_deploy` | `true` | Deploy `nginx.conf` from template on every run |
| `nginx_default_vhost_enabled` | `true` | Deploy and enable the default static vhost |
| `nginx_default_vhost_port` | `80` | Listen port for default vhost |
| `nginx_default_vhost_root` | `/var/www/html` | Document root |
| `nginx_default_vhost_server_name` | `"_"` | `server_name` (`_` = catch-all) |
| `nginx_default_vhost_access_log` | `"/var/log/nginx/access.log"` | Access log for default vhost |
| `nginx_default_vhost_error_log` | `"/var/log/nginx/error.log"` | Error log for default vhost |

### HTTP → HTTPS redirect

| Variable | Default | Description |
|---|---|---|
| `nginx_redirect_http_to_https` | `false` | Deploy redirect vhost (301 all HTTP → HTTPS) |
| `nginx_redirect_http_port` | `80` | Port to listen on for HTTP redirect |

> **Note:** `nginx_redirect_http_to_https: true` and `nginx_default_vhost_enabled: true` on the same
> port will cause a `duplicate default_server` conflict — the role will fail with a clear error message.
> Set `nginx_default_vhost_enabled: false` or change `nginx_default_vhost_port` when enabling redirect.

### Service

| Variable | Default | Description |
|---|---|---|
| `nginx_service_name` | `nginx` | systemd service name |
| `nginx_service_enabled` | `true` | Enable at boot |
| `nginx_service_state` | `started` | Service state |

### Advanced

| Variable | Default | Description |
|---|---|---|
| `nginx_extra_conf_options` | `""` | Extra directives injected into the global block |
| `nginx_extra_events_conf_options` | `""` | Extra directives injected into `events {}` |
| `nginx_extra_http_options` | `""` | Extra directives injected into `http {}` before vhost includes |
| `nginx_proxy_cache_path_enabled` | `false` | Enable `proxy_cache_path` directive |
| `nginx_proxy_cache_path` | `""` | Full `proxy_cache_path` value (e.g. `"/tmp/cache levels=1:2 ..."`) |

## Example Playbook

```yaml
- hosts: webservers
  roles:
    - role: ansible-role-nginx
```

### Custom settings

```yaml
- hosts: webservers
  roles:
    - role: ansible-role-nginx
      vars:
        nginx_worker_processes: "2"
        nginx_default_vhost_port: 8080
        nginx_default_vhost_root: /var/www/mysite
```

### HTTP → HTTPS redirect (disable default vhost to avoid conflict)

```yaml
- hosts: webservers
  roles:
    - role: ansible-role-nginx
      vars:
        nginx_redirect_http_to_https: true
        nginx_default_vhost_enabled: false
```

### Used as dependency (disable default vhost)

```yaml
- hosts: vpn_servers
  roles:
    - role: ansible-role-nginx
      vars:
        nginx_default_vhost_enabled: false
```

## Notes

- Designed to work standalone or as a dependency for other roles (e.g. `ansible-role-remnawave`)
- When used with `ansible-role-remnawave`, set `nginx_default_vhost_enabled: false` — remnawave deploys its own fallback vhost on `127.0.0.1:8080`
- `/var/www/html/index.html` is created only on fresh install (`force: false`) — safe to run repeatedly
- `nginx.conf` is deployed on every run when `nginx_config_deploy: true`; config is validated with `nginx -t` after deploy
