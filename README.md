# PhishGuard SOC

PhishGuard SOC is a defensive cybersecurity SaaS-style project for phishing email reporting, automated indicator extraction, rule-based risk scoring, SOC triage, audit logging, and PDF incident reports.

## Problem Statement

Employees often forward suspicious emails to security teams through scattered channels. PhishGuard SOC centralizes those reports, safely parses email metadata and content, assigns risk, lets analysts document the investigation, and gives admins visibility into users, detection rules, reports, and audit activity.

## Features

- Employee registration, login, suspicious email submission, and personal report tracking
- SOC incident queue with status, severity, verdict, notes, IOCs, triggered rules, and PDF generation
- Admin user management, rule management, generated report listing, audit logs, and dashboard statistics
- Safe parser for headers, URLs, domains, IPs, email addresses, authentication results, attachment names, and risky extensions
- Rule-based phishing risk score from 0 to 100 with Low, Medium, High, and Critical severities
- Defensive threat enrichment records for domains, URLs, IPs, hashes, and brand impersonation indicators
- Explainable detection output with score summaries, evidence items, triggered rule impact, and score contribution charts
- QR phishing analysis for uploaded images and email attachments, with payload extraction only and no link visits
- Brand watchlist management with lookalike, punycode, homoglyph, keyword, and subdomain-abuse detection
- BEC and payment fraud workflow with structured financial-risk fields and analyst verification checklist
- Campaign correlation engine that groups incidents by subject similarity, sender domain, IOC overlap, attachment hashes, brand match, and time window
- MITRE ATT&CK-style and NIST incident lifecycle mappings stored per incident and included in PDF reports
- Executive dashboard for reports over time, SLA breaches, targeted brands, sender domains, campaign severity, verdict rates, and response timing
- Safe rule-based SOC copilot placeholder that does not call external AI APIs and treats email content as untrusted evidence
- SOAR-style playbook simulator for defensive actions such as block domain, search mailbox, notify users, reset password, escalate, and export to SIEM
- Enterprise case queues with SLA timers, campaign labels, queue status, and assignment metadata
- SSO, email gateway, threat intelligence, sandbox, and SIEM integration management screens
- Bulk campaign import, attachment hash capture, sandbox metadata-ready IOCs, and SIEM export queues
- Docker Compose stack with FastAPI, React/Vite, and PostgreSQL
- Pytest coverage for auth, RBAC, analyzer logic, incident creation, PDF generation, and admin rule updates

## Architecture

```text
React + Vite UI
  -> Axios JWT API calls
FastAPI backend
  -> SQLAlchemy ORM
  -> PostgreSQL in Docker, SQLite-friendly local tests
  -> Safe email analyzer and rule engine
  -> ReportLab PDF incident reports
```

## Tech Stack

- Backend: Python, FastAPI, SQLAlchemy, PostgreSQL, JWT, passlib/bcrypt, Pydantic, pytest
- Frontend: React, Vite, Tailwind CSS, Recharts, Axios, React Router, lucide-react
- Other: Docker, docker-compose, Alembic scaffold, PDF report generation, `.env` configuration

## Folder Structure

```text
phishguard-soc/
  backend/
    app/
      main.py
      database.py
      models.py
      schemas.py
      auth.py
      security.py
      routers/
      services/
      tests/
    alembic/
    requirements.txt
    Dockerfile
  frontend/
    src/
      components/
      context/
      pages/
      services/
      App.jsx
      main.jsx
    package.json
    Dockerfile
  samples/
  docker-compose.yml
  .env.example
  README.md
```

## Run With Docker

```bash
cd phishguard-soc
cp .env.example .env
docker compose up --build
```

- Frontend: `http://localhost:5173`
- Backend API: `http://localhost:8000`
- API docs: `http://localhost:8000/docs`

## Run Backend Manually

```bash
cd phishguard-soc/backend
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
set DATABASE_URL=sqlite:///./phishguard.db
set JWT_SECRET_KEY=local-development-secret
uvicorn app.main:app --reload
```

On macOS/Linux, use `source .venv/bin/activate` and `export` instead of `set`.

## Run Frontend Manually

```bash
cd phishguard-soc/frontend
npm install
npm run dev
```

If your backend is not on `http://localhost:8000`, create `frontend/.env`:

```bash
VITE_API_URL=http://localhost:8000
```

## Demo Credentials

The backend seeds these accounts on startup:

| Role | Email | Password |
| --- | --- | --- |
| Admin | `admin@phishguard.local` | `AdminPass123!` |
| SOC Analyst | `analyst@phishguard.local` | `AnalystPass123!` |
| Employee | `employee@phishguard.local` | `EmployeePass123!` |

## API Documentation

Key endpoints:

