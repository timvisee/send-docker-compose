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

*Note: for plain Docker usage without Compose, see: https://github.com/timvisee/send/blob/master/docs/docker.md*

## Usage

1. Install Docker and Docker Compose on your system: https://get.docker.com/
2. Clone this repository `git clone https://github.com/timvisee/send-docker-compose && cd send-docker-compose`
3. Run `cp .env.example .env` and edit `.env` and `docker-compose.yaml` to setup your configuration (see [example](#example-env))
4. Run `docker-compose up` to start the containers (add `-d` to start it in the background)
5. Wait 30sec for it to start, then visit your domain verify the UI is up and running, e.g. `https://send.example.com`

<img src="https://i.imgur.com/eyvrWAP.png" alt="Screenshot of succesfully running Send UI" width="450px"/>

## Example .env

```bash
# Docker image to use for Send
# - Latest Send version by @timvisee: registry.gitlab.com/timvisee/send:latest
# - Outdated (archived) Send version by Mozilla: mozilla/send:latest
DOCKER_SEND_IMAGE=registry.gitlab.com/timvisee/send:latest

# Host to expose Send on
HOST=send.example.com

# Base URL for Send
SEND_BASE_URL=https://send.example.com

# Optional: for LetsEncrypt SSL, same as HOST
LETSENCRYPT_HOST=send.example.com

# Optional: for LetsEncrypt SSL, your email address
LETSENCRYPT_EMAIL=ssl@send.example.com
```

## Configuration

All the config options and their defaults can be found here: https://github.com/timvisee/send/blob/master/server/config.js

For more documentation about the config options available and their defaults, see: https://github.com/timvisee/send/blob/master/docs/docker.md

Config options expecting array values (e.g. `EXPIRE_TIMES_SECONDS`, `DOWNLOAD_COUNTS`) should be set as bare comma separated values, and the first entry is used as the default.

Other options should be set as unquoted strings, integers, booleans, etc., for example:
```yaml
...
services:
  ...
  send:
    ...
    environment:
      ...

      # strings numbers, bools, etc. should all be set as bare unquoted values
      - BASE_URL=https://send.example.com
      - DHPARAM_GENERATION=false
      - MAX_DOWNLOADS=250000

      # use values set in the .env using ${VARNAME} bash syntax
      - VIRTUAL_HOST=${HOST}
      
      # time values are all in seconds, e.g. 365d * 60*60*24 = 31,536,000 seconds
      - MAX_EXPIRE_SECONDS=31536000
      - DEFAULT_EXPIRE_SECONDS=86400

      # size values are are in bytes, e.g. 10GB * 1024*1024*1024 = 10,747,904,000 bytes
      - MAX_FILE_SIZE=10747904000

      # array configs are set as CSV (first entry is the default for the UI dropdown)
      - EXPIRE_TIMES_SECONDS=86400,3600,86400,604800,2592000,31536000,157680000
      - DOWNLOAD_COUNTS=10,1,2,5,10,15,25,50,100,1000,10000,100000,250000
```
