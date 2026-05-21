# IICS Setup Guide - Zurich Insurance Project

## Prerequisites

### Required Software & Accounts
- ✅ Informatica IICS Cloud Account (Admin access)
- ✅ Salesforce Developer Account
- ✅ Oracle Cloud Account or local Oracle Database
- ✅ AWS Account (for S3 access)
- ✅ Snowflake Account
- ✅ Git and GitHub Desktop
- ✅ Text Editor (VS Code, Sublime, etc.)

### System Requirements
- **Operating System**: Windows/Mac/Linux
- **Java**: Version 8 or higher
- **RAM**: Minimum 8GB
- **Network**: VPN access to source systems

---

## Step 1: IICS Cloud Organization Setup

### 1.1 Create Cloud Organization

```
1. Login to Informatica Cloud
   URL: https://www.informaticacloud.com

2. Navigate to Administration → Shared Objects

3. Create new Organization
   Name: Zurich_Insurance_PROD
   Description: Zurich Insurance IICS Implementation
   Time Zone: UTC

4. Enable required modules:
   ✅ Data Integration
   ✅ Data Quality
   ✅ Master Data Management
   ✅ API Management
   ✅ Governance
```

### 1.2 Configure User Roles

```
Roles to Create:

1. IICS_Admin
   - Full access to all objects
   - Organization settings
   - User management

2. IICS_Developer
   - Create/Edit mappings
   - Create/Edit workflows
   - Test data integration jobs
   
3. IICS_Viewer
   - Read-only access
   - View monitoring dashboards
   - View logs

4. IICS_Operator
   - Execute scheduled tasks
   - View monitoring and alerts
   - Perform manual task runs

Role Assignments:
- Admin: your_admin_email
- Developer: your_dev_email
- Operator: operations_team_email
- Viewer: business_analyst_email
```

### 1.3 Setup Security

```
Security Settings:

1. Enable Multi-Factor Authentication (MFA)
   - Navigate to: Administration → Security
   - Enable MFA for all users
   - Type: Time-based OTP (TOTP)
   - Authenticator App: Google Authenticator, Authy

2. Set Password Policy
   - Minimum length: 12 characters
   - Complexity required: Yes
   - Include uppercase, lowercase, numbers, special chars
   - Rotation: Every 90 days
   - History: Remember last 5 passwords

3. Configure IP Whitelisting
   - Add office IP ranges: 203.0.113.0/24
   - Add VPN IP range: 198.51.100.0/24
   - Enable: Strict IP validation

4. Enable API Security
   - Generate API keys in console
   - Set key rotation policy: 90 days
   - Implement key versioning
   - Track API key usage
```

---

## Step 2: Cloud Connectors Configuration

### 2.1 Secure Agent Setup

#### Install Secure Agent

```
Steps:
1. Download Secure Agent from IICS UI
   - Administration → Secure Agent → Download
   
2. Extract to installation directory
   - Windows: C:\Program Files\Informatica\Agent
   - Linux: /opt/informatica/agent
   - Mac: /Users/[user]/informatica/agent

3. Configure with IICS credentials
   - Edit: infagent.ini
   - Add IICS URL and credentials
   - Set SSL certificate path

4. Start Secure Agent
   Windows: net start InformaticaAgent
   Linux: ./infaagent.sh -d -f infagent.ini

5. Verify Status
   - Check IICS console for agent status
   - Should show as "Running" within 2 minutes
```

#### Secure Agent Configuration File

```ini
; infagent.ini - Example Configuration
[AGENT]
AgentName=ZURICH_AGENT_01
LISTENHOST=localhost
LISTENPORT=6005
LOGDIR=/var/log/informatica/agent
LOGLEVEL=info
SSLCERTFILE=/etc/ssl/informatica.pem

[IICS_CLOUD]
CLOUDURL=https://app.informaticacloud.com
CLOUDUSER=your_admin_email@zurich.com
CLOUDPASSWORD=your_encrypted_password
CLOUDORG=Zurich_Insurance_PROD
```

### 2.2 Salesforce Connector

#### Setup Salesforce Connection

```
Connection Details:
- Name: SFDC_Production
- Type: Salesforce
- Environment: Production
- Username: sfdc_integration_user@zurich.com
- Password: [encrypted password]
- Security Token: [Salesforce token]
- Client ID: [OAuth Client ID from Connected App]
- Client Secret: [OAuth Client Secret]

Test Connection:
1. Click "Test Connection"
2. Verify successful connection
3. Grant permission to access Salesforce objects
4. Confirm access to: Accounts, Contacts, Opportunities
```

#### Create Salesforce Connected App

