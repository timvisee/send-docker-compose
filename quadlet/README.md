# Send on Podman Quadlet

Rootless [Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html)
version of the compose setup. [Send](https://gitlab.com/timvisee/send) runs as
systemd user services, with [Caddy](https://caddyserver.com/) handling HTTPS
(Let's Encrypt) instead of nginx-proxy, so no container socket is needed.

Tested with Podman 5.4.2 on Debian 13.

## Files

| File | Purpose |
|------|---------|
| `send.container` | Send (image, env, uploads) |
| `redis.container` | Redis, data in the `send-redis` volume |
| `caddy.container` | Reverse proxy on ports 80/443 |
| `send.network` | Private network, containers reach each other by name |
| `*.volume` | Volumes: `send-redis`, `caddy-data`, `caddy-config` |
| `Caddyfile` | Proxies `$HOST` to `send:1234` |
| `send.env.example` | Settings to edit: `HOST`, `BASE_URL`, `LETSENCRYPT_EMAIL` |
| `send-cleanup.container` | Deletes leaked uploads (replaces `../gc.cron`) |
| `send-cleanup.timer` | Runs the cleanup hourly |

## Install

Run from this directory, as the user that will run Send (not root).

1. Install Podman 5.x.
2. Allow ports 80/443 without root, and start services at boot without a login:
   ```bash
   echo 'net.ipv4.ip_unprivileged_port_start=80' | sudo tee /etc/sysctl.d/50-unpriv-ports.conf
   sudo sysctl --system
   sudo loginctl enable-linger "$USER"
   ```
3. Copy the files:
   ```bash
   mkdir -p ~/.config/containers/systemd ~/.config/systemd/user ~/.config/send ~/send/uploads
   cp *.container *.network *.volume ~/.config/containers/systemd/
   cp send-cleanup.timer ~/.config/systemd/user/
   cp Caddyfile ~/.config/send/Caddyfile
   cp send.env.example ~/.config/send/send.env
   ```
4. Edit `~/.config/send/send.env`.
5. Start (`caddy` pulls in `send` and `redis`; all start on boot):
   ```bash
   systemctl --user daemon-reload
   systemctl --user start caddy
   systemctl --user enable --now send-cleanup.timer
   systemctl --user enable --now podman-auto-update.timer
   ```

The domain must point to this host, with ports 80/443 reachable.

## Configuration

- **Image**: set `Image=` in `send.container` (Quadlet can't read it from the env file).
- **Auto-update**: `send`, `redis` and `caddy` use `AutoUpdate=registry`. The
  `podman-auto-update.timer` checks daily, pulls new images and restarts the
  services. Remove the line to pin an image.
- **Send options** (limits, S3, ...): uncomment or add `Environment=` lines in
  `send.container`. See the [Send config](https://github.com/timvisee/send/blob/master/server/config.js);
  arrays are comma-separated.
- **Uploads**: stored in `~/send/uploads`. `UserNS=keep-id:uid=1000,gid=1000` maps
  your user to the image's `app` user, otherwise the directory isn't writable.
- **Cleanup**: hourly, deletes uploads older than 7 days. Adjust `Exec=` in
  `send-cleanup.container` or `OnCalendar=` in `send-cleanup.timer`. Quadlet has
  no timer type, so the timer is a plain systemd unit.

## Operations

```bash
systemctl --user status caddy send redis   # status
journalctl --user -u send -f               # logs
systemctl --user list-timers               # next cleanup and update
podman auto-update --dry-run               # pending image updates
systemctl --user start send-cleanup        # run cleanup now
```

After editing a unit, copy it again, run `systemctl --user daemon-reload` and
restart the service. To check units without starting them:
`/usr/lib/systemd/system-generators/podman-system-generator --user --dryrun`.

## Testing

Checked in a throwaway Debian 13 VM: unit generation, HTTPS and redirect, Redis
persistence, an upload/download round trip with [ffsend](https://github.com/timvisee/ffsend),
auto-update, cleanup and autostart after reboot. Not tested: real Let's Encrypt
issuance (the VM used Caddy's local CA).

With ffsend, pass your server (`-h https://your.host`), or it uploads to a public instance.
