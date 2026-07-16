# JSM Shiksha Academy ERP — Version History & Changelog

This file tracks all major updates, releases, and changes to the JSM Shiksha Academy ERP system over time.

> **Note:** This file is stored outside the project directory (`E:\deploy\jsm_docs\`) and is **not tracked by Git**. It serves as a local, private record for deployment notes, hotfixes, and version management.

---

## How to Use This File

Each version entry should include:
- **Version number** (use Semantic Versioning: `MAJOR.MINOR.PATCH`)
- **Release date**
- **Git commit hash** (run `git log --oneline -1` to get the latest)
- **Type of changes** — `feat` (new feature), `fix` (bug fix), `ui` (design/UX), `perf` (performance), `security` (security patch), `chore` (maintenance)
- **Summary of changes made**
- **Deployment notes** (any manual steps required after deploying)

---

## Version Log

---

### v1.5.0 — 2026-07-16
**Git Commit:** *(run `git log --oneline -1` to fill in)*  
**Type:** `ui` · `fix`

#### Changes
- **UI:** Upgraded Status badges on Contact Messages (`/dashboard/admin/contact-messages/`) and Inquiries (`/dashboard/admin/inquiries/`) pages from blank/solid Bootstrap `.badge` capsules to premium **Soft Pill + SVG Icon** design:
  - New / Unread → Blue soft pill with envelope SVG
  - In Progress / Contacted → Amber soft pill with clock SVG
  - Resolved / Admitted → Green soft pill with check-circle SVG
  - Closed → Gray soft pill with check SVG
- **UI:** Removed cartoonish emoji headers (✉️, 📝) from Contact Messages and Inquiries page banners. Replaced with clean SVG icon badges embedded in the navy header card.
- **UI:** Replaced emoji filter labels (`🔍 Search`, `🚦 Status Filter`) with uppercase clean labels (`text-xs font-semibold text-gray-600 uppercase tracking-wider`) using Font Awesome icons.
- **Fix:** Used `display: inline-flex !important` inline CSS to override Bootstrap `.badge`'s `display: inline-block` which was causing the soft pills to render as blank solid capsules.

#### Deployment Notes
- No database migrations required.
- No environment variable changes required.
- Static file collection may be needed if CSS was changed: `python manage.py collectstatic --noinput`

---

### v1.4.0 — 2026-07-14
**Git Commit:** `6fd1072`  
**Type:** `ui` · `fix`

#### Changes
- **UI:** Fixed routing bugs on Admin Overview Quick Action buttons — "Add Class", "Create Subject", "Add User", "Publish Announcement" now correctly route to custom ERP portal routes instead of the raw `/admin/...` Django backend.
- **UI:** Modernised Payments Ledger (`/dashboard/admin/payments/`) header banner: refactored "Configure Class Fees" and "Collection Reports" buttons to high-contrast SaaS-style design.
- **UI:** Fixed Payments Ledger stats cards overflow — large currency values (e.g. ₹829,000) no longer clip against card borders.
- **UI:** Upgraded all Fee Plans page (`/dashboard/admin/fee-plans/`) labels, inputs, and buttons — removed emoji, standardised filter control heights, replaced "Configure New Structure" with a clean dark-slate button with plus SVG icon.
- **UI:** Upgraded Actions column in Payments Ledger table — compact "Collect / View" button with print icon aligned in a flex row.
- **UI:** Upgraded Status badges on Payments Ledger table — Paid (green), Partial (amber), Unpaid (red) soft pill + SVG icons.
- **Perf:** Refactored Teacher and Student list views to use a single global DOM modal populated via JS instead of generating one modal per table row.
- **Perf:** Pre-fetched related fields in `admin_teachers` view to reduce N+1 database query loads.

#### Deployment Notes
- No database migrations required.

---

### v1.3.0 — 2026-07-10
**Git Commit:** *(historical)*  
**Type:** `ui` · `feat`

#### Changes
- **UI:** Global "Clean Soft Pill + SVG Icon" Status badge design applied across Student Database, Teacher Roster, Users list, and modal JS badge updates.
- **UI:** Replaced all heavy green block + checkmark emoji (✅) status indicators across the dashboard.
- **feat:** Implemented Single Global Modal Pattern for Student Profile and Teacher Profile modals — moved modal HTML to bottom of templates, outside all loops and table wrappers.
- **Fix:** Resolved Teacher Profile modal vertical clipping issue — modal was rendering inside scrollable dashboard content hierarchy causing top section (Avatar, Name, Email) to clip above viewport.

#### Deployment Notes
- No database migrations required.

---

### v1.2.0 — 2026-07-01
**Git Commit:** *(historical)*  
**Type:** `feat` · `fix`

#### Changes
- **feat:** Admin Classroom Management CRUD fully integrated — modal-based classroom creation, updating, and deletion in the Admin Portal.
- **feat:** ID Card verification module and print templates added.
- **Fix:** Resolved `TypeError` in Announcements admin workflow.
- **Fix:** Verified and corrected dynamic real data loading on the Overview portal.

#### Deployment Notes
- Run migrations: `python manage.py migrate`

---

### v1.1.0 — 2026-06-29
**Git Commit:** *(historical)*  
**Type:** `feat` · `chore`

#### Changes
- **feat:** Initial full portal deployment — Student, Teacher, Admin portals live.
- **feat:** Public CMS (Home, About, Academics, Courses, Gallery, Facilities, Contact, Admission) deployed.
- **feat:** Finance module (Fee Plans, Payments, Receipts, Student Ledger) deployed.
- **feat:** Attendance bulk processing and sessions deployed.
- **feat:** Learning module (Assignments, Notes, Video Lectures, Quizzes, Results) deployed.
- **feat:** REST API layer with JWT authentication and DRF ViewSets deployed.
- **chore:** Pushed initial production-ready codebase to GitHub.

#### Deployment Notes
- Run: `python manage.py migrate`
- Run: `python manage.py seed_demo` to populate demo data.
- Run: `python manage.py collectstatic --noinput`
- Set `DJANGO_SECRET_KEY`, `DJANGO_DEBUG=False`, and `DATABASE_URL` in production `.env`.

---

## Upcoming / Planned Changes

| Priority | Type | Description |
|---|---|---|
| 🔴 High | security | Remove hardcoded `SECRET_KEY` fallback; raise `ImproperlyConfigured` if not set |
| 🔴 High | security | Change `DEBUG` default from `True` to `False` in `settings.py` |
| 🔴 High | security | Add `backend/db.sqlite3` to `.gitignore` |
| 🟡 Medium | perf | Split `dashboard/views.py` (3,058 lines) into `admin_views.py`, `teacher_views.py`, `student_views.py` |
| 🟡 Medium | perf | Add server-side pagination to Student Database and Payments Ledger list views |
| 🟡 Medium | perf | Configure Redis/Memcached caching for dashboard summary stats |
| 🟡 Medium | model | Replace `TeacherProfile.assigned_class` CharField with `ForeignKey` to `ClassRoom` |
| 🟡 Medium | model | Replace `TeacherProfile.subjects_taught` CharField with `ManyToManyField` to `Subject` |
| 🟠 Low | feat | Implement dark mode CSS using the existing `User.theme` preference field |
| 🟠 Low | feat | Add soft-delete (archive) pattern for Student and Teacher records |
| 🟠 Low | ux | Add loading spinners/skeleton loaders on View Profile and Collect/View action buttons |
| 🟠 Low | devops | Add `Dockerfile` and `docker-compose.yml` for local development parity |
| 🟠 Low | devops | Configure Django `LOGGING` dict with rotating file handlers |
| 🟠 Low | devops | Configure email backend (`EMAIL_HOST`, `EMAIL_PORT`, etc.) for password resets |
| 🟠 Low | fix | Fix broken gallery images: `classroom-session.jpg` and `k-literacy.jpg` (HTTP 404) |
| 🟠 Low | ux | Improve modal accessibility — add `aria-modal`, `aria-label`, and keyboard focus trapping |
