# IICS Best Practices & Troubleshooting Guide

## 1. IICS Development Best Practices

### 1.1 Mapping Design Best Practices

**Single Responsibility Principle**
```
GOOD:
  Mapping: M_POLICY_EXTRACT
  - Purpose: Extract policy data only
  - Input: Oracle POLICY table
  - Output: Staged policy data
  
  Mapping: M_POLICY_TRANSFORM
  - Purpose: Apply transformations only
  - Input: Staged policy data
  - Output: Transformed policy data
  
  Mapping: M_POLICY_LOAD
  - Purpose: Load to targets only
  - Input: Transformed policy data
  - Output: Loaded to Salesforce/Snowflake

BAD:
  Mapping: M_POLICY_ALL_IN_ONE
  - Extract + Transform + Load in single mapping
  - Difficult to test and debug
  - Hard to reuse components
```

**Error Handling**
```
GOOD:
  1. Add input validation
  2. Implement try-catch blocks
  3. Log errors with context
  4. Quarantine failed records
  5. Send notifications
  6. Continue processing (don't fail on first error)

BAD:
  1. No validation
  2. Let errors bubble up and fail mapping
  3. Generic error messages
  4. Lose failed records
  5. No notification
```

**Performance Optimization**
```
Techniques:
  1. Use lookups with caching
  2. Implement parallel processing
  3. Use batch operations (INSERT all vs individual)
  4. Avoid large memory transformations
  5. Use connection pooling
  6. Index critical columns
  7. Partition large data sets

Example - Good:
  Batch Size: 10,000
  Parallel Threads: 4
  Cache Lookups: Enabled
  Processing Time: 15 minutes for 1M records

Example - Bad:
  Batch Size: 1
  Parallel Threads: 1
  Cache Lookups: Disabled
  Processing Time: 4 hours for 1M records
```

### 1.2 Workflow Design Best Practices

**Clear Workflow Structure**
```
GOOD Workflow Structure:

Start Task
    ↓
Pre-flight Checks (Database connectivity, Agent status)
    ↓
Run Core Mapping (Extract/Transform/Load)
    ↓
Error Handling (Capture failures, retry logic)
    ↓
Validation (Verify data loaded correctly)
    ↓
Notification (Send email/alerts)
    ↓
End Task

Simple, Linear, Easy to follow
```

**Error Handling in Workflows**
```
Best Practice:
  1. Use conditional logic for error paths
  2. Implement retry logic with backoff
  3. Log all errors with timestamps
  4. Quarantine failed records
  5. Notify on critical failures
  6. Continue non-critical failures (don't stop workflow)

Example:
  IF mapping_failed THEN
    Retry 2 times with 5-min interval
    IF still_failed THEN
      Log error
      Quarantine records
      Send email alert
      Continue to next mapping
    ENDIF
  ENDIF
```

### 1.3 Naming Conventions

```
Mappings:
  M_[SOURCE]_[TARGET]_[ACTION]
  Examples:
    M_ORACLE_SALESFORCE_POLICY_LOAD
    M_CUSTOMER_CONSOLIDATION
    M_PREMIUM_CALCULATION

Workflows:
  W_[PROCESS]_[FREQUENCY]_[ACTION]
  Examples:
    W_DAILY_BATCH_POLICY_SYNC
    W_REALTIME_CLAIMS_PROCESSING
    W_MONTHLY_COMPLIANCE_REPORT

Connections:
  [SYSTEM]_[ENVIRONMENT]
  Examples:
    ORACLE_PROD
    SALESFORCE_DEV
    SNOWFLAKE_STAGING

Folders/Projects:
  [BUSINESS_AREA]_[PURPOSE]
  Examples:
    POLICY_MANAGEMENT
    CUSTOMER_MASTER_DATA
    CLAIMS_PROCESSING
```

---

## 2. Troubleshooting Guide

### 2.1 Common Connection Issues

