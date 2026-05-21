# IICS Best Practices & Troubleshooting Guide

## 1. Development Best Practices

### Mapping Design

**Single Responsibility Principle**
- Extract-only mapping
- Transform-only mapping
- Load-only mapping
- Easier to test and debug
- Reusable components

**Error Handling**
- Add input validation
- Implement try-catch blocks
- Log errors with context
- Quarantine failed records
- Send notifications
- Continue processing (don't fail on first error)

**Performance Optimization**
- Use lookups with caching
- Implement parallel processing
- Use batch operations
- Avoid large memory transformations
- Use connection pooling
- Index critical columns
- Partition large datasets

### Naming Conventions

```
Mappings: M_[SOURCE]_[TARGET]_[ACTION]
Example: M_ORACLE_SALESFORCE_POLICY_LOAD

Workflows: W_[PROCESS]_[FREQUENCY]_[ACTION]
Example: W_DAILY_BATCH_POLICY_SYNC

Connections: [SYSTEM]_[ENVIRONMENT]
Example: ORACLE_PROD, SALESFORCE_DEV
```

## 2. Common Issues & Solutions

### Connection Issues

**Connection Timeout**
```
Symptoms:
- Task fails with timeout error
- Intermittent failures
- More frequent during peak hours

Root Causes:
1. Database is slow
2. Network latency too high
3. Connection pool exhausted
4. Firewall blocking

Solutions:
1. Check database performance
2. Increase timeout setting (300s → 600s)
3. Increase connection pool (20 → 30-50)
4. Verify network connectivity
5. Check firewall rules
```

**Authentication Failed**
```
Symptoms:
- "Invalid username or password" error
- Connection test fails
- Suddenly stops working

Root Causes:
1. Password expired
2. Credential vault corrupted
3. User account locked
4. Credential not encrypted properly

Solutions:
1. Verify credentials in target system
2. Update connection with new credentials
3. Check credential vault
4. Unlock user account (if locked)
5. Restart Secure Agent
```

### Data Quality Issues

**Unexpected Null Values**
```
Root Causes:
1. Source data has nulls
2. Join condition not met
3. Lookup failed silently
4. Mapping error in transformation

Solutions:
1. Query source system
2. Check join conditions
3. Review mapping logic
4. Add default value in mapping
5. Add data quality check
```

**Duplicate Records**
```
Root Causes:
1. Workflow executed multiple times
2. Mapping logic creates duplicates
3. Upsert logic not working
4. Join creating cartesian product

Solutions:
1. Fix upsert logic
2. Specify correct key for upsert
3. De-duplicate target
4. Prevent duplicate execution
5. Add execution lock mechanism
```

### Performance Issues

**Slow Mapping Execution**
```
Root Causes:
1. Data volume increased
2. Database statistics outdated
3. Index fragmentation
4. Poor query execution plan
5. Connection pool exhausted
6. Memory pressure

Solutions:
1. Update table statistics
2. Rebuild fragmented indexes
3. Archive old data
4. Implement partitioning
5. Optimize queries
6. Increase JVM memory
7. Reduce batch size
```

**High Memory Usage**
```
Root Causes:
1. Processing large result set in memory
2. Memory leak in mapping
3. Large lookup table not cached correctly
4. Batch size too large

Solutions:
1. Increase JVM memory
2. Reduce batch size (50K → 10K)
3. Implement streaming
4. Optimize caching
5. Monitor memory usage
```

## 3. Monitoring & Diagnostics

### Useful Queries

**Policy Data Validation**
```sql
-- Count records by status
SELECT STATUS, COUNT(*) FROM POLICY GROUP BY STATUS;

-- Find policies with invalid dates
SELECT * FROM POLICY WHERE START_DATE > END_DATE;

-- Find null critical fields
SELECT * FROM POLICY WHERE POLICY_ID IS NULL;
```

**Workflow Execution Analysis**
```sql
-- Count executions by status
SELECT STATUS, COUNT(*) FROM WORKFLOW_EXECUTION GROUP BY STATUS;

-- Find slow executions
SELECT * FROM WORKFLOW_EXECUTION 
WHERE EXECUTION_TIME > 1800 
ORDER BY EXECUTION_TIME DESC;
```

### Log Analysis

**Where to Look**:
1. IICS Execution Logs (most detailed)
2. Secure Agent Logs (/opt/informatica/agent/logs/)
3. Application Logs (/var/log/informatica/)
4. Database Logs (varies by database)

**What to Look For**:
- ERROR keywords
- WARNING keywords
- Stack traces
- FAILED status
- TimeoutException
- Connection refused
- Authentication failed

## 4. Data Reconciliation

**Procedure**:

1. **Count Records**
   - Source Count vs Target Count
   - If different: Check ERROR_QUARANTINE

2. **Field Validation**
   - Compare distinct values
   - Identify missing fields

3. **Quality Check**
   - Check for null values
   - Check for invalid formats
   - Check for zero-length strings

4. **Sample Validation**
   - Compare sample records field-by-field
   - Verify transformation results

## 5. Quick Reference

### Common Error Messages

| Error | Cause | Solution |
|-------|-------|----------|
| Connection Timeout | DB slow | Check DB perf/increase timeout |
| Invalid Credentials | Wrong password | Update credentials |
| Out of Memory | Too much data | Reduce batch size |
| Duplicate Key | Upsert error | Fix key specification |
| No Matching Records | Join failed | Verify join condition |
| Permission Denied | Insufficient privileges | Grant privileges |
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
Time: Premium support 24/7
```

**Last Updated**: May 2026