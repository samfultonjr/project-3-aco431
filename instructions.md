# COMPLETE STEP-BY-STEP GUIDE: Project 3 TLS Stage 1

## EXACT EXECUTION CHECKLIST

Due: November 17, 2025 before class starts

---

## PART A: SETUP AND PREPARATION

### A1. Network Setup
- [ ] **Server (Kali VM - Guest)**: Identify IP address
  - Run: `ip addr` or `ifconfig`
  - Note the IP: ________________

- [ ] **Client (Host Computer)**: Identify IP and OS
  - Note your host OS: ________________
  - Note browser to use: ________________

- [ ] **VMware Mode**: Confirm networking mode
  - Note mode (NAT/Bridged/Host-only): ________________

- [ ] **Verify connectivity**: Ping from host to guest
  - From host, ping the Kali VM IP

### A2. Choose Your Group Name
- [ ] Decide on group name: ________________
- [ ] Server will be: `www.yourgroupname.com`
- [ ] CA will be: `YourGroupName Root CA` (different from server)

---

## PART B: STEP 1 - SET UP CERTIFICATE AUTHORITY (ON KALI VM)

### B1. Prepare CA Directory Structure
```bash
# Run these commands on Kali VM
mkdir -p ~/ca/{private,certs,newcerts,csr}
cd ~/ca
touch index.txt
echo 1000 > serial
```

### B2. Create OpenSSL Configuration File
- [ ] Create file: `nano ~/ca/openssl.cnf`
- [ ] Paste this configuration (adjust paths as needed):

```ini
[ ca ]
default_ca = CA_default

[ CA_default ]
dir              = /home/kali/ca
certs            = $dir/certs
crl_dir          = $dir/crl
new_certs_dir    = $dir/newcerts
database         = $dir/index.txt
serial           = $dir/serial
RANDFILE         = $dir/private/.rand
private_key      = $dir/private/ca-key.pem
certificate      = $dir/certs/ca-cert.crt
crlnumber        = $dir/crlnumber
crl              = $dir/crl.pem
crl_extensions   = crl_ext
default_crl_days = 30
default_md       = sha256
name_opt         = ca_default
cert_opt         = ca_default
default_days     = 365
preserve         = no
policy           = policy_loose

[ policy_loose ]
countryName            = optional
stateOrProvinceName    = optional
localityName           = optional
organizationName       = optional
organizationalUnitName = optional
commonName             = supplied
emailAddress           = optional

[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
string_mask        = utf8only
default_md         = sha256
x509_extensions    = v3_ca

[ req_distinguished_name ]
countryName                    = Country Name (2 letter code)
stateOrProvinceName            = State or Province Name
localityName                   = Locality Name
organizationName               = Organization Name
organizationalUnitName         = Organizational Unit Name
commonName                     = Common Name
emailAddress                   = Email Address

[ v3_ca ]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ v3_intermediate_ca ]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ server_cert ]
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = www.yourgroupname.com
```

- [ ] **IMPORTANT**: Replace `www.yourgroupname.com` in `[alt_names]` section with YOUR actual server name
- [ ] **IMPORTANT**: Replace `/home/kali/ca` with your actual path if different (run `pwd` in ~/ca to confirm)
- [ ] Save file (Ctrl+O, Enter, Ctrl+X)

### B3. Generate Root CA Private Key
```bash
cd ~/ca
openssl genrsa -aes256 -out private/ca-key.pem 4096
```
- [ ] **Record command in deliverables**
- [ ] Enter and confirm a strong passphrase
- [ ] **Store passphrase securely**: ________________

**Purpose**: Generates a 4096-bit RSA private key with AES-256 encryption for the root CA

### B4. Generate Root CA Certificate
```bash
openssl req -config openssl.cnf -key private/ca-key.pem -new -x509 -days 3650 -sha256 -extensions v3_ca -out certs/ca-cert.crt
```

