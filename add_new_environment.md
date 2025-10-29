# Adding New Environment Variables

This guide covers the process of adding new environment variables to your self-hosted services.

## Prerequisites

- Access to encrypted environment files
- GPG encryption tools installed
- Password manager with encryption passphrase
- Cloud storage access for environment files

## Steps

1. **Download encrypted environments zip** from cloud storage
2. **Unzip** the downloaded file
3. **Create or edit environment files** with new variables
4. **Encrypt files** using `gpg -c <new_file>` and provide the passphrase from password manager
5. **Zip encrypted files** using `zip file.1.gpg file2.gpg`
6. **Upload environment files** back to secure cloud storage

## Security Notes

- Never store unencrypted environment files
- Use strong passphrases from password manager
- Regularly rotate encryption keys
- Limit access to encrypted files

---

**Created**: October 2025
