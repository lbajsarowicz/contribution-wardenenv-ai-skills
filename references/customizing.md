# Customizing Warden Environments

## How `.warden/warden-env.yml` Works

Warden assembles a Docker Compose stack from multiple partial YAML files. The `.warden/warden-env.yml` file in the project root is loaded **last**, giving it the highest merge precedence. Docker Compose merge semantics apply: later definitions override earlier ones.

### Load Order

1. Built-in partials (`environments/includes/` and `environments/{type}/`)
2. User home overrides (`~/.warden/environments/`)
3. Project overrides (`.warden/environments/`)
4. **`.warden/warden-env.yml`** — final override
5. **`.warden/warden-env.{darwin|linux}.yml`** — OS-specific final override

Preview the merged result: `warden env config`

## Custom PHP Image

Override the default PHP-FPM image (e.g., to use a custom registry or add extensions at build time):

```yaml
# .warden/warden-env.yml
services:
  php-fpm:
    image: my-registry/custom-php:8.3
  php-debug:
    image: my-registry/custom-php:8.3-debug
```

## PHP Configuration Override

Mount a custom `php.ini` to change runtime settings:

```yaml
# .warden/warden-env.yml
services:
  php-fpm:
    volumes:
      - ./.warden/php/zz-config.ini:/etc/php.d/zz-config.ini
  php-debug:
    volumes:
      - ./.warden/php/zz-config.ini:/etc/php.d/zz-config.ini
```

Example `.warden/php/zz-config.ini`:

```ini
memory_limit = 4G
max_execution_time = 600
error_reporting = E_ALL & ~E_DEPRECATED
```

Default PHP settings (overridable):

| Setting | Default |
|---------|---------|
| `date.timezone` | `UTC` |
| `max_execution_time` | `3600` |
| `max_input_vars` | `10000` |
| `memory_limit` | `2G` |
| `display_errors` | `On` |
| `post_max_size` | `25M` |
| `upload_max_filesize` | `25M` |

## Nginx Configuration Override

Mount a custom Nginx config included after default configs:

```yaml
# .warden/warden-env.yml
services:
  nginx:
    volumes:
      - ./.warden/nginx/custom.conf:/etc/nginx/default.d/custom.conf
```

The custom config is included via: `include /etc/nginx/default.d/*.conf`

## Multiple Domains

Add extra Traefik routing labels for additional domains:

```yaml
# .warden/warden-env.yml
services:
  nginx:
    labels:
      - traefik.http.routers.${WARDEN_ENV_NAME}-extra.rule=Host(`extra.${TRAEFIK_DOMAIN}`)
      - traefik.http.routers.${WARDEN_ENV_NAME}-extra.tls=true
```

## Adding Extra Services

Define additional services not included by default:

```yaml
# .warden/warden-env.yml
services:
  adminer:
    image: adminer:latest
    labels:
      - traefik.enable=true
      - traefik.http.routers.${WARDEN_ENV_NAME}-adminer.rule=Host(`adminer.${TRAEFIK_DOMAIN}`)
      - traefik.http.routers.${WARDEN_ENV_NAME}-adminer.tls=true
```

## OS-Specific Overrides

Use platform-specific files for macOS/Linux differences:

- `.warden/warden-env.darwin.yml` — macOS only
- `.warden/warden-env.linux.yml` — Linux only

Common use case: different volume mount options or sync settings per platform.

## Agent Guidelines

When a user asks to customize their Warden environment:

1. Check if `.warden/warden-env.yml` already exists in the project
2. Use Docker Compose merge semantics (don't duplicate the full config, only override what's needed)
3. Always apply changes to both `php-fpm` and `php-debug` for PHP-related overrides
4. Suggest `warden env config` to preview the result before restarting
5. Restart with `warden env up -d` to apply changes
