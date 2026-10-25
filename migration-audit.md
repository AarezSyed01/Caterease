# CaterEase migration audit

## Scope and evidence
Audit of the current source and accessible saved files on 25 October 2026. No application screens, API behavior, credentials or live records were changed. The exact requested stack will be delivered as a separate local HTML/CSS/JavaScript/Bootstrap + Node.js/Express/MySQL application; the Lovable-hosted project must retain TanStack Start and its managed integrations.

The earlier downloadable Express/MySQL package is not available in the current Files mount or inspected temporary locations. Comments and architecture notes describe it, but no backend directory, Express entrypoint or MySQL adapter exists in the current source. Previous conversation claims are not evidence that this package can currently be verified.

## Dependency inventory
| Location | Dependency or purpose | Migration treatment |
| --- | --- | --- |
| `package.json` | `@supabase/supabase-js`, `postgres`, `drizzle-orm`, `drizzle-kit`; TanStack/React/Vite shell | Exclude from the standalone submission, retain in hosted preview |
| `src/lib/cloud-db.server.ts` | Service-client CRUD adapter and provisioning RPCs | Replace with parameterized `mysql2` adapter and real transactions locally |
| `src/integrations/supabase/` | Generated browser/admin clients, types, auth middleware/attacher, preview storage and cron helper | Do not copy to the local package; do not edit generated hosted files |
| `src/start.ts` | Cloud auth attachment and server-function CSRF middleware | Express must provide its own cookie/session and CSRF protection |
| `src/routes/api/public/$.ts` | REST entrypoint dynamically loading the shared dispatcher and Cloud adapter | Express mounts the same API contract under `/api` |
| `src/server.ts` | TanStack server entry and error response wrapper | Not needed locally |
| `drizzle/migrations/*.sql`, `drizzle.config.ts` | PostgreSQL schema, constraints and provisioning functions | Translate into versioned MySQL schema/migrations |
| `public/app/config.js` | Frontend API base is `/api/public` | Local copy targets `/api` |
| `public/app/*.js` | REST requests; custom JWT session state | Update request/auth plumbing only, retain layouts and styling |

No application `createServerFn` or deployed Edge Function implementation was found. Cloud token middleware is separate from the actual CaterEase REST login.

### Configuration names only
Current configuration references: `SUPABASE_PROJECT_ID`, `SUPABASE_URL`, `SUPABASE_PUBLISHABLE_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `VITE_SUPABASE_PROJECT_ID`, `VITE_SUPABASE_URL`, `VITE_SUPABASE_PUBLISHABLE_KEY`, `LOVABLE_DB_MIGRATION_URL`, `JWT_SECRET`, `ADMIN_SETUP_KEY`, `LOVABLE_CRON_SECRET`, `LOVABLE_CRON_SECRET_PREVIOUS`. Values are deliberately omitted. Generated Cloud configuration will not be distributed with the standalone application.

## Authentication and authorization
- Existing account passwords are bcrypt hashes. Login accepts email or legacy username, then issues an eight-hour signed bearer JWT.
- Every protected request verifies the JWT, reloads the user and resolves membership from `business_roles`. Editable `users.role` is ignored.
- Existing roles are owner, admin and manager; owners present as business administrators. Staff are operational records, not a demonstrated staff-login role. Do not add an invented staff login or permission model.
- User creation and settings updates require business administrator permissions. Other protected operations are scoped to the authenticated business.
- Registration creates an account, placeholder business, owner membership and incomplete-details flag. New accounts complete the details page before entering the dashboard; existing completed accounts enter directly.
- Current logout returns success but does not revoke an already issued JWT. Local server-backed sessions should support real logout revocation.
- Current browser storage contains session state; it is not the server's role authority. The local version will use HttpOnly cookies instead of storing authentication tokens in browser storage.

## API and data inventory
Shared code is in `src/core/{auth,routes,services,tenant,util}.js`. All service queries run through `all/get/insert/update/remove/tx`, with optional atomic provisioning adapter methods.

Public APIs: authentication status/setup/login/registration, business information, inquiry submission and booking-status lookup.

Protected APIs: current user/logout/password change; user listing/creation; business details/settings; dashboard/search; customers/events/menus/staff CRUD; booking creation/update/status; guests; staff assignments; payments/receipts; feedback; reports.

Tables to migrate: businesses, users, business_roles, business_settings, legacy settings, customers, events, menus, bookings, guests, staff, staff_assignments, payments and feedback. There is no independently implemented orders module; preserve booking workflows rather than inventing orders. Preserve all six existing report variants.

## Integrity hazards
- PostgreSQL UUID, identity, timestamp, JSON, boolean, checks and procedural functions need explicit MySQL equivalents.
- Preserve primary IDs, bcrypt hashes, legacy business membership, business/key uniqueness and tenant-aware composite foreign keys.
- Preserve parameter validation, booking price snapshots, payment balances, booking transitions, idempotency and deletion protections.
- MySQL provisioning and multistep writes need real transactions and rollback tests. The current Cloud adapter's generic `tx()` does not supply atomicity; only dedicated provisioning RPCs do.
- Session cookies require CSRF controls, strict allowed origins, appropriate SameSite/Secure settings and no wildcard credentialed CORS.
- Concurrency tests must cover duplicate registrations, first-admin setup and payments against a shared balance.
- Source data export must be private and server-only. Exporting password hashes for account preservation must never place them in a public archive or frontend asset.
- Inquiry email delivery remains unconfigured; sender setup and the owner's recipient address are still required. Database migration alone does not enable emails.

## Verification baseline
Ran `bunx vitest run src/test/business-isolation.test.ts src/test/catering-rules.test.ts src/test/app-routing.test.tsx`: **41 tests passed across three files**. These are in-memory/unit tests, not proof of a functioning MySQL migration.

Not yet verified: live schema and data reconciliation; Express startup; actual MySQL CRUD, transactions, tenant constraints and persistence across restart; cookie sessions/CSRF; migrated browser workflows; MacBook installation. Do not describe the migration as complete until these checks pass, and distinguish sandbox checks from tests on the user's MacBook.