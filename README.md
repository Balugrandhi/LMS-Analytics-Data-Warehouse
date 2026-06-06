# LMS Analytics Data Warehouse

## 📋 Project Overview

A comprehensive **Star Schema data warehouse** designed for Learning Management System (LMS) analytics and reporting. This project aggregates and analyzes learner activity data from multiple source systems to provide institutional insights on student enrollment, course engagement, and learning outcomes.

**Organization:** Educational Analytics & Reporting  
**Repository:** LMS-Analytics-Data-Warehouse  
**Created:** 2026  
**Status:** Active Development

---

## 🏗️ Architecture Overview

### System Design Pattern

This project implements a **dimensional modeling approach** using a Star Schema architecture optimized for OLAP (Online Analytical Processing) queries and business intelligence reporting.

```
┌─────────────────────────────────────┐
│   Multiple Source Systems           │
│ (SIS, LMS, Authentication, etc.)    │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│   ETL Pipeline Layer                │
│ (Extract, Transform, Load)          │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│   Data Warehouse (Star Schema)      │
│                                     │
│  ┌──────────────────────────────┐   │
│  │    Fact Tables               │   │
│  │ • FactEnrollment             │   │
│  │ • FactCourseActivity         │   │
│  └──────────────────────────────┘   │
│                                     │
│  ┌──────────────────────────────┐   │
│  │    Dimension Tables          │   │
│  │ • DimStudent                 │   │
│  │ • DimCourse                  │   │
│  │ • DimTime                    │   │
│  │ • DimInstructor              │   │
│  └──────────────────────────────┘   │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│   Analytics & Reporting             │
│ (Dashboards, Reports, Analysis)     │
└─────────────────────────────────────┘
```

---

## 📊 Data Model

### Fact Tables

#### **FactEnrollment**
Central fact table tracking student enrollment events and lifecycle management.

**Purpose:** Track student registrations, enrollments, status changes, and withdrawal events

**Key Measures:**
- Enrollment Count
- Enrollment Status
- Enrollment Date
- Withdrawal Date
- Credit Hours Enrolled

**Dimensions Connected:**
- DimStudent
- DimCourse
- DimTime
- DimInstructor

---

#### **FactCourseActivity**
Granular fact table capturing detailed learner interactions and engagement within courses.

**Purpose:** Record learning activities, engagement metrics, and interaction patterns

**Key Measures:**
- Activity Count
- Login Frequency
- Assignment Submission Count
- Forum Participation Count
- Assessment Score
- Time Spent (minutes)
- Completion Status

**Dimensions Connected:**
- DimStudent
- DimCourse
- DimTime
- DimInstructor

---

### Dimension Tables

#### **DimStudent**
Student profile and demographic information dimension.

**Attributes:**
- StudentID (Primary Key)
- StudentNumber (Natural Key)
- FirstName, LastName
- Email
- DateOfBirth
- Gender
- Ethnicity
- ClassificationLevel (Freshman, Sophomore, Junior, Senior)
- Cohort
- MajorCode, MajorDescription
- MinorCode, MinorDescription
- GPA
- EnrollmentStatus
- StudentType (Full-Time, Part-Time, Online)
- InternationalStudent (Y/N)
- FirstGenerationStudent (Y/N)
- DateEnrolled
- DateInactive
- IsActive (Y/N)

---

#### **DimCourse**
Course metadata and organizational structure dimension.

**Attributes:**
- CourseID (Primary Key)
- CourseCode (Natural Key)
- CourseTitle
- CourseDescription
- DepartmentCode
- DepartmentName
- CollegeCode
- CollegeName
- CreditHours
- CourseLevel (100, 200, 300, 400)
- InstructionType (Lecture, Lab, Hybrid, Online)
- MaxEnrollment
- Prerequisites
- CoRequisites
- EffectiveStartDate
- EffectiveEndDate
- IsActive (Y/N)

---

#### **DimTime** (Recommended)
Time dimension for temporal analysis and aggregations.

**Attributes:**
- DateKey
- FullDate
- Year
- Quarter
- Month
- Week
- DayOfWeek
- IsWeekday
- Semester
- AcademicYear

---

#### **DimInstructor** (Recommended)
Instructor profile dimension for course delivery analysis.

**Attributes:**
- InstructorID (Primary Key)
- InstructorNumber (Natural Key)
- FirstName, LastName
- Email
- Department
- Title
- HireDate
- IsActive

---

## 🔄 ETL Pipeline

### Overview

The ETL (Extract, Transform, Load) pipeline automates the process of:
1. **Extracting** data from heterogeneous source systems
2. **Transforming** and normalizing data to dimensional model
3. **Loading** cleaned data into warehouse tables

### Data Flow

