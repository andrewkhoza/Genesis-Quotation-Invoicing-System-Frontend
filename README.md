<div align="center">

# Genesis — Quotation & Invoicing Frontend

[![React](https://img.shields.io/badge/React-18-61DAFB.svg?logo=react)](https://react.dev/)
[![Vite](https://img.shields.io/badge/Vite-5-646CFF.svg?logo=vite)](https://vitejs.dev/)
[![JavaScript](https://img.shields.io/badge/JavaScript-ES2022-F7DF1E.svg?logo=javascript)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![License](https://img.shields.io/badge/License-BSD--3--Clause-blue.svg)](LICENSE.md)
[![Status](https://img.shields.io/badge/Status-Live%20%26%20Active-brightgreen.svg)]()

The React/Vite frontend for the Genesis Quotation & Invoicing platform. A multi-tenant SPA that consumes the [Genesis API](../api/README.md) — providing role-scoped dashboards for Owners, Admins, and Staff to manage quotes, invoices, credit notes, customers, and products.

> **This repository contains the README only.** The source code is proprietary. Interested parties are welcome to [get in touch](#contact--demos) for a live walkthrough or code demo.

</div>

---

## Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [User Roles & Views](#user-roles--views)
- [Key Features](#key-features)
- [API Integration](#api-integration)
- [Authentication Flow](#authentication-flow)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Available Scripts](#available-scripts)
- [Deployment](#deployment)
- [License](#license)
- [Contact & Demos](#contact--demos)

---

## Overview

Genesis frontend is a single-page application (SPA) built with React and Vite. It connects exclusively to the Genesis REST API backend via JWT bearer token authentication. All business data is tenant-scoped — each logged-in user sees only their company's data.

The app handles two distinct access patterns:

- **Authenticated routes** — role-gated dashboards for Owners, Admins, and Staff
- **Public routes** — unauthenticated document views for clients accessing shared quote/invoice links

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | React 18 |
| Build Tool | Vite 5 |
| Language | JavaScript (ES2022) |
| Routing | React Router v6 |
| State Management | Zustand |
| HTTP Client | Axios |
| UI Components | shadcn/ui + Tailwind CSS |
| Form Handling | React Hook Form + Zod |
| PDF Preview | iframe embed |
| Notifications | Sonner (toast) |
| Icons | Lucide React |
| Date Handling | Day.js |

---

## Project Structure

```
├── public/                   # Static assets
├── src/
│   ├── api/                  # Axios instance, interceptors, endpoint functions
│   │   ├── client.js         # Base Axios config + JWT interceptor
│   │   ├── auth.js
│   │   ├── quotes.js
│   │   ├── invoices.js
│   │   └── ...
│   ├── components/           # Shared UI components
│   │   ├── ui/               # shadcn/ui primitives
│   │   ├── layout/           # AppShell, Sidebar, Navbar
│   │   └── ...
│   ├── features/             # Feature-scoped modules
│   │   ├── auth/
│   │   ├── dashboard/
│   │   ├── quotes/
│   │   ├── invoices/
│   │   ├── credit-notes/
│   │   ├── customers/
│   │   ├── products/
│   │   ├── staff/
│   │   ├── settings/
│   │   └── public/           # Unauthenticated document views
│   ├── hooks/                # Custom React hooks
│   ├── stores/               # Zustand state stores (auth, tenant)
│   ├── utils/                # Formatters, helpers, constants
│   ├── router/               # Route definitions and guards
│   │   ├── index.jsx
│   │   ├── ProtectedRoute.jsx
│   │   ├── RoleRoute.jsx
│   │   └── GuestRoute.jsx
│   ├── App.jsx
│   └── main.jsx
├── .env.example
├── index.html
├── tailwind.config.js
└── vite.config.js
```

---

## User Roles & Views

| Role | Access |
|---|---|
| **Owner** | Full access — tenant settings, staff management, all documents, billing |
| **Admin** | Staff management, all documents, email templates, sequences |
| **Staff** | Quotes, invoices, customers, products — scoped to own tenant |
| **Public** | Read-only document view via UUID link — no login required |

### Route Structure

```
/login                        → GuestRoute
/forgot-password              → GuestRoute
/reset-password/:token        → GuestRoute

/dashboard                    → ProtectedRoute (all roles)
/quotes                       → ProtectedRoute (all roles)
/quotes/:id                   → ProtectedRoute (all roles)
/invoices                     → ProtectedRoute (all roles)
/invoices/:id                 → ProtectedRoute (all roles)
/credit-notes                 → ProtectedRoute (all roles)
/customers                    → ProtectedRoute (all roles)
/products                     → ProtectedRoute (all roles)
/staff                        → RoleRoute (owner, admin)
/settings                     → RoleRoute (owner)
/settings/email-templates     → RoleRoute (owner, admin)

/p/quote/:uuid                → Public (no auth)
/p/invoice/:uuid              → Public (no auth)
/p/credit-note/:uuid          → Public (no auth)
```

---

## Key Features

<details>
<summary><strong>Dashboard</strong></summary>

- KPI cards — total quotes, invoices, outstanding amounts, overdue count
- Recent activity feed
- Quick-action shortcuts for common tasks

</details>

<details>
<summary><strong>Quote Management</strong></summary>

- Create quotes with dynamic line items, tax selection, and custom terms
- Status pipeline — Draft → Sent → Accepted / Declined / Expired
- PDF preview and direct email send to client
- One-click conversion to invoice
- Duplicate existing quotes as templates
- Shareable public link generation

</details>

<details>
<summary><strong>Invoice Management</strong></summary>

- Full invoice lifecycle — Draft → Sent → Paid → Closed
- Overdue and voided states
- PDF preview and email delivery
- Credit note issuance against existing invoices
- Public link sharing for clients

</details>

<details>
<summary><strong>Credit Notes</strong></summary>

- Issued against specific invoices
- PDF preview and email delivery
- Public UUID link for client access

</details>

<details>
<summary><strong>Customer & Product Management</strong></summary>

- Full customer directory with contact and billing details
- Product/service catalogue with activation toggles
- Both scoped per tenant automatically

</details>

<details>
<summary><strong>Staff Management</strong></summary>

- Invite team members by email
- Activate / deactivate accounts
- Role assignment (Admin or Staff)
- Owner-only destructive actions

</details>

<details>
<summary><strong>Settings & Templates</strong></summary>

- Tenant branding — company name and logo upload
- Email template editor with live placeholder preview (`{{client_name}}`, `{{quote_total}}`, etc.)
- Document sequence configuration

</details>

<details>
<summary><strong>Public Document Views</strong></summary>

- Branded, read-only quote/invoice/credit note pages accessible via UUID link
- Client quote response — Accept or Decline with a single click
- No login required

</details>

---

## API Integration

All API calls go through a centralised Axios instance at `src/api/client.js`. It handles:

- Base URL injected from environment variable
- JWT `Authorization: Bearer <token>` header on every request
- Automatic token refresh on `401` responses via a request queue and interceptor
- Redirect to `/login` on refresh failure

```javascript
// src/api/client.js (simplified)
const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
});

apiClient.interceptors.request.use((config) => {
  const token = useAuthStore.getState().accessToken;
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});
```

---

## Authentication Flow

```
User submits login form
        │
        ▼
POST /auth/login
        │
        ▼
Store accessToken (memory) + refreshToken (localStorage)
        │
        ▼
Bootstrap user profile + tenant → populate Zustand auth store
        │
        ▼
Redirect to /dashboard

On 401:
  → Queue request → POST /auth/refresh → retry queue → or redirect /login
```

> Access tokens are short-lived. Refresh tokens are rotated on each use and revoked on logout.

---

## Requirements

- Node.js >= 18
- npm >= 9 or pnpm >= 8
- A running instance of the [Genesis API backend](../api/README.md)

---

## Installation

### 1. Clone and install dependencies

```bash
git clone <repository-url> genesis-frontend
cd genesis-frontend
npm install
```

### 2. Configure environment variables

```bash
cp .env.example .env.local
```

Edit `.env.local`:

```env
VITE_API_BASE_URL=http://localhost/genesis/api/v1
```

For production:

```env
VITE_API_BASE_URL=https://api.yourdomain.com/v1
```

### 3. Start the development server

```bash
npm run dev
```

The app will be available at `http://localhost:5173`.

---

## Configuration

### Environment Variables

| Variable | Description | Example |
|---|---|---|
| `VITE_API_BASE_URL` | Genesis API base URL | `https://api.yourdomain.com/v1` |

> Only variables prefixed with `VITE_` are exposed to the browser bundle. Never put secrets here.

---

## Available Scripts

| Script | Description |
|---|---|
| `npm run dev` | Start Vite dev server with HMR |
| `npm run build` | Production build to `dist/` |
| `npm run preview` | Preview the production build locally |
| `npm run lint` | Run ESLint |

---

## Deployment

The app builds to a static `dist/` folder and can be deployed to any static host.

### Recommended hosts

- **Vercel** — zero-config, connects to GitHub, automatic preview deployments
- **Cloudflare Pages** — fast global CDN, generous free tier

### SPA routing

Since React Router handles routing client-side, your host must redirect all requests to `index.html`. For Nginx:

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

For Vercel or Cloudflare Pages this is handled automatically.

### Production checklist

- `VITE_API_BASE_URL` points to the production API
- API CORS `allowedOrigins` includes the production frontend URL
- SSL enabled on both frontend and API domains
- No `.env.local` or dev credentials committed to the repository

---

## License

Licensed under the **BSD-3-Clause License**. See [LICENSE.md](LICENSE.md) for details.

---

## Contact & Demos

This repository is intentionally published as a README-only showcase — the full source code is proprietary and actively running in production.

If you're interested in any of the following, feel free to reach out:

- **Live demo or walkthrough** — a full presentation of the platform's features and workflows
- **Code review** — a private demo of the codebase for vetting or evaluation purposes
- **Similar build** — commissioning a comparable system for your business

📧 **[andrew@genesisdigital.co.za](mailto:andrew@genesisdigital.co.za)**
🌐 **[genesisdigital.co.za](https://genesisdigital.co.za)**

> Built by [Genesis Digital Solutions](https://genesisdigital.co.za)
