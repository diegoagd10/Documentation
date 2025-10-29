# Adding New Environment Variables

This guide covers the process of adding new environment variables to your self-hosted services.

## Prerequisites

- Access to encrypted environment files
- GPG encryption tools installed
- Password manager with encryption passphrase
- Cloud storage access for environment files

## Steps

1. **Download encrypted environments zip** from cloud storage
2. **Unzip** the downloaded file:
   ```bash
   unzip environments.zip
   ```
3. **Decrypt existing files** (if editing existing ones):
   ```bash
   gpg --decrypt .env.gpg > .env
   ```
4. **Create or edit environment files** with new variables:
   ```bash
   nano .env
   # Add new variables in KEY=VALUE format
   ```
5. **Encrypt files** using `gpg -c <new_file>` and provide the passphrase from password manager:
   ```bash
   gpg -c .env
   # Enter passphrase when prompted
   ```
6. **Remove unencrypted files** for security:
   ```bash
   rm .env
   ```
7. **Zip encrypted files** using `zip file.1.gpg file2.gpg`:
   ```bash
   zip environments.zip *.gpg
   ```
8. **Upload environment files** back to secure cloud storage

## Security Notes

- Never store unencrypted environment files
- Use strong passphrases from password manager
- Regularly rotate encryption keys
- Limit access to encrypted files

---

**Created**: October 2025
