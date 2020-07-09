# Firefox Send in Docker compose

### Usage
- Install Docker with Docker compose
- Clone repository
- Run `cp .env.example .env` and configure it (see [example](#example-env))
- Run `docker-compose up`
- Visit your domain

### Example .env
```bash
# Host to expose Send on
HOST=send.example.org

# Optional: for LetsEncrypt SSL, your email address
LETSENCRYPT_EMAIL=mail@example.org

# Optional: for LetsEncrypt SSL, same as HOST
LETSENCRYPT_HOST=
```