```
┌──────────────┐
│ Source System│
│   (SIS/LMS)  │
└──────┬───────┘
       │
       ▼
┌─────────────────────┐
│ EXTRACT PHASE       │
│ • Read data         │
│ • Handle connections│
│ • Error handling    │
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│ TRANSFORM PHASE     │
│ • Data validation   │
│ • Cleansing         │
│ • Normalization     │
│ • Business rules    │
│ • Surrogate keys    │
│ • SCD handling      │
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│ LOAD PHASE          │
│ • Insert/Update     │
│ • Referential integ.│
│ • Fact aggregation  │
└──────┬──────────────┘
       │
       ▼
┌──────────────┐
│ Data         │
│ Warehouse    │
└──────────────┘
```

### Key ETL Components

1. **Source System Connectors**
   - Student Information System (SIS) connector
   - Learning Management System (LMS) connector
   - Authentication system connector
   - Course management connector

2. **Data Validation & Cleansing**
   - Null/missing value handling
   - Data type validation
   - Business rule enforcement
   - Duplicate detection and resolution
   - Referential integrity checks

3. **Transformations**
   - Student dimension construction
   - Course dimension construction
   - Surrogate key generation
   - Slowly Changing Dimensions (SCD) Type 2 handling
   - Grain-level aggregations

4. **Load Strategies**
   - Full load (initial warehouse population)
   - Incremental load (daily/weekly updates)
   - Upsert operations
   - Dimension member processing
   - Fact table population

### Scheduling & Execution

- **Frequency:** Daily ETL runs (configurable)
- **Window:** Off-peak hours (recommend 2 AM - 6 AM)
- **Monitoring:** Automated alerts on failure
- **Logging:** Detailed execution logs and error tracking
- **Recovery:** Checkpointing and restart capabilities

---

## 📁 Project Structure

```
LMS-Analytics-Data-Warehouse/
│
├── README.md                          # Project documentation (this file)
│
├── database-schema/
│   ├── schema_definition.sql          # Complete DDL statements
│   ├── fact_tables/
│   │   ├── fact_enrollment.sql        # FactEnrollment table definition
│   │   └── fact_course_activity.sql   # FactCourseActivity table definition
│   ├── dimension_tables/
│   │   ├── dim_student.sql            # DimStudent table definition
│   │   ├── dim_course.sql             # DimCourse table definition
│   │   ├── dim_time.sql               # DimTime table definition
│   │   └── dim_instructor.sql         # DimInstructor table definition
│   ├── indexes/
│   │   └── create_indexes.sql         # Index definitions for optimization
│   └── views/
│       └── analytics_views.sql        # Pre-built analytical views
│
├── etl-pipeline/
│   ├── README.md                      # ETL documentation
│   ├── config/
│   │   ├── config.example.yaml        # Configuration template
│   │   └── logging_config.yaml        # Logging configuration
│   ├── scripts/
│   │   ├── extract_sis_data.py        # SIS data extraction
│   │   ├── extract_lms_data.py        # LMS data extraction
│   │   ├── transform_students.py      # Student dimension transformation
│   │   ├── transform_courses.py       # Course dimension transformation
│   │   ├── load_dimensions.py         # Dimension table loading
│   │   ├── load_facts.py              # Fact table loading
│   │   └── etl_orchestrator.py        # Main ETL orchestration
│   ├── sql/
│   │   ├── dimension_queries.sql      # Dimension processing queries
│   │   └── fact_queries.sql           # Fact table processing queries
│   └── logs/
│       └── etl_execution.log          # ETL execution logs
│
├── architecture/
│   ├── ARCHITECTURE.md                # Detailed architecture documentation
│   ├── data_model_diagram.md          # Data model ERD description
│   ├── system_design.md               # System design patterns
│   └── deployment_guide.md            # Deployment instructions
│
├── documentation/
│   ├── data_dictionary.md             # Complete data dictionary
│   ├── etl_process_guide.md           # ETL process documentation
│   ├── reporting_guide.md             # Reporting and query examples
│   ├── troubleshooting.md             # Troubleshooting guide
│   └── faq.md                         # Frequently asked questions
│
├── analytics/
│   ├── sample_queries.sql             # Example analytical queries
│   ├── dashboards/
│   │   ├── enrollment_dashboard.md    # Enrollment analytics
│   │   ├── course_activity_dashboard.md   # Course activity analytics
│   │   └── student_success_dashboard.md   # Student success metrics
│   └── reports/
│       ├── enrollment_report.sql      # Enrollment report queries
│       ├── engagement_report.sql      # Engagement report queries
│       └── performance_report.sql     # Performance report queries
│
├── tests/
│   ├── data_quality_tests.sql         # Data quality validation
│   ├── etl_unit_tests.py              # ETL unit tests
│   └── integration_tests.py           # Integration tests
│
├── requirements.txt                   # Python dependencies
├── LICENSE                            # License information
└── CONTRIBUTING.md                    # Contribution guidelines
```

