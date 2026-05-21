# Security & Compliance Framework - IICS Zurich Insurance

## 1. Data Encryption Standards

### 1.1 In-Transit Encryption

**TLS Configuration**
```
Protocol Version: TLS 1.2 or higher
Cipher Suites:
  - TLS_RSA_WITH_AES_256_GCM_SHA384
  - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
  - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384

Certificate:
  - Type: CA-signed X.509 certificate
  - Key Length: 2048-bit RSA or 256-bit ECDSA
  - Validity: 1 year
  - Renewal: 30 days before expiry
```

**API Communication**
```
All APIs must use HTTPS:
- Salesforce: OAuth 2.0 over HTTPS
- Oracle Cloud: TLS 1.2 minimum
- AWS S3: HTTPS + bucket policies
- Snowflake: HTTPS + TLS 1.2 minimum
```

### 1.2 At-Rest Encryption

**Database Encryption**
```
Oracle Database:
  - Transparent Data Encryption (TDE)
  - Master key stored in Oracle Key Vault
  - Encryption algorithm: AES-256
  - Encrypted tablespaces: POLICY, CUSTOMER, CLAIMS

Snowflake:
  - Native Snowflake encryption
  - Default: AES-256-CBC
  - Multi-tenancy isolation
  - Automatic key rotation

AWS S3:
  - Server-Side Encryption (SSE-S3)
  - Encryption type: AES-256
  - Bucket encryption: Mandatory
  - Key management: AWS KMS
```

### 1.3 Column-Level Encryption

**Sensitive Data Fields**
```
Encrypted Columns:
  - CUSTOMER.EMAIL
  - CUSTOMER.PHONE
  - CUSTOMER.SSN
  - POLICY.CUSTOMER_ID
  - CLAIMS.CLAIM_DETAILS

Encryption Method:
  - Algorithm: AES-256 with HMAC
  - Key: Stored in secure credential vault
  - Application-level: Encrypt before storage
```

---

## 2. Authentication & Authorization

### 2.1 Multi-Factor Authentication (MFA)

**Setup**
```
Requirement: Mandatory for all users
Type: Time-based OTP (TOTP)
Supported Apps:
  - Google Authenticator
  - Microsoft Authenticator
  - Authy
  - FreeOTP

Setup Process:
1. Generate QR code in IICS console
2. Scan with authenticator app
3. Verify 6-digit code
4. Save backup codes (10)
5. Confirm MFA activation
```

### 2.2 Role-Based Access Control (RBAC)

**Role Definitions**

| Role | Create | Edit | Execute | View | Admin |
|------|--------|------|---------|------|-------|
| **Admin** | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Developer** | ✓ | ✓ | ✓ (test) | ✓ | ✗ |
| **Operator** | ✗ | ✗ | ✓ | ✓ | ✗ |
| **Viewer** | ✗ | ✗ | ✗ | ✓ | ✗ |
| **Auditor** | ✗ | ✗ | ✗ | ✓ (logs) | ✗ |

### 2.3 API Key Management

**Key Generation**
```
Steps:
1. Navigate to: Administration → API Management
2. Click: "Generate New API Key"
3. Provide: Description, expiration date
4. Copy: API Key ID and Secret
5. Store: In secure credential vault

Key Rotation:
- Frequency: Every 90 days
- Overlap period: 1 week
- Emergency rotation: On compromise
```

---

## 3. Audit & Compliance

### 3.1 Audit Logging

**Logged Operations**
```
User Actions:
  - Login/Logout: User, time, IP, status
  - Create object: User, object type, name, time
  - Modify object: User, changes, before/after values
  - Delete object: User, object, reason
  - Execute task: User, task, start time, end time, status

Data Access:
  - Query execution: User, query, table, time, record count
  - Export data: User, data, format, time
  - View sensitive field: User, field, record, time

System Events:
  - Configuration changes: What changed, who, when
  - Security events: MFA activation, key rotation
  - Error events: Error type, component, timestamp
  - Performance events: Slow queries, bottlenecks
```

**Log Storage**
```
Location: AWS CloudWatch Logs
Retention: 2 years (compliance requirement)
Rotation: Monthly archival to S3 Glacier
Encryption: AES-256 at rest
Access: Restricted to authorized auditors
Backup: Daily automated backup
```

### 3.2 Data Lineage

**Lineage Tracking**
```
For each record, track:
  - Source system and table
  - Extraction timestamp
  - Transformations applied
  - Target system and table
  - Load timestamp
  - Load status (success/failure)
  - Error details (if failed)

Example:
  Oracle POLICY table
    → M_POLICY_EXTRACT_LOAD mapping
    → Validation rules applied
    → Snowflake POLICY_DIM table
    → Loaded at 2024-05-21 02:15:30 UTC
    → Status: Success (145,230 records)
```

### 3.3 Compliance Tracking

**SOX Compliance**
```
Requirements:
  - Financial data access control
  - Change management and approval
  - System audit trails
  - Access reviews (quarterly)
  - Segregation of duties

Implementation:
  - All changes require approval
  - Four-eyes principle: 2 approvals needed
  - Audit trail: Complete and immutable
  - Access review: Quarterly listing
```

