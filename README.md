# rcmnt

> Rclone mount manager with health checks, cache management, and monitoring

A bash script that makes rclone mounting easy with an interactive menu, automatic health checks, cache management, and systemd integration for auto-start.

## Features

- **Interactive Menu** - User-friendly TUI with keyboard navigation
- **Health Monitoring** - Automatic checks for mount health and network connectivity
- **Cache Management** - Clean old cache files to free up space
- **Bandwidth Control** - Limit upload/download speeds
- **Log Management** - View and rotate mount logs
- **Systemd Integration** - Auto-start on boot with user services
- **Resource Monitor** - Real-time CPU, memory, and thread monitoring
- **Stale Mount Recovery** - Automatic cleanup of orphaned mounts

## Prerequisites

- [rclone](https://rclone.org/install/) configured with at least one remote
- FUSE support (fuse3 or fuse)
- `bc` for byte formatting
- Linux or macOS

### Install Dependencies

```bash
# Debian/Ubuntu
sudo apt install rclone fuse3 bc

# Arch Linux
sudo pacman -S rclone fuse3 bc

# macOS
brew install rclone coreutils
```

**Note:** On Linux, ensure `user_allow_other` is enabled in `/etc/fuse.conf` for multi-user access:

```bash
echo "user_allow_other" | sudo tee -a /etc/fuse.conf
```

## Installation

### Quick Install

```bash
curl -fsSL https://raw.githubusercontent.com/whonixnetworks/rcmnt/main/rcmnt -o ~/rcmnt
chmod +x ~/rcmnt
```

Or clone and install:

```bash
git clone https://github.com/whonixnetworks/rcmnt.git
cd rcmnt
chmod +x rcmnt
./rcmnt --setup
```

### Install to System Path (optional)

```bash
sudo cp rcmnt /usr/local/bin/rcmnt
sudo chmod +x /usr/local/bin/rcmnt
```

## First-Time Setup

Run the setup wizard to configure your remote:

```bash
rcmnt --setup
```

You'll be prompted for:
- **Remote name** - Your rclone remote (e.g., `GCrypt`, `GDrive`)
- **Remote path** - Subdirectory to mount (optional, leave empty for root)
- **Mount point** - Where to mount the drive (default: `~/mnt/rclone`)
- **Systemd service** - Option to enable auto-start on boot

## Usage

### Interactive Menu (Default)

```bash
rcmnt
```

```
╭─────────────────────────────────────╮
║       RCLONE CONTROLS               ║
╠─────────────────────────────────────┤
│  [M]ount       [U]nmount      [R]estart
│  [S]tatus      [W]atch        [H]ealth
│  [C]lean Cache [L]ogs         [Q]uit
```

### Command Line Options

| Command | Description |
|---------|-------------|
| `rcmnt --setup` | First-run setup wizard |
| `rcmnt --mount` | Mount the remote drive |
| `rcmnt --unmount` | Unmount the remote drive |
| `rcmnt --restart` | Restart the mount |
| `rcmnt --status` | Show detailed status |
| `rcmnt --health` | Run health check |
| `rcmnt --watch` | Monitor resources in real-time |
| `rcmnt --logs` | View logs (use `--errors` for errors only) |
| `rcmnt --clean-cache` | Clean old cache files |
| `rcmnt --bwlimit 10M` | Set bandwidth limit |
| `rcmnt --doctor` | Check system dependencies |
| `rcmnt --menu` | Launch interactive menu |
| `rcmnt --help` | Show all options |

### Bandwidth Control

```bash
# Limit to 10 MB/s
rcmnt --bwlimit 10M

# Limit to 512 KB/s
rcmnt --bwlimit 512K

# Remove limit
rcmnt --bwlimit off
```

### Log Management

```bash
# View last 50 lines
rcmnt --logs

# View errors only
rcmnt --logs --errors

# Follow log in real-time
rcmnt --logs --tail

# View full log
rcmnt --logs --full
```

### Systemd Service (Auto-Start)

Install a user service that starts the mount on login:

```bash
rcmnt --install-service
systemctl --user enable rclone-<remote>.service
systemctl --user start rclone-<remote>.service
```

Uninstall:

```bash
rcmnt --uninstall-service
```

## Configuration

Config file location: `~/.config/rcmnt/config.conf`

```bash
REMOTE="GCrypt"
REMOTE_PATH="media"
MOUNT_PATH="$HOME/mnt/rclone"
LOG_FILE="$HOME/.local/share/rcmnt/mount.log"
BWLIMIT=""
```

## File Locations

| Purpose | Default Path |
|---------|--------------|
| Config | `~/.config/rcmnt/config.conf` |
| Logs | `~/.local/share/rcmnt/mount.log` |
| VFS Cache | `~/.cache/rcmnt/vfs/` |
| Mount Point | `~/mnt/rclone` (or custom) |

## Troubleshooting

### Mount fails with "permission denied"

Ensure `user_allow_other` is enabled in `/etc/fuse.conf`:

```bash
echo "user_allow_other" | sudo tee -a /etc/fuse.conf
```

### Stale mount detected

Run health check:

```bash
rcmnt --health
```

Or restart:

```bash
rcmnt --restart
```

### Check dependencies

```bash
rcmnt --doctor
```

### View errors

```bash
rcmnt --logs --errors
```

## License

MIT License - see [LICENSE](LICENSE) for details.
