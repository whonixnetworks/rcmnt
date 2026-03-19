# rcmnt

Rclone mount manager with health checks, cache management, and systemd integration.

---

## Quick Install

```bash
wget https://raw.githubusercontent.com/whonixnetworks/rcmnt/main/rcmnt
chmod +x rcmnt
sudo mv rcmnt /usr/local/bin/rcmnt
```

<details>
<summary>Clone the repo instead</summary>

```bash
git clone https://github.com/whonixnetworks/rcmnt.git
cd rcmnt
sudo cp rcmnt /usr/local/bin/rcmnt
sudo chmod +x /usr/local/bin/rcmnt
```

</details>

---

## Commands

| Command | Description |
|---------|-------------|
| `rcmnt` | Launch interactive menu |
| `rcmnt --setup` | First-run setup wizard |
| `rcmnt --mount` | Mount the configured remote |
| `rcmnt --unmount` | Unmount the remote |
| `rcmnt --restart` | Restart the mount |
| `rcmnt --status` | Show detailed mount status |
| `rcmnt --health` | Run health check |
| `rcmnt --watch` | Monitor resources in real-time |
| `rcmnt --logs` | View mount logs |
| `rcmnt --logs --errors` | View errors only |
| `rcmnt --logs --tail` | Follow log in real-time |
| `rcmnt --logs --full` | View full log |
| `rcmnt --clean-cache` | Clean old VFS cache files |
| `rcmnt --bwlimit 10M` | Set bandwidth limit (e.g. `512K`, `10M`, `off`) |
| `rcmnt --install-service` | Install systemd user service |
| `rcmnt --uninstall-service` | Remove systemd user service |
| `rcmnt --doctor` | Check system dependencies |
| `rcmnt --help` | Show all options |

---

## ⚠️ Warnings

<details>
<summary>FUSE requires user_allow_other for multi-user access</summary>

Without `user_allow_other` in `/etc/fuse.conf`, mounts are only accessible to the user who created them. Other users (including root via sudo) will get permission denied.

**Fix:**
```bash
echo "user_allow_other" | sudo tee -a /etc/fuse.conf
```

</details>

<details>
<summary>Stale mounts can block remounting after a crash</summary>

If the process dies uncleanly, the mount point may appear occupied even though nothing is mounted. Attempting to mount again will fail.

**Fix:**
```bash
rcmnt --health
# or force-clear the stale mount:
rcmnt --restart
```

</details>

---

## Uninstall

```bash
sudo rm /usr/local/bin/rcmnt
rm -rf ~/.config/rcmnt
rm -rf ~/.local/share/rcmnt
rm -rf ~/.cache/rcmnt
```

To also remove the systemd service if installed:

```bash
rcmnt --uninstall-service
```

---

<details>
<summary>Requirements</summary>

- Linux (or macOS with FUSE support)
- [rclone](https://rclone.org/install/) configured with at least one remote
- `fuse3` (or `fuse` on older systems / macOS)
- `bc` for byte formatting

**Install dependencies:**

```bash
# Debian/Ubuntu
sudo apt install rclone fuse3 bc

# Arch Linux
sudo pacman -S rclone fuse3 bc

# macOS
brew install rclone coreutils
```

</details>

<details>
<summary>How it works</summary>

On first run, `--setup` writes a config to `~/.config/rcmnt/config.conf` with your remote name, remote path, mount point, and optional systemd service details.

On mount, rcmnt launches rclone with VFS caching enabled and records the PID. Health checks poll the mount point and verify the rclone process is still alive. If a stale mount is detected (mount point occupied but process dead), rcmnt automatically runs `fusermount -uz` to clear it before attempting recovery.

The systemd integration creates a user-scoped service (`~/.config/systemd/user/`) so the mount starts on login without requiring root.

Bandwidth limits are applied live via rclone's RC interface without requiring a remount.

</details>

<details>
<summary>Configuration</summary>

Config file: `~/.config/rcmnt/config.conf`

| Variable | Default | Description |
|----------|---------|-------------|
| `REMOTE` | *(set by --setup)* | rclone remote name (e.g. `GCrypt`) |
| `REMOTE_PATH` | *(empty)* | Subdirectory on the remote to mount |
| `MOUNT_PATH` | `~/mnt/rclone` | Local mount point |
| `LOG_FILE` | `~/.local/share/rcmnt/mount.log` | Path to mount log |
| `BWLIMIT` | *(empty)* | Bandwidth limit (e.g. `10M`, `512K`) |

Changes take effect the next time the mount is started or restarted.

</details>

<details>
<summary>File locations</summary>

| Purpose | Path |
|---------|------|
| Config | `~/.config/rcmnt/config.conf` |
| Logs | `~/.local/share/rcmnt/mount.log` |
| VFS Cache | `~/.cache/rcmnt/vfs/` |
| Mount Point | `~/mnt/rclone` (or as configured) |
| Systemd Service | `~/.config/systemd/user/rclone-<remote>.service` |

</details>