**GDPR Compliance**
```
Key Requirements:
  - Data subject rights (access, deletion, portability)
  - Data minimization
  - Privacy by design
  - Data protection impact assessments
  - Breach notification (72 hours)

Implementation:
  - Customer data marked with tag: PII
  - Data retention policy: 5 years max
  - Anonymization: Remove identifiers
  - Right to be forgotten: Automated deletion
  - Data export: GDPR format (JSON/CSV)
```

**ISO 27001 Compliance**
```
Domains:
  - Access control
  - Cryptography
  - Incident management
  - Business continuity

Implementation:
  - Document information security policy
  - Risk assessment (annual)
  - Access control matrix
  - Incident response plan
  - Business continuity plan
  - Security training (annual)
```

---

## 4. Credential Management

### 4.1 Secure Credential Storage

**Credential Vault**
```
Location: Informatica Secure Agent credential vault
Encryption: AES-256 with HMAC
Access: Only Secure Agent can decrypt
Audit Trail: All access logged

Stored Credentials:
  - Database passwords
  - API keys
  - OAuth tokens
  - SSH keys
  - SSL certificates
  - Custom encryption keys
```

**Password Policy**
```
Requirements:
  - Minimum length: 12 characters
  - Complexity: Must include:
    * Uppercase letters (A-Z)
    * Lowercase letters (a-z)
    * Numbers (0-9)
    * Special characters (!@#$%^&*)
  - No dictionary words
  - No repeating characters

Rotation:
  - System accounts: Every 30 days
  - Service accounts: Every 90 days
  - Manual rotation: On employee departure
  - Emergency rotation: On suspected compromise
```

---

## 5. Network Security

### 5.1 Firewall & Network Policies

**Inbound Rules**
```
HTTPS (443):
  - Source: Office IP range 203.0.113.0/24
  - Source: VPN IP range 198.51.100.0/24
  - Destination: IICS cloud services
  - Allow: Yes

Database Port (1521 for Oracle):
  - Source: Secure Agent IP 10.0.1.5
  - Destination: Oracle DB 10.0.2.10
  - Allow: Yes

SSH (22):
  - Source: Specific admin IPs
  - Destination: Secure Agent server
  - Allow: Yes
```

**Outbound Rules**
```
HTTPS (443):
  - Destination: Informatica Cloud
  - Destination: Salesforce
  - Destination: AWS S3
  - Destination: Snowflake
  - Allow: Yes

DNS (53):
  - Destination: Corporate DNS servers
  - Allow: Yes

Email (587/SMTP):
  - Destination: Email server
  - Allow: Yes (for notifications)
```

---

## 6. Security Best Practices

### 6.1 Development Practices

```
Code Review:
  - All mappings must be peer-reviewed
  - Security checklist:
    * No hardcoded credentials
    * Proper error handling
    * Input validation
    * SQL injection prevention
    * XSS prevention
  - Approval required: 2 reviewers

Testing:
  - Security testing: SAST (Static Analysis)
  - Dependency scanning: Check for vulnerabilities
  - Penetration testing: Annual for critical systems
  - Data masking: Mask PII in test data
```

### 6.2 Incident Response

**Incident Classification**
```
Level 1 (Critical):
  - Data breach affecting > 1000 customers
  - System outage > 4 hours
  - Ransomware detected
  - Response Time: 15 minutes

Level 2 (High):
  - Data breach affecting < 1000 customers
  - System outage < 4 hours
  - Security vulnerability exploited
  - Response Time: 1 hour

Level 3 (Medium):
  - Unauthorized access attempt (blocked)
  - Failed authentication (multiple)
  - Misconfiguration found
  - Response Time: 4 hours

Level 4 (Low):
  - Informational security event
  - Policy violation (minor)
  - Warning-level anomaly
  - Response Time: 1 business day
```

**Response Procedure**
```
1. Detect → Alert (automated or manual)
2. Triage → Classify severity level
3. Contain → Isolate affected systems
4. Investigate → Determine root cause
5. Remediate → Apply fix
6. Recover → Restore systems
7. Notify → Inform stakeholders
8. Review → Post-incident analysis
```

---

## Compliance Checklist

- [ ] Data encryption implemented (in-transit, at-rest, column-level)
- [ ] MFA enabled for all users
- [ ] RBAC configured with principle of least privilege
- [ ] API keys generated and stored securely
- [ ] Audit logging enabled and monitored
- [ ] Data lineage tracking implemented
- [ ] SOX compliance controls in place
- [ ] GDPR requirements documented and implemented
- [ ] ISO 27001 policies documented
- [ ] Network security policies configured
- [ ] VPN connectivity established
- [ ] Incident response plan created
- [ ] Security training completed by all users
- [ ] Penetration testing scheduled
- [ ] Backup encryption verified
- [ ] Credential vault configured
- [ ] OAuth authentication tested
- [ ] Certificate management process established

**Version**: 1.0  
**Last Updated**: May 2026
