# PKI-Manager

A comprehensive set of bash scripts for managing a complete Public Key Infrastructure (PKI). Create Certificate Authorities (CA), generate server and client certificates, and manage Certificate Revocation Lists (CRL) with ease.

## Features

- ✅ **Certificate Authority (CA) Generation** - Create self-signed root CAs with RSA or ECC algorithms
- ✅ **Server & Client Certificates** - Generate X.509 certificates for servers and PKCS#12 containers for clients
- ✅ **CRL Management** - Revoke certificates and maintain Certificate Revocation Lists
- ✅ **Flexible Configuration** - Customize certificate attributes via environment variables or custom OpenSSL config files
- ✅ **Subject Alternative Names (SAN)** - Support for multiple DNS names and IP addresses
- ✅ **Algorithm Support** - RSA (4096-bit) and Elliptic Curve (secp384r1) cryptography

## Prerequisites

- **OpenSSL** (version 1.1.1 or later recommended)
- **Bash** (version 4.0 or later)
- **Unix-like environment** (Linux, macOS, WSL on Windows)

### Verify Installation

```bash
openssl version
bash --version
```

## Project Structure

```
pki-manager/
├── bin/              # Executable scripts
│   ├── pki-ca        # Certificate Authority management
│   ├── pki-cert      # Certificate generation
│   └── pki-crl       # Certificate Revocation List management
├── lib/              # Common library functions
│   └── common.sh     # Shared utilities
├── templates/        # Configuration templates
│   └── ca.cnf.template
├── examples/         # Example configurations
│   └── example_cert.cnf
├── pki               # Main entry point wrapper
└── README.md         # This file
```

## Quick Start

### Using the wrapper script (recommended)

```bash
# Create a Certificate Authority
./pki ca create -p demo -a ec

# Generate a server certificate
./pki cert -p demo -t server -a ec -n "*.example.com"

# Revoke a certificate and update CRL
./pki crl -p demo -r 01 -u
```

### Using scripts directly

```bash
# Create a Certificate Authority
./bin/pki-ca create -p demo -a ec

# Generate certificates
./bin/pki-cert -p demo -t server -a ec -n "*.example.com"
./bin/pki-cert -p demo -t client -a ec -n "John Doe"

# Manage CRL
./bin/pki-crl -p demo -r 01 -u
```

### 1. Create a Certificate Authority

```bash
./pki ca create -p demo -a ec
```

This creates a new project directory `demo/` containing:
- CA private key (`ca.key`) - password protected
- CA certificate (`ca.crt`) - self-signed root certificate
- CRL infrastructure - directories and configuration files

### 2. Generate Certificates

**Server certificate:**
```bash
./pki cert -p demo -t server -a ec -n "*.example.com"
```

**Client certificate:**
```bash
./pki cert -p demo -t client -a ec -n "John Doe"
```

### 3. Manage Certificate Revocation

```bash
# Revoke a certificate and update CRL
./pki crl -p demo -r 01 -u
```

## Scripts Overview

