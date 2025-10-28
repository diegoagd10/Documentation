# Server Setup Guide

This guide covers the complete setup of Ubuntu Server for self-hosting, including installation, initial configuration, and security hardening.

## Hardware Requirements

### Dell Inspiron 3558 Specifications
- **CPU**: Intel Core i3-5015U @ 2.10GHz (2 cores, 4 threads)
- **RAM**: 4GB DDR3 @ 1600MHz (1 empty slot available for upgrade)
- **Storage**: 1TB Toshiba HDD
- **Network**:
  - Ethernet: Realtek RTL810xE
  - WiFi: Intel Wireless 3160
- **Graphics**: Intel HD Graphics 5500

## Ubuntu Server Installation

### Prerequisites
- Ubuntu Server 24.04 LTS ISO downloaded
- USB drive (8GB minimum)
- USB-to-Ethernet adapter (if needed for installation)

### Create Bootable USB

#### On Windows:
1. Download [Rufus](https://rufus.ie/) or [balenaEtcher](https://balena.io/etcher)
2. Select the Ubuntu Server ISO file
3. Select your USB drive
4. Set partition scheme to GPT and target system to UEFI
5. Click Start to flash the drive

#### On macOS:
```bash
# Identify your USB drive
diskutil list

# Unmount the drive (replace diskN with your disk identifier)
diskutil unmountDisk /dev/diskN

# Flash the Ubuntu ISO (replace diskN and the ISO path)
sudo dd if=~/Downloads/ubuntu-24.04-live-server-amd64.iso of=/dev/rdiskN bs=1m

# Eject the drive
diskutil eject /dev/diskN
```

### Installation Steps

1. Insert the USB drive and boot from it (press F12 during startup on Dell Inspiron)
2. Select "Try or Install Ubuntu Server"
3. Choose your language
4. Select your keyboard layout
5. For installation type, select "Ubuntu Server" with third-party drivers checked
6. Configure network:
   - Connect Ethernet cable or skip for wireless configuration later
   - Leave proxy blank
   - Use default mirror
7. For storage, select "Use entire disk" (this erases all data)
8. Set up your profile: Create username and password
9. Enable SSH server installation
10. Wait for installation to complete (10-20 minutes)
11. Remove USB drive when prompted and reboot

## First Boot Configuration

### Verify Network Connection

```bash
# Check network interfaces
ip link show

# Display IP addresses
ip addr show

# Show hostname and IP
hostname -I
```

### Update System

```bash
sudo apt update
sudo apt upgrade -y
```

## Network Configuration

The Dell Inspiron's Ethernet typically works automatically via DHCP. To verify:

```bash
# Test network connectivity
ping -c 4 google.com

# Check DNS resolution
nslookup google.com
```

## SSH Configuration

OpenSSH server installs automatically during Ubuntu setup.

### Generate SSH Key Pair (on your client machine)

```bash
# Generate Ed25519 key (recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy public key to server
ssh-copy-id -i ~/.ssh/id_ed25519.pub username@server-ip
```

### Optional SSH Client Configuration

On your client machine, edit `~/.ssh/config`:

```
Host ubuntu-server
    HostName server-ip-or-hostname
    User username
    IdentityFile ~/.ssh/id_ed25519
```

Connect using: `ssh ubuntu-server`

## Hostname and mDNS Configuration

### Install Avahi for mDNS

```bash
sudo apt install avahi-daemon -y
sudo systemctl enable avahi-daemon
sudo systemctl start avahi-daemon
```

Now connect using hostname: `ssh username@hostname.local`

### Discover Server on Network (from macOS)

```bash
# List SSH services
dns-sd -B _ssh._tcp

# Ping by hostname
ping hostname.local
```

## Lid-Closed Operation Configuration

Configure the server to remain running when the laptop lid closes.

### Edit Systemd Logind Configuration

```bash
sudo nano /etc/systemd/logind.conf
```

Uncomment and modify these lines:
```
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
```

### Restart Systemd-Logind Service

```bash
sudo systemctl restart systemd-logind
```

### Disable All Sleep Modes (Extra Protection)

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

### Verify Configuration

```bash
systemctl show systemd-logind | grep HandleLid
```

Expected output:
```
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
```

### Reboot to Apply Changes

```bash
sudo reboot
```

## Security Hardening

### Firewall (UFW)

```bash
# Install UFW
sudo apt install ufw -y

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (critical - do this first)
sudo ufw allow ssh

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status verbose

# View listening services
sudo ss -tulpn
```

Add additional services as needed:
```bash
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw allow 445/tcp   # Samba
```

### Disable Root Login

Verify root account status:
```bash
# Check if root is locked
sudo passwd -S root
# Should show 'L' (locked)

# Verify SSH configuration
grep "PermitRootLogin" /etc/ssh/sshd_config
# Should show 'prohibit-password' or 'no'
```

Lock root account if needed:
```bash
sudo passwd -l root
```

### Fail2Ban (Brute Force Protection)

```bash
# Install Fail2Ban
sudo apt install fail2ban -y
```

Create local configuration:
```bash
sudo nano /etc/fail2ban/jail.local
```

Add configuration:
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

Enable and start service:
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

### ClamAV Antivirus

```bash
# Install ClamAV
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

Add content:
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

Make executable and schedule:
```bash
sudo chmod +x /usr/local/bin/weekly-scan.sh

# Edit crontab for weekly scan (Sunday at 2 AM)
sudo crontab -e
```

Add line:
```
0 2 * * 0 /usr/local/bin/weekly-scan.sh
```

Check scan logs:
```bash
sudo tail -f /var/log/clamav/weekly-scan.log
```

### Automatic Security Updates

Verify unattended-upgrades:
```bash
systemctl status unattended-upgrades
```

Install if needed:
```bash
sudo apt install unattended-upgrades apt-listchanges -y
```

Configure updates:
```bash
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

Key settings:
```
Unattended-Upgrade::Automatic-Reboot "false";
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
};
Unattended-Upgrade::Automatic-Reboot-Time "03:00";
```

Enable updates:
```bash
sudo dpkg-reconfigure -plow unattended-upgrades
# Select "Yes"
```

## System Information Commands

### Hardware and System Info

```bash
# Hardware overview
sudo lshw -short

# CPU information
lscpu

# Memory information
free -h

# Disk space
df -h

# Disk usage by directory
du -sh /*

# System summary (install neofetch first)
sudo apt install neofetch
neofetch

# Interactive monitoring (install htop first)
sudo apt install htop
htop

# Temperature monitoring (install lm-sensors first)
sudo apt install lm-sensors
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

# Network speed test (install speedtest-cli first)
sudo apt install speedtest-cli
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

# Fail2Ban status
sudo fail2ban-client status sshd

# View authentication logs
sudo tail -f /var/log/auth.log

# Check failed login attempts
sudo grep "Failed password" /var/log/auth.log

# List logged-in users
who

# View login history
last

# Check for security updates
sudo apt update
apt list --upgradable
```

## Maintenance

### Regular System Updates

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
# SSH keys and configuration
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

# Fail2Ban log
sudo tail -f /var/log/fail2ban.log

# ClamAV scan log
sudo tail -f /var/log/clamav/weekly-scan.log
```

## Troubleshooting

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
# Check SSH service status
sudo systemctl status ssh

# Test SSH configuration
sudo sshd -t

# Restart SSH service
sudo systemctl restart ssh

# View SSH logs
sudo journalctl -u ssh -f
```

### Lid Close Issues

```bash
# Verify lid settings
systemctl show systemd-logind | grep HandleLid

# Check if sleep is masked
systemctl status sleep.target

# Re-mask sleep targets if needed
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

## Next Steps

After completing server setup and hardening:

1. Install Docker and Docker Compose
2. Configure firewall for Docker networks
3. Set up user accounts for services
4. Proceed to network configuration

---

**Default Credentials**:
- Username: `pikachu10` (or your configured username)
- Hostname: `svc-01` or `svc-02`
- Access: `ssh username@hostname.local`

**Notes**:
- Keep servers plugged in 24/7
- Ensure good ventilation
- Check server status occasionally
- Monitor logs regularly

**Created**: October 2025
**OS**: Ubuntu Server 24.04 LTS