- [ ] **Record command in deliverables**
- [ ] Enter the passphrase from B3
- [ ] Fill in details:
  - Country Name: **US** (or your country)
  - State: ________________
  - Locality: ________________
  - Organization: **[YourGroupName] Root CA**
  - Organizational Unit: **Certificate Authority**
  - Common Name: **[YourGroupName] Root CA** (CRITICAL - must be different from server)
  - Email: ________________

**Purpose**: Creates a self-signed root CA certificate valid for 10 years using SHA-256

### B5. View and Analyze Root CA Certificate
```bash
openssl x509 -in certs/ca-cert.crt -text -noout
```

- [ ] **Record command in deliverables**
- [ ] **Save output to file**: `openssl x509 -in certs/ca-cert.crt -text -noout > ca-cert-readable.txt`
- [ ] **Screenshot the output**

**Purpose**: Displays the CA certificate in human-readable format

---

## PART C: GENERATE SERVER CERTIFICATE (ON KALI VM)

### C1. Generate Server Private Key
```bash
cd ~/ca
openssl genrsa -out private/server-key.pem 2048
```

- [ ] **Record command in deliverables**

**Purpose**: Generates a 2048-bit RSA private key for the web server (unencrypted for Apache)

### C2. Create Server Certificate Signing Request (CSR)
```bash
openssl req -config openssl.cnf -key private/server-key.pem -new -sha256 -out csr/server-csr.pem
```

- [ ] **Record command in deliverables**
- [ ] Fill in details:
  - Country Name: **US** (or match CA)
  - State: ________________
  - Locality: ________________
  - Organization: **[YourGroupName]**
  - Organizational Unit: **Web Services**
  - Common Name: **www.yourgroupname.com** (CRITICAL - exact server domain)
  - Email: ________________

**Purpose**: Creates a certificate signing request for the server with SHA-256

### C3. Sign Server Certificate with Root CA
```bash
openssl ca -config openssl.cnf -extensions server_cert -days 365 -notext -md sha256 -in csr/server-csr.pem -out certs/server-cert.crt
```

- [ ] **Record command in deliverables**
- [ ] Enter CA passphrase from B3
- [ ] Type **y** to sign the certificate
- [ ] Type **y** to commit

**Purpose**: Issues a server certificate signed by the root CA, valid for 1 year, with server extensions including SAN

### C4. View and Analyze Server Certificate
```bash
openssl x509 -in certs/server-cert.crt -text -noout
```

- [ ] **Record command in deliverables**
- [ ] **Save output**: `openssl x509 -in certs/server-cert.crt -text -noout > server-cert-readable.txt`
- [ ] **Screenshot the output**

**Purpose**: Displays the server certificate in human-readable format

---

## PART D: ANSWER CERTIFICATE ANALYSIS QUESTIONS

### D1. Root CA Certificate Analysis
From `ca-cert-readable.txt`:

- [ ] **Question 4a**: How can you tell a certificate belongs to a root CA?
  - **Look for**: `Issuer:` equals `Subject:` (self-signed)
  - **Look for**: `CA:TRUE` in Basic Constraints
  - **Look for**: Key Usage includes `Certificate Sign, CRL Sign`
  - **Write answer using YOUR certificate data**

### D2. Server Certificate Analysis
From `server-cert-readable.txt`:

- [ ] **Question 4b**: How can you tell a certificate belongs to a web server?
  - **Look for**: `Issuer:` different from `Subject:`
  - **Look for**: `CA:FALSE` in Basic Constraints
  - **Look for**: Extended Key Usage includes `TLS Web Server Authentication`
  - **Look for**: Subject Alternative Name includes your domain
  - **Write answer using YOUR certificate data**

- [ ] **Question 4c-i**: Extract and record the public key
  - Run: `openssl x509 -in certs/server-cert.crt -pubkey -noout`
  - **Copy entire public key block**

