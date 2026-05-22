# Real-Time IICS CDI & CAI Practice Project - Zurich Insurance

## 🎯 Project Objective

This is a **comprehensive real-time IICS project** designed to help you understand and practice:

- ✅ **Cloud Data Integration (CDI)** - Real-time data synchronization
- ✅ **Cloud Application Integration (CAI)** - Real-time API-based integrations
- ✅ **File Watching & Event Triggers** - Real-time file monitoring
- ✅ **Mapping Tasks & Taskflows** - Complex data transformation orchestration
- ✅ **Error Handling & Recovery** - Automated retry mechanisms
- ✅ **Workflow Automation** - Scheduling and automated triggers

---

## 📋 Real-Time Use Cases

### 1. **Policy Updates - Real-Time Sync (CDI)**
- Monitor policy changes in Oracle DB
- Real-time sync to Salesforce CRM
- Immediate notification to customers
- Error handling with automatic retry

### 2. **Claims Submission - Real-Time Processing (CAI)**
- REST API receives claim submission
- Real-time validation and enrichment
- Immediate approval/escalation decision
- Instant notification to customer

### 3. **File-Based Policy Upload (CDI + File Watch)**
- Monitor S3/SFTP for policy files
- File arrives → Auto-trigger mapping
- If file doesn't arrive by schedule → Alert & retry
- Parse and load policies in real-time

### 4. **Premium Calculation - Real-Time API (CAI)**
- REST API receives premium calculation request
- Real-time calculation with business rules
- Response with premium amount
- Error handling with fallback

---

## 🏗️ Architecture - Real-Time Flow

```
┌─────────────────────────────────────────────────────────────┐
│           REAL-TIME DATA SOURCES                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Oracle DB    │  │ REST APIs    │  │ File Watch   │      │
│  │ (CDC)        │  │ (Requests)   │  │ (SFTP/S3)    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                         ↓ (Event-based)
┌─────────────────────────────────────────────────────────────┐
│     INFORMATICA INTELLIGENT CLOUD SERVICES                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │     CDI (Cloud Data Integration)                     │   │
│  │  ┌─────────────────────────────────────────────────┐ │   │
│  │  │ Mapping Tasks → Taskflow → Real-Time Sync       │ │   │
│  │  │ • File Watch (Event triggered)                  │ │   │
│  │  │ • CDC (Change Data Capture)                     │ │   │
│  │  │ • Error Handling & Auto-Retry                   │ │   │
│  │  │ • Workflow Scheduling                           │ │   │
│  │  └─────────────────────────────────────────────────┘ │   │
│  ├──────────────────────────────────────────────────────┤   │
│  │     CAI (Cloud Application Integration)             │   │
│  │  ┌─────────────────────────────────────────────────┐ │   │
│  │  │ REST APIs → Mapping Tasks → Response            │ │   │
│  │  │ • Real-time request processing                  │ │   │
│  │  │ • Synchronous response                          │ │   │
│  │  │ • Error handling with HTTP status codes         │ │   │
│  │  │ • Automated retries on failure                  │ │   │
│  │  └─────────────────────────────────────────────────┘ │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                         ↓ (Real-time)
┌─────────────────────────────────────────────────────────────┐
│        REAL-TIME TARGETS                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Salesforce   │  │ Oracle Cloud │  │ Notification│      │
│  │ (CRM)        │  │ (Data Lake)  │  │ Service     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

---

## 📁 Project Structure

```
realtime-iics-cdi-cai-project/
│
├── README.md                                    # This file
│
├── docs/
│   ├── 01_CDI_CONCEPTS.md                      # CDI fundamentals
│   ├── 02_CAI_CONCEPTS.md                      # CAI fundamentals
│   ├── 03_FILE_WATCH_SETUP.md                  # File watching configuration
│   ├── 04_MAPPING_TASKS.md                     # Mapping task creation
│   ├── 05_TASKFLOWS.md                         # Taskflow orchestration
│   ├── 06_ERROR_HANDLING.md                    # Error handling patterns
│   ├── 07_WORKFLOW_SCHEDULING.md               # Workflow scheduling
│   └── 08_BEST_PRACTICES.md                    # Real-time best practices
│
├── sample_data/
│   ├── source_data/
│   │   ├── policy_updates.csv                  # Sample policy data
│   │   ├── claims_submissions.json             # Sample claims data
│   │   ├── premium_requests.json               # Sample API requests
│   │   └── README.md                           # Data format guide
│   │
│   └── api_payloads/
│       ├── claim_submission_payload.json
│       ├── premium_calculation_payload.json
│       ├── policy_update_payload.json
│       └── README.md
│
├── mappings/
│   ├── CDI_Mappings/
│   │   ├── M_CDC_POLICY_REALTIME.md            # CDC policy mapping
│   │   ├── M_FILE_WATCH_POLICY_UPLOAD.md       # File watch mapping
│   │   ├── M_CUSTOMER_REALTIME_SYNC.md         # Customer real-time sync
│   │   └── M_CLAIMS_REALTIME_LOAD.md           # Claims real-time load
│   │
│   └── CAI_Mappings/
│       ├── API_POLICY_UPDATE.md                # Policy update API mapping
│       ├── API_CLAIM_SUBMISSION.md             # Claim submission API mapping
│       ├── API_PREMIUM_CALCULATION.md          # Premium calc API mapping
│       └── API_ERROR_RESPONSE.md               # API error response mapping
│
├── taskflows/
│   ├── TASKFLOW_POLICY_REALTIME_SYNC.md        # Policy real-time taskflow
│   ├── TASKFLOW_CLAIM_PROCESSING.md            # Claim processing taskflow
│   ├── TASKFLOW_FILE_WATCH_ERROR_HANDLING.md   # File watch error handling
│   └── TASKFLOW_SCHEDULED_RETRY.md             # Scheduled retry taskflow
│
├── workflows/
│   ├── WF_REALTIME_CDI_POLICY_SYNC.md          # CDI policy workflow
│   ├── WF_REALTIME_CAI_CLAIMS_API.md           # CAI claims API workflow
│   ├── WF_FILE_WATCH_WITH_FALLBACK.md          # File watch with fallback
│   ├── WF_ERROR_RECOVERY_AUTOMATION.md         # Error recovery automation
│   └── WF_SCHEDULED_TRIGGER_ON_FAILURE.md      # Scheduled trigger on failure
│
├── error_handling/
│   ├── ERROR_PATTERNS.md                       # Common error patterns
│   ├── RETRY_STRATEGIES.md                     # Retry mechanisms
│   ├── RECOVERY_PROCEDURES.md                  # Recovery procedures
│   ├── AUTOMATION_TRIGGERS.md                  # Automated trigger setup
│   └── MONITORING_ALERTS.md                    # Monitoring & alerts
│
├── configuration/
│   ├── SCHEDULER_CONFIG.md                     # Scheduler configuration
│   ├── FILE_WATCH_CONFIG.md                    # File watch setup
│   ├── CDC_CONFIG.md                           # CDC configuration
│   ├── RETRY_CONFIG.md                         # Retry policy configuration
│   └── NOTIFICATION_CONFIG.md                  # Notification setup
│
└── interview_prep/
    ├── COMMON_QUESTIONS.md                     # Common CDI/CAI interview Q&A
    ├── REAL_TIME_SCENARIOS.md                  # Real-time interview scenarios
    ├── TROUBLESHOOTING_GUIDE.md                # Troubleshooting common issues
    └── CASE_STUDIES.md                         # Real-world case studies
