# Backup and Restoration Guide

This guide provides step-by-step instructions for restoring your self-hosted services from backups. It covers installing Portainer, restoring Docker stacks, decrypting environment files, and restoring databases.

## 1. Install Portainer

Portainer is a web-based Docker management interface. Install it using the following command:

```bash
docker run -d \
  --name portainer \
  --restart always \
  -p 8000:8000 \
  -p 9443:9443 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

Access Portainer at `https://your-server-ip:9443` after installation.

## 2. Restore Portainer Stacks

Depending on which server you are restoring to, use the appropriate Docker Compose files:

- For `svc-01` (n8n service): Use `svc-01/n8n.yml`
- For `svc-02` (databases): Use `svc-02/databases.yml`

In Portainer:
1. Navigate to "Stacks"
2. Create a new stack
3. Upload or paste the contents of the relevant `.yml` file
4. Set environment variables as needed
5. Deploy the stack

## 3. Decrypt Environment Files

Environment files are stored encrypted in your password manager. Decrypt them using GPG:

```bash
gpg --decrypt .env.gpg
```

This command will output the decrypted `.env` file to the current directory.

## 4. Restore Databases

After restoring applications, restore the associated databases.

### 4.1 Locate Backup Files

Database backups are stored on Google Drive in the "backups" directory as encrypted `.tar.gz.gpg` files.

### 4.2 Decrypt the Backup File

Use the decryption key from your password manager to decrypt the file:

```bash
gpg --output=backup.tar.gz --decrypt backup.tar.gz.gpg
```

### 4.3 Extract the Backup Archive

Extract the tar archive:

```bash
tar -xzf backup.tar.gz
```

This will create a directory named with the backup date in `YYYY-MM-DD` format.

### 4.4 Navigate to Backup Directory

Change into the extracted backup directory:

```bash
cd YYYY-MM-DD  # Replace with actual date
```

### 4.5 Extract Database Dump

Unzip the PostgreSQL backup file:

```bash
gunzip postgre_backup.sql.gz
```

### 4.6 Restore Database

Run the following command on the PostgreSQL container to restore the database. Replace `<POSTGRES_CONTAINER_ID>` with the actual container ID or name, and `<ROOT_USER>` with the appropriate database user:

```bash
docker exec -i <POSTGRES_CONTAINER_ID> psql -U <ROOT_USER> postgres < postgre_backup.sql
```

## 5. Restore n8n Workflows

To restore n8n workflows and credentials, follow the official n8n documentation:

[https://docs.n8n.io/hosting/cli-commands/#workflows_1](https://docs.n8n.io/hosting/cli-commands/#workflows_1)

Use the n8n CLI commands to import workflows from your backup files.