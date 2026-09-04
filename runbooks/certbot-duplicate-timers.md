# Checking for duplicate certbot renewal timers

**Why this matters:** the `eu-central-1` host was found running two independent certbot renewal
timers at once — an apt-packaged `certbot.timer` (active since initial setup) and a
`snap.certbot.renew.timer` (installed later, independently). Two renewal paths racing against
the same certs is a real risk; check any host you're about to touch TLS on, don't assume it's
clean just because one path works.

## Steps

1. List active timers:
   ```bash
   systemctl list-timers | grep certbot
   ```
2. If more than one is present, check each renewal config's `version =` stamp in
   `/etc/letsencrypt/renewal/*.conf` to see which certbot binary it expects.
3. Confirm which one actually works before disabling the other:
   ```bash
   sudo <certbot-binary> renew --dry-run --no-random-sleep-on-renew
   ```
   The `--no-random-sleep-on-renew` flag matters — without it certbot's built-in multi-minute
   jitter makes a working dry-run look like a hang.
4. Don't assume the newer-versioned client is automatically the right one to keep — verify it can
   actually renew *every* cert on that host before masking the older timer.

Applies to both production hosts (`servers/eu-central-1.md`, `servers/ap-southeast-1.md`) —
confirmed present on the EU host; worth checking `ap-southeast-1` too rather than assuming.
