# Configuration

Post-install customization. This file is the single source for *where*
settings live and *how* to change them. Baseline system layout:
[ARCHITECTURE.md](ARCHITECTURE.md). Routine operations: [MAINTENANCE.md](MAINTENANCE.md).

## Where settings live

| Setting | Location |
|---|---|
| Hostname | `sudo hostnamectl set-hostname <name>` |
| IP (static) | `/etc/systemd/network/10-smachine.net` |
| WireGuard | `/etc/wireguard/wg0.conf` |
| Cockpit port / access | `/etc/cockpit/cockpit.conf` |
| Samba shares | `/etc/samba/smb.conf` |
| Samba passwords | `sudo smbpasswd -a <user>` |
| Users | `sudo useradd` / `sudo userdel`; sudo via `sudo usermod -aG wheel <user>` |
| RetroDECK (ROM paths, cores, controls) | RetroDECK Configurator, reachable from inside ES-DE |
| Update schedule | you, manually — see [MAINTENANCE.md § Updates](MAINTENANCE.md#updates) |
| Snapshot policy | `smachine-btrfs-snapshot.timer` — see [MAINTENANCE.md § Snapshots](MAINTENANCE.md#snapshots) |

## Users

- `sudeste` — admin. Sudo. Runs the console session. Default password equals
  the username; change it on first login: `passwd sudeste`.
- `games` — Samba-only content account. Give it a real password for file
  sharing: `sudo smbpasswd games`.

## Samba

Share: `\\<machine>\roms` — ROMs, saves, BIOS.

```bash
# change/verify the games password
sudo smbpasswd games

# reload after editing /etc/samba/smb.conf
sudo systemctl reload smbd

# quick test from any machine on the network
smbclient -L //<machine> -U games
```

Add extra shares by editing `/etc/samba/smb.conf` and reloading `smbd`.

## SSH

```bash
sudo systemctl status sshd
ssh sudeste@<machine>
```

Copy your key: `ssh-copy-id sudeste@<machine>`, then consider disabling
password auth in `/etc/ssh/sshd_config`.

## WireGuard

sMachine is a client only; bring your own WireGuard server.

```bash
# point the config at your server, then:
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0

# status
wg show
```

Checklist when it does not work: server sees the peer (`wg show` on the
server), allowed IPs match your network, and the config landed as
`/etc/wireguard/wg0.conf`.

## Cockpit (optional)

Web dashboard on port **42069** (not the default 9090):

```bash
sudo systemctl status cockpit
# https://<machine>:42069 — first load uses a self-signed cert; install the
# provided CA or accept the warning in the browser.
```

Port and allowed identities live in `/etc/cockpit/cockpit.conf`.
Log in as `sudeste`.

## RetroDECK

Configure ROM paths, cores, and inputs from inside the app:
ES-DE → RetroDECK/Configurator. Flatpak app id: `com.retrodeck.RetroDeck`.
From a shell, if ever needed: `flatpak run com.retrodeck.RetroDeck`.
