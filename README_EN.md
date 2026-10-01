# 3x-ui-pro

🇷🇺 [Русская версия](README.md)

Automated installer for the [3x-ui](https://github.com/MHSanaei/3x-ui) panel with nginx, SSL, Clash subscription and network diagnostics.

- Debian 12 / Ubuntu 24
- Two domains or subdomains (one for the panel, one for REALITY)
- Automatic SSL certificate renewal
- VLESS+REALITY, VLESS+XHTTP and Trojan+gRPC through 443/TCP
- Hysteria 2 through 443/UDP and AmneziaWG through a separate UDP port (printed by the installer)
- AmneziaWG uses version 3.1 settings; host records for AmneziaWG and Hysteria 2 are created automatically

---

## What gets installed

| Component | Description |
|-----------|-------------|
| 3x-ui | VPN panel with web UI; installer pre-creates empty AmneziaWG and Hysteria 2 inbounds |
| nginx | Reverse proxy, SNI routing |
| certbot | Let's Encrypt SSL |
| Clash subscription | Serves `clash.yaml` based on User-Agent |
| Diagnostics | MTR tracer + in-browser speed test |
| Fake site | Random HTML cover site |
| Backup | Backup / restore script |
| AdGuard Home | Optional: ad-blocking DNS (DoH) — separate script |

---

## Installation

**Install the 3x-ui panel**

```bash
sudo wget -qO x-ui-latest.sh https://raw.githubusercontent.com/drafwodgaming/3x-ui-pro/main/x-ui-latest.sh && sudo bash x-ui-latest.sh -install y
```

---
## Patch

Apply current fixes to an existing installation (no DB changes):

```bash
sudo wget -qO x-ui-patch.sh https://raw.githubusercontent.com/drafwodgaming/3x-ui-pro/main/x-ui-patch.sh && sudo bash x-ui-patch.sh
```

---

## AdGuard Home (optional)

Installs [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) on the panel domain — no separate domain or open ports, everything goes through the existing 443:

- **DNS-over-HTTPS** for clients: `https://<panel-domain>/dns-query`
- **Admin UI** — at a random `/adg-<random>/` path (login and password are printed by the script)

```bash
sudo wget -qO x-ui-adguard.sh https://raw.githubusercontent.com/drafwodgaming/3x-ui-pro/main/x-ui-adguard.sh && sudo bash x-ui-adguard.sh
```

Re-running is safe (settings and password are kept). After the installer or the patch, run this script again — they rewrite the nginx config.

Uninstall:

```bash
sudo wget -qO x-ui-adguard.sh https://raw.githubusercontent.com/drafwodgaming/3x-ui-pro/main/x-ui-adguard.sh && sudo bash x-ui-adguard.sh -uninstall y
```

---

## Uninstall

**Remove the 3x-ui panel**

```bash
sudo wget -qO x-ui-latest.sh https://raw.githubusercontent.com/drafwodgaming/3x-ui-pro/main/x-ui-latest.sh && sudo bash x-ui-latest.sh -uninstall y
```

---

## Command-line options

| Option | Description |
|--------|-------------|
| `-install y` | Install |
| `-subdomain <domain>` | Panel and subscription domain |
| `-reality_domain <domain>` | REALITY destination domain |
| `-version <version>` | Install a specific 3x-ui version (AmneziaWG requires `3.7.0` or newer), default — latest |
| `-uninstall y` | Full uninstall |

---

## Clash subscription

Works via User-Agent detection — one URL, different behavior:

- **Clash / Mihomo / Stash** → get a ready-to-use `clash.yaml` config
- **Regular browser / other clients** → get the standard 3x-ui subscription page

The import link is printed by the script after installation.

---

## Backup and restore

**Install the backup script**

```bash
sudo wget -qO x-ui-backup.sh https://raw.githubusercontent.com/drafwodgaming/3x-ui-pro/main/assets/backup/x-ui-backup.sh && sudo install -m 0755 x-ui-backup.sh /usr/local/bin/x-ui-backup
```

**Create a backup**

```bash
sudo x-ui-backup backup
```

**List backups**

```bash
sudo x-ui-backup list
```

**Restore from a backup** (on a clean server; packages are installed automatically)

```bash
sudo x-ui-backup restore /var/backups/x-ui/x-ui-backup-20260101-120000.tar.gz
```

The backup includes: nginx configs, panel DB, 3x-ui binary, SSL certificates, web content, systemd units, cron, UFW rules.

---

## Network diagnostics

Available after installation at the link printed by the script. Includes:

- MTR trace to your IP
- Download and upload speed test (512 MB test files)