| Script | Purpose | Key Features |
|--------|---------|--------------|
| `pki ca` | Create Certificate Authority | Generates CA key/cert, initializes CRL infrastructure |
| `pki cert` | Generate certificates | Server (X.509) or Client (PKCS#12) certificates |
| `pki crl` | Manage CRL | Revoke certificates, update revocation lists |

## Detailed Documentation

### 1. Certificate Authority Generation

**Command:** `pki ca create` or `bin/pki-ca create`

#### Syntax

```bash
./pki ca create -p <project_name> -a <algorithm>
# or
./bin/pki-ca create -p <project_name> -a <algorithm>
```

#### Options

| Option | Description | Required |
|--------|-------------|----------|
| `-p <project_name>` | Project name (creates a directory with this name) | ✅ Yes |
| `-a <algorithm>` | Encryption algorithm: `rsa` or `ec` | ✅ Yes |

#### Algorithm Comparison

| Algorithm | Key Size | Performance | Use Case |
|-----------|----------|-------------|----------|
| **RSA** | 4096 bits | Slower | Maximum compatibility |
| **EC** (secp384r1) | 384 bits | Faster | Modern systems, better performance |

#### Environment Variables

Customize the CA certificate subject with these environment variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `CA_C` | Country code (2 letters) | `FR` |
| `CA_L` | Locality/City | `Paris` |
| `CA_O` | Organization | `France` |
| `CA_OU` | Organizational Unit | `DevOps` |
| `CA_CN` | Common Name | Project name |
| `CA_EXPIRE_DAYS` | CA validity period (days) | `365` |
| `CRL_EXPIRE_DAYS` | CRL validity period (days) | `30` |

#### Examples

**Basic CA creation:**
```bash
./pki ca create -p myproject -a rsa
```

**Customized CA:**
```bash
export CA_O="My Company" CA_OU="IT Security" CA_EXPIRE_DAYS=730
./pki ca create -p production -a ec
```

#### Generated Files Structure

```
project_name/
├── ca.key              # CA private key (password protected, chmod 400)
├── ca.crt              # CA certificate (self-signed, chmod 444)
├── ca.pass             # CA key password (randomly generated)
├── ca.cnf              # OpenSSL CA configuration file
├── ca.srl              # Serial number file (auto-incremented)
├── index.txt           # Certificate database (issued/revoked)
├── index.txt.attr      # Index attributes
├── crlnumber           # CRL serial number
├── crl/
│   └── ca.crl          # Certificate Revocation List
└── newcerts/           # Copies of issued certificates (serial.pem)
```

#### Viewing CA Information

```bash
# Show CA certificate details
openssl x509 -nameopt multiline,-esc_msb,utf8 -in demo/ca.crt -text -noout | \
    egrep -i -v '^\s+([0-9a-z]{2}:){15,}'

# Show CRL details
openssl crl -in demo/crl/ca.crl -text -noout
```

---

### 2. Certificate Generation

**Command:** `pki cert` or `bin/pki-cert`

#### Syntax

```bash
./pki cert -p <project_name> -t <type> -a <algorithm> [-n <name>] [-c <cnf_file>]
```

#### Options

| Option | Description | Required |
|--------|-------------|----------|
| `-p <project_name>` | Project name (must match existing CA project) | ✅ Yes |
| `-t <type>` | Certificate type: `server` or `client` | ✅ Yes |
| `-a <algorithm>` | Algorithm: `rsa` or `ec` | ✅ Yes |
| `-n <name>` | Common Name (CN) | ❌ Optional |
| `-c <cnf_file>` | Custom OpenSSL configuration file | ❌ Optional |

#### Certificate Types

**Server Certificates:**
- Format: X.509 (`.crt`)
- Private key: Passwordless by default (for automated services)
- Use case: Web servers, API endpoints, TLS/SSL services

**Client Certificates:**
- Format: PKCS#12 (`.p12` / `.pfx`)
- Private key: Password protected by default
- Use case: Client authentication, email signing, VPN access

#### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `CRT_C` | Country code | `FR` |
| `CRT_L` | Locality | `Paris` |
| `CRT_O` | Organization | `France` |
| `CRT_OU` | Organizational Unit | `DevOps` |
| `CRT_CN` | Common Name | Overridden by `-n` if provided |
| `CRT_EXPIRE_DAYS` | Certificate validity (days) | `365` |
| `CRT_SAN` | Subject Alternative Names | None |
| `NO_PASSWD` | Disable password for private key | `true` (server), `false` (client) |

#### Subject Alternative Names (SAN)

SAN allows a certificate to be valid for multiple domain names and IP addresses:

```bash
export CRT_SAN="DNS:example.com,DNS:www.example.com,DNS:*.example.com,IP:192.168.1.1"
./pki cert -p demo -t server -a ec -n "example.com"
```

#### Examples

**Basic server certificate:**
```bash
./pki cert -p demo -t server -a ec -n "api.example.com"
```

**Server certificate with SAN:**
```bash
export CRT_SAN="DNS:api.example.com,DNS:www.example.com,IP:192.168.1.1"
./pki cert -p demo -t server -a ec -n "api.example.com"
```

**Client certificate:**
```bash
export CRT_O="My Company" CRT_OU="Development"
./pki cert -p demo -t client -a ec -n "John Doe"
```

**Using custom configuration file:**
```bash
./pki cert -p demo -t server -a ec -n "example.com" -c example_cert.cnf
```

#### Generated Files

**For server certificates:**
- `<type>Certificate_<timestamp>_<name>.key` - Private key
- `<type>Certificate_<timestamp>_<name>.csr` - Certificate Signing Request
- `<type>Certificate_<timestamp>_<name>.crt` - Signed certificate

**For client certificates:**
- All files above, plus:
- `<type>Certificate_<timestamp>_<name>.p12` - PKCS#12 container (for browser/OS import)
- `<type>Certificate_<timestamp>_<name>.p12.pass` - P12 file password

#### Viewing Certificate Information

```bash
# Show certificate details
openssl x509 -in certificate.crt -text -noout

# Show certificate subject
openssl x509 -in certificate.crt -noout -subject

# Show certificate serial number
openssl x509 -in certificate.crt -noout -serial

# Verify certificate against CA
openssl verify -CAfile demo/ca.crt certificate.crt

# For PKCS#12 files
openssl pkcs12 -info -in certificate.p12 -passin file:certificate.p12.pass
```

---

### 3. CRL Management

**Command:** `pki crl` or `bin/pki-crl`

#### Syntax

```bash
./pki crl -p <project_name> [-r <serial_number>] [-u]
```

#### Options

| Option | Description | Required |
|--------|-------------|----------|
| `-p <project_name>` | Project name (must match existing CA project) | ✅ Yes |
| `-r <serial_number>` | Serial number of certificate to revoke (hex format) | ❌ Optional |
| `-u` | Update the CRL (use with `-r` or alone) | ❌ Optional |

#### Examples

**Revoke a certificate and update CRL:**
```bash
./pki crl -p demo -r 01 -u
```

**Update CRL without revoking:**
```bash
./pki crl -p demo -u
```

**Revoke only (CRL will be updated automatically):**
```bash
./pki crl -p demo -r 02
```

#### Finding Certificate Serial Numbers

**From certificate file:**
```bash
openssl x509 -in certificate.crt -noout -serial
```

**From CA database:**
```bash
cat demo/index.txt
```

**List all certificates:**
```bash
# Show all issued certificates
cat demo/index.txt | grep "^V"

# Show revoked certificates
cat demo/index.txt | grep "^R"
```

#### Verifying CRL

```bash
# Display CRL content
openssl crl -in demo/crl/ca.crl -text -noout

# Count revoked certificates
openssl crl -in demo/crl/ca.crl -text -noout | grep -c "Serial Number:"

# Check if specific serial is revoked
openssl crl -in demo/crl/ca.crl -text -noout | grep -A 2 "Serial Number: 01"
```

---

## Custom Configuration Files

### Using Custom `.cnf` Files

You can use custom OpenSSL configuration files to define advanced certificate extensions, Subject Alternative Names (SAN), and other options.

#### Example Configuration File

See `example_cert.cnf` for a complete example. The script automatically detects these sections:

- `[v3_server]` - For server certificates
- `[v3_client]` - For client certificates  
- `[v3_req]` - Generic section (fallback)

#### Usage

```bash
# Copy and customize the example
cp example_cert.cnf my_cert.cnf
# Edit my_cert.cnf according to your needs

# Use with certificate generation
./pki cert -p demo -t server -a ec -n "example.com" -c my_cert.cnf
```

#### Configuration File Structure

```ini
[ req ]
default_bits = 4096
distinguished_name = req_distinguished_name
req_extensions = v3_req

[ req_distinguished_name ]
countryName = Country Name (2 letter code)
countryName_default = FR
# ... other DN fields

[ v3_server ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = example.com
DNS.2 = www.example.com
DNS.3 = *.example.com
IP.1 = 192.168.1.1
```

---

## Complete Workflow Example

This example demonstrates creating a complete PKI for a project called "webapp":

### Step 1: Create the Certificate Authority

```bash
export CA_O="Acme Corp" CA_OU="IT Security" CA_EXPIRE_DAYS=1825
./01_generate_CA.sh -p webapp -a ec
```

### Step 2: Generate Server Certificates

```bash
cd webapp

# Main web server
export CRT_SAN="DNS:webapp.com,DNS:www.webapp.com"
./pki cert -p webapp -t server -a ec -n "webapp.com"

# API server
export CRT_SAN="DNS:api.webapp.com,DNS:*.api.webapp.com"
./pki cert -p webapp -t server -a ec -n "api.webapp.com"
```

### Step 3: Generate Client Certificates

```bash
# Developer certificates
export CRT_O="Acme Corp" CRT_OU="Development"
./pki cert -p webapp -t client -a ec -n "Alice Developer"
./pki cert -p webapp -t client -a ec -n "Bob Developer"

# Admin certificates
export CRT_OU="Administration"
./pki cert -p webapp -t client -a ec -n "Admin User"
```

### Step 4: Verify Certificates

```bash
# Verify server certificates
for cert in webapp/server*.crt; do
    echo "=== $cert ==="
    openssl x509 -in "$cert" -noout -subject -dates
    openssl verify -CAfile webapp/ca.crt "$cert"
done

# List all issued certificates
cat webapp/index.txt
```

### Step 5: Revoke a Certificate (if needed)

```bash
# Find serial number
openssl x509 -in webapp/clientCertificate_*.crt -noout -serial

# Revoke and update CRL
./pki crl -p webapp -r 02 -u

# Verify revocation
openssl crl -in webapp/crl/ca.crl -text -noout
```

---

## File Structure Reference

```
project_name/
├── ca.key              # CA private key (password protected, chmod 400)
├── ca.crt              # CA certificate (chmod 444)
├── ca.pass             # CA key password (randomly generated)
├── ca.cnf              # OpenSSL CA configuration
├── ca.srl              # Serial number counter
├── index.txt           # Certificate database
├── index.txt.attr      # Index attributes
├── crlnumber           # CRL serial number
├── crl/
│   └── ca.crl          # Certificate Revocation List
├── newcerts/           # Copies of issued certificates
│   ├── 01.pem
│   ├── 02.pem
│   └── ...
└── <certificate_files> # Generated certificates
    ├── serverCertificate_20240101_webapp.com.key
    ├── serverCertificate_20240101_webapp.com.csr
    ├── serverCertificate_20240101_webapp.com.crt
    ├── clientCertificate_20240101_JohnDoe.key
    ├── clientCertificate_20240101_JohnDoe.csr
    ├── clientCertificate_20240101_JohnDoe.crt
    ├── clientCertificate_20240101_JohnDoe.p12
    └── clientCertificate_20240101_JohnDoe.p12.pass
```

---

## Security Best Practices

⚠️ **IMPORTANT SECURITY CONSIDERATIONS:**

1. **Private Keys & Passwords**
   - Never commit private keys (`.key`) or password files (`.pass`) to version control
   - Use restrictive permissions: `chmod 600` for keys, `chmod 400` for CA key
   - Store passwords securely (consider using a password manager)

2. **CA Protection**
   - Backup CA files (`ca.key`, `ca.crt`, `ca.pass`) in a secure, encrypted location
   - The CA private key is the root of trust - if compromised, all certificates are compromised
   - Consider using hardware security modules (HSM) for production CAs

3. **Certificate Validity**
   - Set appropriate expiration dates (shorter for certificates, longer for CA)
   - Regularly update CRLs before they expire
   - Monitor certificate expiration dates

4. **Repository Security**
   - Use `.gitignore` to exclude sensitive files
   - Consider using a private, encrypted Git repository
   - Never share CA private keys or passwords

5. **Access Control**
   - Limit access to CA directory to authorized personnel only
   - Use separate CAs for different environments (dev, staging, production)

---

## Troubleshooting

### Common Issues

**Error: "variable has no value" in ca.cnf**
- This was a bug that has been fixed. Ensure you're using the latest version of the script.

**Error: "project doesn't exist yet"**
- Make sure you've created the CA first using `pki ca create`
- Verify the project name matches exactly (case-sensitive)

**Error: "ca.cnf not found"**
- Regenerate the CA using the updated `pki ca create` command
- The CA must be created with a recent version that includes CRL support

**Certificate verification fails**
- Ensure you're using the correct CA certificate: `openssl verify -CAfile project/ca.crt certificate.crt`
- Check certificate expiration: `openssl x509 -in certificate.crt -noout -dates`
- Verify the certificate chain is complete

**CRL not updating**
- Ensure you have write permissions in the project directory
- Check that `ca.key` and `ca.pass` are accessible
- Verify the `crl/` directory exists

**PKCS#12 import fails**
- Verify the password file exists and contains the correct password
- Try: `openssl pkcs12 -info -in file.p12 -passin file:file.p12.pass`
- Some systems require the password to be entered interactively

### Getting Help

Each script provides help information:

```bash
./pki ca --help
./pki cert -h
./pki crl -h
```

### Debugging

Enable verbose output by checking OpenSSL commands in the scripts. You can also test OpenSSL configuration:

```bash
# Test CA configuration
openssl ca -config demo/ca.cnf -help

# Verify certificate
openssl x509 -in certificate.crt -text -noout

# Check CRL
openssl crl -in demo/crl/ca.crl -text -noout
```

---

## License

See the `LICENSE.md` file for license information.

---

## Contributing

Contributions are welcome! Please ensure:
- Scripts follow bash best practices
- Error handling is comprehensive
- Documentation is updated
- Security considerations are maintained
