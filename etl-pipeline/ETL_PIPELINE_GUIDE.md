# ETL Pipeline Documentation

## 📋 Overview

The ETL (Extract, Transform, Load) pipeline automates the process of ingesting data from multiple Learning Management System sources, transforming it according to business rules, and loading it into the star schema data warehouse.

**Pipeline Characteristics:**
- **Frequency:** Daily (configurable)
- **Duration:** ~45 minutes (depends on data volume)
- **Execution Window:** 2 AM - 6 AM (off-peak hours)
- **Status:** Active production pipeline

---

## 🔄 ETL Process Flow

### High-Level Flow

```
┌─────────────────────────────────────────────────────────────┐
│ START ETL PROCESS                                           │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ EXTRACT PHASE                                               │
│ ├─ Connect to SIS                                           │
│ ├─ Connect to LMS                                           │
│ ├─ Connect to Authentication System                         │
│ └─ Store in staging area                                    │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ TRANSFORM PHASE                                             │
│ ├─ Data validation                                          │
│ ├─ Data cleansing                                           │
│ ├─ Build dimensions                                         │
│ ├─ Prepare facts                                            │
│ └─ Error handling                                           │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ LOAD PHASE                                                  │
│ ├─ Load dimensions                                          │
│ ├─ Load facts                                               │
│ ├─ Verify referential integrity                             │
│ └─ Update aggregations                                      │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ VALIDATION PHASE                                            │
│ ├─ Count comparisons                                        │
│ ├─ Data quality checks                                      │
│ ├─ Aggregate validations                                    │
│ └─ Generate report                                          │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ END ETL PROCESS / SUCCESS                                   │
└─────────────────────────────────────────────────────────────┘
```

---

## 📥 EXTRACT PHASE

### Data Sources

#### 1. Student Information System (SIS)

**Data Elements:**
- Student demographics (name, ID, DOB, etc.)
- Enrollment records
- Academic history
- Transcript data
- Major/minor information

**Connection Method:**
- Database direct connection (SQL Server, Oracle)
- REST API (if available)
- File export (CSV, Excel)

---

#### 2. Learning Management System (LMS)

**Data Elements:**
- Enrollment information
- Course sections
- Student activity logs
- Assignment submissions
- Discussion forum posts
- Quiz attempts and scores
- Video viewing records

**Connection Method:**
- LMS API (Canvas, Blackboard, Moodle)
- Direct database connection
- Bulk export files

---

#### 3. Authentication System

**Data Elements:**
- User account status
- Department assignments
- Role information
- Last login timestamps

**Connection Method:**
- LDAP/Active Directory queries
- API authentication service
- SAML provider

---

## 🔄 TRANSFORM PHASE

### Data Validation

**Validation Rules:**
- Required fields check
- Data type validation
- Business rule enforcement
- Referential integrity validation

---

### Data Cleansing

**Tasks:**
- Trim and standardize whitespace
- Handle encoding issues
- Standardize codes and values
- Remove duplicates
- Correct known data quality issues

---

### Dimension Building

#### Building DimStudent

Transform SIS student data into dimension format with Slowly Changing Dimension (SCD) Type 2 handling.

#### Building FactEnrollment

Transform SIS enrollment data into fact table format with proper dimension key lookups.

#### Building FactCourseActivity

Transform LMS activity data into fact table format with aggregations and engagement calculations.

---

## 📤 LOAD PHASE

### Dimension Loading

**SCD Type 2 Implementation:**
- Identify changed attributes
- Create new records with effective dates
- Mark old records as inactive
- Maintain complete history

### Fact Loading

- Insert new facts
- Update existing facts (upsert)
- Handle late-arriving dimension members
- Maintain referential integrity

### Data Quality Checks Post-Load

- Verify referential integrity
- Check for null required fields
- Check for duplicate records
- Compare aggregate values

---

## 🔍 VALIDATION PHASE

### Pre-Load Validation

- Null key validation
- Duplicate key detection
- Referential integrity checks
- Business rules validation

### Post-Load Validation

- Row count comparison
- Aggregate comparison
- Data quality metrics

---

## 📊 Monitoring & Logging

### ETL Execution Log

Every ETL run is logged with details including:
- Start and end times
- Records processed/loaded/rejected
- Error messages
- Duration
- Success/failure status

---

## 🚨 Error Handling & Recovery

### Common Errors and Resolutions

| Error | Cause | Resolution |
|-------|-------|-----------|
| Connection timeout | Source system unavailable | Retry connection, escalate if persistent |
| Duplicate key | Data conflict in warehouse | Review and merge records |
| Referential integrity violation | Missing dimension member | Add missing dimension or investigate source |
| Data type mismatch | Incompatible data format | Update transformation logic |
| Row count variance > 5% | Data extraction issues | Investigate source data quality |

---

## 🔄 Scheduling

### Daily ETL Schedule

```
02:00 AM - Start ETL process
02:15 AM - Extract SIS data
02:30 AM - Extract LMS data
02:45 AM - Transform student dimension
03:00 AM - Transform course dimension
03:15 AM - Transform activity data
03:30 AM - Load dimension tables
03:45 AM - Load fact tables
04:00 AM - Run data quality checks
04:15 AM - Generate execution report
04:30 AM - Complete ETL process
```

---

**Last Updated:** June 2026  
**Version:** 1.0
