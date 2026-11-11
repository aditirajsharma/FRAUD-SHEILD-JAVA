🌐 Live Website: https://detectionfruad.netlify.app
# FraudShield — AI-Powered Fraud Detection & Security Intelligence System

An enterprise-grade fraud detection, security monitoring, and financial safety platform built with **Spring Boot 3**, **Java 17**, **Thymeleaf**, **Raw JDBC & JPA dual data layers**, **Jakarta Servlets & Filters**, and **Multithreaded Ingestion Pipelines**.

---

## 🌟 Key Highlights & Architecture

### 1. Dual-Persona Web Experience
- **Client/User Portal (`/user/dashboard`)**:
  - **Overview**: Real-time spending safety score, dynamic risk insights, 30-day spending vs flagged anomaly trend, quick protection actions, and AI assistant query copilot.
  - **My Transactions**: Searchable, filterable ledger with instant transaction status chips, risk scoring badges, and CSV export.
  - **Submit / Simulate Transaction (`/user/transactions/new`)**: Live simulation form with instant scoring, risk factor inspection, and receipt breakdown.
  - **Fraud & Security Alerts (`/user/alerts`)**: Notification center for held/blocked transactions with dispute actions.
  - **Account & Security Settings (`/user/profile`)**: Dynamic 2FA toggle, daily spend limit controls, and risk sensitivity threshold sliders.
- **Admin Command Center (`/admin/dashboard`)**:
  - **System Metrics**: Total volume, flagged rates, model accuracy, and system health status.
  - **Interactive Analytics**: Monthly fraud volume, case resolution distribution, anomaly timelines, and ML model performance.
  - **Reports & Case Investigation (`/admin/reports`)**: Paginated case queue with review modals (`CONFIRMED_FRAUD` / `FALSE_POSITIVE`).
  - **Algorithms (`/admin/algorithms`)**: Track model versions, review feedback loops, and trigger automated weight tuning.
  - **Configuration (`/admin/config`)**: Real-time threshold adjustment (amount limit, velocity weights, location jump penalties).

### 2. Dual Data Access: Raw JDBC DAO & Spring Data JPA
- **Centralised Connection Pool (`DataSourceProvider`)**: Manages pooled HikariCP connections for raw JDBC operations.
- **Explicit JDBC Layer (`JdbcTransactionDao`, `JdbcUserDao`, `JdbcFraudCaseDao`)**:
  - Uses `PreparedStatement` with parameterized SQL for injection prevention.
  - Explicit transaction control (`conn.setAutoCommit(false)`, `conn.commit()`, `conn.rollback()`).
  - Column-by-column `ResultSet` mapping and `Statement.RETURN_GENERATED_KEYS`.
  - Try-with-resources resource cleanup ensuring zero connection leaks.