- [ ] **Question 4c-ii**: Find signing algorithm
  - **Look for**: `Signature Algorithm:` (likely sha256WithRSAEncryption)
  - **Note**: Signed with CA's private key
  - **Write answer**

- [ ] **Question 4c-iii**: Does certificate include private key?
  - **Answer**: NO
  - **Reason**: Certificates only contain public keys. Private keys must remain secret and are stored separately (in server-key.pem)
  - **Write detailed answer**

---

## PART E: STEP 2 - CONFIGURE APACHE WEB SERVER (ON KALI VM)

### E1. Install Apache and SSL Module
```bash
sudo apt update
sudo apt install apache2 -y
```

- [ ] Confirm Apache installed
- [ ] **Screenshot installation**

### E2. Copy Certificates to Apache Directory
```bash
sudo mkdir -p /etc/apache2/ssl
sudo cp ~/ca/certs/server-cert.crt /etc/apache2/ssl/
sudo cp ~/ca/private/server-key.pem /etc/apache2/ssl/
sudo chmod 600 /etc/apache2/ssl/server-key.pem
```

- [ ] Verify files copied: `ls -l /etc/apache2/ssl/`

### E3. Configure SSL Virtual Host
```bash
sudo nano /etc/apache2/sites-available/default-ssl.conf
```

- [ ] Find and modify these lines (uncomment if commented):
```apache
SSLCertificateFile      /etc/apache2/ssl/server-cert.crt
SSLCertificateKeyFile   /etc/apache2/ssl/server-key.pem
```

- [ ] Also verify `ServerName` line exists (add if not):
```apache
ServerName www.yourgroupname.com:443
```

- [ ] Save file (Ctrl+O, Enter, Ctrl+X)
- [ ] **Screenshot the configuration file**

### E4. Enable SSL Module and Site
```bash
sudo a2enmod ssl
sudo a2ensite default-ssl
sudo apache2ctl configtest
```

- [ ] Verify output says **Syntax OK**
- [ ] **Screenshot the output**

### E5. Restart Apache
```bash
sudo systemctl restart apache2
```

- [ ] Check Apache status: `sudo systemctl status apache2`
- [ ] Should show **active (running)**
- [ ] **Screenshot the status**

### E6. Verify SSL Port is Listening
```bash
netstat -na | grep 443
```

- [ ] Should show `0.0.0.0:443` or `:::443` in LISTEN state
- [ ] **Screenshot the output**

### E7. Create Test Web Page (Optional but Recommended)
```bash
echo "<h1>Secure Web Server - [YourGroupName]</h1><p>TLS is working!</p>" | sudo tee /var/www/html/index.html
```

- [ ] Creates a simple test page

---

## PART F: CONFIGURE CLIENT COMPUTER (HOST)

### F1. Configure Hosts File

**On Windows Host:**
```bash
# Run as Administrator
notepad C:\Windows\System32\drivers\etc\hosts
```

**On Mac/Linux Host:**
```bash
sudo nano /etc/hosts
```

- [ ] Add this line at the end:
```
[KALI_VM_IP]    www.yourgroupname.com
```

- [ ] **Replace `[KALI_VM_IP]`** with actual Kali IP from step A1
- [ ] **Replace `yourgroupname`** with YOUR group name
- [ ] Save file
- [ ] **Screenshot the hosts file**

### F2. Test Domain Resolution
```bash
ping www.yourgroupname.com
```

- [ ] Should ping the Kali VM IP
- [ ] **Screenshot ping result**

### F3. Import Root CA Certificate to Browser

**For Firefox:**
1. [ ] Copy `ca-cert.crt` from Kali to host computer
   - Can use: `scp kali@[KALI_IP]:~/ca/certs/ca-cert.crt ~/Desktop/`
