# Mini Truck CMS — SINOTRUK Parts Catalog

[Đọc bản tiếng Việt](README-vi.md)

![React](https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=111111)
![Vite](https://img.shields.io/badge/Vite-646CFF?style=flat-square&logo=vite&logoColor=white)
![TypeScript](https://img.shields.io/badge/Admin_TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Express](https://img.shields.io/badge/Express-000000?style=flat-square&logo=express&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase_Storage-3ECF8E?style=flat-square&logo=supabase&logoColor=white)
![Sharp](https://img.shields.io/badge/Sharp-99CC00?style=flat-square)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)

Mini Truck CMS is a product-catalog and admin system for SINOTRUK/HOWO truck parts. Its real problem is not just showing product cards; it has to keep thousands of part codes searchable, map each part to both a part category and one or more vehicle lines, protect product images with controlled watermark downloads, and let an operator import or correct catalog data without touching source code.

The current repository is a transition system. The active public site is a Vite React app. The active admin is a separate React/TypeScript app mounted under `/secret`. Image upload and watermarking exist in two runtime shapes: Vercel serverless functions backed by Supabase Storage, and a Docker/Express/PostgreSQL runtime under `deploy/`. A Laravel/MySQL system is kept in `backend/dongha.sinotruk-hanoi.com` as historical source and domain evidence, not as the primary app path.

## Preview

![Homepage](docs/assets/homepage-current.png)

Admin login preview:

![Admin login](docs/assets/admin-login-current.png)

## What It Does

- Public catalog site with homepage, product listing, product detail, technical article catalog, image library, about, and contact pages.
- Product search by name, product code, manufacturer code, part category, and vehicle line.
- Admin panel for product CRUD, category/vehicle-line management, article publishing, image library, company settings, and watermark settings.
- Excel import/export for bulk product maintenance, including embedded image handling in the import modal.
- Image proxy and watermark download flow using `Sharp`; normal browsing can serve clean/proxied images while protected downloads request `?watermark=true`.
- Deployment paths for Vercel and for a Docker Compose stack with PostgreSQL, Express API, and Nginx.

## Technical Shape

```mermaid
flowchart LR
  User["Customer browser"] --> Public["React public site"]
  Admin["Admin browser /secret"] --> AdminUI["React TS admin"]
  Public --> API["/api routes"]
  AdminUI --> API
  API --> Store["Supabase Storage or local uploads"]
  API --> DB["PostgreSQL catalog tables"]
  API --> Sharp["Sharp watermark pipeline"]
  Legacy["Laravel 5.4 + MySQL archive"] -.domain history.-> DB
```

The public and admin apps deliberately share the catalog API surface but keep separate UI builds. The split keeps the public catalog light while the admin can carry heavier tooling such as Editor.js, ExcelJS, XLSX, and Recharts.

## Tech Stack

| Layer | Stack |
|---|---|
| Public frontend | ![React](https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=111111) ![Vite](https://img.shields.io/badge/Vite-646CFF?style=flat-square&logo=vite&logoColor=white) ![GSAP](https://img.shields.io/badge/GSAP-88CE02?style=flat-square) ![Framer Motion](https://img.shields.io/badge/Framer_Motion-0055FF?style=flat-square) ![Three.js](https://img.shields.io/badge/Three.js-000000?style=flat-square&logo=threedotjs&logoColor=white) |
| Admin frontend | ![React](https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=111111) ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white) ![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=flat-square&logo=tailwindcss&logoColor=white) ![Editor.js](https://img.shields.io/badge/Editor.js-111111?style=flat-square) |
| API and image processing | ![Express](https://img.shields.io/badge/Express-000000?style=flat-square&logo=express&logoColor=white) ![Vercel](https://img.shields.io/badge/Vercel_Functions-000000?style=flat-square&logo=vercel&logoColor=white) ![Sharp](https://img.shields.io/badge/Sharp-99CC00?style=flat-square) ![Multer](https://img.shields.io/badge/Multer-333333?style=flat-square) |
| Data and storage | ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white) ![Supabase](https://img.shields.io/badge/Supabase_Storage-3ECF8E?style=flat-square&logo=supabase&logoColor=white) ![Cloudinary](https://img.shields.io/badge/Cloudinary-3448C5?style=flat-square&logo=cloudinary&logoColor=white) |
| Deploy | ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white) ![Nginx](https://img.shields.io/badge/Nginx-009639?style=flat-square&logo=nginx&logoColor=white) |
| Legacy evidence | ![Laravel](https://img.shields.io/badge/Laravel_5.4-FF2D20?style=flat-square&logo=laravel&logoColor=white) ![MySQL](https://img.shields.io/badge/MySQL_5.7-4479A1?style=flat-square&logo=mysql&logoColor=white) |

## Run Locally

Public site:

```bash
npm install
npm run dev
```

Admin UI:

```bash
cd admin_ui
npm install
npm run dev
```

Docker runtime:

```bash
cd deploy
docker-compose up -d
```

The browser apps expect an API at `/api` or `VITE_API_URL`. Without a running API, product and admin data surfaces degrade to empty/error states even if the frontend builds successfully.

## Read Next

- [Technical specification](docs/01-technical-specification.md)
- [Product and admin workflows](docs/02-product-and-admin-workflows.md)
- [Operations, risks, and maintenance](docs/03-operations-and-maintenance.md)
