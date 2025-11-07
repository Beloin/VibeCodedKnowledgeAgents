# Network Security - Comprehensive Technical Guide

## Fundamental Security Concepts

### CIA Triad - Core Security Principles

**Confidentiality**:
- **Definition**: Protection of data from unauthorized access
- **Techniques**: Encryption, access controls, data classification
- **Implementation**: AES, RSA, access control lists

**Integrity**:
- **Definition**: Assurance that data has not been modified
- **Techniques**: Hash functions, digital signatures, checksums
- **Implementation**: SHA-256, HMAC, digital certificates

**Availability**:
- **Definition**: Ensuring systems and data are accessible when needed
- **Techniques**: Redundancy, backups, DDoS protection
- **Implementation**: Load balancing, clustering, failover systems

### Authentication, Authorization, and Accounting (AAA)

**Authentication**:
- **Methods**: Passwords, biometrics, tokens, certificates
- **Protocols**: RADIUS, TACACS+, Kerberos, OAuth
- **Multi-factor Authentication**: Something you know, have, are

**Authorization**:
- **Role-Based Access Control (RBAC)**: Permissions based on roles
- **Attribute-Based Access Control (ABAC)**: Dynamic permission evaluation
- **Policy Enforcement**: Access control lists, firewall rules

**Accounting**:
- **Logging**: User activities and system events
- **Audit Trails**: Comprehensive activity records
- **Compliance**: Regulatory requirement fulfillment

## Network Security Threats and Attacks

### Common Attack Vectors

#### Layer 2 Attacks

**MAC Flooding**:
- **Mechanism**: Overwhelm switch MAC address table
- **Impact**: Switch reverts to hub-like behavior
- **Prevention**: Port security, MAC address limiting

**ARP Spoofing/Poisoning**:
- **Mechanism**: Send fake ARP replies to associate attacker MAC with victim IP
- **Impact**: Man-in-the-middle attacks, traffic interception
- **Prevention**: Dynamic ARP inspection, static ARP entries

**VLAN Hopping**:
- **Mechanism**: Exploit VLAN trunking to access restricted VLANs
- **Impact**: Unauthorized access to network segments
- **Prevention**: Disable unused ports, proper VLAN configuration

#### Layer 3/4 Attacks

**IP Spoofing**:
- **Mechanism**: Forge source IP address in packets
- **Impact**: Bypass IP-based filtering, DDoS amplification
- **Prevention**: Ingress/egress filtering, BCP 38/84

**TCP SYN Flood**:
- **Mechanism**: Send SYN packets without completing handshake
- **Impact**: Exhaust server connection resources
- **Prevention**: SYN cookies, rate limiting, firewalls

**DNS Cache Poisoning**:
- **Mechanism**: Inject false DNS records into cache
- **Impact**: Redirect users to malicious sites
- **Prevention**: DNSSEC, random source ports, query ID randomization

#### Application Layer Attacks

**SQL Injection**:
- **Mechanism**: Inject malicious SQL code through user input
- **Impact**: Database manipulation, data theft
- **Prevention**: Input validation, parameterized queries, WAF

**Cross-Site Scripting (XSS)**:
- **Mechanism**: Inject malicious scripts into web pages
- **Impact**: Session hijacking, credential theft
- **Prevention**: Input sanitization, Content Security Policy

**Cross-Site Request Forgery (CSRF)**:
- **Mechanism**: Trick users into executing unwanted actions
- **Impact**: Unauthorized transactions, data modification
- **Prevention**: Anti-CSRF tokens, SameSite cookies

### Advanced Persistent Threats (APTs)

**Characteristics**:
- **Targeted**: Specific organizations or individuals
- **Persistent**: Long-term presence in network
- **Stealthy**: Advanced evasion techniques
- **Multi-vector**: Combination of attack methods

**Detection and Prevention**:
- **Behavioral Analysis**: Anomaly detection
- **Threat Intelligence**: Known attack patterns
- **Network Segmentation**: Limit lateral movement
- **Endpoint Protection**: Advanced antivirus, EDR

## Encryption and Cryptography

### Symmetric Encryption

**Advanced Encryption Standard (AES)**:
- **Key Sizes**: 128, 192, 256 bits
- **Block Size**: 128 bits
- **Modes**: ECB, CBC, CFB, OFB, CTR, GCM
- **Performance**: Hardware acceleration support

**Triple DES (3DES)**:
- **Key Size**: 168 bits (effective 112 bits)
- **Block Size**: 64 bits
- **Security**: Deprecated, use AES instead

**ChaCha20**:
- **Stream Cipher**: High performance on mobile devices
- **Key Size**: 256 bits
- **Use Cases**: TLS 1.3, mobile applications

### Asymmetric Encryption