2. [ ] Open Firefox
3. [ ] Go to: Settings → Privacy & Security → Certificates → View Certificates
4. [ ] Click **Authorities** tab
5. [ ] Click **Import**
6. [ ] Select `ca-cert.crt`
7. [ ] Check **Trust this CA to identify websites**
8. [ ] Click **OK**
9. [ ] **Screenshot the imported certificate**

**For Chrome/Edge:**
1. [ ] Copy `ca-cert.crt` from Kali to host computer
2. [ ] Open Chrome/Edge
3. [ ] Go to: Settings → Privacy and security → Security → Manage certificates
4. [ ] Go to **Trusted Root Certification Authorities** tab
5. [ ] Click **Import**
6. [ ] Follow wizard, select `ca-cert.crt`
7. [ ] Confirm import
8. [ ] **Screenshot the imported certificate**

**For Safari (Mac):**
1. [ ] Copy `ca-cert.crt` from Kali to Mac
2. [ ] Double-click `ca-cert.crt`
3. [ ] Keychain Access opens
4. [ ] Double-click the certificate
5. [ ] Expand **Trust** section
6. [ ] Set **When using this certificate** to **Always Trust**
7. [ ] Close and enter password
8. [ ] **Screenshot the trusted certificate**

---

## PART G: TEST HTTPS CONNECTION

### G1. Access HTTPS Server
- [ ] Open browser on host computer
- [ ] Navigate to: `https://www.yourgroupname.com`
- [ ] **Should load without security warnings**
- [ ] **Screenshot the browser showing:**
  - Green padlock/secure icon
  - Full URL in address bar
  - Web page content

### G2. Verify Certificate in Browser
- [ ] Click the padlock icon
- [ ] View certificate details
- [ ] **Screenshot showing:**
  - Certificate is valid
  - Issued to: www.yourgroupname.com
  - Issued by: [YourGroupName] Root CA
  - Valid dates
  - Certificate path/chain

### G3. Additional Verification
```bash
# On Kali VM, check Apache logs
sudo tail -f /var/log/apache2/access.log
```

- [ ] Refresh browser page
- [ ] Should see GET request in logs
- [ ] **Screenshot the log entries**

---

## PART H: PREPARE DELIVERABLES

### H1. Document OpenSSL Commands
Create a document with ALL commands and their purposes:

- [ ] **Root CA Commands:**
  1. `openssl genrsa -aes256 -out private/ca-key.pem 4096`
     - Purpose: [your explanation]
  2. `openssl req -config openssl.cnf -key private/ca-key.pem -new -x509 -days 3650 -sha256 -extensions v3_ca -out certs/ca-cert.crt`
     - Purpose: [your explanation]
  3. `openssl x509 -in certs/ca-cert.crt -text -noout`
     - Purpose: [your explanation]

- [ ] **Server Certificate Commands:**
  1. `openssl genrsa -out private/server-key.pem 2048`
     - Purpose: [your explanation]
  2. `openssl req -config openssl.cnf -key private/server-key.pem -new -sha256 -out csr/server-csr.pem`
     - Purpose: [your explanation]
  3. `openssl ca -config openssl.cnf -extensions server_cert -days 365 -notext -md sha256 -in csr/server-csr.pem -out certs/server-cert.crt`
     - Purpose: [your explanation]
  4. `openssl x509 -in certs/server-cert.crt -text -noout`
     - Purpose: [your explanation]

### H2. Include Readable Certificates
- [ ] Attach `ca-cert-readable.txt`
- [ ] Attach `server-cert-readable.txt`

### H3. Create Network Topology Diagram
Draw diagram showing:
- [ ] Host computer (Client)
  - OS: ________________
  - Browser: ________________
  - IP: ________________
- [ ] Kali VM (Server + CA)
  - OS: Kali Linux
  - IP: ________________
  - Services: Apache HTTPS, OpenSSL CA
- [ ] VMware networking mode
- [ ] HTTPS connection (port 443) between them
- [ ] **Include this diagram**

