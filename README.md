# Send in Docker compose
This repository provides a basic Docker compose configuration to host a public
[Send](https://gitlab.com/timvisee/send) instance on your own domain.

- Hosts on your own domain
- Provides automatic SSL certificates through LetsEncrypt

This configuration exposes a reverse proxy on ports 80 and 443, so these must be
available.

This uses the latest Send version from
[`timvisee/send`](https://gitlab.com/timvisee/send) because
[`mozilla/send`](https://github.com/mozilla/send) has been archived.
This is configurable in your [`.env`](.env.example) file.

See [docker-compose.yaml](./docker-compose.yaml).

### Usage
- Install Docker with Docker compose
- Clone repository
- Run `cp .env.example .env` and configure it (see [example](#example-env))
- Run `docker-compose up`
- Visit your domain

### Example .env
```bash
# Docker image to use for Send
# - Latest Send version by @timvisee: registry.gitlab.com/timvisee/send:latest
# - Outdated (archived) Send version by Mozilla: mozilla/send:latest
DOCKER_SEND_IMAGE=registry.gitlab.com/timvisee/send:latest

# Host to expose Send on
HOST=send.example.org

# Base URL for Send
SEND_BASE_URL=https://send.example.org

# Optional: for LetsEncrypt SSL, same as HOST
LETSENCRYPT_HOST=

# Optional: for LetsEncrypt SSL, your email address
LETSENCRYPT_EMAIL=mail@example.org
```