**RSA (Rivest-Shamir-Adleman)**:
- **Key Sizes**: 2048, 3072, 4096 bits
- **Security**: Based on integer factorization
- **Performance**: Slower than symmetric encryption
- **Use Cases**: Key exchange, digital signatures

**Elliptic Curve Cryptography (ECC)**:
- **Key Sizes**: 256, 384, 521 bits
- **Security**: Based on elliptic curve discrete logarithm
- **Performance**: Faster than RSA with equivalent security
- **Use Cases**: Mobile devices, IoT, modern TLS

### Hash Functions

**SHA-2 Family**:
- **SHA-256**: 256-bit output
- **SHA-384**: 384-bit output  
- **SHA-512**: 512-bit output
- **Security**: Currently secure, widely adopted

**SHA-3**:
- **Design**: Sponge construction
- **Security**: Different mathematical foundation than SHA-2
- **Adoption**: Growing but slower than SHA-2

**Message Authentication Codes (MAC)**:
- **HMAC**: Hash-based MAC
- **CMAC**: Cipher-based MAC
- **GMAC**: Galois/Counter Mode MAC

## Network Security Protocols

### TLS/SSL Protocol Suite

#### TLS 1.3 Protocol

**Handshake Process**:
1. **Client Hello**: Supported cipher suites, key share
2. **Server Hello**: Selected cipher suite, key share
3. **Finished**: Authentication and key confirmation

**Key Improvements**:
- **1-RTT Handshake**: Faster connection establishment
- **Forward Secrecy**: Mandatory ephemeral key exchange
- **Simplified Cipher Suites**: Removed weak algorithms
- **0-RTT**: Optional zero round-trip time resumption

**Cipher Suite Examples**:
- **TLS_AES_128_GCM_SHA256**: AES-128-GCM with SHA-256
- **TLS_AES_256_GCM_SHA384**: AES-256-GCM with SHA-384
- **TLS_CHACHA20_POLY1305_SHA256**: ChaCha20 with Poly1305

#### Certificate Management

**X.509 Certificate Structure**:
- **Version**: Certificate format version
- **Serial Number**: Unique identifier
- **Signature Algorithm**: Hash and signature algorithm
- **Issuer**: Certificate authority
- **Validity Period**: Not before/after dates
- **Subject**: Entity the certificate belongs to
- **Public Key**: Subject's public key
- **Extensions**: Additional information

**Certificate Authorities (CAs)**:
- **Public CAs**: Let's Encrypt, DigiCert, GlobalSign
- **Private CAs**: Internal enterprise CAs
- **Certificate Chains**: Root CA → Intermediate CA → End-entity

### IPsec Protocol Suite

#### Security Associations (SA)

**SA Parameters**:
- **Security Parameter Index (SPI)**: Unique identifier
- **Destination IP**: Target of SA
- **Security Protocol**: AH or ESP
- **Encryption Algorithm**: AES, 3DES, etc.
- **Authentication Algorithm**: SHA-256, MD5, etc.

#### IPsec Modes

**Transport Mode**:
- **Encryption**: IP payload only
- **Use Cases**: Host-to-host communication
- **Header Preservation**: Original IP header intact

**Tunnel Mode**:
- **Encryption**: Entire IP packet
- **Use Cases**: Site-to-site VPN, gateway-to-gateway
- **New Header**: New IP header added

#### Internet Key Exchange (IKE)

**IKEv2 Protocol**:
- **Phase 1**: Establish IKE SA
- **Phase 2**: Establish IPsec SA
- **Features**: MOBIKE (mobility), EAP authentication
- **Performance**: Faster than IKEv1

### Wireless Security Protocols

#### WPA3 Security

**Personal Mode**:
- **Simultaneous Authentication of Equals (SAE)**:
  - Resistant to dictionary attacks
  - Forward secrecy
  - Protection against offline attacks

**Enterprise Mode**:
- **192-bit Security Suite**:
  - Suite B cryptographic algorithms
  - 384-bit elliptic curve
  - Required for government and enterprise

**Enhanced Open**:
- **Opportunistic Wireless Encryption (OWE)**:
  - Encryption without authentication
  - Protection in public Wi-Fi
  - Individualized encryption

## Firewall Technologies

### Stateful Firewalls

#### Connection Tracking

**State Table Management**:
- **Entry Creation**: When new connection detected
- **State Maintenance**: Track connection progress
- **Entry Removal**: Timeout or connection termination
- **Memory Management**: Efficient state table handling

**Stateful Inspection Process**:
1. **Packet Arrival**: Receive incoming packet
2. **State Lookup**: Check state table for existing connection
3. **Policy Check**: Verify against security policy
4. **State Update**: Update connection state
5. **Forward/Block**: Allow or deny based on state and policy

