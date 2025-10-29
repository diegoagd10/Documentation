# Backup and Restoration Guide

This guide covers creating, encrypting, and restoring backups of your self-hosted services, including databases, configurations, and application data.

## Backup Strategy Overview

- **Frequency**: Weekly automated backups
- **Storage**: Encrypted files stored off-site (Google Drive)
- **Encryption**: GPG encryption with password manager keys
- **Scope**: Databases, configurations, and critical application data

## Prerequisites

- GPG encryption keys set up
- Password manager with backup decryption keys
- Google Drive or similar off-site storage
- Access to all servers and services

## Creating Backups

### Database Backups

#### PostgreSQL Backup

```bash
# Create backup directory
mkdir -p ~/backups/$(date +%Y-%m-%d)

# Backup PostgreSQL database
docker exec postgres pg_dumpall -U postgres > ~/backups/$(date +%Y-%m-%d)/postgres_backup.sql

# Compress the backup
cd ~/backups/$(date +%Y-%m-%d)
gzip postgres_backup.sql
```

#### Redis Backup

Redis uses append-only file (AOF) persistence. The data is automatically persisted to the `redis_data` volume.

#### Qdrant Backup

```bash
# Create snapshot
curl -X POST http://qdrant:6333/snapshots

# Copy snapshot from container
docker cp qdrant:/qdrant/snapshots/snapshot_name ~/backups/$(date +%Y-%m-%d)/
```

### Application Data Backups

#### n8n Workflows and Credentials

```bash
# Export workflows using n8n CLI
docker exec n8n n8n export:workflows --output=/tmp/workflows.json
docker cp n8n:/tmp/workflows.json ~/backups/$(date +%Y-%m-%d)/

# Export credentials
docker exec n8n n8n export:credentials --output=/tmp/credentials.json
docker cp n8n:/tmp/credentials.json ~/backups/$(date +%Y-%m-%d)/
```

#### Docker Volumes Backup

```bash
# Backup all Docker volumes
docker run --rm -v /var/lib/docker/volumes:/volumes -v $(pwd):/backup alpine tar czf /backup/volumes_backup.tar.gz -C /volumes .
```

### Configuration Backups

#### System Configuration

```bash
# Backup important system files
sudo tar czf ~/backups/$(date +%Y-%m-%d)/system_config.tar.gz \
  /etc/ssh/sshd_config \
  /etc/fail2ban/jail.local \
  /etc/ufw/ufw.conf \
  /home/username/.ssh/ \
  /etc/crontab
```

#### Docker Compose Files

```bash
# Backup all compose files
cp -r ~/docker-compose-files ~/backups/$(date +%Y-%m-%d)/
```

### Environment Variables Backup

```bash
# Create encrypted backup of environment variables
# Store in password manager or encrypted file
echo "BACKUP_DATE=$(date)" > ~/backups/$(date +%Y-%m-%d)/env_backup.txt
# Add all environment variables manually or from secure source
```

## Encryption and Storage

### Encrypt Backup Archive

```bash
# Create compressed archive
cd ~/backups
tar czf $(date +%Y-%m-%d)_backup.tar.gz $(date +%Y-%m-%d)/

# Encrypt with GPG
gpg --encrypt --recipient your_email@example.com $(date +%Y-%m-%d)_backup.tar.gz

# Remove unencrypted files
rm -rf $(date +%Y-%m-%d)/ $(date +%Y-%m-%d)_backup.tar.gz
```

### Upload to Off-site Storage

```bash
# Upload to Google Drive (using rclone or similar)
rclone copy $(date +%Y-%m-%d)_backup.tar.gz.gpg remote:backups/

# Verify upload
rclone ls remote:backups/
```

## Automated Backup Script

Create a comprehensive backup script:

