<!-- LOVABLE:BEGIN -->
> [!IMPORTANT]
> This project is connected to [Lovable](https://lovable.dev). Avoid rewriting
> published git history — force pushing, or rebasing/amending/squashing commits
> that are already pushed — as it rewrites history on Lovable's side and the
> user will likely lose their project history.
>
> Commits you push to the connected branch sync back to Lovable and show up in
> the editor, so keep the branch in a working state.
<!-- LOVABLE:END -->

## Architecture
- Business logic lives in plain-JS `src/core/*.js` (services, auth, route table) behind a tiny db adapter interface; it is shared verbatim with the downloadable Express/MySQL project, so keep it framework-free.
- The manager UI is a vanilla HTML/Bootstrap app in `public/app/` (required by the capstone brief); `/` redirects there.
- Manager-UI images stay as local files inside `public/app/` (never CDN asset pointers), so the downloadable Express/MySQL package renders them offline.
- Preview API is the splat server route `src/routes/api/public/$.ts` using `src/lib/cloud-db.server.ts` (service role, RLS-locked tables); protected endpoints verify the app's own JWT, not Cloud Auth.
- All protected services receive a tenant-bound adapter derived from verified business_roles membership; public inquiry endpoints bind a validated business link, so callers cannot select another business through request fields.
- Authorization roles live only in business_roles; users.role is deprecated and ignored, preserving existing accounts without trusting editable role fields.
- Cloud business registration uses a service-only atomic SQL function after shared validation and bcrypt hashing; adapter fallbacks use a real transaction to avoid partially created accounts.
- Tenant settings use business_settings with surrogate setting_id and unique business/key; the shared wrapper translates existing setting-key operations without changing service contracts.
- Registration is two-step: account-only signup (name, email, password) creates a placeholder business with business_settings.details_complete='false', then the authenticated PUT /business/details endpoint collects business details and sets it 'true'; the UI routes users with details_complete=false to details.html before the dashboard.
- The downloadable Express backend routes every endpoint through the shared dispatch() from src/core/routes.js (no per-route controllers), so tenant scoping and auth stay identical to the preview.
- Standalone capstone deliveries retain the hosted TanStack app unchanged and use a separate Express/mysql2 package; this satisfies the local stack without breaking Lovable hosting.
- Local Express authentication uses MySQL-backed HttpOnly cookie sessions with server-only signed tokens, CSRF/origin checks and membership revalidation; browser storage is never an authorization authority.
- Local MySQL writes run through real transactions and a serialization lock, preserving multistep rollback and concurrent payment/account integrity; migration exports remain private and outside source archives.
