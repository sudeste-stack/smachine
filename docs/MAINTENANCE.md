# Maintenance

Day-to-day operations: updates, snapshots, rollbacks, troubleshooting.
Changeable settings: [CONFIGURATION.md](CONFIGURATION.md).

## Updates

Two update lanes, deliberately separate:

- **Base system** — you decide. No unattended arch upgrades.
  ```bash
  sudo pacman -Syu
  ```
  Weekly snapshots stand behind every reboot, so a broken upgrade is a rollback
  away (below).
- **RetroDECK** — automatic. The Flatpak updates itself in the background;
  nothing to do.

## Snapshots

Weekly via `smachine-btrfs-snapshot.timer` (Sunday 03:00). Last **14** kept;
older removed automatically. Snapshots of `/retrodeck` are taken the same way
on its own filesystem when a dual-disk layout is installed.

```bash
ls /.snapshots/                          # list, newest last
sudo btrfs subvolume list -s /           # details
sudo du -sh /.snapshots/* | sort -h      # sizes
```

Manual snapshot before a risky change:
```bash
sudo btrfs subvolume snapshot / "/.snapshots/manual-$(date +%Y%m%d-%H%M)/root"
```

## Rollback

Restore a known-good system subvolume. This rewrites `/` — current changes
are lost (the old root remains in place if you ever need to swap back).

```bash
# pick a snapshot
ls /.snapshots/

# swap / onto it
sudo mount -t tmpfs tmpfs /
sudo mount --bind /boot /boot
sudo btrfs subvolume snapshot /.snapshots/smachine-2026-08-15/root /root
sudo systemctl reboot
```

RetroDECK content (`/retrodeck`, ROMs/saves) is on its own subvolume and is
not affected by a system rollback.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Boots to black screen / no ES-DE | autolaunch or seat session failed | boot a TTY: `journalctl -u smachine-autolaunch -u lightdm`; re-run `flatpak run com.retrodeck.RetroDeck` to confirm the app itself works |
| Samba asks for password repeatedly | `games` has no Samba password | `sudo smbpasswd games` |
| WireGuard connects then no traffic | wrong `AllowedIPs`, or server-side peer missing | `wg show` on machine and server; compare |
| After an Arch update, things broke | broken package or config | [Rollback](#rollback) |
| Cockpit unreachable | wrong port / firewall | `systemctl status cockpit`, check https port 42069 and the LAN firewall |
| Disk full on `/` vs `/retrodeck` confusion | snapshots count against usage | `btrfs filesystem usage /`; remove old manual snapshots |