**Problem: Connection Timeout**
```
Symptoms:
  - Task fails with "Connection Timeout" error
  - Occurs intermittently
  - More frequent during peak hours

Root Causes:
  1. Database is slow/unresponsive
  2. Network latency too high
  3. Connection pool exhausted
  4. Firewall blocking connection

Solutions:
  1. Check database performance
     SELECT * FROM v$session;  -- Check active sessions
     SELECT * FROM v$sql;       -- Check running queries
  
  2. Increase timeout setting
     Current: 300 seconds → Try: 600 seconds
  
  3. Increase connection pool size
     Current: 20 → Try: 30-50
  
  4. Verify network connectivity
     ping -c 3 database_host
     telnet database_host 1521
  
  5. Check firewall rules
     Verify port 1521 (Oracle) is open
     Verify source IP is whitelisted

Testing:
  1. Manually connect to database from Secure Agent server
  2. Run connection test in IICS UI
  3. Monitor response time
  4. Verify no network packet loss
```

**Problem: Authentication Failed**
```
Symptoms:
  - "Invalid username or password" error
  - Connection test fails
  - Suddenly stops working (was working before)

Root Causes:
  1. Password expired
  2. Credential vault corrupted
  3. User account locked
  4. Credential not properly encrypted

Solutions:
  1. Verify credentials in target system
     Oracle: SELECT * FROM dba_users WHERE username='INFORMATICA_USER';
     Salesforce: Reset password in Security → Users
  
  2. Update connection with new credentials
     Delete old connection
     Create new connection with fresh credentials
     Test connectivity
  
  3. Check credential vault
     Restart Secure Agent
     Verify credential files are intact
  
  4. Unlock user account (if locked)
     Oracle: ALTER USER informatica_user ACCOUNT UNLOCK;
     Salesforce: Admin → Users → Unlock user

Prevention:
  - Set password expiry 90+ days
  - Monitor user lockouts
  - Have backup account for emergencies
```

### 2.2 Data Quality Issues

**Problem: Unexpected Null Values**
```
Symptoms:
  - Some fields showing NULL in target
  - Was populated in source
  - Intermittent issue

Root Causes:
  1. Source data has nulls
  2. Join condition not met
  3. Lookup failed silently
  4. Mapping error in transformation

Investigation:
  1. Query source system
     SELECT * FROM POLICY WHERE POLICY_ID = 'POL-123';
     Check if field is actually null
  
  2. Check join conditions
     Verify join keys match
     Check for case sensitivity
     Check data types match
  
  3. Review mapping logic
     Check lookup join condition
     Verify error handling on lookup
     Test with sample data
  
  4. Check for special characters
     Some systems may interpret whitespace as null
     Check encoding issues

Solutions:
  1. Add default value in mapping
     IF field IS NULL THEN 'DEFAULT_VALUE'
  
  2. Add data quality check
     Validate non-null before loading
  
  3. Log null occurrences
     Create null_tracking table
     Log every null value found
     Investigate patterns
```

**Problem: Duplicate Records**
```
Symptoms:
  - Same record appears multiple times in target
  - Count in target > count in source
  - Occurs after certain operations

Root Causes:
  1. Workflow executed multiple times
  2. Mapping logic creates duplicates
  3. Upsert logic not working correctly
  4. Join creating cartesian product

Investigation:
  1. Count records
     Source: SELECT COUNT(*) FROM POLICY;
     Target: SELECT COUNT(*) FROM POLICY_DIM;
     Check if target > source
  
  2. Check duplicate key
     SELECT POLICY_ID, COUNT(*) FROM POLICY_DIM 
     GROUP BY POLICY_ID HAVING COUNT(*) > 1;
  
  3. Review workflow execution log
     Check if mapping ran multiple times
     Check for manual re-runs

Solutions:
  1. Fix upsert logic
     Specify correct key for upsert
     Verify key uniqueness
  
  2. De-duplicate target
     DELETE FROM POLICY_DIM WHERE POLICY_ID IN 
     (SELECT POLICY_ID FROM POLICY_DIM 
      GROUP BY POLICY_ID HAVING COUNT(*) > 1);
  
  3. Prevent duplicate execution
     Add execution lock mechanism
     Track last execution timestamp
```

### 2.3 Performance Issues

