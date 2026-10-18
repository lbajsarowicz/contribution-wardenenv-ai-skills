# Global Services & URLs

## Service URLs (default domain: warden.test)

| Service | URL |
|---------|-----|
| Traefik Dashboard | https://traefik.warden.test |
| Webmail (Mailpit) | https://webmail.warden.test |
| Portainer | https://portainer.warden.test |
| phpMyAdmin | https://phpmyadmin.warden.test |
| SSH Tunnel | `tunnel.warden.test:2222` (for DB clients) |

Per-project: RabbitMQ, Elasticsearch, OpenSearch at `https://{service}.{project}.test`

## SSH Tunnel (Database Clients)

Warden runs an SSH tunnel service for connecting external database clients (TablePlus, Sequel Pro, PhpStorm, DBeaver, etc.) to project database containers.

| Setting | Value |
|---------|-------|
| SSH Host | `tunnel.warden.test` |
| SSH Port | `2222` |
| SSH User | `user` |
| SSH Key | `~/.warden/tunnel/ssh_key` |
| MySQL Host | container name from `warden env ps` |
| MySQL Port | `3306` |

Auto-configured during `warden install`. No manual setup needed.

## Global Configuration (~/.warden/.env)

| Variable | Default | Description |
|----------|---------|-------------|
| `WARDEN_SERVICE_DOMAIN` | warden.test | Base domain for global services |
| `WARDEN_DNSMASQ_ENABLE` | 1 | Enable DNS for .test domains |
| `WARDEN_PORTAINER_ENABLE` | 0 | Enable Portainer UI |
| `WARDEN_PHPMYADMIN_ENABLE` | 1 | Enable phpMyAdmin |
| `WARDEN_MUTAGEN_ENABLE` | 1 (macOS) / 0 (Linux) | Mutagen file sync |

## Changing the Global Domain

1. Edit `~/.warden/.env`
2. Set `WARDEN_SERVICE_DOMAIN=your-domain.local`
3. Restart: `warden svc down && warden svc up -d`
4. Update DNS resolver if needed (e.g. `/etc/resolver/local` on macOS)

## Email (Mailpit)

No SMTP configuration needed. Warden configures sendmail to route to Mailpit. All emails appear at https://webmail.warden.test.

For Magento: default sendmail transport works. No SMTP module required.
