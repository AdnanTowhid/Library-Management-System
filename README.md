# Library Book Lending System

A lightweight, robust Library Book Lending System built for a university Software Engineering project. The application follows clean architectural boundaries, separating core domain logic and design patterns into dedicated service layers while keeping views thin.

---

## 1. Project Overview

The system allows university librarians to catalog books, manage registered members, issue and return book loans with loan limits (max 3 per member) and availability checks, calculate overdue fines via extensible strategies, notify observers on return events, and generate administrative summary reports.

### Five Functional Modules
1. **Book Catalog:** Full CRUD operations and search by title or author. Displays total copies and live available copies.
2. **Member Management:** Member registration (name, email, phone) and member listing (no member login/signup).
3. **Issue and Return:** Loan lifecycle management enforcing copy availability and 3-active-loan limits. Includes interactive fine simulation.
4. **Fine Calculation:** Extensible overdue fine calculation using pluggable strategies (daily rate with caps, lost book flat rate).
5. **Reports:** Aggregate metrics and breakdown activity generated via builder factories.

---

## 2. Design Patterns

The core domain logic cleanly isolates design patterns using Python Abstract Base Classes (`abc.ABC`):

| Design Pattern | Class / Abstraction | File Location | Rationale & Responsibility |
| :--- | :--- | :--- | :--- |
| **Strategy** | `FineStrategy`, `DailyRateFine`, `LostBookFine` | `backend/loans/strategies.py` | Encapsulates fine computation algorithms. Allows swapping calculation rules (e.g. daily rate vs. flat lost book fee) without modifying the loan return service. |
| **Observer** | `LoanObserver`, `EmailReminderObserver`, `AuditLogObserver` | `backend/loans/observers.py` | Decouples loan lifecycle events from side-effects. Observers are notified exactly once on return to send fine reminder emails and write audit records. |
| **Factory** | `ReportFactory`, `ReportBuilder`, `MonthlyReport`, `OverdueReport` | `backend/reports/factories.py` | Encapsulates the instantiation of reporting builders based on request parameters (`type="monthly"`, `type="overdue"`), shielding client views from builder construction details. |

### Supporting Architecture
- **Injected Clock (`today_fn`):** `backend/loans/services.py` accepts a `today_fn` parameter defaulting to `date.today`. This makes fine calculations and loan durations 100% deterministic during unit testing.
- **Email Service Abstraction:** `backend/notifications/services.py` defines `EmailService` and `ConsoleEmailService` to prevent hitting real SMTP servers in development or testing.

---

## 3. Tech Stack

- **Backend:** Python 3.12, Django 5.x, Django REST Framework, SQLite, `rest_framework.authtoken`, `django-cors-headers`, `coverage.py`.
- **Frontend:** React 18, Vite, plain JavaScript/JSX, `react-router-dom`, native `fetch`, Tailwind CSS.
- **CI:** GitHub Actions (`.github/workflows/ci.yml`).

---

## 4. Setup and Installation

### Prerequisites
- Python 3.11+
- Node.js 18+ and npm

### Backend Setup
```bash
# 1. Navigate to backend directory
cd backend

# 2. Create and activate a virtual environment
python -m venv ../venv
# On Windows PowerShell:
..\venv\Scripts\Activate.ps1
# On Linux/macOS:
# source ../venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run database migrations
python manage.py migrate

# 5. Create default librarian user
python manage.py create_librarian
# Default credentials: username 'librarian', password 'password123'
```

### Frontend Setup
```bash
# 1. Navigate to frontend directory
cd frontend

# 2. Install dependencies
npm install
```

---

## 5. How to Run Tests

Run the full Django test suite with coverage report:

```bash
cd backend
coverage run manage.py test
coverage report -m
```
*Current test suite achieves **99% line coverage** across 30 unit and integration tests.*

---

## 6. How to Run the Application

Run the application with two terminal windows:

### Terminal 1: Backend API Server
```bash
cd backend
python manage.py runserver
```

### Terminal 2: Frontend Development Server
```bash
cd frontend
npm run dev
```

Then open `http://localhost:5173` and login with username `librarian` / password `password123`.

---

## 7. Git Branching Strategy

This project uses the following branch management approach:

| Branch | Purpose |
| :--- | :--- |
| `main` | Stable production branch. All tested and validated code merges here. |
| `develop` | Integration branch for ongoing development work. |
| `feature/*` | Short-lived branches for specific feature development. |

### Commit Convention
Commits follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:
- `feat:` — New features
- `fix:` — Bug fixes
- `test:` — Test additions or modifications
- `ci:` — CI/CD configuration changes
- `docs:` — Documentation updates
- `chore:` — Maintenance tasks
- `refactor:` — Code restructuring without behavior change

---

## 8. GitHub Actions CI/CD

### CI Pipeline (`.github/workflows/ci.yml`)

The CI pipeline runs automatically on:
- Push to `main` branch
- Pull requests targeting `main`

#### Backend Job
1. Checkout code
2. Set up Python 3.12
3. Install pip dependencies from `requirements.txt`
4. Run Django test suite with `coverage`
5. Enforce minimum 80% code coverage

#### Frontend Job
1. Checkout code
2. Set up Node.js 20
3. Install npm dependencies with `npm ci`
4. Build frontend with `npm run build`

---

## 9. Release Management

Releases follow [Semantic Versioning](https://semver.org/):

| Version | Description |
| :--- | :--- |
| `v0.1.0` | Initial backend API with all five modules |
| `v0.2.0` | Frontend UI and full-stack integration |
| `v1.0.0` | Complete application with tests, CI/CD, and documentation |

---

## 10. Project Structure

```
Library-Management-System/
├── .github/workflows/ci.yml    # GitHub Actions CI pipeline
├── backend/
│   ├── manage.py               # Django management script
│   ├── requirements.txt        # Python dependencies
│   ├── library_system/         # Django project configuration
│   │   ├── settings.py
│   │   ├── urls.py
│   │   ├── auth_views.py
│   │   └── tests/              # API integration tests
│   ├── books/                  # Book catalog module
│   ├── members/                # Member management module
│   ├── loans/                  # Loan lifecycle module
│   │   ├── strategies.py       # Strategy pattern
│   │   ├── observers.py        # Observer pattern
│   │   ├── services.py         # Domain service layer
│   │   └── tests/              # Unit tests
│   ├── reports/                # Report generation module
│   │   └── factories.py        # Factory pattern
│   └── notifications/          # Email service abstraction
├── frontend/
│   ├── package.json
│   ├── vite.config.js
│   ├── tailwind.config.js
│   └── src/
│       ├── App.jsx             # Main application component
│       ├── api.js              # API utility module
│       └── pages/              # Page components
├── .gitignore
└── README.md
```

---

## License

This project is developed for academic purposes as part of CSE 3206 — Software Engineering Sessional at RUET.