```bash
#!/bin/bash
# Comprehensive backup script

BACKUP_DATE=$(date +%Y-%m-%d)
BACKUP_DIR=~/backups/$BACKUP_DATE
LOG_FILE=~/backups/backup_$BACKUP_DATE.log

# Create backup directory
mkdir -p $BACKUP_DIR

echo "=== Backup Started: $BACKUP_DATE ===" > $LOG_FILE

# Database backups
echo "Backing up PostgreSQL..." >> $LOG_FILE
docker exec postgres pg_dumpall -U postgres > $BACKUP_DIR/postgres_backup.sql 2>> $LOG_FILE
gzip $BACKUP_DIR/postgres_backup.sql

echo "Backing up Redis data..." >> $LOG_FILE
docker run --rm -v redis_data:/data -v $BACKUP_DIR:/backup alpine tar czf /backup/redis_backup.tar.gz -C /data . 2>> $LOG_FILE

# Application data
echo "Backing up n8n data..." >> $LOG_FILE
docker exec n8n n8n export:workflows --output=/tmp/workflows.json 2>> $LOG_FILE
docker cp n8n:/tmp/workflows.json $BACKUP_DIR/ 2>> $LOG_FILE

# System configuration
echo "Backing up system config..." >> $LOG_FILE
sudo tar czf $BACKUP_DIR/system_config.tar.gz /etc/ssh/sshd_config /etc/fail2ban/ /etc/ufw/ /home/$USER/.ssh/ 2>> $LOG_FILE

# Compress and encrypt
echo "Compressing and encrypting..." >> $LOG_FILE
cd ~/backups
tar czf ${BACKUP_DATE}_backup.tar.gz $BACKUP_DATE/ 2>> $LOG_FILE
gpg --encrypt --recipient your_email@example.com ${BACKUP_DATE}_backup.tar.gz 2>> $LOG_FILE

# Upload to cloud
echo "Uploading to cloud storage..." >> $LOG_FILE
rclone copy ${BACKUP_DATE}_backup.tar.gz.gpg remote:backups/ 2>> $LOG_FILE

# Cleanup
rm -rf $BACKUP_DIR ${BACKUP_DATE}_backup.tar.gz

echo "=== Backup Completed: $BACKUP_DATE ===" >> $LOG_FILE
echo "Backup saved to: remote:backups/${BACKUP_DATE}_backup.tar.gz.gpg" >> $LOG_FILE
```

Make executable and schedule:

```bash
chmod +x ~/backup-script.sh
sudo crontab -e
# Add: 0 2 * * 0 ~/backup-script.sh  # Weekly at 2 AM Sunday
```

## Restoration Process

### Download and Decrypt Backup

```bash
# Download from cloud storage
rclone copy remote:backups/YYYY-MM-DD_backup.tar.gz.gpg .

# Decrypt backup
gpg --output backup.tar.gz --decrypt YYYY-MM-DD_backup.tar.gz.gpg

# Extract backup
tar -xzf backup.tar.gz
cd YYYY-MM-DD
```

### Restore Databases

#### PostgreSQL Restoration

```bash
# Extract PostgreSQL dump
gunzip postgres_backup.sql.gz

# Restore to database
docker exec -i postgres psql -U postgres postgres < postgres_backup.sql
```

#### Redis Restoration

```bash
# Stop Redis container
docker stop redis redis-commander

# Restore volume data
docker run --rm -v redis_data:/data -v $(pwd):/backup alpine sh -c "cd /data && tar xzf /backup/redis_backup.tar.gz"

# Start Redis container
docker start redis redis-commander
```

#### Qdrant Restoration

```bash
# Copy snapshot back to container
docker cp snapshot_name qdrant:/qdrant/snapshots/

# Restore from snapshot
curl -X POST http://qdrant:6333/snapshots/snapshot_name/restore
```

### Restore Application Data

#### n8n Workflows and Credentials

```bash
# Copy files to n8n container
docker cp workflows.json n8n:/tmp/
docker cp credentials.json n8n:/tmp/

# Import using n8n CLI
docker exec n8n n8n import:workflows --input=/tmp/workflows.json
docker exec n8n n8n import:credentials --input=/tmp/credentials.json
```