**Problem: Slow Mapping Execution**
```
Symptoms:
  - Mapping takes 2+ hours (used to take 30 min)
  - Throughput dropped significantly
  - More resources being used

Root Causes:
  1. Data volume increased
  2. Database statistics outdated
  3. Index fragmentation
  4. Poor query execution plan
  5. Connection pool exhausted
  6. Memory pressure

Investigation:
  1. Check data volume
     SELECT COUNT(*) FROM POLICY;
     Compare to last week/month
  
  2. Check query plan
     EXPLAIN PLAN for SELECT ... FROM POLICY;
     Look for full table scans
  
  3. Check index fragmentation
     Oracle: SELECT * FROM index_stats;
  
  4. Monitor resource usage
     CPU usage: top -p [process_id]
     Memory usage: free -h
     Disk I/O: iostat -x 1 5

Solutions:
  1. Update table statistics
     Oracle: ANALYZE TABLE POLICY COMPUTE STATISTICS;
  
  2. Rebuild indexes
     Oracle: ALTER INDEX policy_idx REBUILD;
  
  3. Archive old data
     Move historical data to archive
     Reduce table size
  
  4. Implement partitioning
     Partition by date
     Parallel processing on partitions
  
  5. Optimize query
     Add missing indexes
     Rewrite inefficient joins
     Use hints if necessary
```

**Problem: High Memory Usage**
```
Symptoms:
  - Secure Agent runs out of memory
  - Mapping fails with OutOfMemory error
  - Agent crashes or becomes unresponsive

Root Causes:
  1. Processing large result set in memory
  2. Memory leak in mapping
  3. Large lookup table not cached correctly
  4. Batch size too large

Solutions:
  1. Increase JVM memory
     Edit infagent.ini:
     AGENT_JVM_MEM_MAX = 2048 (increase to 4096)
  
  2. Reduce batch size
     Current: 50,000 → Try: 10,000
  
  3. Implement streaming
     Read source in chunks
     Process and write immediately
     Don't load all in memory
  
  4. Cache optimization
     Cache only necessary lookups
     Limit cache size
     Clear cache between runs
  
  5. Monitor memory
     Enable memory monitoring
     Alert when usage > 80%
     Implement garbage collection tuning
```

### 2.4 Security Issues

**Problem: Credentials Not Secure**
```
Symptoms:
  - Found password in logs
  - Found password in error message
  - Credentials visible in mapping

Investigation:
  1. Check log files
     grep -r "password" /var/log/informatica/
     grep -r "secret" /var/log/informatica/
  
  2. Check configuration files
     grep "password" *.ini *.conf
  
  3. Review error messages
     Check if sensitive data in exception

Solutions:
  1. Use credential vault
     Store passwords in vault
     Reference in mappings
  
  2. Mask sensitive data in logs
     Implement masking rules
     Never log full credentials
  
  3. Update credentials
     Change any exposed passwords
     Rotate API keys
     Enable auditing
```

---

## 3. Monitoring & Troubleshooting Utilities

### 3.1 Useful Queries

**Policy Data Validation**
```sql
-- Count records by status
SELECT STATUS, COUNT(*) FROM POLICY GROUP BY STATUS;

-- Find policies with invalid dates
SELECT * FROM POLICY WHERE START_DATE > END_DATE;

-- Find null critical fields
SELECT * FROM POLICY WHERE POLICY_ID IS NULL OR CUSTOMER_ID IS NULL;

-- Premium range check
SELECT POLICY_TYPE, MIN(PREMIUM), MAX(PREMIUM), AVG(PREMIUM) 
FROM POLICY GROUP BY POLICY_TYPE;
```

**Workflow Execution Analysis**
```sql
-- Count executions by status
SELECT STATUS, COUNT(*) FROM WORKFLOW_EXECUTION GROUP BY STATUS;

-- Find slow executions
SELECT * FROM WORKFLOW_EXECUTION 
WHERE EXECUTION_TIME > 1800 
ORDER BY EXECUTION_TIME DESC;

-- Error analysis
SELECT ERROR_TYPE, COUNT(*) FROM WORKFLOW_ERRORS 
GROUP BY ERROR_TYPE ORDER BY COUNT DESC;
```

### 3.2 Diagnostic Tools

```
IICS Task Monitoring:
  - Real-time execution view
  - Execution history
  - Error details and logs
  - Performance metrics

IICS Logs:
  - Cloud execution logs
  - Application logs
  - System logs
  - Export for analysis

External Tools:
  - Apache JMeter: Performance testing
  - Splunk: Log analysis
  - Prometheus: Metrics monitoring
  - Grafana: Visualization

Debug Mode:
  - Enable debug logging in mapping
  - Capture intermediate data
  - Log all transformations
  - Useful for development/troubleshooting
```