### H4. Document Server Configuration
Write detailed description including:
- [ ] Apache configuration file location: `/etc/apache2/sites-available/default-ssl.conf`
- [ ] SSL certificate location: `/etc/apache2/ssl/server-cert.crt`
- [ ] SSL key location: `/etc/apache2/ssl/server-key.pem`
- [ ] Commands used: `a2enmod ssl`, `a2ensite default-ssl`, `apache2ctl configtest`, `systemctl restart apache2`
- [ ] Verification command: `netstat -na | grep 443`

### H5. Document Client Configuration
Write detailed description including:
- [ ] Hosts file location and modification
- [ ] Root CA certificate import process for your specific browser
- [ ] How domain name resolution works
- [ ] Why CA certificate must be trusted

### H6. Answer Analysis Questions
- [ ] Question 4a: How to identify root CA certificate (with examples from YOUR cert)
- [ ] Question 4b: How to identify web server certificate (with examples from YOUR cert)
- [ ] Question 4c-i: Server's public key
- [ ] Question 4c-ii: Signing algorithm and key used
- [ ] Question 4c-iii: Private key in certificate? Why/why not?

### H7. Compile All Screenshots
Required screenshots:
- [ ] CA certificate in readable format
- [ ] Server certificate in readable format
- [ ] Apache SSL configuration file
- [ ] `apache2ctl configtest` output showing Syntax OK
- [ ] Apache status showing active (running)
- [ ] `netstat` output showing port 443 listening
- [ ] Hosts file configuration
- [ ] Ping test to domain name
- [ ] Browser certificate import process
- [ ] HTTPS website loading successfully (with padlock)
- [ ] Certificate details in browser
- [ ] Apache access logs showing connection
- [ ] Network environment details (IP addresses, OS versions, VMware mode)

---

## TROUBLESHOOTING CHECKLIST

If HTTPS doesn't work:

- [ ] **Firewall**: `sudo ufw allow 443/tcp` on Kali
- [ ] **Apache SSL enabled**: `sudo apache2ctl -M | grep ssl` should show ssl_module
- [ ] **Certificate paths correct**: Verify in `/etc/apache2/sites-available/default-ssl.conf`
- [ ] **File permissions**: `ls -l /etc/apache2/ssl/` - key file should be 600
- [ ] **Domain name matches**: Certificate CN and hosts file entry match
- [ ] **CA certificate trusted**: Check browser certificate manager
- [ ] **Apache errors**: `sudo tail /var/log/apache2/error.log`
- [ ] **Certificate validity**: `openssl x509 -in /etc/apache2/ssl/server-cert.crt -noout -dates`
- [ ] **SAN included**: `openssl x509 -in /etc/apache2/ssl/server-cert.crt -noout -ext subjectAltName`

---

## FINAL CHECKLIST

- [ ] All commands documented with purposes (15 points)
- [ ] CA certificate in readable format (5 points)
- [ ] Root CA analysis answer (5 points)
- [ ] Server certificate in readable format (5 points)
- [ ] Web server certificate analysis answer (5 points)
- [ ] Server public key extracted (5 points)
- [ ] Signing algorithm identified (5 points)
- [ ] Private key explanation (5 points)
- [ ] Apache server configuration documented (10 points)
- [ ] Client hosts file configuration documented (10 points)
- [ ] HTTPS access successful from browser (10 points)
- [ ] Network topology diagram (10 points)
- [ ] Network environment description (10 points)
- [ ] All required screenshots (10 points)

**TOTAL: 100 points**

---

## SUBMISSION READY

- [ ] All deliverables compiled into one document
- [ ] Document formatted professionally
- [ ] All screenshots clear and labeled
- [ ] All questions answered with evidence
- [ ] Group member names included
- [ ] Submit before November 17, 2025, before class starts

---

**You now have a complete execution plan. Follow each checkbox in order, and you will successfully complete Project 3 Stage 1!**
