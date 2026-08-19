# Architecture

How sMachine is put together. This file is the single source of truth for
system layout: design decisions, boot flow, disks, services, and users.
Other docs link here instead of repeating.

> Status: design document. The installer implements this layout; until the
> installer code lands in this repo, treat the details below as the spec it
> must follow.

## Design Decisions

| Decision | Why |
|---|---|
| Arch Linux base | Up-to-date drivers and packages, one install flow. Risk (rolling release) mitigated by weekly btrfs snapshots and user-controlled update timing. |
| RetroDECK via Flatpak | Self-contained emulator stack. Updates stay inside the Flatpak without touching the base system. |
| Btrfs | Copy-on-write snapshots used for backups and update rollback. |
| No desktop environment | LightDM autologin into a minimal session that launches ES-DE. The machine is a console, not a PC. |
| Optional WireGuard and Cockpit | sMachine works standalone by default. VPN and web dashboard plug in only if configured at setup. |
| Two users (`sudeste`, `games`) | Management and content are separated. The `games` account cannot administer the system. |

## Boot Flow

1. Arch kernel + initramfs.
2. systemd starts enabled units from the [services table](#services).
3. `smachine-first-boot.service` — runs only if `/etc/smachine/.first-boot`
   does not exist. Shows the one-time setup menu (see [SETUP.md](SETUP.md)),
   then creates the marker and masks itself.
4. `smachine-autolaunch.service` — every boot. LightDM autologins as
   `sudeste`; the minimal session starts `flatpak run com.retrodeck.RetroDeck --esde`.

## Disks

### System disk (default `/dev/sda`, SSD or NVMe)

| Mount | Type | Size | Notes |
|---|---|---|---|
| `/boot` | FAT32 (EFI) | 1 GiB | UEFI required |
| `/` | btrfs | rest | Subvolumes: system + `/.snapshots` |

### Storage disk (optional, default `/dev/sdb`)

| Mount | Type | Notes |
|---|---|---|
| `/retrodeck` | btrfs | RetroDECK content |

`/retrodeck` subvolumes: `roms`, `saves`, `bios`, `screenshots`.

Single-disk installs put these RetroDECK directories under `/retrodeck` on the
system disk instead. The RetroDECK Flatpak is told where to find them; the
layout itself is the same either way.

## Services

| Unit | Purpose | When active |
|---|---|---|
| `smachine-first-boot.service` | One-time setup menu | Until `/etc/smachine/.first-boot` exists, then masked |
| `smachine-autolaunch.service` | Autologin + ES-DE launch | Every boot |
| `smachine-btrfs-snapshot.timer` | Weekly snapshots (policy in [MAINTENANCE.md #snapshots](MAINTENANCE.md#snapshots)) | Installed by default |
| `wg-quick@wg0.service` | WireGuard | Only if a config is provided at setup |
| `smbd.service` | Samba ROM shares | Enabled at setup |
| `sshd.service` | Remote shell | Enabled at setup |
| `cockpit.service` | Web dashboard (port 42069) | Optional |

## Users

| User | Role | Notes |
|---|---|---|
| `sudeste` | Admin, sudo, runs RetroDECK | Interactive use |
| `games` | Samba-only content account | No sudo, no login by default |

## Decoupling

sMachine installs no VPN server and no Cockpit hub of its own. Point
`wg0.conf` at your own WireGuard server, or access Cockpit per-machine on
port 42069. Nothing in sMachine requires external infrastructure.