- **Reproducible DDL & DML**: Self-contained [schema.sql](file:///src/main/resources/schema.sql) and [data.sql](file:///src/main/resources/data.sql).

### 3. Native Jakarta Servlets & Servlet Filters
- Registered alongside Spring Boot MVC using `ServletRegistrationBean`, `FilterRegistrationBean`, and `ServletListenerRegistrationBean`.
- **Servlets (`/servlet/**`)**:
  - `TransactionServlet` (`/servlet/transactions`)
  - `FraudCaseServlet` (`/servlet/fraud-cases`)
  - `DashboardStatsServlet` (`/servlet/dashboard-stats`)
  - `ConfigServlet` (`/servlet/config`)
  - `LoginServlet` (`/servlet/login`)
  - `LogoutServlet` (`/servlet/logout`)
- **Filters**:
  - `CharacterEncodingFilter` (`/*`, UTF-8 enforcement)
  - `RequestLoggingFilter` (`/servlet/*`, timing and request auditing)
  - `AuthenticationFilter` (`/servlet/*`, session & principal validation)
  - `AuthorizationFilter` (`/servlet/*`, administrative role enforcement)
- **Lifecycle Listener**: `AppServletContextListener` hooks into application initialization and destruction.

### 4. Advanced Concurrency, Multithreading & Collections
- **Thread Pool (`FraudScanService`)**: Fixed thread pool of 4 named worker threads for asynchronous transaction scanning (`scanAsync()`) returning `Future<Result>`.
- **Scheduled Background Tasks (`StatisticsRefreshTask`)**: `ScheduledExecutorService` executing recurring statistics caching every 60 seconds with resilient error handling.
- **Producer-Consumer Pipeline (`TransactionProcessingPipeline`)**: Bounded `ArrayBlockingQueue<Transaction>`, `ReentrantLock`, and worker threads for non-blocking ingestion.
- **Lock-Free Atomic Counters (`FraudMetricsCounter`)**: Uses `AtomicLong` and `AtomicInteger` for real-time transaction throughput and fraud totals.
- **Thread-Safe Cache (`ThreadSafeCache<K, V>`)**: Backed by `ConcurrentHashMap` with atomic `computeIfAbsent()` caching.
- **Diverse Collections**: Employs `HashSet` for $O(1)$ device/location lookups, `ArrayDeque` for velocity FIFO time windows, and `TreeMap` for chronological spending timelines.

---

## 🚀 Quick Start & How to Run

### Prerequisites
- Java 17+
- Maven 3.9+ (or use included `mvnw.cmd` / `./mvnw`)

### Running Locally
1. Start the Spring Boot server:
   ```bash
   .\mvnw.cmd spring-boot:run
   ```
2. Open your browser and navigate to:
   [http://localhost:8080](http://localhost:8080)

### Running Automated Test Suite
To execute the comprehensive unit and integration test suite:
```bash
.\mvnw.cmd test
```
All 14 tests across rules, JDBC DAOs, service layer, and multithreading concurrency execute cleanly.

### Running Dev-Only UI Overflow Detector
To inspect all User and Admin pages across responsive viewports (`390px`, `768px`, `1280px`, `1440px`, `1920px`, `2560px`) for document scroll, card boundary overflows, and button label wrapping:
```bash
# Using npm
npm run ui-overflow-check

# Or directly with Node / scripts:
node scripts/ui-overflow-check.js
# On Windows:
.\scripts\ui-overflow-check.cmd
# On Linux/macOS:
./scripts/ui-overflow-check
```
Ensures 0 horizontal scrollbars, 0 card overflows, and strictly single-line button labels across all breakpoints.

---

## 🔑 Demo Credentials

| Role | Email | Password | Default Redirect |
|---|---|---|---|
| **User (Demo)** | `demo@fraudshield.io` | `Demo@123` | `/user/dashboard` |
| **System Admin** | `admin@fraud.com` | `Admin@123` | `/admin/dashboard` |

---

## 📡 Endpoints Summary

### Web Application Routes (Thymeleaf UI)
- `GET /login` — Secure login screen with quick demo auto-fill
- `GET /register` — User registration portal
- `GET /user/dashboard` — User main dashboard (supports `?tab=overview|transactions|alerts|security|settings`)
- `GET /user/transactions/new` — Transaction simulator & tester
- `GET /user/export?format=csv` — Export personal transactions to CSV
- `GET /admin/dashboard` — Admin analytics & KPI monitoring
- `GET /admin/reports` — Fraud investigation reports & case review
- `GET /admin/reports/export?format=csv` — Export fraud cases to CSV
- `GET /admin/algorithms` — Model accuracy & version history
- `GET /admin/config` — Real-time detection configuration

### Jakarta Servlet Endpoints (`/servlet/**`)
- `GET /servlet/transactions` — Authenticated user's transaction history
- `GET /servlet/fraud-cases?status=OPEN` — Administrative fraud cases list
- `GET /servlet/dashboard-stats` — System detection report statistics
- `GET /servlet/config` — Current detection thresholds and rule weights
- `POST /servlet/login` — Servlet-based JSON authentication establishing `HttpSession`
- `POST /servlet/logout` — Servlet-based session termination

### REST API Endpoints (`/api/**`)
- `POST /api/auth/register` — Public registration
- `POST /api/auth/login` — Public JWT authentication
- `POST /api/transactions` — Submit transaction (scored in real-time)
- `GET /api/transactions` — User transactions list
- `GET /api/transactions/alerts` — User fraud alerts
- `GET /api/charts/user-trend` — Live 30-day user trend data
- `GET /api/charts/fraud-trend` — Live monthly fraud trend
- `GET /api/charts/monthly-cases` — Monthly case volume data
- `GET /api/charts/status-distribution` — Donut distribution data
- `GET /api/charts/algorithm-performance` — ML version accuracy comparison

---

## 📚 Technical Documentation & Verification
- **Architecture & Design Document:** [docs/DESIGN.md](file:///docs/DESIGN.md) (Architecture, ER Diagram, Class Diagram, Sequence Diagram, Algorithm Math)
- **Grading Rubric Matrix:** [RUBRIC_COMPLIANCE.md](file:///RUBRIC_COMPLIANCE.md) (Line-by-line verification of every rubric criterion)
