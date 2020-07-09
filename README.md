# Firefox Send in Docker compose

### Usage
- Install Docker with Docker compose
- Clone repository
- Run `cp .env.example .env` and configure it (see [example](#example-env))
- Run `docker-compose up`
- Visit your domain

### Example .env
```.env
# Host to expose Send on
HOST=myhost

# Configure these to enable automatic LetsEncrypt certificate generation
LETSENCRYPT_EMAIL=mail@example.org
LETSENCRYPT_HOST=
```
