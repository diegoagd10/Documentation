# Ubuntu Server Setup Guide - Dell Inspiron 3558

Complete setup guide for converting an old Dell Inspiron laptop into a home Ubuntu Server.

## Hardware Specifications

- **Model**: Dell Inspiron 3558
- **CPU**: Intel Core i3-5015U @ 2.10GHz (2 cores, 4 threads)
- **RAM**: 4GB DDR3 @ 1600MHz (1 empty slot for upgrade)
- **Storage**: 1TB Toshiba HDD
- **Network**: 
  - Ethernet: Realtek RTL810xE
  - WiFi: Intel Wireless 3160
- **Graphics**: Intel HD Graphics 5500

## 1. Ubuntu Server Installation

### Prerequisites
- Ubuntu Server ISO downloaded
- USB drive (8GB minimum)
- USB-to-Ethernet adapter (if needed)

### Create Bootable USB

**On Windows:**
1. Download [Rufus](https://rufus.ie/) or [balenaEtcher](https://balena.io/etcher)
2. Select Ubuntu Server ISO
3. Select USB drive
4. Settings: GPT partition scheme, UEFI target system
5. Flash the drive

**On Mac:**
```bash
# Find USB drive
diskutil list

# Unmount it (replace diskN with your disk number)
diskutil unmountDisk /dev/diskN

# Flash Ubuntu ISO
sudo dd if=~/Downloads/ubuntu-*.iso of=/dev/rdiskN bs=1m

# Eject when done
diskutil eject /dev/diskN
```

### Installation Steps

1. Insert USB drive and boot from it (hold F12 during startup)
2. Select "Try or Install Ubuntu Server"
3. Choose language
4. Select keyboard layout
5. **Installation type**: Ubuntu Server (with third-party drivers checked)
6. **Network**: Connect Ethernet or skip for now
7. **Proxy**: Leave blank
8. **Mirror**: Use default
9. **Storage**: "Use entire disk" - erases all data
10. **Profile setup**: Create username and password
11. **SSH setup**: Install OpenSSH server
12. Wait for installation (10-20 minutes)
13. Remove USB when prompted and reboot

## 2. First Boot Configuration

### Check Network Connection

```bash
# View network interfaces
ip link show

# View IP addresses
ip addr show

# View network configuration
hostname -I
```

### Update System

```bash
sudo apt update
sudo apt upgrade -y
```

## 3. Network Configuration

The Dell Inspiron's Ethernet worked automatically via DHCP - no configuration needed! ✓

If you need to check your network status:

```bash
# View network interfaces
ip link show

# View IP addresses
ip addr show

# Test connectivity
ping -c 4 google.com
```

## 4. SSH Configuration

OpenSSH server was already installed during Ubuntu installation (step 11). ✓

### Generate SSH Key on Client (Mac/Linux)

```bash
# Generate Ed25519 key (modern, secure)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy public key to server
ssh-copy-id -i ~/.ssh/id_ed25519.pub username@server-ip
```

### Configure SSH Client (Optional)

On your client machine:
```bash
nano ~/.ssh/config
```

Add:
```
Host ubuntu-server
    HostName server-ip-or-hostname
    User username
    IdentityFile ~/.ssh/id_ed25519
```

Now connect with: `ssh ubuntu-server`

## 5. Hostname and mDNS Configuration

### Install Avahi for mDNS (.local domain)

```bash
sudo apt install avahi-daemon -y
sudo systemctl enable avahi-daemon
sudo systemctl start avahi-daemon
```

Now you can connect using: `ssh username@svc-02.local`

### Discover Server on Network (from Mac)

```bash
# List SSH services
dns-sd -B _ssh._tcp

# Ping by hostname
ping svc-02.local
```

## 6. Lid-Closed Operation

Configure the laptop to stay on when the lid is closed:

### Edit Login Configuration

```bash
sudo nano /etc/systemd/logind.conf
```

Find and modify these lines (remove # and change suspend to ignore):
```
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
```

### Restart Service

```bash
sudo systemctl restart systemd-logind
```

### Disable All Sleep Modes (Extra Protection)

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

### Verify Settings

```bash
systemctl show systemd-logind | grep HandleLid
```

Should show:
```
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
```

### Reboot to Apply

```bash
sudo reboot
```

## 7. Security Hardening

### 7.1 Firewall (UFW)

```bash
# Install UFW
sudo apt install ufw -y

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (IMPORTANT - do this first!)
sudo ufw allow ssh

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status verbose

# See what's listening
sudo ss -tulpn
```

Add other services as needed:
```bash
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw allow 445/tcp   # Samba
```

### 7.2 Disable Root Login

Check root status:
```bash
# Verify root is locked
sudo passwd -S root
# Should show 'L' (locked)

# Check SSH configuration
grep "PermitRootLogin" /etc/ssh/sshd_config
# Should show 'prohibit-password' or 'no'
```

If needed, lock root account:
```bash
sudo passwd -l root
```

### 7.3 Fail2Ban (Brute Force Protection)

Install and configure:
```bash
sudo apt install fail2ban -y
```

Create local configuration:
```bash
sudo nano /etc/fail2ban/jail.local
```

Add:
```ini
[DEFAULT]
# Ban for 1 week (604800 seconds)
bantime = 604800

# Time window to count failures (24 hours)
findtime = 86400

# Max retries before ban
maxretry = 5

# Ban action
banaction = iptables-multiport

# Don't ban local network
ignoreip = 127.0.0.1/8 ::1 192.168.1.0/24

[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 5
```

Enable and start:
```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Check status:
```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

View logs:
```bash
sudo tail -f /var/log/fail2ban.log
```

Manually unban IP if needed:
```bash
sudo fail2ban-client set sshd unbanip 192.168.1.100
```

### 7.4 ClamAV Antivirus

Install ClamAV:
```bash
sudo apt install clamav clamav-daemon -y
```

Update virus definitions:
```bash
sudo systemctl stop clamav-freshclam
sudo freshclam
sudo systemctl start clamav-freshclam
```

Create weekly scan script:
```bash
sudo nano /usr/local/bin/weekly-scan.sh
```

Add:
```bash
#!/bin/bash
# Weekly ClamAV scan script

SCAN_DIR="/"
LOG_FILE="/var/log/clamav/weekly-scan.log"
DATE=$(date +%Y-%m-%d)

echo "=== ClamAV Scan Started: $DATE ===" >> $LOG_FILE

clamscan -r -i \
  --exclude-dir=/sys \
  --exclude-dir=/proc \
  --exclude-dir=/dev \
  $SCAN_DIR >> $LOG_FILE 2>&1

echo "=== ClamAV Scan Completed: $DATE ===" >> $LOG_FILE
echo "" >> $LOG_FILE
```

Make executable:
```bash
sudo chmod +x /usr/local/bin/weekly-scan.sh
```

Schedule weekly scan (every Sunday at 2 AM):
```bash
sudo crontab -e
```

Add:
```
0 2 * * 0 /usr/local/bin/weekly-scan.sh
```

Check scan logs:
```bash
sudo tail -f /var/log/clamav/weekly-scan.log
```

### 7.5 Automatic Security Updates

Unattended-upgrades is usually pre-installed. Verify:
```bash
systemctl status unattended-upgrades
```

If not installed:
```bash
sudo apt install unattended-upgrades apt-listchanges -y
```

Configure (optional):
```bash
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

Key settings:
```
// Auto-reboot if needed (false for servers)
Unattended-Upgrade::Automatic-Reboot "false";

// Only security updates (recommended)
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
};

// Reboot time if enabled
Unattended-Upgrade::Automatic-Reboot-Time "03:00";
```

Enable:
```bash
sudo dpkg-reconfigure -plow unattended-upgrades
```

Choose "Yes"

## 8. Useful Commands

### System Information

```bash
# Hardware overview
sudo lshw -short

# CPU info
lscpu

# Memory info
free -h

# Disk space
df -h

# Disk usage by directory
du -sh /*

# System summary (install first: sudo apt install neofetch)
neofetch

# Interactive resource monitor (install first: sudo apt install htop)
htop

# Temperature monitoring (install first: sudo apt install lm-sensors)
sudo sensors-detect  # Answer YES to all
sensors
```

### Network Commands

```bash
# Show network interfaces
ip link show

# Show IP addresses
ip addr show

# Show routing table
ip route show

# Test DNS
nslookup google.com

# Test connectivity
ping -c 4 google.com

# Show listening ports
sudo ss -tulpn

# Network speed test (install first: sudo apt install speedtest-cli)
speedtest-cli
```

### Service Management

```bash
# Check service status
sudo systemctl status service-name

# Start service
sudo systemctl start service-name

# Stop service
sudo systemctl stop service-name

# Restart service
sudo systemctl restart service-name

# Enable service (start on boot)
sudo systemctl enable service-name

# Disable service
sudo systemctl disable service-name

# View service logs
sudo journalctl -u service-name -f
```

### Security Commands

```bash
# Check open ports
sudo ss -tulpn

# UFW firewall status
sudo ufw status verbose

# Fail2ban status
sudo fail2ban-client status sshd

# View authentication logs
sudo tail -f /var/log/auth.log

# Check failed login attempts
sudo grep "Failed password" /var/log/auth.log

# List currently logged in users
who

# View last logins
last

# Check for security updates
sudo apt update
apt list --upgradable
```

## 9. Maintenance

### Regular Updates

```bash
# Update package list and upgrade all packages
sudo apt update && sudo apt upgrade -y

# Remove old packages
sudo apt autoremove -y

# Clean package cache
sudo apt clean
```

### Backup Important Files

```bash
# SSH keys and config
/home/username/.ssh/

# System configuration
/etc/

# Cron jobs
crontab -l

# Service configurations
/etc/systemd/system/
```

### Monitor Logs

```bash
# System log
sudo journalctl -f

# Authentication log
sudo tail -f /var/log/auth.log

# Fail2ban log
sudo tail -f /var/log/fail2ban.log

# ClamAV scan log
sudo tail -f /var/log/clamav/weekly-scan.log
```

## 10. Troubleshooting

### Network Issues

```bash
# Restart networking
sudo systemctl restart systemd-networkd

# Reapply netplan
sudo netplan apply

# Check DNS resolution
systemd-resolve --status
```

### SSH Issues

```bash
# Check SSH service
sudo systemctl status ssh

# Test SSH configuration
sudo sshd -t

# Restart SSH
sudo systemctl restart ssh

# View SSH logs
sudo journalctl -u ssh -f
```

### System Won't Stay On When Lid Closed

```bash
# Verify lid settings
systemctl show systemd-logind | grep HandleLid

# Check if sleep is masked
systemctl status sleep.target

# Re-mask if needed
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

## 11. Next Steps

Now that your server is set up and secured, you can install services:

- **File Server**: Samba, NFS
- **Web Server**: Nginx, Apache
- **Media Server**: Plex, Jellyfin
- **Docker**: For containerized applications
- **VPN**: WireGuard, OpenVPN
- **Git Server**: Gitea, GitLab
- **Home Automation**: Home Assistant
- **Ad Blocker**: Pi-hole

## Security Summary

Your server is now protected with:

✅ Firewall (UFW) - Only SSH open  
✅ Root login disabled  
✅ Fail2Ban - Blocks brute force attempts (5 attempts/24hrs → 1 week ban)  
✅ ClamAV - Weekly virus scans  
✅ Automatic security updates  
✅ SSH key authentication  
✅ mDNS hostname resolution  

## Notes

- Default username: `pikachu10`
- Hostname: `svc-02`
- Access via: `ssh pikachu10@svc-02.local` or `ssh ubuntu-server`
- Keep laptop plugged in 24/7
- Ensure good ventilation
- Check on server occasionally to ensure it's running smoothly

---

**Created**: October 2025  
**Server**: Dell Inspiron 3558  
**OS**: Ubuntu Server 24.04 LTS
