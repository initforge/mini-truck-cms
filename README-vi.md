# Mini Truck CMS — Hệ thống catalog phụ tùng SINOTRUK

[Read in English](README.md)

![React](https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=111111)
![Vite](https://img.shields.io/badge/Vite-646CFF?style=flat-square&logo=vite&logoColor=white)
![TypeScript](https://img.shields.io/badge/Admin_TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Express](https://img.shields.io/badge/Express-000000?style=flat-square&logo=express&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase_Storage-3ECF8E?style=flat-square&logo=supabase&logoColor=white)
![Sharp](https://img.shields.io/badge/Sharp-99CC00?style=flat-square)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)

Mini Truck CMS là hệ thống catalog và quản trị phụ tùng xe tải SINOTRUK/HOWO. Bài toán chính không chỉ là dựng vài thẻ sản phẩm; hệ thống phải làm cho hàng nghìn mã phụ tùng có thể tra cứu được, gắn mỗi phụ tùng với cả nhóm linh kiện lẫn dòng xe tương thích, bảo vệ ảnh sản phẩm bằng luồng tải watermark, và cho người vận hành nhập/sửa dữ liệu catalog mà không cần đụng mã nguồn.

Repo này đang ở trạng thái chuyển tiếp có kiểm soát. Website public là Vite React app. Admin là một React/TypeScript app riêng, mount dưới `/secret`. Upload ảnh và watermark có hai hình thái runtime: Vercel serverless function dùng Supabase Storage, và bộ Docker/Express/PostgreSQL trong `deploy/`. Phần Laravel/MySQL trong `backend/dongha.sinotruk-hanoi.com` là nguồn lịch sử/domain evidence, không phải đường chạy chính của app mới.

## Preview

![Homepage](docs/assets/homepage-current.png)

Preview admin login:

![Admin login](docs/assets/admin-login-current.png)

## Hệ thống làm gì

- Website catalog public gồm trang chủ, danh sách sản phẩm, chi tiết sản phẩm, bài viết kỹ thuật, thư viện ảnh, giới thiệu và liên hệ.
- Tra cứu phụ tùng theo tên, mã sản phẩm, mã nhà sản xuất, nhóm phụ tùng và dòng xe.
- Admin quản lý sản phẩm, danh mục/dòng xe, bài viết, thư viện ảnh, thông tin công ty và cấu hình watermark.
- Import/export Excel cho bảo trì dữ liệu hàng loạt, có xử lý ảnh nhúng trong file import.
- Proxy ảnh và luồng tải watermark bằng `Sharp`; người dùng xem ảnh qua proxy, còn hành động tải ảnh có thể yêu cầu `?watermark=true`.
- Hai hướng triển khai: Vercel cho serverless API, hoặc Docker Compose với PostgreSQL, Express API và Nginx.

## Hình thái kỹ thuật

```mermaid
flowchart LR
  User["Khách truy cập"] --> Public["React public site"]
  Admin["Người quản trị /secret"] --> AdminUI["React TS admin"]
  Public --> API["/api routes"]
  AdminUI --> API
  API --> Store["Supabase Storage hoặc local uploads"]
  API --> DB["PostgreSQL catalog tables"]
  API --> Sharp["Sharp watermark pipeline"]
  Legacy["Laravel 5.4 + MySQL archive"] -.dấu vết nghiệp vụ.-> DB
```

Public site và admin dùng chung bề mặt API catalog, nhưng tách build UI. Public giữ vai trò giới thiệu/tra cứu nhanh; admin gánh các công cụ nặng hơn như Editor.js, ExcelJS, XLSX và Recharts.

## Tech Stack

| Layer | Stack |
|---|---|
| Public frontend | ![React](https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=111111) ![Vite](https://img.shields.io/badge/Vite-646CFF?style=flat-square&logo=vite&logoColor=white) ![GSAP](https://img.shields.io/badge/GSAP-88CE02?style=flat-square) ![Framer Motion](https://img.shields.io/badge/Framer_Motion-0055FF?style=flat-square) ![Three.js](https://img.shields.io/badge/Three.js-000000?style=flat-square&logo=threedotjs&logoColor=white) |
| Admin frontend | ![React](https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=111111) ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white) ![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=flat-square&logo=tailwindcss&logoColor=white) ![Editor.js](https://img.shields.io/badge/Editor.js-111111?style=flat-square) |
| API và xử lý ảnh | ![Express](https://img.shields.io/badge/Express-000000?style=flat-square&logo=express&logoColor=white) ![Vercel](https://img.shields.io/badge/Vercel_Functions-000000?style=flat-square&logo=vercel&logoColor=white) ![Sharp](https://img.shields.io/badge/Sharp-99CC00?style=flat-square) ![Multer](https://img.shields.io/badge/Multer-333333?style=flat-square) |
| Data và storage | ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white) ![Supabase](https://img.shields.io/badge/Supabase_Storage-3ECF8E?style=flat-square&logo=supabase&logoColor=white) ![Cloudinary](https://img.shields.io/badge/Cloudinary-3448C5?style=flat-square&logo=cloudinary&logoColor=white) |
| Deploy | ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white) ![Nginx](https://img.shields.io/badge/Nginx-009639?style=flat-square&logo=nginx&logoColor=white) |
| Legacy evidence | ![Laravel](https://img.shields.io/badge/Laravel_5.4-FF2D20?style=flat-square&logo=laravel&logoColor=white) ![MySQL](https://img.shields.io/badge/MySQL_5.7-4479A1?style=flat-square&logo=mysql&logoColor=white) |

## Chạy local

Website public:

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

Các app frontend cần API ở `/api` hoặc `VITE_API_URL`. Nếu chưa chạy API, giao diện vẫn build được nhưng danh sách sản phẩm/admin data sẽ rơi vào trạng thái rỗng hoặc lỗi tải dữ liệu.

## Đọc tiếp

- [Đặc tả kỹ thuật](docs/01-technical-specification.md)
- [Luồng sản phẩm và quản trị](docs/02-product-and-admin-workflows.md)
- [Vận hành, rủi ro và bảo trì](docs/03-operations-and-maintenance.md)