### Restore System Configuration

```bash
# Extract system configuration
tar -xzf system_config.tar.gz

# Restore SSH configuration
sudo cp etc/ssh/sshd_config /etc/ssh/
sudo systemctl restart ssh

# Restore firewall rules
sudo cp -r etc/ufw/* /etc/ufw/
sudo ufw reload

# Restore fail2ban configuration
sudo cp -r etc/fail2ban/* /etc/fail2ban/
sudo systemctl restart fail2ban
```

## Portainer Stack Restoration

### Install Portainer (if needed)

Deploy Portainer using the following Docker command:

```bash
docker run -d --name portainer --restart always -p 9000:9000 -p 9443:9443 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

**Note**: This command deploys Portainer with default settings. For production environments, consider additional security configurations and network isolation.

### Restore Docker Stacks

1. Access Portainer web interface
2. Navigate to "Stacks"
3. Create new stacks using your backed-up compose files
4. Set environment variables from your secure backup
5. Deploy stacks

## Testing Restoration

### Verify Database Integrity

```bash
# Test PostgreSQL
docker exec postgres psql -U postgres -c "SELECT version();"

# Test Redis
docker exec redis redis-cli ping

# Test Qdrant
curl http://qdrant:6333/health
```

### Verify Application Functionality

1. Access each service through Traefik URLs
2. Test n8n workflows
3. Verify Homepage dashboard loads
4. Check Uptime Kuma monitors

### Validate Configurations

```bash
# Check SSH configuration
sudo sshd -t

# Verify firewall rules
sudo ufw status

# Test fail2ban
sudo fail2ban-client status
```

## Backup Verification

### Automated Verification Script

```bash
#!/bin/bash
# Backup verification script

BACKUP_FILE=$1
LOG_FILE=~/backup_verification.log

echo "=== Backup Verification Started: $(date) ===" > $LOG_FILE

# Check file exists and is readable
if [ ! -f "$BACKUP_FILE" ]; then
    echo "ERROR: Backup file not found" >> $LOG_FILE
    exit 1
fi

# Test decryption
gpg --decrypt $BACKUP_FILE > /dev/null 2>> $LOG_FILE
if [ $? -ne 0 ]; then
    echo "ERROR: Cannot decrypt backup file" >> $LOG_FILE
    exit 1
fi

# Test archive integrity
gpg --decrypt $BACKUP_FILE | tar -tzf - > /dev/null 2>> $LOG_FILE
if [ $? -ne 0 ]; then
    echo "ERROR: Backup archive is corrupted" >> $LOG_FILE
    exit 1
fi

echo "SUCCESS: Backup file is valid and readable" >> $LOG_FILE
echo "=== Backup Verification Completed: $(date) ===" >> $LOG_FILE
```

## Disaster Recovery Plan

1. **Immediate Actions**:
   - Assess damage and determine recovery scope
   - Contact hosting provider if applicable
   - Secure backup files from cloud storage

2. **Server Recovery**:
   - Reinstall Ubuntu Server following setup guide
   - Restore system configuration from backups
   - Reinstall Docker and services

3. **Data Recovery**:
   - Decrypt and extract backup files
   - Restore databases in correct order
   - Restore application data and configurations

4. **Service Recovery**:
   - Deploy services using Portainer
   - Test all functionality
   - Update DNS and monitoring

5. **Verification**:
   - Run comprehensive tests
   - Update documentation
   - Communicate with users

## Best Practices

- **Test Restorations**: Regularly test backup restoration procedures
- **Multiple Locations**: Store backups in multiple geographic locations
- **Encryption**: Always encrypt sensitive backups
- **Documentation**: Keep backup procedures documented and updated
- **Monitoring**: Monitor backup success/failure
- **Retention**: Implement backup retention policies

---

**Note**: Always test backup and restoration procedures in a development environment before relying on them for production systems.

**Created**: October 2025