#### Advanced Firewall Features

**Application Awareness**:
- **Deep Packet Inspection (DPI)**:
  - Protocol identification
  - Application signature matching
  - Behavioral analysis

**User Identity Integration**:
- **Active Directory Integration**:
  - User-based policies
  - Group membership evaluation
  - Time-based access controls

**Threat Intelligence**:
- **Reputation Services**:
  - IP reputation scoring
  - Domain categorization
  - Malware hash checking

### Next-Generation Firewalls (NGFW)

#### Core Capabilities

**Integrated Intrusion Prevention**:
- **Signature-based Detection**: Known attack patterns
- **Anomaly-based Detection**: Behavioral deviations
- **Heuristic Analysis**: Rule-based threat identification

**SSL/TLS Inspection**:
- **Certificate Validation**: Verify server certificates
- **Decryption/Re-encryption**: Inspect encrypted traffic
- **Performance Impact**: Hardware acceleration required

**Sandboxing**:
- **File Analysis**: Execute suspicious files in isolated environment
- **Behavior Monitoring**: Track file activities
- **Threat Scoring**: Risk assessment based on behavior

#### Deployment Considerations

**Performance Requirements**:
- **Throughput**: Maximum data transfer rate
- **Connections per Second**: New connection establishment rate
- **Concurrent Connections**: Active connection capacity
- **SSL Inspection Performance**: Decryption throughput

**High Availability**:
- **Active/Active**: Both units processing traffic
- **Active/Passive**: One unit active, one standby
- **State Synchronization**: Session state replication

## Intrusion Detection and Prevention Systems

### Network-based IDS/IPS (NIDS/NIPS)

#### Detection Methods

**Signature-based Detection**:
- **Pattern Matching**: Known attack signatures
- **Protocol Analysis**: Protocol violation detection
- **String Matching**: Specific content patterns

**Anomaly-based Detection**:
- **Statistical Analysis**: Baseline behavior modeling
- **Machine Learning**: Pattern recognition algorithms
- **Behavioral Analysis**: Deviation from normal patterns

**Heuristic Analysis**:
- **Rule-based Systems**: Custom detection rules
- **Expert Systems**: Knowledge-based analysis
- **Fuzzy Logic**: Approximate reasoning

#### Deployment Strategies

**Inline Deployment**:
- **Position**: Directly in traffic path
- **Advantages**: Real-time blocking capability
- **Disadvantages**: Potential performance impact

**Passive Deployment**:
- **Position**: Network TAP or SPAN port
- **Advantages**: No performance impact
- **Disadvantages**: Detection only, no prevention

### Host-based IDS/IPS (HIDS/HIPS)

#### Host Monitoring

**File Integrity Monitoring**:
- **Checksum Verification**: File hash comparison
- **Change Detection**: File modification alerts
- **Baseline Creation**: Initial system state recording

**Log Analysis**:
- **System Logs**: Operating system events
- **Application Logs**: Software-specific events
- **Security Logs**: Authentication and access events

**Process Monitoring**:
- **Process Creation**: New process detection
- **Memory Analysis**: Suspicious memory patterns
- **Network Activity**: Process network connections

## Virtual Private Networks (VPNs)

### Site-to-Site VPNs

#### IPsec VPN Configuration

**Phase 1 Parameters**:
```
Encryption: AES-256
Hash: SHA-256
DH Group: 14 (2048-bit)
Lifetime: 28800 seconds
Authentication: Pre-shared key
```

**Phase 2 Parameters**:
```
Encryption: AES-256
Hash: SHA-256
PFS: Group 14
Lifetime: 3600 seconds
Protocol: ESP
Mode: Tunnel
```

#### Dynamic Multipoint VPN (DMVPN)

**Components**:
- **Hub**: Central connection point
- **Spokes**: Remote sites
- **mGRE**: Multipoint Generic Routing Encapsulation
- **NHRP**: Next Hop Resolution Protocol

**Benefits**:
- **Scalability**: Easy addition of new sites
- **Dynamic Routing**: Automatic route propagation
- **Spoke-to-Spoke**: Direct communication between spokes

### Remote Access VPNs

#### SSL VPN Technologies

**Clientless Access**:
- **Web Portal**: Browser-based access
- **Application Translation**: Webification of applications
- **File Access**: Secure file sharing

**Full Tunnel Clients**:
- **Always-on VPN**: Continuous connection
- **Split Tunneling**: Selective traffic routing
- **Endpoint Security**: Client security checks

#### Authentication Methods

**Multi-factor Authentication**:
- **Hardware Tokens**: Physical security devices
- **Software Tokens**: Mobile app-based tokens
- **Biometric Authentication**: Fingerprint, facial recognition