```

---

## 🚀 Quick Start Guide

### Phase 1: Understand CDI & CAI
1. Read: `docs/01_CDI_CONCEPTS.md`
2. Read: `docs/02_CAI_CONCEPTS.md`
3. Understand the differences and use cases

### Phase 2: Learn Real-Time Concepts
1. Read: `docs/03_FILE_WATCH_SETUP.md`
2. Read: `docs/04_MAPPING_TASKS.md`
3. Read: `docs/05_TASKFLOWS.md`

### Phase 3: Explore Sample Mappings
1. Study: `mappings/CDI_Mappings/M_CDC_POLICY_REALTIME.md`
2. Study: `mappings/CAI_Mappings/API_CLAIM_SUBMISSION.md`
3. Review sample data in `sample_data/`

### Phase 4: Understand Error Handling
1. Read: `docs/06_ERROR_HANDLING.md`
2. Study: `error_handling/RETRY_STRATEGIES.md`
3. Learn: `error_handling/AUTOMATION_TRIGGERS.md`

### Phase 5: Workflow Automation
1. Read: `docs/07_WORKFLOW_SCHEDULING.md`
2. Study: `workflows/WF_FILE_WATCH_WITH_FALLBACK.md`
3. Learn: `workflows/WF_ERROR_RECOVERY_AUTOMATION.md`

### Phase 6: Interview Preparation
1. Review: `interview_prep/COMMON_QUESTIONS.md`
2. Study: `interview_prep/REAL_TIME_SCENARIOS.md`
3. Practice: `interview_prep/CASE_STUDIES.md`

---

## 🎓 Learning Outcomes

After completing this project, you will understand:

### CDI (Cloud Data Integration)
- ✅ Real-time data synchronization concepts
- ✅ Change Data Capture (CDC) patterns
- ✅ File watching and event-based triggers
- ✅ Incremental data loading
- ✅ Real-time performance optimization

### CAI (Cloud Application Integration)
- ✅ REST API-based real-time integration
- ✅ Synchronous request-response processing
- ✅ API payload transformation
- ✅ HTTP status code handling
- ✅ Real-time error responses

### Advanced Concepts
- ✅ Mapping tasks and their configuration
- ✅ Taskflow orchestration and execution
- ✅ Error handling and recovery strategies
- ✅ Automatic retry mechanisms
- ✅ Scheduled triggers on data task failure
- ✅ File arrival monitoring and fallback actions
- ✅ Real-time workflow automation

### Interview Ready
- ✅ Real-time scenario handling
- ✅ Performance troubleshooting
- ✅ Error recovery procedures
- ✅ Best practices and design patterns
- ✅ Real-world use case solutions

---

## 🔧 Technologies Covered

- **Informatica Cloud** (IICS)
  - Cloud Data Integration (CDI)
  - Cloud Application Integration (CAI)
  - Cloud Secure Agent
  - REST APIs & Webhooks

- **Data Sources**
  - Oracle Database (CDC)
  - Salesforce (Real-time sync)
  - REST APIs (Request/Response)
  - File Systems (File watch)
  - AWS S3 (Event-based)

- **Concepts**
  - Event-driven architecture
  - Real-time data synchronization
  - Error handling & recovery
  - Workflow automation
  - Performance optimization

---

## 📊 Sample Use Cases

### Use Case 1: Real-Time Policy Updates (CDI)
```
Oracle DB Policy Change 
    ↓ (CDC detects change)