- `POST /auth/register`
- `POST /auth/login`
- `GET /auth/me`
- `POST /reports/submit`
- `GET /reports/my`
- `GET /reports/{id}`
- `GET /incidents`
- `GET /incidents/{id}`
- `PATCH /incidents/{id}/status`
- `PATCH /incidents/{id}/severity`
- `PATCH /incidents/{id}/verdict`
- `POST /incidents/{id}/notes`
- `GET /incidents/{id}/iocs`
- `PATCH /incidents/{id}/bec-checklist/{item_key}`
- `POST /incidents/{id}/copilot-summary`
- `GET /incidents/{id}/playbooks`
- `POST /incidents/{id}/playbooks/run`
- `POST /incidents/{id}/generate-report`
- `GET /reports/{id}/download`
- `GET /campaigns`
- `GET /campaigns/{id}`
- `POST /campaigns/{id}/merge`
- `POST /campaigns/{id}/close`
- `GET /admin/users`
- `PATCH /admin/users/{id}/role`
- `GET /admin/rules`
- `PATCH /admin/rules/{id}`
- `GET /admin/brand-watchlist`
- `POST /admin/brand-watchlist`
- `PATCH /admin/brand-watchlist/{id}`
- `DELETE /admin/brand-watchlist/{id}`
- `GET /admin/audit-logs`
- `GET /dashboard/employee`
- `GET /dashboard/soc`
- `GET /dashboard/admin`
- `GET /dashboard/executive`

## Submit A Suspicious Email

1. Log in as `employee@phishguard.local`.
2. Open **Submit Email**.
3. Paste one of the fake educational samples from `samples/`, upload it as a `.eml` file, or upload a QR image for safe QR payload extraction.
4. Add the subject, sender, and report reason.
5. Submit the report. The platform creates an incident, extracts IOCs, triggers rules, assigns a risk score, and stores explainable evidence for analyst review.

## Generate An Incident Report

1. Log in as `analyst@phishguard.local` or `admin@phishguard.local`.
2. Open **Incidents** and choose an incident.
3. Add notes, update status, severity, and verdict as needed.
4. Click **Generate PDF**.
5. Download the generated report from the incident detail page or the admin reports page.

## Test The Email Analyzer

Run the automated tests:

```bash
cd phishguard-soc/backend
pytest
```

QR image tests require the optional QR dependencies from `backend/requirements.txt`, including `opencv-python-headless` and `qrcode[pil]`.

Run a quick analyzer check:

```bash
cd phishguard-soc/backend
python -c "from app.services.email_analyzer import parse_email, calculate_risk_score; raw=open('../samples/fake_password_reset.eml').read(); parsed=parse_email(raw); print(calculate_risk_score(parsed))"
```

## Screenshots Placeholder

Add screenshots here after running the app:

- Landing page
- Employee submission page
- SOC incident details page
- Admin dashboard
- PDF incident report

## Resume Bullet Points

- Built PhishGuard SOC, a full-stack defensive phishing triage platform using FastAPI, React, PostgreSQL, JWT auth, and Docker.
- Implemented safe email parsing for headers, URLs, domains, IPs, attachment metadata, authentication results, and phishing keywords without executing files or opening links.
- Designed a rule-based risk engine that maps phishing indicators to scores, severities, verdict suggestions, triggered-rule evidence, and remediation guidance.
- Developed SOC analyst workflows for incident status, severity, verdicts, investigation notes, extracted IOCs, audit logs, and PDF incident reports.
- Created role-based access control for employees, SOC analysts, and admins with protected API endpoints and frontend routes.

## Implemented Enterprise Upgrades

- Optional defensive-only threat intelligence enrichment for domains, URLs, IPs, hashes, and brand indicators
- Explainable phishing detection engine with evidence, triggered rule impact, severity, and verdict suggestions
- Quishing analyzer that decodes QR payloads locally, stores QR IOCs, and never visits QR URLs
- Admin brand watchlist with seeded enterprise and banking brands plus optional college-domain monitoring
- BEC/payment fraud triage workflow with financial-risk extraction and SOC checklist tracking
- Campaign correlation, campaign detail timelines, and campaign filters in the incident queue
- MITRE ATT&CK/NIST incident mapping panels plus PDF report inclusion
- Executive dashboard for SOC business metrics and response performance
- Deterministic SOC copilot summary generator with prompt-injection safety notes
- SOAR playbook simulator with stored action results and audit logging
- SSO configuration records for enterprise identity providers
- Case assignment queues and SLA timers
- Email gateway integration records for internal reporting buttons
- Attachment hashing and sandbox metadata ingestion fields without local execution
- Bulk incident import and correlation by campaign
- SIEM export queues through syslog, JSON, or webhook connectors
- More advanced HTML URL display parsing and brand impersonation checks

## Commercial Hosting Notes

PhishGuard SOC now includes product-facing pages, pricing-style plan cards, enterprise operations screens, integration configuration records, SLA queue workflows, enrichment history, bulk import, and export queues. These make it a stronger SaaS MVP for authorized defensive services.

Before charging customers in production, replace demo secrets, configure HTTPS, connect a real database backup strategy, add tenant isolation, connect payment/subscription billing, configure real SSO providers, review compliance requirements, and run the full backend/frontend test suites in a healthy Python and Node environment.

## Security Disclaimer

PhishGuard SOC is for authorized defensive security analysis. It does not create phishing kits, send phishing emails, collect credentials, build fake login pages, execute attachments, download external content, or attack external websites.
