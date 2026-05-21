# IICS Setup Guide - Zurich Insurance Project

## Prerequisites

### Required Software & Accounts
- Informatica IICS Cloud Account (Admin access)
- Salesforce Developer Account
- Oracle Database or Oracle Cloud Account
- AWS Account (for S3 access)
- Snowflake Account
- Git and GitHub
- Text Editor (VS Code preferred)

### System Requirements
- Operating System: Windows/Mac/Linux
- Java: Version 8 or higher
- RAM: Minimum 8GB
- Network: VPN access to source systems

## Step 1: IICS Cloud Organization Setup

### Create Cloud Organization
1. Login to Informatica Cloud: https://www.informaticacloud.com
2. Navigate to Administration → Shared Objects
3. Create new Organization: "Zurich_Insurance_PROD"
4. Enable modules:
   - Data Integration
   - Data Quality
   - Master Data Management
   - API Management

### Configure User Roles

**Role Definitions**:
1. **IICS_Admin**: Full access to all objects
2. **IICS_Developer**: Create/Edit mappings and workflows
3. **IICS_Viewer**: Read-only access
4. **IICS_Operator**: Execute tasks and monitor

### Setup Security
1. Enable Multi-Factor Authentication (MFA)
2. Set password policy:
   - Minimum: 12 characters
   - Complexity: Required
   - Rotation: Every 90 days
3. Configure IP Whitelisting
4. Enable API security with key rotation

## Step 2: Secure Agent Setup

### Download & Install
1. Download Secure Agent from IICS console
2. Extract to installation directory
3. Configure infagent.ini with IICS credentials
4. Start Secure Agent service
5. Verify status in IICS console (should show "Running")

### Agent Configuration
```ini
[AGENT]
AgentName=ZURICH_AGENT_01
LISTENHOST=localhost
LISTENPORT=6005
LOGDIR=/var/log/informatica/agent
CLOUDURL=https://app.informaticacloud.com
CLOUDORG=Zurich_Insurance_PROD
```

## Step 3: Cloud Connectors Configuration

### Salesforce Connector
1. Create Connected App in Salesforce
2. Configure OAuth settings
3. Get Consumer Key & Secret
4. Create IICS connection: SFDC_Production
5. Test connection

### Oracle Database Connector
1. Create INFORMATICA user in Oracle
2. Grant required privileges
3. Create IICS connection: OracleDB_Production
4. Configure connection pool (min: 5, max: 50)
5. Test connectivity

### AWS S3 Connector
1. Create IAM user with S3 permissions
2. Get Access Key ID & Secret Access Key
3. Create IICS connection: AWS_S3_DataLake
4. Configure bucket: zurich-insurance-datalake
5. Verify encryption settings

### Snowflake Connector
1. Create INFORMATICA user in Snowflake
2. Grant warehouse and database access
3. Create IICS connection: Snowflake_DW
4. Configure warehouse: COMPUTE_WH
5. Test connection

## Step 4: Create Source & Target Connections

### Source Connections
- Oracle_Policy_Source (POLICY, POLICY_DETAILS tables)
- Oracle_Claims_Source (CLAIMS, CLAIM_DETAILS tables)
- SFDC_Accounts_Source (Account, Contact objects)
- AWS_S3_Source (/raw-data/ path)

### Target Connections
- Salesforce_Target (Opportunities, Accounts)
- Snowflake_Target (Analytics database)
- S3_Archival (/processed-data/ path)

## Step 5: Reference Data Setup

### Lookup Tables (Create in Snowflake)
1. **POLICY_TYPE_LOOKUP**
   - Policy Type ID, Name, Coverage Type, Base Premium

2. **CUSTOMER_SEGMENT_LOOKUP**
   - Segment ID, Name, Risk Profile, Premium Adjustment

3. **CLAIM_TYPE_LOOKUP**
   - Claim Type ID, Name, Processing Rules, SLA

## Step 6: Monitoring & Logging Setup

### Enable Monitoring
1. Navigate to: Monitoring → Task Monitoring
2. Enable task-level logging
3. Set log retention: 30 days
4. Configure performance metrics
5. Setup error notifications

### Configure Alerting
1. Email alerts for task failures
2. Slack notifications for warnings
3. Dashboard alerts for SLA violations
4. Escalation rules for critical issues

## Step 7: Testing

### Unit Testing
- Test individual mappings with sample data
- Verify transformation logic
- Validate data quality checks

### Integration Testing
- End-to-end workflow execution
- Verify data consistency across targets
- Test error recovery procedures

### UAT Testing
- Business user validation
- Data accuracy verification
- Performance acceptance testing

## Pre-Deployment Checklist

- [ ] All connections tested successfully
- [ ] Secure Agent running and healthy
- [ ] API keys generated and secured
- [ ] SSL certificates installed
- [ ] Database users created with permissions
- [ ] Backup procedures documented
- [ ] Monitoring configured
- [ ] Security audit completed

## Next Steps

1. Complete all setup steps
2. Validate all connections
3. Create test mappings
4. Run validation tests
5. Review documentation
6. Practice with sample data
7. Proceed to advanced mappings

**For detailed mapping examples, see**: mappings/ADVANCED_MAPPING_EXAMPLES.md

**Last Updated**: May 2026