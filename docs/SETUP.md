# First-Boot Setup

The one-time menu (`smachine-first-boot.service`) runs on the first boot,
asks a few questions, and then the machine boots straight into ES-DE
forever after. Canonical prompt layout:
[first-boot-menu.screen](../repo/screens/first-boot-menu.screen).

Installed by default, regardless of your answers:
autolaunch into ES-DE, weekly btrfs snapshots, the `sudeste` and `games`
users. Everything else below is a choice.

## Questions

### Hostname

Default `smachine-01`. Used for SSH, Samba (`\\smachine-01\roms`), and as the
WireGuard peer name. Change later with `hostnamectl set-hostname <name>`.

### Network: DHCP or static

- **DHCP** — simple path. Needs the target network to hand out addresses.
- **Static** — choose this if you run WireGuard, have no DHCP server, or need
  a fixed address for Samba. The menu collects IP, prefix, gateway, DNS and
  writes `systemd-networkd` configuration.

### Storage: single or dual disk

- **Single disk** — system and RetroDECK content share one btrfs root. Fine for
  most machines.
- **Dual disk** — system on the first disk, all ROMs/saves/BIOS on the second
  disk under `/retrodeck`. Keeps the OS fast and gives you a disk you can
  move between machines. Disk layout: [ARCHITECTURE.md § Disks](ARCHITECTURE.md#disks).

### WireGuard: config file or skip

Select a `.conf` you already generated for this machine, or skip. If selected:
the installer places it at `/etc/wireguard/wg0.conf` and enables
`wg-quick@wg0.service`. sMachine does not run a server — bring your own.
Details and troubleshooting: [CONFIGURATION.md § WireGuard](CONFIGURATION.md#wireguard).

### Samba: enable or disable

Enables `\\<machine>\roms` for the `games` user, so you can copy ROMs from a
PC without USB drives. Details: [CONFIGURATION.md § Samba](CONFIGURATION.md#samba).

### SSH: enable or disable

Enables `sshd` for the `sudeste` user. Recommended if you plan remote
management: [CONFIGURATION.md § SSH](CONFIGURATION.md#ssh).

## After the menu

The installer does the work (partitioning, install, services) and reboots. If
an answer was wrong, reconfigure per [CONFIGURATION.md](CONFIGURATION.md).
Full reset means a reinstall.
