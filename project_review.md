# JSM Shiksha Academy ERP — Project Review Report

**Reviewed On:** 2026-07-16  
**Reviewer:** Antigravity AI  
**Project Path:** `E:\deploy\jsm_production`  
**Repository:** `vishalbhakt/jsmpro`  
**Tech Stack:** Django 6.0.5 · DRF 3.17.1 · Bootstrap 5 · SQLite / PostgreSQL · Gunicorn · WhiteNoise

---

## Executive Summary

JSM Shiksha Academy ERP is a **production-grade monolithic Django 6** school management system covering three user portals (Admin, Teacher, Student), a public-facing CMS, a payments ledger, attendance tracking, learning management, and a full REST API layer. The project is functionally rich and architecturally reasonable for a single-institution deployment. This document outlines its key strengths and the areas that need attention before long-term scale or public production use.

---

## ✅ PROS — Strengths of the Project

### 1. Architecture & Code Organisation
- **Clean Domain-Driven App Structure**: The project is split into focused Django apps — `users`, `academics`, `attendance`, `finance`, `learning`, `communication`, `cms`, `verification`, `dashboard` — each with a clear single responsibility. This keeps models, views, and serializers well-separated and easy to navigate.
- **Dual API + Template Surface**: The project simultaneously serves **server-side HTML views** (for the browser dashboard) and a **REST API** (via Django REST Framework). This dual surface means the same data models power both the portal UI and any future mobile/frontend app without code duplication.
- **Custom User Model with RBAC**: The `User` model correctly extends `AbstractUser` with a `role` field (`admin`, `teacher`, `student`) and uses Django's built-in session authentication augmented with a custom `EmailOrUsernameModelBackend` — allowing login via either email or username, which is a solid UX improvement over the default.
- **Profile Auto-Creation via Signals**: `StudentProfile` and `TeacherProfile` records are automatically created on user save using Django `post_save` signals, preventing orphaned user accounts.
- **Profile Completion Tracking**: Both `StudentProfile` and `TeacherProfile` include a `completion_percentage` property and `is_profile_complete` flag, driven by a detailed field checklist. The `ProfileOnboardingMiddleware` restricts access to sensitive portal pages until the profile is complete — a thoughtful UX guard.
- **Comprehensive Finance Module**: The `FeePlan` → `StudentFee` → `Payment` model chain is well-designed. It auto-calculates totals, supports multiple fee components (tuition, transport, exam, etc.), applies discounts/scholarships, and tracks payment status (`Unpaid`, `Partial`, `Paid`) through computed properties — clean and auditable.
- **JWT + Session Dual Authentication**: The REST API supports both session cookies (for browser) and JWT tokens (for future mobile apps), with token rotation, refresh, and blacklisting configured.
- **WhiteNoise + Gunicorn for Production**: Static assets are served via compressed WhiteNoise, and Gunicorn is the WSGI server — a proper production setup requiring no Nginx dependency for static files.
- **Environment Variable Configuration**: Settings are cleanly driven by environment variables with sensible defaults (via the custom `env()` / `env_bool()` helpers), and PostgreSQL is activated automatically when `DATABASE_URL` or `POSTGRES_DB` is set.
- **API Throttling**: DRF throttle classes are configured for both anonymous (`60/min`) and authenticated (`600/min`) users, protecting against brute-force and abuse.
- **Comprehensive Admin Templates**: The admin dashboard covers 41 page templates across all modules with a cohesive premium UI — Users, Students, Teachers, Classes, Subjects, Attendance, Assignments, Notes, Videos, Quizzes, Results, Fee Plans, Payments, Announcements, Notifications, Gallery, Facilities, Courses, Contact Messages, Inquiries, ID Cards, and more.
- **PDF & ID Card Module**: A dedicated `verification` app and `id_card_print.html` / `receipt_pdf.html` template exist for generating printable ID cards and payment receipts — a practical real-world feature.
- **Attendance Bulk Processing**: Attendance sessions support bulk write operations, which is performant compared to row-by-row inserts.
- **Global Single Modal Pattern**: Profile modals (Teacher/Student) were refactored to use a single DOM-level modal populated by JavaScript `data-*` attributes, reducing DOM bloat caused by per-row modal generation inside template loops.

