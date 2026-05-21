# Advanced IICS Mappings - Real-World Implementations

## Mapping 1: M_POLICY_EXTRACT_TRANSFORM_LOAD

### Purpose
Extract policy data from Oracle, apply complex transformations, and load to Salesforce and Snowflake.

### Source Extraction
**Source Table**: POLICY (Oracle DB)
- Filter: Active and Inactive policies only
- Modified: Last 24 hours
- Expected: ~150,000 records daily

### Transformation Logic

#### Premium Calculation
```
IF policy_type = 'Comprehensive' THEN
  final_premium = base_premium * 0.95  (5% discount)
ELSE IF policy_type = 'Third Party' THEN
  final_premium = base_premium * 0.90  (10% discount)
ELSE
  final_premium = base_premium
END IF
```

#### Date Formatting
- Input: DD-MON-YY
- Output: YYYY-MM-DD
- Example: 21-MAY-24 → 2024-05-21

#### Status Mapping
| Source | Target |
|--------|--------|
| Active | Active |
| Inactive | Suspended |
| Expired | Closed |
| Pending | Pending Approval |

#### Data Enrichment
- Lookup customer master data
- Add customer name
- Add branch information
- Add customer segment

### Validation Rules
1. POLICY_ID: NOT NULL
2. CUSTOMER_ID: EXISTS in CUSTOMER table
3. PREMIUM_AMOUNT: > 0
4. START_DATE < END_DATE
5. STATUS: IN valid list

**On Failure**:
- Write to ERROR_QUARANTINE table
- Email data team
- Continue processing remaining records

### Target Loads

**Target 1: Salesforce Opportunities**
- Operation: Upsert (external key: Policy_ID)
- Updates: Status, Amount, Dates
- Expected: 150,000 records processed

**Target 2: Snowflake POLICY_DIM**
- Operation: Insert/Update (key: POLICY_ID)
- Includes: All policy attributes + metadata
- Timestamp: Creation and update dates

**Target 3: AWS S3 Backup**
- Format: Parquet with Snappy compression
- Partition: By POLICY_TYPE and effective_date
- Path: s3://zurich-insurance-datalake/processed-data/

### Error Handling

**Invalid Customer ID**
- Action: Quarantine record
- Notification: Email to data team
- Resolution: Fix customer data or skip

**Calculation Error**
- Action: Log error details
- Fallback: Use original premium
- Notification: Alert for manual review

**Target Load Failure**
- Action: Retry 3 times (exponential backoff)
- If fails: Quarantine and notify

### Performance Targets
- Throughput: 5,000 records/minute
- Processing time: < 30 minutes (150K records)
- Error rate: < 1%
- Data loss: 0%

---

## Mapping 2: M_CUSTOMER_CONSOLIDATION_MDM

### Purpose
Consolidate customer data from multiple sources with deduplication.

### Multiple Source Inputs

**Source 1: Salesforce Accounts**
- Account_ID, Company_Name, Email, Phone, Country

**Source 2: Oracle Customers**
- CUSTOMER_ID, CUST_NAME, CUST_EMAIL, CUST_PHONE

**Source 3: Legacy ERP**
- CustomerCode, CompanyName, ContactEmail, ContactPhone

### Matching Rules (Priority Order)

1. **Exact Match** (100% confidence)
   - Account_ID = CUSTOMER_ID = CustomerCode

2. **Email Match** (95% confidence)
   - Email exact match + Company fuzzy > 90%

3. **Phone Match** (90% confidence)
   - Phone exact match + Country match

4. **Company + Country** (80% confidence)
   - Company similarity > 85% + Country match

5. **Manual Review** (< 80% confidence)
   - Flag for manual consolidation

### Deduplication

**Master Record Selection**:
1. Salesforce (Primary)
2. Oracle (Secondary)
3. Legacy ERP (Tertiary)

**Merge Process**:
- Combine non-null fields from all sources
- Keep most recent data
- Preserve source record IDs
- Create audit trail

### Standardization

**Name**: Trim, proper case, remove special chars
**Email**: Lowercase, validate format, remove duplicates
**Phone**: Extract digits, add country code, format
**Address**: Parse, validate, geocode

### Target: MDM Hub (Snowflake)

**Table**: CUSTOMER_GOLDEN_RECORD
- customer_key (PK)
- customer_number (unique)
- customer_name, email, phone, country
- source_ids (from all systems)
- data_quality_score
- merge tracking fields

---

## Mapping 3: M_PREMIUM_CALCULATION_ENGINE

### Purpose
Complex premium calculation with risk assessment and discounts.

### Calculation Formula

#### Step 1: Base Premium
```
BasePremium = ReferenceTable[vehicle_type, coverage_type]
- Sedan + Comprehensive = $800
- SUV + Comprehensive = $950
- Sports Car + Comprehensive = $1,500
```

#### Step 2: Risk Assessment
```
RiskScore = Σ(RiskFactor × Weight)

Factors:
1. Age (30% weight): 18-25 (+30pts), 26-35 (+5pts)
2. Driving Record (40% weight): Clean (0pts), 1 accident (+10pts)
3. Vehicle (20% weight): Sedan (0pts), Sports car (+20pts)
4. Location (10% weight): Low theft (0pts), High (+15pts)
```

#### Step 3: Discount Application
```
Loyalty: 0-10%
Multi-Policy: 0-10%
Safety Features: 0-10%
Claim-Free: 0-15%
Bundling: 0-15%

Max Total Discount: 40%
```

#### Step 4: Tax Calculation
```
Insurance Tax = Premium × 5%
GST = (Premium + Insurance Tax) × 18%
Final = Premium + Taxes
```

### Example
```
Base Premium:        1000
Risk Surcharge (20%): +200
Loyalty Discount:     -60
Insurance Tax:        +57
GST:                 +215
─────────────────────────
FINAL PREMIUM:       1412

Payment Schedule:
- Annual: 1412
- Quarterly: 353
- Monthly: 117.66
```

### Output

**Target**: PREMIUM_QUOTATION table
- quotation_id, policy_id, customer_id
- base_premium, risk_surcharge, total_discounts
- insurance_tax, gst_amount
- final premium amounts (annual, quarterly, monthly)
- effective_date, expiry_date

---

## Performance Optimization

### Techniques
- Batch size: 10,000 records
- Parallel threads: 4-8
- Connection pool: 20-50
- Lookup caching: Enabled
- Query optimization: Indexed columns

### Testing
- Unit test: Individual transformations
- Integration test: End-to-end flow
- Performance test: Load testing with production volume
- UAT: Business user validation

**Last Updated**: May 2026