# Maintenance Guide

This guide covers ongoing maintenance tasks for your self-hosted infrastructure, including monitoring, updates, troubleshooting, and system management.

## Regular Maintenance Schedule

### Daily Tasks
- Check service status and logs
- Monitor resource usage
- Review security alerts
- Verify backup completion

### Weekly Tasks
- Update system packages
- Review and clean up Docker resources
- Check SSL certificate expiration
- Monitor disk space usage

### Monthly Tasks
- Test backup restoration procedures
- Review and update security configurations
- Analyze performance metrics
- Update documentation

### Quarterly Tasks
- Major version updates for services
- Security audit and hardening review
- Hardware maintenance and checks
- Disaster recovery plan testing

## System Monitoring

### Uptime Kuma Dashboard

Access at `https://kuma.local.yourdomain.com`

#### Setting Up Monitors

1. **Service Monitors**:
   - HTTP monitors for web services
   - TCP monitors for databases
   - Ping monitors for server availability

2. **Notification Setup**:
   - Email notifications
   - Discord/Slack webhooks
   - Pushover notifications

#### Monitor Configuration Examples

**Traefik Monitor**:
- URL: `https://traefik.local.yourdomain.com`
- Monitoring interval: 60 seconds
- Expected status code: 200

**PostgreSQL Monitor**:
- Monitor type: TCP
- Hostname: `192.168.1.19`
- Port: `5432`
- Monitoring interval: 30 seconds

**n8n Monitor**:
- URL: `https://n8n.local.yourdomain.com`
- Keyword monitoring: "n8n"

### Resource Monitoring

#### Docker Resource Usage

```bash
# Monitor container resource usage
docker stats

# Check container health
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# View container logs
docker logs --tail 100 container_name
```

#### System Resource Monitoring

```bash
# CPU and memory usage
top
htop

# Disk space
df -h

# System load
uptime

# Network usage
ip -s link
```

#### Portainer Monitoring

Use Portainer dashboard to monitor:
- Container CPU/memory usage
- Network I/O
- Disk I/O
- Container logs and events

## System Updates

### Ubuntu System Updates

```bash
# Update package list
sudo apt update

# Upgrade all packages
sudo apt upgrade -y

# Remove unnecessary packages
sudo apt autoremove -y

# Clean package cache
sudo apt clean

# Check for security updates
sudo apt list --upgradable | grep security
```

### Docker Updates

```bash
# Update Docker images
docker images

# Pull latest images for running containers
docker pull image_name:latest

# Update containers (requires downtime)
docker-compose pull
docker-compose up -d
```

### Service Updates

#### Update Strategies

1. **Rolling Updates** (recommended):
   ```bash
   # Update with zero downtime
   docker-compose up -d --no-deps service_name
   ```

2. **Blue-Green Deployment**:
   - Deploy new version alongside old
   - Test new version
   - Switch traffic
   - Remove old version

#### Service-Specific Updates

**PostgreSQL**:
```bash
# Backup before updating
docker exec postgres pg_dumpall -U postgres > backup.sql

# Update minor version
docker-compose pull postgres
docker-compose up -d postgres
```

**n8n**:
```bash
# Check for updates
docker pull n8nio/n8n:latest

# Update with backup
docker-compose up -d n8n
```

**Traefik**:
```bash
# Update Traefik
docker-compose pull traefik
docker-compose up -d traefik

# Verify SSL certificates renew automatically
```

## Security Maintenance

### SSL Certificate Management

```bash
# Check certificate expiration
echo | openssl s_client -connect yourdomain.com:443 2>/dev/null | openssl x509 -noout -dates

# Force certificate renewal (if needed)
docker exec traefik traefik healthcheck
```

### Firewall Management

```bash
# Check firewall status
sudo ufw status verbose

# Review blocked connections
sudo ufw status | grep DENY

# Update rules if needed
sudo ufw allow from 192.168.1.0/24 to any port 22
```

### Fail2Ban Management

```bash
# Check status
sudo fail2ban-client status

# View banned IPs
sudo fail2ban-client status sshd

# Unban IP if needed
sudo fail2ban-client set sshd unbanip 192.168.1.100

# Check logs
sudo tail -f /var/log/fail2ban.log
```

### Antivirus Scans

```bash
# Manual scan
sudo clamscan -r --bell -i /

# Check scan results
sudo tail -f /var/log/clamav/weekly-scan.log

# Update virus definitions
sudo freshclam
```

## Backup Management

### Backup Verification

```bash
# Check backup script logs
tail -f ~/backups/backup_$(date +%Y-%m-%d).log

# Verify backup file integrity
gpg --decrypt YYYY-MM-DD_backup.tar.gz.gpg | tar -tzf - > /dev/null && echo "Backup is valid"

# Test restoration (monthly)
# Follow restoration procedures in backup guide
```

### Backup Rotation

```bash
# List old backups
rclone ls remote:backups/

# Remove backups older than 30 days (adjust as needed)
rclone delete remote:backups/ --min-age 30d

# Verify retention policy
rclone ls remote:backups/ | wc -l
```

