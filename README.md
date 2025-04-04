# Generate Local SSL Certificate Authority (CA) and Domain Certificate

## Overview

This script automates the process of creating a local Certificate Authority (CA) and generating an SSL certificate signed by that CA. This can be useful for development environments where you need to use HTTPS with custom domain names.

## Prerequisites

1. **OpenSSL Installed**: Ensure that OpenSSL is installed on your system. You can check this by running `openssl version`. If it's not installed, you can install it using your package manager:
   - On Ubuntu/Debian: `sudo apt-get install openssl`
   - On macOS with Homebrew: `brew install openssl`

2. **Bash Environment**: The script is written in Bash and requires a Bash environment.

3. **Root Privileges (Optional)**: Depending on your system's configuration, you might need root or sudo privileges to create directories and files in certain locations.

## Setup

1. **Set Variables**:
   - `CANAME`: The name of your Certificate Authority. This will be used for the directory name and CA certificate filename.
   - `MYCERT`: The domain name for which you want to generate the SSL certificate.

2. **Create Directory for Certificates**:
   - A new directory will be created to store all generated certificates, keys, and configuration files.

## Steps

1. **Generate AES-Encrypted Private Key for the CA**

   This step creates a private key for your CA that is encrypted with AES-256. The key will be 4096 bits long.

2. **Create a Self-Signed Root CA Certificate**

   A self-signed root CA certificate is created using the previously generated private key. It is valid for 5 years (1826 days).

3. **Generate a Private Key and CSR for Your Domain**

   This step generates a new private key and a Certificate Signing Request (CSR) for your specified domain name.

4. **Create a v3.ext File**

   A configuration file (`v3.ext`) is created to include Subject Alternative Name (SAN) properties, which are necessary for modern SSL certificates.

5. **Sign the CSR with the Root CA and Create the Domain Certificate**

   The root CA signs the previously generated CSR, creating the final domain certificate that is valid for 2 years (730 days).

## Instructions

1. **Set your CA and certificate variables**:
   ```bash
   CANAME="Local"
   MYCERT="local.dev.com"
   ```

2. **Optional, create a directory for your certificates**:
   ```bash
   mkdir $CANAME
   cd $CANAME
   ```

3. **Generate AES-encrypted private key for the CA**:
   ```bash
   openssl genrsa -aes256 -out $CANAME.key 4096
   ```
   You will be prompted to enter a passphrase for encrypting the private key.

4. **Create a self-signed root CA certificate, valid for 5 years (1826 days)**:
   ```bash
   openssl req -x509 -new -nodes -key $CANAME.key -sha256 -days 1826 -out $CANAME.crt \
     -subj "/CN=Local Root CA/C=AT/ST=Vienna/L=Vienna/O=LocalOrganisation"
   ```

5. **Generate a private key and CSR for your domain**:
   ```bash
   openssl req -new -nodes -out $MYCERT.csr -newkey rsa:4096 -keyout $MYCERT.key \
     -subj "/CN=$MYCERT/C=AT/ST=Vienna/L=Vienna/O=LocalOrganisation"
   ```

6. **Create a v3.ext file to include SAN properties**:
   ```bash
   cat > $MYCERT.v3.ext << EOF
   authorityKeyIdentifier=keyid,issuer
   basicConstraints=CA:FALSE
   keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
   subjectAltName = @alt_names
   [alt_names]
   DNS.1 = $MYCERT
   EOF
   ```

7. **Use the root CA to sign the CSR and create the certificate, valid for 2 years (730 days)**:
   ```bash
   openssl x509 -req -in $MYCERT.csr -CA $CANAME.crt -CAkey $CANAME.key -CAcreateserial \
     -out $MYCERT.crt -days 730 -sha256 -extfile $MYCERT.v3.ext
   ```

8. **Add the CA Certificate to Trusted Root Certificates**:

   For Ubuntu:
   ```bash
   sudo apt install -y ca-certificates
   sudo cp $CANAME.crt /usr/local/share/ca-certificates
   sudo update-ca-certificates
   ```

After running the script, you will have the following files in your directory:

- `$CANAME.key`: The private key for your CA.
- `$CANAME.crt`: The self-signed root CA certificate.
- `$MYCERT.key`: The private key for your domain.
- `$MYCERT.csr`: The Certificate Signing Request for your domain.
- `$MYCERT.v3.ext`: The configuration file used to specify SAN properties.
- `$MYCERT.srl`: A serial number file generated by OpenSSL.
- `$MYCERT.crt`: The final SSL certificate signed by the root CA.

You can now use these certificates in your development environment. For example, you can configure a web server to use `$MYCERT.key` and `$MYCERT.crt`.

## Notes

- **Passphrase**: Remember the passphrase you set for the CA private key (`$CANAME.key`). You will need it if you want to access or modify this key.
- **SAN Properties**: The `v3.ext` file includes a basic SAN configuration. You can expand this file to include additional domains or IP addresses if needed.

## Conclusion

This script provides an easy way to generate a local SSL CA and a domain certificate, which is particularly useful for development environments. Feel free to modify the variables and configurations as per your requirements.
