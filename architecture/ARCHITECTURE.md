# LMS Analytics Data Warehouse - Architecture Documentation

## 📐 System Architecture Overview

### High-Level Architecture Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                    DATA SOURCES                              │
├──────────────────────────────────────────────────────────────┤
│  • Student Information System (SIS)                          │
│  • Learning Management System (LMS)                          │
│  • Authentication System (Active Directory, LDAP)            │
│  • Course Management System                                  │
│  • Assessment Platform                                       │
│  • Attendance System                                         │
│  • Third-party Integrations (Canvas, Blackboard, etc.)      │
└────────────────┬─────────────────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────────────────┐
│           ETL PIPELINE LAYER                                 │
├──────────────────────────────────────────────────────────────┤
│  ┌────────────────┬──────────────────┬──────────────────┐    │
│  │  EXTRACT       │    TRANSFORM     │     LOAD         │    │
│  │                │                  │                  │    │
│  │ • Connect to   │ • Data           │ • Dimension      │    │
│  │   sources      │   validation     │   loading        │    │
│  │ • Read data    │ • Cleansing      │ • Fact table     │    │
│  │ • Staging      │ • Normalization  │   population     │    │
│  │ • Error        │ • Business       │ • Referential    │    │
│  │   handling     │   rules          │   integrity      │    │
│  │ • Logging      │ • SCD            │ • Aggregations   │    │
│  │                │   processing     │                  │    │
│  └────────────────┴──────────────────┴──────────────────┘    │
└────────────────┬─────────────────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────────────────┐
│        STAR SCHEMA DATA WAREHOUSE                            │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│              ┌─────────────────────────┐                     │
│              │      DimTime            │                     │
│              │ (Date, Year, Quarter)   │                     │
│              └────────┬────────────────┘                     │
│                       │                                      │
│  ┌────────────────┬───┴────────┬─────────────────┐          │
│  │                │            │                 │          │
│  ▼                ▼            ▼                 ▼          │
│ ┌────────────┐ ┌──────────────────────┐ ┌────────────────┐ │
│ │ DimStudent │ │  FactEnrollment      │ │ DimCourse      │ │
│ │            │ │  FactCourseActivity  │ │                │ │
│ └────────────┘ └──────────────────────┘ └────────────────┘ │
│                       │                                      │
│  ┌────────────────┬───┴────────┐                            │
│  │                │            │                            │
│  ▼                ▼            ▼                            │
│ ┌─────────────────────────────────────┐                    │
│ │      DimInstructor                  │                    │
│ │  (Faculty, Department, Contact)     │                    │
│ └─────────────────────────────────────┘                    │
│                                                               │
└────────────────┬─────────────────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────────────────┐
│     ANALYTICS & REPORTING LAYER                              │
├──────────────────────────────────────────────────────────────┤
│  • BI Tools (Tableau, Power BI, Looker)                     │
│  • Ad-hoc SQL Queries                                       │
│  • Pre-built Reports & Dashboards                           │
│  • Data Visualization                                       │
│  • Analytics Views & Materialized Views                     │
└──────────────────────────────────────────────────────────────┘
```

---

## 🏗️ Dimensional Modeling Approach

### Star Schema Design

The data warehouse implements a **Star Schema** (also called Dimensional Model) with:

**Central Fact Tables:**
- FactEnrollment
- FactCourseActivity

**Supporting Dimension Tables:**
- DimStudent
- DimCourse
- DimTime
- DimInstructor

### Star Schema Benefits

1. **Query Performance** - Fewer joins, simpler queries
2. **Intuitive Design** - Business stakeholders understand the structure
3. **Scalability** - Efficient for large datasets
4. **Aggregation** - Pre-aggregation at fact table level
5. **Flexibility** - Easy to add new dimensions or metrics

---

## 📊 Fact Tables Details

### FactEnrollment

**Purpose:** Record enrollment transactions and student-course relationships

**Grain:** One row per student-course-semester enrollment

**Dimensions:**
- StudentKey (FK to DimStudent)
- CourseKey (FK to DimCourse)
- InstructorKey (FK to DimInstructor)
- DateKey (FK to DimTime)

**Measures (Additive):**
- CreditHoursEnrolled
- CreditHoursEarned
- GradeNumeric
- GradePoints

**Key Attributes:**
- EnrollmentStatus
- EnrollmentDate
- WithdrawalDate
- GradeLetterGrade
- PassingGrade

---

### FactCourseActivity

**Purpose:** Record granular learner interactions and engagement activities

**Grain:** One row per student-course-activity-day

**Dimensions:**
- StudentKey (FK to DimStudent)
- CourseKey (FK to DimCourse)
- InstructorKey (FK to DimInstructor)
- DateKey (FK to DimTime)

**Measures (Additive):**
- LoginsCount
- TimeSpentMinutes
- AssignmentSubmissions
- ForumPostsCreated
- ResourcesViewed
- VideoWatchMinutes

**Key Attributes:**
- ActivityType
- CompletionPercentage
- EngagementLevel
- ParticipationFlag

---

## 🔄 ETL Architecture

### ETL Phases

#### 1. Extract Phase
- Connect to source systems (SIS, LMS, Authentication)
- Read transactional data
- Apply initial filtering
- Store in staging area

#### 2. Transform Phase
- Data validation against business rules
- Data cleansing and standardization
- Dimension transformations (SCD Type 2)
- Fact table transformations
- Surrogate key generation

#### 3. Load Phase
- Dimension loading with SCD Type 2 handling
- Fact table loading (insert/update)
- Referential integrity verification
- Data quality checks

---

## 🔐 Security Architecture

### Data Access Control

**Role-Based Access:**
- **Data Administrator** - Full access to all data
- **Analyst** - Read access to all data and reports
- **Department Chair** - Read access to department courses only
- **Student** - Read access to own enrollment records

### Compliance

**FERPA Compliance:**
- Student education records isolated with access controls
- Audit trail for all data access
- PII masking in non-production environments
- Encryption of sensitive data fields

---

## 📈 Performance Considerations

### Query Optimization

**Star Schema Benefits:**
- Pre-aggregation at fact table level
- Reduced number of joins required
- Denormalized dimensions for faster access
- Optimized for analytical queries (OLAP)

**Index Strategy:**
- Primary key indexes on all tables
- Foreign key indexes on fact tables
- Composite indexes on frequently used dimension combinations
- Index on date fields for time-based queries

### Scalability

**Data Volume Handling:**
- Partition fact tables by date range (monthly/quarterly)
- Archive historical data (>5 years) to separate storage
- Use columnnar storage for analytics when possible

**Projected Growth:**
- Year 1: 1-5M records per fact table
- Year 2: 5-10M records
- Year 3+: 10M+ records with archival strategy

---

## 🔄 Change Management & Evolution

### Schema Evolution

**Adding New Dimensions:**
1. Create new dimension table with proper structure
2. Add foreign key to relevant fact tables
3. Update ETL pipeline to populate dimension
4. Backfill historical data
5. Update analytical views and reports

**Adding New Measures:**
1. Add new column(s) to fact table
2. Update ETL transformation logic
3. Validate data quality with checks
4. Update views and report queries
5. Test with reporting tools

---

## 📚 Data Model Relationships

### Fact-to-Dimension Relationships

```
FactEnrollment
  ├─ DimStudent (many-to-one)
  ├─ DimCourse (many-to-one)
  ├─ DimInstructor (many-to-one)
  └─ DimTime (many-to-one)

FactCourseActivity
  ├─ DimStudent (many-to-one)
  ├─ DimCourse (many-to-one)
  ├─ DimInstructor (many-to-one)
  └─ DimTime (many-to-one)
```

---

## 🎯 Key Analytics Capabilities

### Enrollment Analytics
- Total enrollments by course, department, semester
- Enrollment trends and forecasting
- Student retention and completion rates
- Enrollment capacity analysis

### Course Performance
- Course effectiveness metrics
- Student engagement by course
- Grade distribution analysis
- Course utilization rates

### Student Success
- At-risk student identification
- Learning outcome assessment
- Student progression tracking
- Demographic performance analysis

### Institutional Reporting
- Comprehensive enrollment dashboards
- Program-level analytics
- Department performance metrics
- Trend analysis and forecasting

---

**Last Updated:** June 2026  
**Version:** 1.0  
**Status:** Active Development