IICS Mapping Task (Transform)
    ↓ (Real-time processing)
Taskflow (Validation & Load)
    ↓
Salesforce Opportunity (Updated)
Customer Notification (Instant)
```

### Use Case 2: Claims Submission API (CAI)
```
Customer REST API Call
    ↓
IICS Receives Request
    ↓
Mapping Task (Validate & Enrich)
    ↓
Taskflow (Apply Rules & Load)
    ↓
Immediate API Response (Claim ID + Status)
    ↓
Auto-Notification to Customer
```

### Use Case 3: File Watch with Error Handling
```
Policy File Expected at 2 AM
    ↓
File Arrives → Auto-Trigger Taskflow
    ↓ OR
File NOT Arrived by 2:30 AM → Trigger Alert Workflow
    ↓
If Taskflow Fails → Auto-Retry (Max 3 times)
    ↓
If Still Fails → Escalate & Notify Admin
```

---

## 💡 Key Concepts You'll Master

1. **Mapping Tasks** - Transform data in real-time
2. **Taskflows** - Orchestrate multiple mapping tasks
3. **File Watch** - Monitor directories for file arrivals
4. **CDC (Change Data Capture)** - Capture real-time database changes
5. **Event Triggers** - Automatic workflow initiation
6. **Error Handling** - Graceful failure management
7. **Auto-Retry** - Automatic failure recovery
8. **Scheduled Fallback** - Actions when data doesn't arrive
9. **Real-Time Notifications** - Instant alerts and updates
10. **Performance Optimization** - Real-time throughput tuning

---

## 📈 Career Benefits

This project will help you:
- ✅ Prepare for **IICS CDI/CAI job interviews**
- ✅ Understand **real-time integration patterns**
- ✅ Master **error handling in production**
- ✅ Learn **workflow automation**
- ✅ Gain **hands-on experience** with real-world scenarios

---

## 🎯 Next Steps

1. **Start with Phase 1**: Read CDI & CAI concepts
2. **Study the sample data**: Understand the data formats
3. **Review sample mappings**: See how real-time transforms work
4. **Practice with taskflows**: Create and test workflows
5. **Work through error scenarios**: Learn recovery mechanisms
6. **Interview prep**: Review common questions and scenarios

---

## 📚 Documentation Guide

Each document provides:
- **Concepts**: Theory and fundamentals
- **Step-by-Step**: Practical implementation
- **Code Examples**: Real mapping and taskflow code
- **Best Practices**: Production-ready patterns
- **Interview Tips**: Common questions and answers

---

## 🔗 Important Links

- [Informatica Documentation](https://docs.informatica.com/)
- [IICS CDI Guide](https://docs.informatica.com/integration-cloud/)
- [IICS CAI Guide](https://docs.informatica.com/integration-cloud/api-management)
- [Real-Time Best Practices](https://docs.informatica.com/integration-cloud/real-time)

---

**Status**: 🟢 Active Development  
**Last Updated**: May 2026  
**Version**: 1.0  
**Level**: Advanced (Interview Ready)