### 2. Security (Strengths)
- `SECURE_PROXY_SSL_HEADER`, `SECURE_SSL_REDIRECT`, `SESSION_COOKIE_SECURE`, `CSRF_COOKIE_SECURE`, and `X_FRAME_OPTIONS = "DENY"` are all present and configurable via environment variables.
- CSRF middleware is enabled and correctly ordered.
- The `@login_required` decorator is applied across all protected views.
- Role-based access checks are applied at the view function level.

### 3. Developer Experience
- Ships with a `seed_demo` management command for fast local setup with demo users.
- `.env.example` is committed, documenting all required environment variables without leaking real secrets.
- Clean SQLite (dev) → PostgreSQL (prod) switching without manual settings changes.
- Comprehensive `README.md` (416 lines): architecture, folder structure, portal workflows, deployment guide, troubleshooting, and production checklist.
- Basic API smoke test suite (`users/tests.py`) covering auth token flow, role-scoped data access, and public inquiry creation.

---

## ❌ CONS — Areas for Improvement

### 1. Security Risks

| Issue | Risk Level | Details |
|---|---|---|
| **Hardcoded Insecure Secret Key** | 🔴 Critical | `settings.py` has a hardcoded fallback `"django-insecure-local-jsm-shiksha-academy"`. If `.env` is not set in production, this insecure key is live. Should raise `ImproperlyConfigured` if not explicitly set. |
| **DEBUG defaults to True** | 🔴 Critical | `DEBUG = env_bool("DJANGO_DEBUG", True)` — any misconfigured production deployment will expose tracebacks, internal paths, and SQL queries to the public. Default must be `False`. |
| **`db.sqlite3` not in `.gitignore`** | 🟡 Medium | The SQLite database file is not listed in `.gitignore`, meaning real or demo user PII data can be committed to the public GitHub repository. Add `backend/db.sqlite3` immediately. |
| **Stat inflation in public views** | 🟡 Medium | `views.py` artificially inflates public-facing statistics (`students + 120`, `teachers + 15`). This misleads prospective parents/students about institution size. |
| **XSS risk in JS modal injection** | 🟡 Medium | Several JS-driven profile modals use `innerHTML` to inject user-supplied names and emails without sanitisation. A crafted `<script>` payload in a student name could execute. Use `textContent` or `DOMPurify`. |
| **Role check via raw strings** | 🟠 Low | Views use `request.user.role == 'admin'` raw string comparisons. A future model change or typo silently breaks access control. Use `User.Roles.ADMIN` enum values. |

### 2. Performance Issues

| Issue | Risk Level | Details |
|---|---|---|
| **N+1 Query Risk** | 🔴 High | Several list views load related fields (classroom, user, profile) via separate ORM queries per row. Needs a comprehensive `select_related` / `prefetch_related` audit with Django Debug Toolbar. |
| **Monolithic `views.py`** | 🟡 Medium | `dashboard/views.py` is **3,058 lines and ~120 KB**. Slow to parse, maintain, and conflict-prone during team edits. Should be split into `admin_views.py`, `teacher_views.py`, `student_views.py`. |
| **Heavy Page Payloads** | 🟡 Medium | The Payments page and Student database generate 600–850KB HTML responses (no pagination). Should implement server-side paginated queries with a configurable page size. |
| **No Caching Layer** | 🟡 Medium | Dashboard stats (totals, counts, aggregates) are recalculated live on every page request. No Redis, Memcached, or Django cache framework is configured. |
| **Media not served in production** | 🟠 Low | Media files are only routed in `DEBUG=True` mode. Without Nginx or S3, profile photos and uploads will 404 in production. |

### 3. Code Quality

