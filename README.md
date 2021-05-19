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

1. Install Docker with Docker Compose https://get.docker.com/
2. Clone this repository `git clone https://github.com/timvisee/send-docker-compose`
3. Run `cp .env.example .env` and configure it (see [example](#example-env))
4. Run `docker-compose up`
5. Visit your domain

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

### Configuration

All the config options and their defaults can be found here: https://github.com/timvisee/send/blob/master/server/config.js.

Config options expecting array values (e.g. `EXPIRE_TIMES_SECONDS`, `DOWNLOAD_COUNTS`) are parsed as CSV, and the first entry is the default.

Other options should be set as unquoted strings, integers, booleans, etc., for example:
```yaml
...
services:
  ...
  send:
    ...
    environment:
      ...

      # strings numbers, bools, etc. shoul all be set as bare unquoted values
      - BASE_URL=https://send.example.com
      - DHPARAM_GENERATION=false
      - MAX_DOWNLOADS=250000

      # use values set in the .env using ${VARNAME} bash syntax
      - VIRTUAL_HOST=${HOST}
      
      # time values are all in seconds, e.g. 365d * 60*60*24 = 31,536,000 seconds
      - MAX_EXPIRE_SECONDS=31536000
      - DEFAULT_EXPIRE_SECONDS=86400

      # array configs are set as CSV (first entry is the default for the UI dropdown)
      - EXPIRE_TIMES_SECONDS=86400,3600,86400,604800,2592000,31536000,157680000
      - DOWNLOAD_COUNTS=10,1,2,5,10,15,25,50,100,1000,10000,100000,250000
      
      # size values are are in bytes, e.g. 10GB * 1024*1024*1024 = 10,747,904,000 bytes
      - MAX_FILE_SIZE=10747904000
```
