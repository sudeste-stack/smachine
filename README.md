# sMachine

**Turn old PCs into instant retro gaming consoles. Zero Linux expertise required.**

sMachine is a lightweight, open-source Linux distribution built on Arch Linux and RetroDECK. Boot an Arch ISO, answer a few interactive questions, and walk away with a fully configured gaming appliance. Supports remote VPN access, automatic backups, fleet management—all optional. Use it standalone or integrate with your own infrastructure.

---

## What Problem Does This Solve?

You have:
- Old computers that still work but serve no purpose.
- A desire to turn them into retro gaming machines.
- No patience for manual Linux configuration.
- Maybe multiple machines to manage across locations.

sMachine solves this with a **single installer script**. No manual partitioning. No copy-paste commands. Just interactive questions and a reboot.

---

## What You Get

- **Complete Gaming System** – RetroDECK + 50+ emulators (NES, SNES, PS1, N64, Dreamcast, and more).
- **Auto-Launch Console UI** – Boots straight to ES-DE. No desktop, no confusion.
- **Network File Sharing** – Add ROMs from your PC via Samba. No USB transfers.
- **Remote Access (Optional)** – WireGuard VPN + Cockpit web dashboard. Manage machines from anywhere.
- **Automatic Backups** – Btrfs snapshots every week. One command to rollback if something breaks.
- **Multi-User Security** – Management user (sudeste, admin) separate from content user (games, Samba-only).
- **Sustainable Updates** – You control when Arch updates run. RetroDECK updates auto-install.
- **Fully Decoupled** – Bring your own WireGuard server, your own Cockpit, or run standalone. No lock-in.
- **Open Source (GPL-3.0)** – Fork it. Modify it. Redistribute it freely.

---

## Installation in 10 Minutes

### What You Need

1. **Arch Linux live ISO** – Download from [archlinux.org](https://archlinux.org)
2. **Target PC** – 20GB+ SSD/HDD (40GB+ if you plan many games)
3. **WireGuard config** (optional, but encouraged)
   - If you have a WireGuard server, generate a `.conf` file for this machine
   - If not, skip it. Works standalone.

### Steps

**1. Boot Arch live USB on target PC**

**2. Get the installer:**
```bash
curl -fsSL https://raw.githubusercontent.com/sudeste-stack/smachine/main/arch-retrogaming-installer.sh | bash
```

**3. Interactive setup menu (gum-based, user-friendly):**
- Hostname: `smachine-01` (or custom)
- Network: DHCP or static IP
- Storage: Single disk or dual (system + games)
- **WireGuard config file:** Select your `.conf` file or skip
- Samba & SSH: Enable/disable

**4. Installer runs:**
- Partitions & formats disks
- Installs Arch Linux base
- Configures Flatpak + RetroDECK
- Sets up users, services, VPN (if config provided)
- Reboots automatically

**5. First boot:**
- System auto-launches ES-DE (RetroDECK)
- Interactive first-time setup (network finalization, storage mount, etc.)
- Ready to add ROMs via Samba

**Done.** You have a gaming console.

---

## After Installation

### Add Games

**From your PC (Windows/Mac/Linux):**
```
Samba share: \\<machine-ip>\roms
User: games
Add your ROM files
```

**Or via SSH (if you're technical):**
```bash
ssh sudeste@<machine-ip>
# Use RetroDECK Configurator to organize
```

### Remote Access (Optional)

**If you set up WireGuard:**
```bash
ssh sudeste@<wireguard-ip>
# or web dashboard:
https://<wireguard-ip>:42069 (Cockpit)
```

### Keep It Running

- **Arch updates:** Run `pacman -Syu` when you login. Snapshots auto-save before updates.
- **RetroDECK updates:** Automatic. Flatpak handles it.
- **Backups:** Automatic weekly snapshots (last 14 kept). Rollback one-liner if needed.

---

## Architecture at a Glance

### Boot Process

1. Arch Linux kernel
2. `smachine-first-boot.service` (once)
   * Interactive menu (hostname, etc.)
   * Format storage disk
   * Configure services
3. `smachine-autolaunch.service`
   * Auto-login sudeste user
      * Launch ES-DE (RetroDECK)

### Disks

1. `/dev/sda` (system)
   * `/boot` (EFI, 1GB)
   * `/` (root, btrfs, ~20GB)
2. `/dev/sdb` (storage, optional)
   * `/retrodeck` (btrfs, ROMs/saves)
      * `/roms`
      * `/saves`
      * `/bios`
      * `/screenshots`
      * (other RetroDECK dirs)

### Services

* `smachine-first-boot.service` (one-time)
* `smachine-autolaunch` (every boot)
* `smachine-btrfs-snapshot.timer` (weekly)
* `smachine-cockpit` (web, port 42069)
* `wireguard-wg0` (if config provided)
* `samba` (network shares)
* `sshd` (remote shell)

### Users

* `sudeste` (sudo access, runs RetroDECK)
* `games` (Samba-only, ROM management)

---

## Who Is This For?

- **Retro gamers** – Want a console, not a Linux lab.
- **IT admins** – Managing gaming rooms, arcades, or kiosks.
- **Business ops** – Gaming cafes with multiple machines. Fleet Cockpit dashboard.
- **Developers** – Extending RetroDECK. Open source, GPL-3.0.
- **Minimalists** – Sustainable reuse of old hardware.

---

## Documentation

- **[INSTALL.md](./docs/INSTALL.md)** – Step-by-step installation guide.
- **[SETUP.md](./docs/SETUP.md)** – First-boot menu options and configuration.
- **[CONFIGURATION.md](./docs/CONFIGURATION.md)** – Customize WireGuard, Cockpit, Samba, SSH.
- **[MAINTENANCE.md](./docs/MAINTENANCE.md)** – Updates, snapshots, rollbacks, troubleshooting.
- **[ARCHITECTURE.md](./docs/ARCHITECTURE.md)** – System design, service flow, btrfs layout.
- **[CONTRIBUTING.md](./docs/CONTRIBUTING.md)** – Fork, extend, contribute back.

---

## License

**GNU General Public License v3.0**

Free to use, modify, and redistribute. See [LICENSE](./LICENSE).

---

## Tech Stack

- **Arch Linux** – Rolling-release, minimal, cutting-edge.
- **RetroDECK** – All-in-one retro gaming Flatpak.
- **Btrfs** – Next-gen filesystem with automatic snapshots.
- **Gum** – Interactive CLI menus (user-friendly setup).
- **Archinstall** – Automated Arch installation.
- **WireGuard** – Modern VPN (optional).
- **Cockpit** – Web system dashboard (optional).
- **Samba** – Network file sharing.

---

## Status

**Beta**. Ready for deployment and testing. Issues and feedback welcome.

---

## Links

- **GitHub:** [sudeste-stack/smachine](https://github.com/sudeste-stack/smachine)
- **Issues:** [Report bugs](https://github.com/sudeste-stack/smachine/issues)
- **RetroDECK:** [retrodeck.readthedocs.io](https://retrodeck.readthedocs.io)
- **Arch Linux:** [archlinux.org](https://archlinux.org)
