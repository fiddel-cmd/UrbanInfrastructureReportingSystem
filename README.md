# 🗺️ CityFix — Urban Infrastructure Reporting System

![Java](https://img.shields.io/badge/Java-21-orange?style=flat-square&logo=java)
![JavaFX](https://img.shields.io/badge/JavaFX-21-blue?style=flat-square)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-17-336791?style=flat-square&logo=postgresql)
![NetBeans](https://img.shields.io/badge/IDE-Apache%20NetBeans-green?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)

---

## 📌 Overview

**CityFix** is a desktop application built to solve a real urban problem — the lack of a structured, accessible way for citizens to report infrastructure issues in their cities. From potholes and broken streetlights to graffiti and illegal dump sites, CityFix gives citizens a platform to report problems and gives administrators the tools to track and resolve them.

The project was built with **Java**, **JavaFX**, and **PostgreSQL**, following the **SOLID principles** of object-oriented design throughout the entire codebase.

---

## 🎯 Problem Statement

Urban infrastructure issues in cities like Nairobi often go unreported or unresolved because:

- There is no easy channel for citizens to formally report problems
- Administrators have no centralized view of outstanding issues
- There is no accountability or status tracking once a report is made

CityFix addresses all three of these gaps.

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| 🔐 User Authentication | Secure login and registration with role-based access (CITIZEN / ADMIN) |
| 📋 Issue Reporting | Citizens submit infrastructure issues with category, title, description and photo |
| 📊 Dashboard | Real-time table of all reported issues with colour-coded status badges |
| 🔍 Filtering | Filter issues by category, status, and date |
| 🔎 Search | Live search across issue titles, descriptions, and categories |
| 🗺️ Map View | Interactive Leaflet map showing issue locations |
| 📤 CSV Export | Export filtered issues to a CSV file |
| 🔄 Status Updates | Admins can update issue status (OPEN → IN PROGRESS → RESOLVED) |
| 🚪 Logout | Secure session logout back to login screen |

---

## 🏗️ Architecture

CityFix follows a **layered architecture** with clear separation of concerns:

```
┌─────────────────────────────────────────┐
│           Presentation Layer            │
│   LoginController   IssueController     │
│        (JavaFX FXML Controllers)        │
├─────────────────────────────────────────┤
│            Service Layer                │
│  IssueFilterService   MapService        │
│  NavigationService    CsvExporter       │
├─────────────────────────────────────────┤
│         Data Access Layer (DAO)         │
│      UserDAO          IssueDAO          │
│    (implements IUserDAO / IIssueDAO)    │
├─────────────────────────────────────────┤
│           Database Layer                │
│    DBConnection → PostgreSQL            │
└─────────────────────────────────────────┘
```

---

## 📁 Project Structure

```
src/
└── cityfix/
    ├── MainApplication.java          ← JavaFX entry point
    ├── controllers/
    │   ├── LoginController.java      ← Login & Register UI
    │   └── IssueController.java      ← Dashboard UI
    ├── dao/
    │   ├── IUserDAO.java             ← User DAO interface
    │   ├── IIssueDAO.java            ← Issue DAO interface
    │   ├── UserDAO.java              ← User DB operations
    │   └── IssueDAO.java             ← Issue DB operations
    ├── db/
    │   └── DBConnection.java         ← PostgreSQL connection factory
    ├── model/
    │   ├── User.java                 ← User data model
    │   └── InfrastructureIssue.java  ← Issue data model
    ├── interfaces/
    │   └── Navigable.java            ← Navigation contract
    └── util/
        ├── CsvExporter.java          ← CSV export logic
        ├── IssueFilterService.java   ← Filter predicate logic
        ├── MapService.java           ← Leaflet map loading
        └── NavigationService.java    ← Scene navigation logic
fxml/
    ├── login.fxml                    ← Login screen layout
    └── dashboard.fxml                ← Dashboard layout
css/
    └── cityfix.css                   ← Application stylesheet
```

---

## 🧱 SOLID Principles Applied

This project was deliberately designed to follow all five SOLID principles:

### S — Single Responsibility
Every class has one reason to change. For example:
- `IssueFilterService` only builds filter predicates
- `CsvExporter` only handles CSV export
- `MapService` only loads the Leaflet map
- `DBConnection` only produces JDBC connections

### O — Open/Closed
New filter types are added by extending predicates in `IssueFilterService` without modifying existing code. New DAO implementations extend the interfaces without touching controllers.

### L — Liskov Substitution
`UserDAO` and `IssueDAO` correctly implement their interfaces `IUserDAO` and `IIssueDAO`. Any subclass can replace the parent without breaking the application.

### I — Interface Segregation
Interfaces are narrow and focused:
- `IUserDAO` has only 2 methods
- `IIssueDAO` has only 3 methods
- `Navigable` has only 1 method

No controller is forced to implement methods it does not use.

### D — Dependency Inversion
Controllers depend on **interfaces**, not concrete classes:

```java
// LoginController depends on IUserDAO interface
private final IUserDAO userDAO;

// IssueController depends on IIssueDAO interface
private final IIssueDAO issueDAO;
```

This means you can swap `UserDAO` for a `MockUserDAO` or `CloudUserDAO` without changing any controller code.

---

## 🗄️ Database Schema

```sql
-- Users table
CREATE TABLE users (
    user_id       SERIAL PRIMARY KEY,
    username      VARCHAR(50)  NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    email         VARCHAR(100) NOT NULL UNIQUE,
    full_name     VARCHAR(100) NOT NULL,
    role          VARCHAR(20)  NOT NULL DEFAULT 'CITIZEN',
    created_at    TIMESTAMP    NOT NULL DEFAULT NOW()
);

-- Issues table
CREATE TABLE issues (
    issue_id      SERIAL PRIMARY KEY,
    user_id       INT          NOT NULL REFERENCES users(user_id),
    title         VARCHAR(150) NOT NULL,
    description   TEXT         NOT NULL,
    location_text VARCHAR(200),
    category      VARCHAR(50)  NOT NULL,
    status        VARCHAR(20)  NOT NULL DEFAULT 'OPEN',
    date_reported TIMESTAMP    NOT NULL DEFAULT NOW(),
    photo_path    VARCHAR(500),
    priority      INT          NOT NULL DEFAULT 0
);
```

---

## ⚙️ Setup & Installation

### Prerequisites

- Java JDK 21+
- JavaFX SDK 21+
- PostgreSQL 15+
- Apache NetBeans IDE 20+
- PostgreSQL JDBC Driver (`postgresql-42.x.x.jar`)

### Steps

**1. Clone the repository**
```bash
git clone https://github.com/fiddel-cmd/UrbanInfrastructureReportingSystem.git
```

**2. Create the database**

Open pgAdmin or SQL Shell and run:
```sql
CREATE DATABASE cityfix_db;
\c cityfix_db
```

Then run the `schema.sql` file provided in the project root.

**3. Configure the database connection**

Open `src/cityfix/db/DBConnection.java` and update:
```java
private static final String DATABASE = "cityfix_db";
private static final String USER     = "postgres";
private static final String PASSWORD = "your_password_here";
```

**4. Add libraries in NetBeans**

Right-click project → Properties → Libraries → Add JAR:
- `postgresql-42.x.x.jar`
- JavaFX SDK lib folder

**5. Add VM options**

Right-click project → Properties → Run → VM Options:
```
--module-path "path/to/javafx-sdk/lib" --add-modules javafx.controls,javafx.fxml,javafx.web
--enable-native-access=javafx.graphics
```

**6. Run**

Right-click `MainApplication.java` → Run File

---

## 👤 Default Login Credentials

| Role | Username | Password |
|------|----------|----------|
| Admin | `admin` | `admin123` |
| Citizen | `john_doe` | `citizen123` |
| Citizen | `jane_doe` | `citizen123` |

---

## 🛠️ Technologies Used

| Technology | Purpose |
|-----------|---------|
| Java 21 | Core application language |
| JavaFX 21 | Desktop UI framework |
| FXML | UI layout definition |
| CSS | UI styling |
| PostgreSQL | Relational database |
| JDBC | Database connectivity |
| Leaflet.js | Interactive map (via WebView) |
| Apache NetBeans | IDE |

---

## 🚀 Future Improvements

- [ ] Password hashing with BCrypt
- [ ] Email notifications when issue status changes
- [ ] Admin panel for user management
- [ ] Mobile companion app
- [ ] Cloud database deployment
- [ ] Issue priority escalation system
- [ ] Photo upload and display in dashboard

---

## 👨‍💻 Author

**Fiddel Omondi**
- GitHub: [@fiddel-cmd](https://github.com/fiddel-cmd)
- Project: [UrbanInfrastructureReportingSystem](https://github.com/fiddel-cmd/UrbanInfrastructureReportingSystem)

---

## 📄 License

This project is licensed under the MIT License — feel free to use, modify and distribute.

---

> *Built to solve a real problem — making urban infrastructure reporting accessible to every citizen.*
