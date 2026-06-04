# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — Start Vite dev server
- `npm run build` — Production build
- `npm run lint` — ESLint (flat config, JS/JSX only)
- `npm run preview` — Preview production build

No test suite is configured.

## Architecture

React 19 SPA using Vite 7, Tailwind CSS 4, and React Router 7. No TypeScript — plain JSX throughout.

### State Management

Two layers of React Context living in two differently-named directories:

- **`src/context/`** — Core user state: `AuthContext` (login/register/localStorage session), `JobContext` (applied/saved/posted jobs + CRUD), `ThemeContext` (dark/light toggle)
- **`src/contexts/`** — Async data caches: `JobsDataContext` (job list with 5-min TTL + auto-refresh on focus), `CompaniesContext` (same pattern)

Provider nesting order (in `App.jsx`): `AuthProvider` → `JobsDataProvider` → `JobProvider` → `CompaniesProvider` → `ThemeProvider`

`JobContext` depends on both `AuthContext` (for the current user) and `JobsDataContext` (to update `applicationsCount` optimistically), so it must be nested inside both.

### Data Layer

All data is **mock only** — no real backend exists. The services in `src/services/` simulate async API calls using `delay()` from `src/utils/delay.js` (random 200–800 ms).

Job data flows from two sources merged at runtime:
- Static mock jobs from `src/data/mockData.js` (read-only)
- Employer-posted jobs from `localStorage.globalPostedJobs` (mutable)

`companyService.fetchAllJobs()` merges both. `JobsDataContext` calls it and caches the result.

Profile data (job seeker) is stored as `userProfile_{userId}` in localStorage and includes base64-encoded profile picture and resume files.

**Profile completion gate**: job seekers cannot apply for jobs until `user.profileComplete === true`. This flag is set on the `AuthContext` user object after checking whether the `profileService` record has all required fields (`jobTitle`, `location`, `experienceLevel`, `professionalBio`, `profilePictureName`, `resumeName`).

### localStorage Keys

| Key | Owner |
|-----|-------|
| `jobPortalUser` | Auth session (parsed user object) |
| `authToken` | Fake JWT string |
| `registeredUsers` | Self-registered accounts |
| `globalPostedJobs` | All employer-posted jobs |
| `jobApplications_{userId}` | Applications per job seeker |
| `savedJobs_{userId}` | Saved jobs per job seeker (via `savedJobService`) |
| `postedJobs_{userId}` | Posted jobs per employer |
| `allApplications_{userId}` | Employer's application cache |
| `userProfile_{userId}` | Job seeker profile (incl. base64 files) |
| `job-portal-theme` | `"light"` or `"dark"` |

### Routing & Roles

Three roles with route protection via `ProtectedRoute` component:
- **ROLE_JOB_SEEKER** — `/profile`, `/applied-jobs`, `/saved-jobs`
- **ROLE_EMPLOYER** — `/post-job`, `/employer/jobs`, `/job-applicants/:jobId`
- **ROLE_ADMIN** — `/admin`, `/admin/companies`, `/admin/employers`, `/admin/contact-messages`

Unauthenticated users are redirected to `/login` with the original path in `location.state.from`. Wrong-role users are redirected to `/`.

### Test Credentials

| Role | Email | Password |
|------|-------|----------|
| Employer | `employer@company.com` | `employer123` |
| Employer | `hr@startup.com` | `hr123` |
| Job Seeker | `jobseeker@email.com` | `jobseeker123` |
| Job Seeker | `candidate@email.com` | `candidate123` |
| Admin | `admin@portal.com` | `admin123` |

Self-registered accounts are stored in `localStorage.registeredUsers` and always get `ROLE_JOB_SEEKER`.

### Key Libraries

- Font Awesome + Lucide React for icons
- react-toastify for notifications

### ESLint

Flat config (`eslint.config.js`). The `no-unused-vars` rule ignores variables starting with uppercase or underscore (`varsIgnorePattern: '^[A-Z_]'`).