```
Salesforce Setup:
1. Login to Salesforce
2. Setup → Apps → App Manager
3. Create New Connected App
   - App Name: IICS Integration
   - API Name: IICS_Integration
   - Contact Email: integration@zurich.com

4. OAuth Settings
   - Enable OAuth Settings: Yes
   - Callback URL: https://app.informaticacloud.com/oauth
   - Selected OAuth Scopes:
     * Access and manage your data (api)
     * Perform requests on your behalf (refresh_token, offline_access)

5. Save and copy:
   - Consumer Key (Client ID)
   - Consumer Secret (Client Secret)
```

### 2.3 Oracle Cloud Connector

#### Database Connection Details

```
Connection: OracleDB_Production
Type: Oracle Database
Connection Method: Secure Network

Hostname: oracle.db.zurich.com
Port: 1521
Database: ZURICH_PROD
Username: informatica_user
Password: [encrypted password]

Connection Pool Settings:
  - Min Size: 5
  - Max Size: 50
  - Idle Timeout: 900 seconds
  - Connection Timeout: 300 seconds
  - Validation Query: SELECT 1 FROM DUAL

SSL/TLS Configuration:
  - Use SSL: Yes
  - Certificate Path: /etc/ssl/oracle-ca.pem
  - Verify Certificate: Yes
```

#### Oracle User Setup

```sql
-- Create INFORMATICA user in Oracle
CREATE USER informatica_user IDENTIFIED BY "Strong_Password_123!";

-- Grant required privileges
GRANT CREATE SESSION TO informatica_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON policy TO informatica_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON customer TO informatica_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON claims TO informatica_user;

-- Grant for staging tables
GRANT CREATE TABLE TO informatica_user;
GRANT UNLIMITED TABLESPACE TO informatica_user;

-- Grant for reading logs
GRANT SELECT ON v$session TO informatica_user;
GRANT SELECT ON v$sql TO informatica_user;
```

### 2.4 AWS S3 Connector

#### S3 Configuration

```
Connection: AWS_S3_DataLake
Type: Amazon S3

Access Key ID: AKIA....... (from AWS IAM)
Secret Access Key: [encrypted key]
Region: us-east-1
Bucket: zurich-insurance-datalake

Folder Structure:
├── /raw-data/
│   ├── /policy/
│   ├── /customer/
│   ├── /claims/
│   └── /premium/
├── /processed-data/
│   ├── /policy/
│   ├── /customer/
│   ├── /claims/
│   └── /premium/
└── /archive/
    └── /historical/

Encryption: AES-256 (SSE-S3)
Versioning: Enabled
Lifecycle Policy: 
  - Move to Glacier after 90 days
  - Delete after 2 years
```

#### AWS IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::zurich-insurance-datalake",
        "arn:aws:s3:::zurich-insurance-datalake/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "kms:Decrypt",
        "kms:Encrypt",
        "kms:GenerateDataKey"
      ],
      "Resource": "arn:aws:kms:us-east-1:ACCOUNT_ID:key/KEY_ID"
    }
  ]
}
```

### 2.5 Snowflake Connector

#### Snowflake Connection

```
Connection: Snowflake_DW
Type: Snowflake Cloud Data Warehouse

Account: zurich_account
Region: us-east-1
Warehouse: COMPUTE_WH
Database: ANALYTICS_DB
Schema: PUBLIC
Username: informatica_user
Password: [encrypted password]

Connection Pool:
  - Min Pool Size: 5
  - Max Pool Size: 30
  - Idle Timeout: 600 seconds

SSL Configuration:
  - Use SSL: Yes
  - Certificate Verification: Yes
```

#### Snowflake User Setup

```sql
-- Create INFORMATICA role and user
CREATE ROLE informatica_role;
CREATE USER informatica_user PASSWORD = 'Strong_Password_123!' DEFAULT_ROLE = informatica_role;

-- Grant warehouse access
GRANT USAGE ON WAREHOUSE COMPUTE_WH TO ROLE informatica_role;
GRANT USAGE ON DATABASE ANALYTICS_DB TO ROLE informatica_role;
GRANT USAGE ON SCHEMA ANALYTICS_DB.PUBLIC TO ROLE informatica_role;

-- Grant table permissions
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA ANALYTICS_DB.PUBLIC TO ROLE informatica_role;
GRANT CREATE TABLE ON SCHEMA ANALYTICS_DB.PUBLIC TO ROLE informatica_role;

-- Assign role to user
GRANT ROLE informatica_role TO USER informatica_user;
```

---

## Step 3: Create Source and Target Connections

### 3.1 Source Connections

```
1. Oracle_Policy_Source
   Type: Oracle Database
   Connection: OracleDB_Production
   Tables: POLICY, POLICY_DETAILS, POLICY_HISTORY
   