| Issue | Details |
|---|---|
| **No type hints** | The entire codebase uses no Python type annotations, reducing IDE support and making large-scale refactoring risky. |
| **Minimal test coverage** | Only 4 smoke tests exist. No tests for finance calculations, attendance bulk writes, form validation, middleware redirects, or any dashboard view. Regressions can be introduced silently. |
| **Inline CSS `!important` proliferation** | Templates use long repeated `style="..."` attribute blocks with `!important` overrides. Global theme changes become extremely tedious. A centralised custom CSS class file would reduce duplication. |
| **Mixed CSS frameworks** | The project mixes **Bootstrap 5** classes (`.badge`, `.btn`, `d-flex`) with **Tailwind CSS** utility classes (`rounded-full`, `text-xs`, `bg-blue-50`) without Tailwind's build step. Without a Tailwind CDN `<script>`, many Tailwind classes silently fail to render. |
| **`assigned_class` stored as `CharField`** | `TeacherProfile.assigned_class` is a plain string rather than a `ForeignKey` to `ClassRoom`. This breaks reporting, filtering, and referential integrity. |
| **`subjects_taught` stored as plain string** | `subjects_taught` is a `CharField(max_length=200)` instead of a `ManyToManyField` to `Subject`. This breaks any subject-level query or cross-reference. |
| **Hardcoded academic year** | Multiple model fields default to `"2026-27"`. Requires manual migration or data fixes every academic year. |
| **No soft delete** | Students, Teachers, and Payments are hard-deleted. Accidentally removing a student permanently destroys all their attendance, results, and payment records. A soft-delete pattern (e.g. `is_deleted` flag) is strongly recommended. |

### 4. Frontend / UX

| Issue | Details |
|---|---|
| **Dark mode not implemented** | The `theme` field on `User` collects light/dark preference but is never applied to any CSS. Effectively a dead field. |
| **Modal accessibility gaps** | Profile modals do not trap keyboard focus, set `aria-modal="true"`, or manage `aria-label` attributes. Screen reader users cannot properly navigate them. |
| **No loading states** | Clicking action buttons (View Profile, Collect / View) shows no spinner or skeleton loader, causing perceived lag during server round-trips. |
| **Broken gallery images** | `classroom-session.jpg` and `k-literacy.jpg` referenced in gallery items return HTTP 404, showing broken placeholder boxes on the public Gallery page. |

### 5. DevOps & Deployment

| Issue | Details |
|---|---|
| **No process manager documented** | Gunicorn with no Supervisor/systemd watchdog is described. A crashed Gunicorn process takes the site offline indefinitely. |
| **No logging configuration** | `settings.py` has no `LOGGING` dict. Production errors go only to Gunicorn's stderr with no rotation, alerting, or structured logging. |
| **No email backend** | No `EMAIL_HOST`, `EMAIL_PORT`, or `EMAIL_USE_TLS` settings are configured. Password reset and notification emails silently fail. |
| **No Docker setup** | No `Dockerfile` or `docker-compose.yml` exists. Onboarding requires manual Python environment setup and version management. |
| **SQLite/PostgreSQL parity gap** | SQLite's `database is locked` concurrency error is explicitly documented as a known issue. Developers should use PostgreSQL locally to avoid environment-specific bugs. |

---

## Summary Scorecard

| Category | Score | Notes |
|---|---|---|
| Architecture | 8 / 10 | Clean app separation, dual API/template surface |
| Security | 5 / 10 | Critical insecure key & DEBUG defaults; XSS risk |
| Performance | 6 / 10 | N+1 risks, no caching, heavy page payloads |
| Code Quality | 6 / 10 | Monolithic views.py, mixed CSS frameworks, no types |
| Test Coverage | 3 / 10 | Only 4 smoke tests across the entire system |
| Frontend / UX | 7 / 10 | Premium design, some accessibility and UX gaps |
| DevOps | 4 / 10 | No Docker, no logging, no email, no process manager |
| Documentation | 9 / 10 | Excellent README; inline code comments adequate |

**Overall: 6 / 10 — A solid, feature-complete ERP foundation. Needs hardening before high-traffic production deployment.**

---

## Top 5 Priority Recommendations

1. 🔴 **Fix `SECRET_KEY` default** — Remove the insecure fallback. Raise `ImproperlyConfigured` if not provided via environment.
2. 🔴 **Set `DEBUG` default to `False`** — Prevent accidental debug mode in production.
3. 🔴 **Add `backend/db.sqlite3` to `.gitignore`** — Stop committing user data to the public repository.
4. 🟡 **Split `dashboard/views.py`** — Break the 3,058-line file into role-scoped modules for maintainability.
5. 🟡 **Fix `assigned_class` and `subjects_taught` model fields** — Replace string fields with proper `ForeignKey` and `ManyToManyField` before more data accumulates.

---

*This report was generated via automated code review of the full project codebase, template inventory, settings, models, views, and server logs. It does not include runtime profiling or penetration testing.*
