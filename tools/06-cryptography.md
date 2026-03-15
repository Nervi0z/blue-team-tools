# 06 — Cryptography Tools

Cryptography supports confidentiality, integrity, and authentication. Blue Teams use these tools for certificate management, file encryption, integrity verification, TLS analysis, and key management.

## Tools in this section

- [ccrypt](#ccrypt)
- [GnuPG (GPG)](#gnupg-gpg)
- [Hashing Utilities](#hashing-utilities)
- [OpenSSL](#openssl)

---

## ccrypt

- **Description:** Command-line utility for symmetric file encryption and decryption using AES (Rijndael). Password-based or keyfile-based. Simple and lightweight for protecting files at rest.
- **Blue Team use:**
  - Encrypt sensitive files before storing or transferring them
  - Decrypt files shared via secure channels
  - Quick protection of specific artifacts during IR without complex key infrastructure
- **Website:** [ccrypt.sourceforge.net](http://ccrypt.sourceforge.net/)
- **Type:** CLI file encryption utility
- **Platform:** Linux, macOS, Windows (via WSL or Cygwin)
- **Installation:**
  ```bash
  sudo apt install ccrypt    # Debian/Ubuntu
  brew install ccrypt        # macOS
  ```
- **Usage:**
  ```bash
  # Encrypt a file (prompts for password)
  ccrypt sensitive_data.txt
  # Creates sensitive_data.txt.cpt

  # Decrypt a file
  ccrypt -d sensitive_data.txt.cpt

  # Encrypt using a keyfile instead of a password
  ccrypt -k keyfile sensitive_data.txt

  # Encrypt to a different output filename
  ccencrypt -o output.cpt sensitive_data.txt
  ```
- **Alternatives:** GPG symmetric mode (`gpg --symmetric`), OpenSSL (`openssl enc`), VeraCrypt/LUKS (disk-level encryption)
- **Notes:** AES-256 by default. There is no password recovery — losing the password or keyfile means losing access to the data permanently. For anything more than quick file protection, GPG provides more flexibility.

---

## GnuPG (GPG)

- **Description:** GNU implementation of the OpenPGP standard. Supports asymmetric and symmetric encryption, digital signatures, and key management. Standard tool for file signing, email encryption, and verifying software releases.
- **Blue Team use:**
  - Encrypt files for a specific recipient using their public key
  - Sign files or artifacts to verify authenticity and integrity
  - Verify signatures on downloaded software or threat intel reports
  - Manage public/private key pairs for secure communication
  - Symmetric encryption for password-protected archives
- **Website:** [gnupg.org](https://gnupg.org/)
- **Type:** CLI encryption and signing tool (GUI frontends: Kleopatra on Windows/Linux, GPG Suite on macOS)
- **Platform:** Linux, macOS, Windows, BSD
- **Installation:**
  ```bash
  sudo apt install gnupg    # Debian/Ubuntu
  brew install gnupg        # macOS
  ```
- **Usage:**
  ```bash
  # Generate a new key pair
  gpg --full-generate-key

  # List public keys
  gpg --list-keys

  # List private keys
  gpg --list-secret-keys

  # Import a public key from a file
  gpg --import recipient_key.asc

  # Encrypt a file for a specific recipient
  gpg --encrypt --recipient analyst@example.com report.pdf
  # Creates report.pdf.gpg

  # Decrypt a file
  gpg --decrypt report.pdf.gpg > report.pdf

  # Sign a file (creates a detached signature)
  gpg --detach-sign --armor malware_sample.zip
  # Creates malware_sample.zip.asc

  # Verify a signature
  gpg --verify malware_sample.zip.asc malware_sample.zip

  # Symmetric encryption (password only, no key pair needed)
  gpg --symmetric --cipher-algo AES256 sensitive.txt
  ```
- **Alternatives:** OpenSSL (overlapping features, less PGP-focused), ccrypt (symmetric only), commercial PGP software
- **Notes:** Protect your private key with a strong passphrase and keep it backed up securely. Verify imported public keys through a trusted channel before using them — key spoofing is a real risk. GPG is the standard tool for verifying signed releases from security projects (Kali, Debian, Tails, etc.).

---

## Hashing Utilities

- **Description:** Built-in OS tools for computing cryptographic hashes (MD5, SHA-1, SHA-256, SHA-512) of files. A hash is a fixed-length fingerprint of data — any change to the file produces a completely different hash.
- **Blue Team use:**
  - Verify integrity of downloaded files against published checksums
  - Hash evidence files to establish a forensic chain of custody
  - Generate file hashes as IOCs for threat intelligence lookups
  - Detect file modifications by comparing hashes before and after
- **Type:** CLI hashing utilities (built-in to OS)
- **Platform:** Linux, macOS, Windows
- **Installation:** Pre-installed on all major platforms.
- **Usage:**

  **Linux / macOS:**
  ```bash
  md5sum file.zip
  sha1sum file.zip
  sha256sum file.zip
  sha512sum file.zip

  # Verify against a published checksum file
  sha256sum -c checksums.sha256

  # Hash all files in a directory recursively
  find /path/to/dir -type f -exec sha256sum {} \; > hashes.txt
  ```

  **Windows PowerShell:**
  ```powershell
  Get-FileHash file.zip -Algorithm MD5
  Get-FileHash file.zip -Algorithm SHA1
  Get-FileHash file.zip -Algorithm SHA256

  # Hash all files in a directory
  Get-ChildItem -Recurse -File | Get-FileHash -Algorithm SHA256
  ```

  **Windows CMD:**
  ```cmd
  certutil -hashfile file.zip MD5
  certutil -hashfile file.zip SHA256
  ```

- **Alternatives:** `openssl dgst -sha256 file.zip`, HashMyFiles (Windows GUI), third-party tools
- **Notes:** MD5 and SHA-1 are cryptographically broken for collision resistance — do not use them for signing or certificates. They remain acceptable for basic integrity checks and as IOC identifiers in threat intel (where you're matching a known value, not creating a signature). Use SHA-256 or SHA-512 for anything new.

---

## OpenSSL

- **Description:** Full-featured toolkit and library for TLS/SSL and general-purpose cryptography. Used by Blue Teams as a Swiss Army knife for certificate inspection, TLS testing, key generation, file encryption, and hashing.
- **Blue Team use:**
  - Inspect TLS certificates on remote services and verify configuration
  - Test cipher suite support and TLS version negotiation
  - Generate private keys and CSRs for certificate requests
  - Inspect certificate files and decode certificate fields
  - Calculate file hashes
  - Encrypt/decrypt files with symmetric ciphers
  - Base64 encode/decode data
- **Website:** [openssl.org](https://www.openssl.org/)
- **Type:** CLI cryptographic toolkit and library
- **Platform:** Linux, macOS, Windows, BSD (pre-installed on most Unix systems)
- **Installation:**
  ```bash
  sudo apt install openssl    # Debian/Ubuntu (usually pre-installed)
  brew install openssl        # macOS
  ```
- **Usage:**
  ```bash
  # Check a remote server's TLS certificate and chain
  openssl s_client -connect example.com:443 -showcerts </dev/null 2>/dev/null

  # Check TLS certificate expiry date
  echo | openssl s_client -connect example.com:443 2>/dev/null \
    | openssl x509 -noout -dates

  # View all details of a certificate file
  openssl x509 -in certificate.crt -text -noout

  # Check certificate subject and issuer only
  openssl x509 -in certificate.crt -noout -subject -issuer

  # Verify a certificate chain against a CA bundle
  openssl verify -CAfile ca-bundle.crt certificate.crt

  # Generate a 4096-bit RSA private key
  openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:4096

  # Generate an EC key (P-256)
  openssl genpkey -algorithm EC -pkeyopt ec_paramgen_curve:P-256 -out ec_key.pem

  # Create a CSR from an existing private key
  openssl req -new -key private_key.pem -out request.csr

  # Calculate SHA-256 hash of a file
  openssl dgst -sha256 file.zip

  # Encrypt a file (AES-256-CBC, prompts for password)
  openssl enc -aes-256-cbc -pbkdf2 -salt -in plaintext.txt -out encrypted.enc

  # Decrypt
  openssl enc -d -aes-256-cbc -pbkdf2 -in encrypted.enc -out decrypted.txt

  # Base64 encode a file
  openssl base64 -in binary_file -out encoded.b64

  # Base64 decode
  openssl base64 -d -in encoded.b64 -out binary_file

  # Test which TLS cipher suites a server supports
  nmap --script ssl-enum-ciphers -p 443 example.com  # or use testssl.sh for a dedicated report
  ```
- **Alternatives:** GnuTLS CLI (`gnutls-cli`), LibreSSL, [testssl.sh](https://testssl.sh/) (TLS testing specifically), platform GUI tools (Keychain Access on macOS, certmgr on Windows)
- **Notes:** Always use `-pbkdf2` with `openssl enc` — without it, key derivation uses a weak legacy method. For comprehensive TLS configuration testing, [testssl.sh](https://testssl.sh/) is more practical than raw `s_client` commands. The `s_client` command is still essential for quick certificate inspection and debugging TLS handshake issues.

---