2. Oracle_Claims_Source
   Type: Oracle Database
   Connection: OracleDB_Production
   Tables: CLAIMS, CLAIM_DETAILS, CLAIM_HISTORY
   
3. SFDC_Accounts_Source
   Type: Salesforce
   Connection: SFDC_Production
   Objects: Account, Contact, Opportunity
   
4. AWS_S3_Source
   Type: Amazon S3
   Connection: AWS_S3_DataLake
   Path: /raw-data/
```

### 3.2 Target Connections

```
1. Salesforce_Target
   Type: Salesforce
   Connection: SFDC_Production
   Operations: Create, Update, Upsert, Delete
   Objects: Account, Contact, Custom Objects
   
2. Snowflake_Target
   Type: Snowflake
   Connection: Snowflake_DW
   Operations: Insert, Update, Delete
   Database: ANALYTICS_DB
   
3. S3_Archival
   Type: Amazon S3
   Connection: AWS_S3_DataLake
   Path: /processed-data/
   Format: Parquet with Snappy compression
```

---

## Step 4: Testing Connections

### 4.1 Validate All Connections

```
IICS Console:
1. Navigate to: Connections
2. For each connection, click "Test"
3. Verify "Test Successful" message
4. Check connection latency and response time

Expected Results:
- Oracle DB: Response < 500ms
- Salesforce: Response < 1s
- AWS S3: Response < 2s
- Snowflake: Response < 1s
```

---

## Step 5: Create Reference Data

### 5.1 Setup Lookup Tables

```
Create in Snowflake:

1. POLICY_TYPE_LOOKUP
   - Policy Type ID
   - Policy Type Name
   - Coverage Type
   - Base Premium
   - Discounts

2. CUSTOMER_SEGMENT_LOOKUP
   - Segment ID
   - Segment Name
   - Risk Profile
   - Premium Adjustment

3. CLAIM_TYPE_LOOKUP
   - Claim Type ID
   - Claim Type Name
   - Processing Rules
   - SLA (hours)
```

---

## Step 6: Monitoring Setup

### 6.1 Enable Monitoring

```
IICS Administration:
1. Monitoring → Task Monitoring
   - Enable: Yes
   - Log Level: Info
   - Retention: 30 days

2. Performance Metrics
   - Track: Record count, Processing time
   - Resolution: 1 minute
   - Retention: 90 days

3. Error Notifications
   - Email alerts on task failure
   - Include: Error details, Log file link
   - Send to: operations@zurich.com
```

---

## Step 7: Security Setup

### 7.1 Configure SSL Certificates

```
1. Download CA certificates for all cloud services
2. Place in secure location: /etc/ssl/informatica/
3. Configure in Secure Agent: infagent.ini
4. Validate certificate chain
5. Test encrypted connections
```

### 7.2 Setup API Keys

```
IICS Console:
1. Administration → API Management
2. Generate new API key
3. Set expiration: 90 days
4. Note down: API Key ID and Secret
5. Store securely in credential vault
6. Rotate every 90 days
```

---

## Step 8: Deployment Checklist

### Pre-Deployment

- [ ] All connections tested and validated
- [ ] Secure Agent running and healthy
- [ ] API keys generated and stored
- [ ] SSL certificates installed
- [ ] Database users created with proper permissions
- [ ] Backup and recovery procedures documented
- [ ] Monitoring and alerting configured
- [ ] Security audit completed

### Post-Deployment

- [ ] Run smoke tests
- [ ] Verify data flow end-to-end
- [ ] Check monitoring dashboard
- [ ] Review error logs
- [ ] Validate security settings
- [ ] Document any issues
- [ ] Schedule follow-up review

---

## Troubleshooting

### Common Issues

#### Connection Timeouts
```
Solution:
1. Check network connectivity
2. Verify firewall rules
3. Increase connection timeout
4. Check cloud agent status
5. Review security group settings
```

#### Authentication Failures
```
Solution:
1. Verify credentials
2. Check password expiry
3. Validate security token (Salesforce)
4. Review IAM permissions (AWS)
5. Check certificate validity
```

#### Data Load Failures
```
Solution:
1. Validate source data format
2. Check target table permissions
3. Review data type mappings
4. Check disk space on target
5. Review error logs in IICS
```

---

## Next Steps

1. ✅ Complete all setup steps
2. ✅ Validate all connections
3. ✅ Create test mappings
4. ✅ Run validation tests
5. ✅ Review documentation
6. ✅ Practice with sample data
7. ✅ Proceed to advanced mappings

For detailed mapping examples, see `../mappings/` directory.

---

**Status**: Setup Complete  
**Last Updated**: May 2026