---

## 4. Advanced Troubleshooting

### 4.1 Data Reconciliation

```
Procedure to verify data consistency:

1. Count Records
   Source Count = SELECT COUNT(*) FROM POLICY;
   Target Count = SELECT COUNT(*) FROM POLICY_DIM;
   
   If Different:
     Check ERROR_QUARANTINE table
     Review mapping error logs
     Identify lost records

2. Field Validation
   FOR EACH field:
     Source Distinct = SELECT COUNT(DISTINCT field) FROM POLICY;
     Target Distinct = SELECT COUNT(DISTINCT field) FROM POLICY_DIM;
     Compare counts
   
   If Different:
     Missing values found
     Check transformation logic
     Verify no filtering occurred

3. Data Quality Check
   SELECT * FROM POLICY_DIM WHERE field IS NULL;
   SELECT * FROM POLICY_DIM WHERE field LIKE '%INVALID%';
   SELECT * FROM POLICY_DIM WHERE LENGTH(field) = 0;
   
   Report findings to data team

4. Sample Validation
   SELECT * FROM POLICY WHERE POLICY_ID = 'POL-123';
   SELECT * FROM POLICY_DIM WHERE POLICY_ID = 'POL-123';
   Compare all fields manually
   Verify transformation results
```

### 4.2 Log Analysis

```
Where to Look:

1. IICS Execution Logs
   - Most detailed info about mapping execution
   - Step-by-step transformation
   - Error details and stack traces

2. Secure Agent Logs
   - Agent startup/shutdown info
   - Connection attempts
   - Security events
   - Location: /opt/informatica/agent/logs/

3. Application Logs
   - Business logic errors
   - Data validation errors
   - Transformation errors
   - Location: /var/log/informatica/

4. Database Logs
   - Connection errors
   - Query errors
   - Lock contention
   - Location: Varies by database

What to Look For:
  - ERROR keywords
  - WARNING keywords
  - Stack traces (starting with "at")
  - FAILED status indicators
  - TimeoutException
  - Connection refused
  - Authentication failed
```

---

## 5. Performance Tuning Checklist

```
☐ Database Performance
  ☐ Run ANALYZE TABLE STATISTICS
  ☐ Check missing indexes
  ☐ Rebuild fragmented indexes
  ☐ Archive old data

☐ Mapping Optimization
  ☐ Increase batch size
  ☐ Enable lookup caching
  ☐ Add parallel processing
  ☐ Use incremental loads

☐ Workflow Optimization
  ☐ Run independent tasks in parallel
  ☐ Reduce retry delays
  ☐ Implement conditional skipping
  ☐ Schedule during off-peak hours

☐ Infrastructure
  ☐ Verify connection pool size
  ☐ Check network bandwidth
  ☐ Monitor CPU and memory
  ☐ Verify disk I/O performance

☐ Configuration
  ☐ Increase JVM memory
  ☐ Tune timeout values
  ☐ Adjust logging level (reduce verbosity in prod)
  ☐ Enable query optimization hints
```

---

## 6. Quick Reference

### Common Error Messages

| Error | Cause | Solution |
|-------|-------|----------|
| Connection Timeout | Database slow | Increase timeout / Check DB |
| Invalid Credentials | Wrong password | Update credentials |
| Out of Memory | Too much data | Reduce batch size |
| Duplicate Key | Upsert logic error | Fix key specification |
| No Matching Records | Join failed | Verify join condition |
| Permission Denied | Insufficient privileges | Grant database privileges |
| Network Unreachable | Firewall blocking | Check firewall rules |

### Escalation Contacts

```
Level 1 - Operations Team
  Issue: Workflow execution, monitoring
  Contact: operations@zurich.com
  Time: 24/7

Level 2 - IICS Development Team
  Issue: Mapping errors, transformation logic
  Contact: iics-dev@zurich.com
  Time: Business hours

Level 3 - Infrastructure Team
  Issue: Server, database, network
  Contact: infrastructure@zurich.com
  Time: 24/7

Level 4 - Informatica Support
  Issue: Platform bug, feature request
  Contact: Informatica Cloud Support
  Time: Business hours (premium support 24/7)
```