---

## 🚀 Getting Started

### Prerequisites

- **Database:** SQL Server, PostgreSQL, MySQL, or compatible RDBMS
- **ETL Tool:** Python 3.8+, or native database procedures
- **Libraries:** pandas, SQLAlchemy, requests (see requirements.txt)
- **Access:** Source system credentials for SIS/LMS

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/Balugrandhi/LMS-Analytics-Data-Warehouse.git
   cd LMS-Analytics-Data-Warehouse
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Configure environment**
   ```bash
   cp etl-pipeline/config/config.example.yaml etl-pipeline/config/config.yaml
   # Edit config.yaml with your database and source system credentials
   ```

4. **Create database schema**
   ```bash
   # Run schema definition scripts in your database
   sqlplus < database-schema/schema_definition.sql
   ```

5. **Run initial ETL**
   ```bash
   python etl-pipeline/scripts/etl_orchestrator.py --full-load
   ```

---

## 📊 Key Business Questions Answered

This data warehouse enables analysis of:

1. **Enrollment Analytics**
   - How many students are enrolled by course, department, level?
   - What is the enrollment trend over time?
   - What is the student retention and completion rate?
   - Which courses have highest/lowest enrollment?

2. **Course Performance**
   - What is the student engagement level by course?
   - How frequently do students access course materials?
   - Which courses have highest participation rates?
   - What is the assignment completion rate by course?

3. **Student Success**
   - Which students are at-risk of non-completion?
   - What learning patterns correlate with success?
   - How does student engagement predict grades?
   - Which populations have disparate outcomes?

4. **Institutional Reporting**
   - What are overall enrollment statistics by college/department?
   - How are student demographics distributed?
   - What is the course capacity utilization?
   - What are program-level learning outcomes?

---

## 🔧 Technology Stack

| Component | Technology |
|-----------|-----------|
| **Database** | SQL Server / PostgreSQL / MySQL |
| **ETL Language** | Python 3.8+ |
| **Data Processing** | Pandas, SQLAlchemy |
| **Orchestration** | Python scripts / Airflow (optional) |
| **Version Control** | Git |
| **Documentation** | Markdown |

---

## 📈 Data Volumes & Performance

- **Expected Data Volume:** Scales from 1000s to millions of records
- **Grain:** Student-Course-Day for activity data
- **Retention:** Recommend 5-7 year rolling history
- **Refresh Frequency:** Daily (configurable)
- **Query Performance Target:** <5 seconds for typical queries

---

## 🔐 Security & Compliance

- **Data Privacy:** PII fields isolated with appropriate access controls
- **FERPA Compliance:** Student education records protected
- **Audit Logging:** All data access logged and auditable
- **Encryption:** Credentials secured in configuration
- **Access Control:** Role-based access to sensitive data

---

## 📝 Maintenance

### Regular Tasks

- **Daily:** Monitor ETL job execution
- **Weekly:** Review data quality metrics
- **Monthly:** Analyze warehouse growth and performance
- **Quarterly:** Review and optimize queries
- **Annually:** Archive historical data, review schema

### Performance Tuning

- Index maintenance and statistics updates
- Query optimization for slow-running reports
- Partition strategy for large fact tables
- Materialized views for common aggregations

---

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

---

## 📞 Support & Issues

For issues, questions, or suggestions:
- Create an issue in the GitHub repository
- Review [TROUBLESHOOTING.md](documentation/troubleshooting.md)
- Check [FAQ.md](documentation/faq.md)

---

## 📄 License

This project is licensed under the MIT License - see [LICENSE](LICENSE) file for details.

---

## 👤 Author

**Balugrandhi**  
GitHub: [@Balugrandhi](https://github.com/Balugrandhi)

---

## 🎯 Roadmap

- [ ] Initial schema and ETL pipeline
- [ ] Data quality framework
- [ ] Sample dashboards and reports
- [ ] Performance optimization
- [ ] Advanced analytics module
- [ ] Real-time data pipeline
- [ ] Machine learning integration

---

## 📚 Additional Resources

- [Architecture Documentation](architecture/ARCHITECTURE.md)
- [Data Dictionary](documentation/data_dictionary.md)
- [ETL Process Guide](documentation/etl_process_guide.md)
- [Sample Queries](analytics/sample_queries.sql)
- [Reporting Guide](documentation/reporting_guide.md)

---

**Last Updated:** June 2026  
**Version:** 1.0.0-beta  
**Status:** Active Development