## Performance Optimization

### Database Optimization

#### PostgreSQL

```bash
# Analyze database performance
docker exec postgres psql -U postgres -c "SELECT * FROM pg_stat_activity;"

# Vacuum database
docker exec postgres psql -U postgres -c "VACUUM ANALYZE;"

# Check table sizes
docker exec postgres psql -U postgres -c "SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) FROM pg_tables ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;"
```

#### Redis

```bash
# Check memory usage
docker exec redis redis-cli info memory

# Monitor connected clients
docker exec redis redis-cli info clients

# Check slow logs
docker exec redis redis-cli slowlog get 10
```

### Docker Optimization

```bash
# Clean up unused images
docker image prune -f

# Clean up unused volumes
docker volume prune -f

# Clean up unused networks
docker network prune -f

# System-wide cleanup
docker system prune -f
```

### System Optimization

```bash
# Check system load
uptime

# Monitor I/O performance
iostat -x 1 5

# Check memory usage
free -h

# Optimize swap usage
sudo sysctl vm.swappiness=10
```

## Troubleshooting

### Service Startup Issues

```bash
# Check service status
sudo systemctl status docker

# View service logs
sudo journalctl -u docker -f

# Check container logs
docker logs container_name --tail 50

# Validate compose configuration
docker-compose config
```

### Network Issues

```bash
# Test DNS resolution
nslookup yourdomain.com

# Check network connectivity
ping -c 4 8.8.8.8

# Verify port availability
sudo netstat -tulpn | grep :80

# Check Traefik configuration
docker logs traefik
```

### Database Connection Issues

```bash
# Test PostgreSQL connection
docker exec postgres pg_isready -U postgres

# Check Redis connectivity
docker exec redis redis-cli ping

# Verify Qdrant health
curl http://qdrant:6333/health
```

### SSL/TLS Issues

```bash
# Check certificate validity
openssl s_client -connect yourdomain.com:443 -servername yourdomain.com < /dev/null 2>/dev/null | openssl x509 -noout -dates

# Test SSL configuration
openssl s_client -connect yourdomain.com:443 -servername yourdomain.com

# Check Traefik SSL logs
docker logs traefik | grep -i ssl
```

### Performance Issues

```bash
# Monitor system resources
htop

# Check disk I/O
iotop

# Analyze network traffic
sudo nload

# Check for memory leaks
docker stats --no-stream
```

## Log Management

### System Logs

```bash
# View system logs
sudo journalctl -f

# Filter by service
sudo journalctl -u ssh -f

# View logs by time range
sudo journalctl --since "1 hour ago"

# Check disk usage by logs
sudo du -sh /var/log/*
```

### Application Logs

```bash
# n8n logs
docker logs n8n --tail 100 -f

# PostgreSQL logs
docker logs postgres

# Traefik access logs
docker logs traefik | grep -E "(GET|POST)"
```

### Log Rotation

```bash
# Check log rotation configuration
cat /etc/logrotate.conf

# Manual log rotation
sudo logrotate -f /etc/logrotate.d/rsyslog

# Clean old logs
sudo find /var/log -type f -name "*.gz" -mtime +30 -delete
```

## Hardware Maintenance

### Temperature Monitoring

```bash
# Check CPU temperature
sensors

# Monitor temperatures over time
watch -n 5 sensors
```

### Disk Health

```bash
# Check disk usage
df -h

# Check disk health (if supported)
sudo smartctl -a /dev/sda

# Check for disk errors
dmesg | grep -i "sda\|disk"
```

### Power Management

```bash
# Check power status
upower -i /org/freedesktop/UPower/devices/battery_BAT0

# Verify lid settings
systemctl show systemd-logind | grep HandleLid
```

## Emergency Procedures

### Service Failure Response

1. **Identify the issue**:
   ```bash
   docker ps -a
   docker logs failing_container
   ```

2. **Attempt restart**:
   ```bash
   docker restart failing_container
   ```

3. **Check dependencies**:
   ```bash
   docker-compose ps
   ```

4. **Review recent changes**:
   ```bash
   docker events --since '1h'
   ```

5. **Escalate if needed**:
   - Check system resources
   - Review error logs
   - Contact support if applicable

### Data Loss Recovery

1. **Stop affected services**
2. **Assess damage extent**
3. **Restore from backup** (see backup guide)
4. **Verify data integrity**
5. **Resume services**

### Security Incident Response

1. **Isolate affected systems**
2. **Document the incident**
3. **Analyze logs for breach details**
4. **Apply security patches**
5. **Change credentials**
6. **Report if required**

## Documentation Updates

### Maintenance Logging

Keep a maintenance log:

```bash
# Create maintenance log entry
echo "$(date): Performed system update and backup verification" >> ~/maintenance.log

# Review maintenance history
tail -20 ~/maintenance.log
```

### Procedure Updates

- Update runbooks after changes
- Document new troubleshooting steps
- Review and update contact information
- Maintain change management records

---

**Note**: Regular maintenance is crucial for system reliability and security. Schedule these tasks and track completion to ensure consistent system health.

**Created**: October 2025