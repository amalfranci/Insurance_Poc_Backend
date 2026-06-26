# 🏗️ SubTraid — Surety Bond & Insurance Underwriting Platform

A production-grade SaaS backend for construction surety bond and insurance underwriting workflows — multi-step applications, financial due diligence, bond lifecycle management, and subscription billing for brokerages.

---

## 🚀 Tech Stack

**Backend:** Node.js (CommonJS), TypeScript, Express.js, Sequelize ORM  
**Database:** MySQL (single database — 37 domain models across users, applications, reports, bonds, and billing)  
**Architecture:** Monolithic REST API · Layered MVC (Router → Controller → Repository → Model)  
**Auth:** JWT · bcrypt · Role-based access control (6 user types)  
**Queue:** Redis · Bull (async OCR / report processing)  
**Real-time:** Socket.IO (online status, report notifications)  
**Infra:** AWS S3 · AWS KMS · Remote MySQL · Redis  
**Integrations:** QuickBooks Online · Dun & Bradstreet Direct+ · ESC/PPSA · Stripe · ConvertAPI  
**Other:** Swagger UI · Nodemailer · Multer · ExcelJS · xlsx · cron

---

## ⚙️ Key Features

### 🏗️ 1. Modular Monolith Architecture

A single deployable Node.js/Express service organized into **11 domain modules** and **30 route groups** (~100 REST endpoints):

| Module | Route Prefix | Responsibility |
|---|---|---|
| Auth | `/api/auth` | Register, login, forgot/reset password, email verification |
| QBO Auth | `/api/qbo` | QuickBooks OAuth2 flow |
| User | `/api/user` | Applicant, broker, brokerage, owner, accountant profiles |
| Permission | `/api/permission` | Accounting data consent workflow |
| Application | `/api/application` | Multi-form underwriting workflow, dashboards, submit/approve |
| Application Forms | `/api/accountingSystem`, `/creditBureau`, `/financialInfo`, `/cms`, `/surety`, `/insurance` | Six-step surety/insurance application forms |
| Reports | `/api/report`, `/wip`, `/wic`, `/ocr`, `/analyticalReports` | Manual upload, WIP/WIC, OCR pipeline, financial ratio analytics |
| QBO Reports | `/api/quickbook`, `/quickbook/report` | QuickBooks data sync and financial report generation |
| Bond | `/api/bond`, `/bondApplication`, `/bondForm`, `/bondActivityTracker` | Bond requests, applications, forms, activity tracking |
| Client Profile | `/api/clientProfile` | Contractor client profile management |
| Admin | `/api/admin`, `/subscription`, `/faq` | Admin operations, Stripe subscription plans, FAQs |

**Layered code structure:**

```
Router → Controller → Repository → Sequelize Model
         ↓
      Helper / Service / Middleware / Validator / DTO
```

| Layer | Count |
|---|---|
| Controllers | 41 |
| Repositories | 38 |
| Sequelize Models | 37 |
| DTOs | 40 |
| Validators | 29 |
| Helpers | 22 |
| Services | 8 |
| Middleware | 10 |

---

### 🏢 2. Brokerage-Scoped Multi-Tenancy

Data is scoped at the **brokerage** level. Applicants and brokers belong to a `brokerageId`; subscription plans and purchased plans are enforced per brokerage. Each user type has its own repository lookup in the `authorize` middleware, ensuring role-specific data access.

---

### 📋 3. Multi-Step Underwriting Application Workflow

Full surety bond and insurance application lifecycle:

1. Broker initiates application and assigns forms to applicant/accountant
2. Six application forms collected: **Accounting System** · **Credit Bureau** · **Financial Info** · **CMS** · **Surety** · **Insurance**
3. Consent/permission flow for accounting data access (QBO integration)
4. Third-party auto-population from QuickBooks, D&B, and ESC/PPSA lien searches
5. Financial reports uploaded manually or synced from QuickBooks
6. Underwriter reviews, approves, or declines application
7. Full audit trail via application history and in-app notifications

**Application statuses:** Draft · Awaiting Input · Awaiting Consent · Submitted · Approved · Declined

---

### 📊 4. Financial Reporting & Analytics Pipeline

**Manual / OCR upload reports:**
- Account Receivable Aging (ARA) · Account Payable Aging (APA)
- Profit & Loss · Balance Sheet · Cashflow
- Banking documents · Corporate org charts · Workers Compensation

**Async OCR pipeline:**
1. User uploads PDF/Excel → stored on **AWS S3**
2. Job enqueued on **Redis/Bull** (`ocrQueue`)
3. **ConvertAPI** converts documents; **ExcelJS/xlsx** parses structured data
4. Structured JSON saved to database
5. **Socket.IO** pushes success/failure notification to user

**QuickBooks Online reports:**
- ARA · APA · P&L · Balance Sheet · WIP · WIC (with fiscal period filters)

**Analytical ratio engine:**
- Profitability · Liquidity · Efficiency · Leverage · Year-over-Year Growth
- Computed from uploaded and QBO-sourced financial data

---

### 🔗 5. Bond Lifecycle Management

| Feature | Detail |
|---|---|
| Bond types | Performance · Payment · Supply · Maintenance · Subdivision · License/Permit · Environmental · Completion · Site Improvement · Material |
| Application statuses | Pending · Assigned · Completed · Approved · Rejected · Downloaded |
| Tracking | Bond activity tracker · client profiles · bond form data |

---

### 🔐 6. Auth & Access Control

- **Six user roles:** `applicant` · `broker` · `brokerage` · `owner` · `accountant` · `admin`
- JWT access tokens with automatic refresh via response headers
- Role-dispatched `authorize` middleware — each role resolves to its own repository
- Email verification, forgot/reset password flows
- Admin-only routes for subscription plan and FAQ management