**Certificate-based Authentication**:
- **Client Certificates**: User-specific digital certificates
- **Machine Certificates**: Device-specific certificates
- **Certificate Revocation**: CRL/OCSP checking

## Security Monitoring and Management

### Security Information and Event Management (SIEM)

#### SIEM Architecture

**Data Collection**:
- **Log Sources**: Firewalls, servers, applications
- **Normalization**: Standardized log format
- **Correlation**: Relationship analysis between events

**Analysis Engine**:
- **Real-time Analysis**: Immediate threat detection
- **Historical Analysis**: Long-term pattern recognition
- **Machine Learning**: Advanced threat detection

**Reporting and Alerting**:
- **Dashboards**: Real-time security status
- **Alerts**: Immediate notification of security events
- **Reports**: Compliance and audit documentation

#### Use Case Examples

**Brute Force Detection**:
- **Pattern**: Multiple failed logins from same source
- **Threshold**: 5+ failures in 5 minutes
- **Response**: Account lockout, source IP blocking

**Data Exfiltration**:
- **Indicators**: Large outbound transfers, unusual protocols
- **Baseline**: Normal data transfer patterns
- **Response**: Connection termination, user notification

### Network Access Control (NAC)

#### NAC Components

**Policy Servers**:
- **Authentication**: User and device verification
- **Authorization**: Access permission determination
- **Accounting**: Activity logging and reporting

**Enforcement Points**:
- **Network Devices**: Switches, routers, wireless controllers
- **Endpoint Agents**: Software on client devices
- **Inline Appliances**: Dedicated NAC appliances

#### NAC Deployment Models

**Pre-admission Control**:
- **Assessment**: Device health check before network access
- **Quarantine**: Isolated network for non-compliant devices
- **Remediation**: Guidance for compliance achievement

**Post-admission Control**:
- **Continuous Monitoring**: Ongoing compliance verification
- **Dynamic Policy Enforcement**: Real-time access adjustment
- **Threat Response**: Immediate access revocation for threats

## Security Configuration Examples

### Firewall Configuration

#### Cisco ASA Firewall Rules
```bash
! Access control lists
access-list outside_in extended permit tcp any host 203.0.113.10 eq 80
access-list outside_in extended permit tcp any host 203.0.113.10 eq 443
access-list outside_in extended deny ip any any

! NAT configuration
object network web-server
 host 192.168.1.10
 nat (inside,outside) static 203.0.113.10

! Apply ACL to interface
access-group outside_in in interface outside
```

#### iptables Firewall Rules
```bash
# Basic firewall rules
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Save rules
iptables-save > /etc/iptables/rules.v4
```

### VPN Configuration

#### OpenVPN Server Configuration
```bash
# Server configuration
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh2048.pem
server 10.8.0.0 255.255.255.0
push "route 192.168.1.0 255.255.255.0"
keepalive 10 120
cipher AES-256-CBC
auth SHA256
user nobody
group nogroup
persist-key
persist-tun
status openvpn-status.log
verb 3
```

#### WireGuard Configuration
```bash
# Server configuration (wg0.conf)
[Interface]
PrivateKey = server_private_key
Address = 10.0.0.1/24
ListenPort = 51820

[Peer]
PublicKey = client_public_key
AllowedIPs = 10.0.0.2/32
```

## Security Best Practices

### Network Segmentation

**Zero Trust Architecture**:
- **Principle**: Never trust, always verify
- **Implementation**: Micro-segmentation, least privilege
- **Benefits**: Reduced attack surface, containment

**DMZ Design**:
- **Public Services**: Web servers, email gateways
- **Semi-trusted Zone**: Application servers
- **Internal Network**: Database servers, management systems

### Security Hardening

**Network Device Hardening**:
- **Disable Unused Services**: Telnet, HTTP, CDP
- **Strong Authentication**: AAA with TACACS+/RADIUS
- **Access Controls**: Role-based CLI access
- **Logging**: Comprehensive audit trails

**Server Hardening**:
- **Patch Management**: Regular security updates
- **Service Minimization**: Disable unnecessary services
- **File System Permissions**: Least privilege principle
- **Network Configuration**: Firewall rules, port restrictions

### Incident Response

**Incident Response Plan**:
- **Preparation**: Tools, team, procedures
- **Identification**: Detection and analysis
- **Containment**: Isolate affected systems
- **Eradication**: Remove threat components
- **Recovery**: Restore normal operations
- **Lessons Learned**: Process improvement

**Forensic Procedures**:
- **Evidence Preservation**: Chain of custody
- **Memory Analysis**: Volatile data collection
- **Disk Imaging**: Complete system capture
- **Log Analysis**: Timeline reconstruction

This comprehensive guide provides detailed technical information about network security, from fundamental concepts and protocols to advanced threat protection and incident response procedures.