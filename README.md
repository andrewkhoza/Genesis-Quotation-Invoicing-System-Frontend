<div align="center">

# Genesis — Quotation & Invoicing Frontend

[![React](https://img.shields.io/badge/React-18-61DAFB.svg?logo=react)](https://react.dev/)
[![Vite](https://img.shields.io/badge/Vite-5-646CFF.svg?logo=vite)](https://vitejs.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6.svg?logo=typescript)](https://www.typescriptlang.org/)
[![License](https://img.shields.io/badge/License-BSD--3--Clause-blue.svg)](LICENSE.md)

The React/Vite frontend for the Genesis Quotation & Invoicing platform. A multi-tenant SPA that consumes the [Genesis API](../api/README.md) — providing role-scoped dashboards for Owners, Admins, and Staff to manage quotes, invoices, credit notes, customers, and products.

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
| Language | TypeScript 5 |
| Routing | React Router v6 |
| State Management | Zustand |
| HTTP Client | Axios |
| UI Components | shadcn/ui + Tailwind CSS |
| Form Handling | React Hook Form + Zod |
| PDF Preview | React PDF / iframe embed |
| Notifications | Sonner (toast) |
| Icons | Lucide React |
| Date Handling | Day.js |

> **Note:** Update this table to match your actual dependencies if they differ.

---

## Project Structure

```
├── public/                   # Static assets
├── src/
│   ├── api/                  # Axios instance, interceptors, endpoint functions
│   │   ├── client.ts         # Base Axios config + JWT interceptor
│   │   ├── auth.ts
│   │   ├── quotes.ts
│   │   ├── invoices.ts
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
│   ├── types/                # TypeScript interfaces and enums
│   ├── utils/                # Formatters, helpers, constants
│   ├── router/               # Route definitions and guards
│   │   ├── index.tsx
│   │   ├── ProtectedRoute.tsx
│   │   ├── RoleRoute.tsx
│   │   └── GuestRoute.tsx
│   ├── App.tsx
│   └── main.tsx
├── .env.example
├── index.html
├── tailwind.config.ts
├── tsconfig.json
└── vite.config.ts
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

- Tenant branding — company name, logo upload
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

All API calls are made through a centralised Axios instance at `src/api/client.ts`. It handles:

- Base URL from environment variable
- JWT `Authorization: Bearer <token>` header injection on every request
- Automatic token refresh on `401` responses via a request queue + interceptor
- Redirect to `/login` on refresh failure

```typescript
// src/api/client.ts (simplified)
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
Store accessToken (memory) + refreshToken (httpOnly cookie or localStorage)
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
| `npm run type-check` | Run TypeScript compiler check (no emit) |

---

## Deployment

The app builds to a static `dist/` folder and can be deployed to any static host.

### Recommended hosts

- **Vercel** — zero-config, connects to GitHub, automatic preview deployments
- **Cloudflare Pages** — fast global CDN, generous free tier
- **Azure Static Web Apps** — if you're within an Azure environment

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