---

### 💳 7. Subscription Billing (Stripe)

Brokerage subscription tiers: **Intro** · **Starter** · **Growth** · **Scale** · **Optimize** · **One-Time Application**

- Stripe Checkout sessions for plan purchase and top-up
- Webhook handler for payment confirmation
- Active plan enforcement per brokerage (`PurchasedPlan` / `SuretyPurchasedPlan`)

---

### 🔌 8. Third-Party Integrations

| Integration | Purpose |
|---|---|
| **QuickBooks Online** | OAuth2, accounting data auto-populate, financial report sync |
| **Dun & Bradstreet Direct+** | Company and credit data for credit bureau forms |
| **ESC / PPSA (Registry Complete)** | Person/Business Debtor Search (lien searches) via SOAP/XML |
| **AWS S3 + KMS** | Document storage, multer-s3 uploads, presigned URLs |
| **Stripe** | Subscription checkout and webhook billing |
| **ConvertAPI** | PDF/document conversion in OCR pipeline |
| **Nodemailer (SMTP)** | Email notifications and consent requests |

---

## 🧠 Architecture Highlights

- **Repository pattern** — 38 repositories abstract all Sequelize data access from controllers
- **Swagger-first API docs** — programmatic generation at startup; served at `/api-docs` (~100 endpoints)
- **Unified error handling** — custom `APIError` class with global Express error middleware
- **Real-time notifications** — Socket.IO for online status and OCR/report completion events
- **Consent workflow** — separate permission module for QBO accounting data access (approve/deny/resend)
- **Financial analytics helpers** — dedicated ratio calculation modules with editable field structures
- **Full-stack deployment** — serves React frontend static build from `./build` alongside the API
- **Cron jobs** — scheduled polling for pending Business Debtor Search (ESC/PPSA) results

---

## 🗄️ Database Domains (37 Sequelize Models)

| Domain | Models |
|---|---|
| **Users & Orgs** | Applicant, Broker, Brokerage, Owner, Accountant, Admin, ProfileHistory, ResetPassword |
| **Application Forms** | Application, AccountingSystem, CreditBureau, FinancialInfo, CMS, Surety, Insurance, Permission, History, Notification, ESC |
| **Financial Reports** | WIP, WIC, UploadReport, PDFSummary, ARA_Report, APA_Report, BSPL_Report, OcrReports, AnalyticalReport |
| **Bonds** | BondRequest, BondApplication, BondForm, BondActivityTracker, ClientProfile |
| **Billing & Admin** | SubscriptionPlan, PurchasedPlan, SuretyPurchasedPlan, FAQ, QuickBook |

---

## 🛠️ Local Setup

### Prerequisites

- Node.js `>= 16.14.2`
- MySQL database
- Redis server (for Bull OCR queue)

### Steps

1. Clone the repository and install dependencies:
   ```bash
   npm install
   ```

2. Copy environment variables:
   ```bash
   cp .env.example .env
   ```

3. Configure `.env` with your credentials:
   - **Database:** `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_DATABASE`
   - **Auth:** `JWT_SECRET`, `JWT_EXPIRESIN`
   - **URLs:** `BACKEND_URL`, `FRONTEND_URL`
   - **SMTP:** `SMTP_EMAIL`, `SMTP_KEY`
   - **QuickBooks:** `QB_Client_ID`, `QB_Client_Secret_ID`, `QB_REDIRECT_URL`, `QB_ENV`
   - **AWS:** `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`, `AWS_BUCKET`, `AWS_KMS_KEY_ID`
   - **Stripe:** `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`
   - **D&B:** `D_B_ConsumerKey`, `D_B_ConsumerSecret`, `D_B_Base64`
   - **ESC/PPSA:** `ESC_USERNAME`, `ESC_PASSWORD`, `ESC_URL`
   - **Redis:** configure in queue connection settings

4. Build and start the server:
   ```bash
   # Development
   npm run dev

   # Production build
   npm run build
   npm start
   ```

5. Open API documentation:
   ```
   http://localhost:<PORT>/api-docs
   ```

### Scripts

| Command | Description |
|---|---|
| `npm run dev` | Start dev server with nodemon (hot reload) |
| `npm run build` | Compile TypeScript to `build/` |
| `npm start` | Run compiled production server |
| `npm run lint` | Run ESLint |
| `npm run lint:fix` | Auto-fix ESLint issues |
| `npm run format` | Format code with Prettier |

---

## 👤 Roles

| Role | Access |
|---|---|
| **Admin** | Platform administration, subscription plan CRUD, FAQ management |
| **Brokerage** | Organization-level account, subscription billing, broker management |
| **Broker** | Create/manage applications, assign forms, underwriting review, bond management |
| **Applicant** | Complete assigned application forms, upload financial documents |
| **Accountant** | Complete accounting-related forms on behalf of applicant |
| **Owner** | Business owner profile linked to applicant |

---

## 📁 Project Structure

```
src/
├── api/
│   ├── controller/     # Request handlers (41 controllers)
│   ├── repository/     # Data access layer (38 repositories)
│   ├── model/          # Sequelize models (37 models)
│   ├── router/         # Express route definitions (30 route groups)
│   ├── middleware/     # Auth, upload, validation middleware
│   ├── validator/      # Joi validation schemas
│   ├── dto/            # Data transfer objects
│   ├── helper/         # Domain helpers (reports, D&B, analytics)
│   └── services/       # Email, file processing, cron, socket
├── connection/         # MySQL / Sequelize configuration
├── docs/               # Swagger API documentation
├── common/             # Shared logger
├── util/               # APIError and utilities
├── app.ts              # Express app setup
└── server.ts           # HTTP server + Socket.IO
```

---

## 📄 License

UNLICENSED — Private project.
