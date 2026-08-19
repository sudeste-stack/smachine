# Installation

Step-by-step install. System layout rationale: [ARCHITECTURE.md](ARCHITECTURE.md).
What each setup question means: [SETUP.md](SETUP.md).

## Requirements

- Arch Linux bootable USB — [archlinux.org](https://archlinux.org)
- Target PC with 20 GB+ disk (40 GB+ if you plan many ROMs), UEFI firmware
- Optional: a second disk for ROM/storage isolation
- Optional: a WireGuard `.conf` for this machine, if you already run a WireGuard server

## Steps

**1. Boot the Arch live USB on the target PC.**

**2. Confirm the network works:**
```bash
ping -c 2 archlinux.org
```

**3. Run the installer:**
```bash
curl -fsSL https://raw.githubusercontent.com/sudeste-stack/smachine/main/arch-retrogaming-installer.sh | bash
```
No manual partitioning. The menu asks a handful of questions (see
[SETUP.md](SETUP.md)), then the installer:
- partitions and formats the selected disks
- installs the Arch base system
- creates the `sudeste` and `games` users
- installs Flatpak + RetroDECK and configures it for the local layout
- enables autolaunch, snapshotting, and any optional services (WireGuard, Samba, SSH)
- reboots

**4. First boot runs the finalization menu once, then launches ES-DE.**

## Verify the install

- ES-DE appears without a login prompt.
- From another machine on the network, open `\\<machine>\roms`, log in as `games`.
- If SSH was enabled: `ssh sudeste@<machine>` (default password equals the username — change it).

## If something fails during install

Reboot the Arch live USB, check the target hardware, and run the installer
again. Re-running wipes and reinstalls the selected disks. Read the warnings
carefully if you put ROMs on a shared disk.
