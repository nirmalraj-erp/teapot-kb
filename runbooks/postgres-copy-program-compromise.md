# Detecting/responding to a Postgres `COPY ... FROM PROGRAM` compromise

**Why this matters:** on 2026-09-15, investigating reported slowness for `nvt`/`gadgets` on
`ap-southeast-1` turned up an ~8-week-old cryptojacking compromise, not a sizing problem. The
`odoo` Postgres role was `SUPERUSER`, which let some still-unidentified bug in the Odoo app reach
`COPY ... FROM PROGRAM` — a superuser-only command that runs arbitrary OS commands. Full incident
record: `tpt-dev-scripts/SECURITY-INCIDENT-2026-09-15.md`. This runbook is the generalized
"what to do" if the same pattern shows up again, here or on `eu-central-1`.

## Detection

1. **Check for the exploit's fingerprint in Postgres logs first** — this is the fastest, most
   reliable signal, and it's what actually cracked the 2026-09-15 incident:
   ```bash
   sudo grep -c "FROM PROGRAM" /var/log/postgresql/postgresql-*-main.log*
   sudo zgrep "FROM PROGRAM" /var/log/postgresql/postgresql-*-main.log*.gz | head
   ```
   Any hit is a real attempt (successful or not — Postgres logs the statement whenever the
   surrounding `COPY` reports an error, which it usually does even when the embedded shell command
   still ran as a side effect). Check every rotated `.gz` file, not just the current log — the
   first hit may be weeks back.
2. **Check `pg_roles` for any client's `db_user` with `rolsuper = t`:**
   ```sql
   SELECT rolname, rolsuper, rolcreatedb, rolcreaterole FROM pg_roles WHERE rolcanlogin;
   ```
   Any app-facing role that's superuser is a live risk even with no active exploit yet.
3. **Look for processes that don't match anything you deployed**, especially ones running as the
   `postgres` OS user:
   ```bash
   ps -eo pid,ppid,user,%cpu,%mem,etime,cmd --sort=-%cpu | head -20
   sudo ss -tnp | grep -v "127.0.0.1\|::1"
   ```
   Don't dismiss something just because its name looks legitimate (this campaign's miner ran as
   `/var/lib/postgresql/supervisor`) or because CPU usage alone looks "explainable" (a genuinely
   legitimate process — `needrestart`, in this case — can also peg a core for days from an
   unrelated bug; verify by binary path + `dpkg -V`, not by vibes).
4. **Scan for fileless/self-deleted processes** — a strong compromise indicator on its own:
   ```bash
   for pid in $(ls /proc | grep -E '^[0-9]+$'); do
     link=$(sudo readlink /proc/$pid/exe 2>/dev/null)
     [[ "$link" == *"(deleted)"* ]] && echo "PID $pid -> $link" && ps -fp $pid
   done
   ```
   Expect some noise (long-running Python/systemd processes show `(deleted)` after a routine
   package upgrade replaces the binary on disk — that's normal). Anything pointing at a
   `memfd:...` or a path inside a Postgres data directory is not normal.

## Containment (does not require stopping any live Odoo instance)

1. Kill whatever rogue processes you found (`kill -9 <pid>`), then **re-scan immediately** —
   these kits commonly have 2+ independent persistence paths, so one respawning under a new PID
   after you kill the first is expected, not a failure.
2. Check **every** user's crontab, not just the one the rogue process runs as:
   ```bash
   for u in $(cut -f1 -d: /etc/passwd); do sudo crontab -u "$u" -l 2>/dev/null && echo "^^ $u"; done
   ```
3. Check systemd for anything unfamiliar, including **dangling enable-symlinks** pointing at an
   already-deleted unit (found once in this incident — the unit file was gone but
   `/etc/systemd/system/multi-user.target.wants/<name>.service` still pointed at it):
   ```bash
   sudo systemctl list-units --all --type=service
   sudo find /etc/systemd/system/*.wants -type l ! -xtype f
   ```
4. Check `docker ps -a` even if you don't think Docker is part of your stack — it wasn't part of
   this one either, until the attacker installed it.
5. Block egress to any C2/pool IPs and ports you found. **AWS Lightsail's built-in firewall is
   inbound-only** — it cannot block outbound traffic. A local `iptables` rule is a same-box
   stopgap only (the host may already be root-compromised, so treat anything set from inside it as
   reversible by the attacker). The durable fix: enable VPC peering for the Lightsail account
   (Account → Advanced features, per region), then add outbound DENY rules to the Network ACL of
   the peered VPC's subnet, ordered before the default allow-all rule.
6. Revoke superuser from the affected role once you've confirmed which extensions its databases
   actually use (`SELECT extname FROM pg_extension;` per db — if they're all already installed,
   revoking is safe for existing traffic):
   ```sql
   ALTER ROLE <role> NOSUPERUSER;
   ```
   Verify: `COPY (SELECT 1) TO PROGRAM 'echo test';` as that role should now fail with
   `must be superuser or a member of the pg_execute_server_program role`.

## After containment

- The application-layer injection point is the actual root cause and is often **not** found during
  containment — don't call the incident closed until it is. Revoking superuser blocks this
  specific technique but doesn't fix whatever let attacker input reach `env.cr.execute(...)` in the
  first place.
- Rotate the affected role's password — coordinate with every client conf that uses it (update
  `.conf` then restart that one `odoo17-*` service) rather than changing the Postgres side first,
  or you'll get connection failures until every conf is updated to match.
- Check `pg_hba.conf` for `trust` entries — even `local ... trust` is a real local-privilege-
  escalation shortcut if any role reachable that way is superuser or otherwise sensitive.
- If retired/unused client databases exist on the same instance, check them too — in this
  incident the exploit hit retired databases (`raybright`, `viable`, `17focus`, old `saizen`
  copies) just as often as live ones. Clean those up entirely (`pg_dump` → drop → remove
  filestore/addon code/conf → update `odoo-conf-summary.md`) rather than leaving them as
  additional attack surface.
