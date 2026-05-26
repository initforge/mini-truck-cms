# Operations, Risks, and Maintenance

## 1. Local run

Public site:

```bash
npm install
npm run dev
```

Admin site:

```bash
cd admin_ui
npm install
npm run dev
```

Express API:

```bash
cd deploy/server
npm install
npm run dev
```

Docker stack:

```bash
cd deploy
docker-compose up -d
```

Important: the frontend can start without the API, but catalog/admin data will not be meaningful. Use `VITE_API_URL` when API is not served from the same origin.

## 2. Build paths

Root scripts:

| Command | Meaning |
|---|---|
| `npm run build` | Build public site into `dist`. |
| `npm run build:admin` | Install/build `admin_ui`. |
| `npm run build:all` | Build admin, build public, copy admin output into `dist/secret`. |

Admin build:

```bash
cd admin_ui
npm run build
```

Docker build uses `deploy/Dockerfile`. It builds public and admin apps, installs production server dependencies, copies public build to `/app/dist`, and copies admin build to `/app/dist/secret`.

## 3. Environment variables

| Variable | Used by | Purpose |
|---|---|---|
| `VITE_API_URL` | Public/admin frontend | Base API URL, defaults to `/api` in root app and `http://localhost:3002/api` in some admin/public pages. |
| `VITE_SUPABASE_URL` | `api/upload.js`, `api/image.js` | Supabase project URL for serverless storage path. |
| `SUPABASE_SERVICE_ROLE_KEY` | `api/upload.js`, `api/image.js` | Preferred server-side Supabase key for storage upload/download. |
| `VITE_SUPABASE_ANON_KEY` | Vercel API fallback | Fallback key; weaker than service role for storage diagnostics. |
| `DATABASE_URL` | `deploy/server/index.js` | PostgreSQL connection string for Express API. |
| `DB_PASSWORD` | `deploy/docker-compose.yml` | Compose database password. |
| `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET` | `admin_ui/api/upload*.js` | Legacy/admin Cloudinary upload functions. |

## 4. Deployment paths

### Vercel

`vercel.json`:

- `buildCommand`: `npm run build:all`
- `outputDirectory`: `dist`
- `/secret/*` rewrites to `/secret/index.html`
- `/api/*` rewrites to serverless functions
- all other routes rewrite to `/index.html`

Vercel path requires Supabase Storage buckets:

- `original`
- `products` as fallback
- `watermarked`

### Docker/VPS

`deploy/docker-compose.yml` starts:

1. `db`: PostgreSQL 16, initialized from `deploy/sinotruk_full_backup.sql`.
2. `api`: Express server on port `3001`.
3. `nginx`: serves frontend/admin static files, proxies `/api`, and serves `/uploads`.

`deploy/nginx/default.conf` routes:

- `/` -> public SPA
- `/secret` -> admin SPA
- `/api/` -> Express API
- `/uploads/` -> upload volume

## 5. Verification commands

Use these before shipping docs or source changes:

```bash
npm run build
```

```bash
cd admin_ui
npm run build
```

```bash
cd deploy/server
npm test
```

API contract checks in `deploy/tests/api.test.js` require a running API:

```bash
node deploy/tests/api.test.js
```

Playwright tests under `deploy/e2e` require both client and admin URLs to be running and configured through `CLIENT_URL` and `ADMIN_URL`.

## 6. Known risks

### Admin auth is not production-grade

`localStorage` route protection and demo fallback credentials are not sufficient for public admin. Treat this as the first security hardening task before exposing `/secret` outside trusted access.

### Image provider split

The repo contains Supabase Storage, local uploads, and Cloudinary paths. This is manageable only if deployment target is documented per environment. Long term, product images and category thumbnails should be consolidated behind one upload service contract.

### Import is partial-success

Excel import creates products row by row from the browser. A failed row does not rollback previous rows. This is acceptable for small admin batches if the operator reviews the result, but a high-volume catalog should move import to a backend job with a durable report.

### Vehicle deep-link mismatch

`VehicleShowcase.jsx` links to `/products?vehicle=...`, while `Products.jsx` reads `category` from the URL. This should be fixed before relying on homepage vehicle links as a guaranteed public workflow.

### Legacy source can mislead stack claims

Laravel/MySQL is real code in the repo, but it is legacy. Do not present Laravel as the active backend for the React public/admin apps unless a deployment explicitly routes to it.

### Encoding quality in old docs/source comments

Several older Markdown/source strings show mojibake when read from the current checkout. New docs should be written cleanly in UTF-8, but source UI text should be fixed only in a separate source-quality task because it may affect product rendering.

## 7. Cleanup policy

The docs cleanup removes or supersedes:

- old shallow docs under `/docs`
- root local-run guide duplicated by this operations doc
- Laravel boilerplate readme
- generated coverage reports and generated deploy build artifacts
- stale API test report artifacts

Do not delete catalog source images under `images/` or `public/images/` casually. Those are product-facing assets, not docs artifacts.

## 8. Maintenance checklist

Before changing products:

- Check add/edit modals, public product list, public detail, Express product routes, and schema SQL together.
- Preserve `manufacturer_code`, `category_id`, `vehicle_ids`, `image`, `thumbnail`, and `show_on_homepage` semantics.

Before changing images:

- Test normal image display.
- Test right-click/download watermark path.
- Test upload path for product image, gallery image, article image, company logo, and vehicle thumbnail.

Before changing deploy:

- Decide Vercel or Docker first.
- Verify `/secret` routing.
- Verify `/api/image` and `/api/upload`.
- Verify static assets and `/uploads`.

Before changing admin auth:

- Remove demo fallback.
- Hash stored passwords.
- Add server-side authorization.
- Replace localStorage-only trust with a server-verifiable token/session.

