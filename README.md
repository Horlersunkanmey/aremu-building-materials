# Aremu Building and Construction Materials — Official Website & Admin Portal

**Tagline:** *Your Vision, Our Priority.*  
**Brand Promise:** *Quality materials and services for a better tomorrow.*

A complete, production-ready business website and private administrator dashboard built with **Next.js 14 (App Router)**, **TypeScript**, **Tailwind CSS**, and **Supabase (PostgreSQL, Authentication & Storage)**.

Designed so the business owner can manage products, iron rod sizes (6 mm – 32 mm), real estate property listings, professional cleaning services, photographs, prices, and business contact details directly from a phone or computer without coding.

---

## 1. Architecture & Key Features

### Public Website Pages
- **Home (`/`)**: Hero section, 4 trust highlights, featured building materials, interactive iron rod size selector (`6 mm` to `32 mm`), 5 real estate service categories (+ published property listings), cleaning services overview, *Why Choose Us* section, and final WhatsApp/Phone CTA.
- **Building Materials Catalogue (`/building-materials`)**: Searchable and filterable catalogue by category (`Aggregates & Sand`, `Cement & Binders`, `Iron Rods & Steel`, etc.) and availability status.
- **Building Iron Rods (`/iron-rods`)**: Dedicated page for reinforcement steel bars (`6 mm`, `8 mm`, `10 mm`, `12 mm`, `16 mm`, `20 mm`, `25 mm`, `32 mm`) with individual stock status and per-length/per-tonne pricing or *"Contact for Price"*.
- **Real Estate (`/real-estate`)**: Showcases the 5 service categories (*Residential Properties*, *Commercial Properties*, *Land Sales*, *Letting and Rentals*, *Property Management*) and displays property cards only when verified listings are published by the owner.
- **Cleaning Services (`/cleaning-services`)**: Showcases *Residential Cleaning*, *Office Cleaning*, *Warehouse Cleaning*, *Company and Commercial Cleaning*, and *Deep Cleaning*.
- **About Us (`/about`)**: Presents the approved company overview and four core service pillars without unverified claims.
- **Contact (`/contact`)**: Click-to-call numbers (`0703 039 1382`, `0913 509 3786`), direct WhatsApp buttons, editable email and address, and a spam-protected enquiry form.

### Private Administrator Portal
- **Administrator Login (`/admin/login`)**: Email & password login, password reset, and session protection. Public registration is disabled.
- **Private Admin Dashboard (`/admin`)**:
  - **Overview**: Real-time summary cards (active products, iron rods, properties, cleaning services, items without prices, recent updates, and customer form enquiries), plus confirmation alerts for provisional flyer details.
  - **Products and Materials**: Add, edit, delete (with confirmation), upload/replace photos, set prices and units (`per bag`, `per tonne`, `per trip`, `per length`), toggle *"Contact for Price"* / *"Hide Price"*, and control availability.
  - **Iron Rods**: Manage individual rod sizes (`6 mm` – `32 mm`), prices, units, and stock confirmation status.
  - **Property Listings**: Add/edit properties, upload multiple property photographs, set location/type/status (`Available`, `Rented`, `Sold`, `Unavailable`), or hide a listing without deleting it.
  - **Cleaning Services**: Edit descriptions, highlights, photographs, sort order, and availability.
  - **Business Information**: Edit business name, logo (toggle between crisp vector SVG logo, cropped flyer logo `/images/flyer-logo.png`, or custom upload), phone numbers, primary WhatsApp number, email address, street address, hero text, and global price visibility.
  - **Image Library**: Drag-and-drop image upload (JPEG, PNG, WebP up to 5 MB with automatic client-side mobile photo optimisation), alt-text management, and **safe deletion** that prevents deleting any image still referenced by a product, property, or service.
  - **Account & Logout**: View active session details, update password, and log out.

---

## 2. Local Development Setup

### Prerequisites
- **Node.js** `v18.18+` or `v20+`
- **npm** `v9+`
- A **Supabase** project (for cloud production) — *Note: For immediate local testing before connecting Supabase, the app also includes a built-in server-side persistent store (`.local-db.json` + `public/uploads`) so all admin edits, price changes, and image uploads work out of the box.*

### Step-by-Step Local Installation
1. Clone the repository and install dependencies:
   ```bash
   npm install
   ```
2. Copy `.env.example` to `.env.local`:
   ```bash
   cp .env.example .env.local
   ```
3. Provision your local administrator account (replace with your chosen email and strong password):
   ```bash
   node scripts/provision-admin.mjs seguosunsanya@gmail.com YourStrongPassword123! "Aremu Owner"
   ```
4. Start the development server:
   ```bash
   npm run dev
   ```
5. Visit:
   - **Public Website**: `http://localhost:3000`
   - **Private Admin Login**: `http://localhost:3000/admin/login`
   - **Private Admin Dashboard**: `http://localhost:3000/admin`

---

## 3. Connecting Supabase (Database, Auth & Storage)

1. Create a new project at [https://supabase.com](https://supabase.com).
2. In the Supabase Dashboard, open the **SQL Editor** and run:
   - First: [`supabase/migrations/001_schema_and_rls.sql`](./supabase/migrations/001_schema_and_rls.sql) — creates all tables (`admin_profiles`, `business_settings`, `products`, `properties`, `cleaning_services`, `media_library`, `customer_enquiries`), indexes, constraints, Row Level Security (RLS) policies, and the `aremu-media` Storage bucket with storage RLS policies.
   - Second: [`supabase/seed.sql`](./supabase/seed.sql) — seeds the initial business settings, 5 building material categories, 8 iron rod sizes (`6 mm` – `32 mm`), 5 cleaning services, and image attribution records.
3. In Supabase Dashboard $\rightarrow$ **Project Settings $\rightarrow$ API**, copy your credentials into `.env.local` (and Vercel Environment Variables):
   ```env
   NEXT_PUBLIC_SITE_URL=https://your-production-domain.vercel.app
   NEXT_PUBLIC_SUPABASE_URL=https://your-project-ref.supabase.co
   NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-public-key
   SUPABASE_SERVICE_ROLE_KEY=your-service-role-secret-key
   NEXT_PUBLIC_SUPABASE_STORAGE_BUCKET=aremu-media
   ADMIN_SETUP_SECRET=your-long-random-setup-secret
   ```

---

## 4. Provisioning & Recovering the Owner's Administrator Account

Public sign-up is intentionally disabled so strangers cannot register as administrators.

### Method A: Via Supabase Dashboard (Recommended for Production)
1. Go to **Supabase Dashboard $\rightarrow$ Authentication $\rightarrow$ Users** and click **Add User $\rightarrow$ Create New User**.
2. Enter the owner's confirmed email address and a strong password, and check **Auto Confirm User**.
3. Copy the newly created user's `UUID` and run this SQL in the **Supabase SQL Editor**:
   ```sql
   INSERT INTO public.admin_profiles (id, email, full_name, role, is_active)
   VALUES ('PASTE-USER-UUID-HERE', 'seguosunsanya@gmail.com', 'Aremu Business Owner', 'super_admin', TRUE)
   ON CONFLICT (id) DO UPDATE SET is_active = TRUE, role = 'super_admin';
   ```

### Method B: Via the Protected Provisioning Endpoint
With `ADMIN_SETUP_SECRET` and `SUPABASE_SERVICE_ROLE_KEY` configured, send a `POST` request once:
```bash
curl -X POST https://your-domain.vercel.app/api/admin/provision \
  -H "Content-Type: application/json" \
  -d '{
    "setupSecret": "your-long-random-setup-secret",
    "email": "seguosunsanya@gmail.com",
    "password": "YourStrongPassword123!",
    "fullName": "Aremu Business Owner"
  }'
```

### Account Recovery Procedure (If the Owner Loses Access)
- **Password Reset Email**: Click *"Forgot password?"* on `/admin/login` to receive a password reset link via Supabase Auth.
- **Direct Owner Recovery**: Log into the Supabase Dashboard $\rightarrow$ **Authentication $\rightarrow$ Users**, select the administrator user, and click **Send password recovery** or **Reset password**.

---

## 5. Deploying to GitHub & Vercel

1. Push this repository to a GitHub repository (`.gitignore` automatically excludes `.env.local` and `.local-db.json`).
2. Log in to [https://vercel.com](https://vercel.com) and click **Add New $\rightarrow$ Project**.
3. Import your GitHub repository.
4. Under **Environment Variables**, add:
   - `NEXT_PUBLIC_SITE_URL`
   - `NEXT_PUBLIC_SUPABASE_URL`
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
   - `SUPABASE_SERVICE_ROLE_KEY`
   - `NEXT_PUBLIC_SUPABASE_STORAGE_BUCKET` (`aremu-media`)
   - `ADMIN_SETUP_SECRET`
5. Click **Deploy**. Once deployed, update `NEXT_PUBLIC_SITE_URL` to match your live Vercel or custom domain URL and add `/admin/login` to your Supabase **Authentication $\rightarrow$ URL Configuration $\rightarrow$ Redirect URLs**.

### Database & Storage Backup Instructions
- **Database Backups**: In Supabase Dashboard $\rightarrow$ **Database $\rightarrow$ Backups**, daily automated backups are enabled on Pro plans, or you can export a full SQL dump anytime using `pg_dump` or the Supabase CLI (`npx supabase db dump -f backup.sql`).
- **Media Backups**: All uploaded product, property, and service photographs reside in the `aremu-media` bucket in Supabase Storage and persist across all Vercel redeployments.

---

## 6. Outstanding Owner Confirmation Checklist (Before Public Launch)

The following items from the printed flyer are pre-populated and flagged in the Admin Dashboard (**Business Information** tab) for the owner's final confirmation:
- [ ] **Business Email Confirmation**: Printed on the flyer as `seguosunsanya@gmail.com` (please confirm whether it should remain `seguosunsanya@gmail.com` or `segunosunsanya@gmail.com`).
- [ ] **Street Address Confirmation**: Printed on the flyer as `5 Ogun Close, off Ojodu-Abiodun, Ojodu Road, off Ojodu-Berger, Lagos`. Editable anytime in **Admin $\rightarrow$ Business Information**.
- [ ] **Primary WhatsApp Number Confirmation**: Currently set to `0703 039 1382` (`2347030391382`), with `0913 509 3786` (`2349135093786`) as the secondary line.
- [ ] **Product & Iron Rod Pricing**: All products default to *"Contact for Price"* so no fabricated prices are shown. The owner can enter confirmed Naira prices and units (`per bag`, `per tonne`, `per trip`, `per length`) in `/admin`.
- [ ] **Owner Photographs**: Initial photographs use licensed commercial stock imagery (documented in [`IMAGE_CREDITS.md`](./IMAGE_CREDITS.md)). The owner can replace any photo from his phone in `/admin`.

---

## 7. Acceptance Test Checklist

- [x] **Public Pages Load**: `/`, `/building-materials`, `/iron-rods`, `/real-estate`, `/cleaning-services`, `/about`, and `/contact` render with 200 OK.
- [x] **WhatsApp Links & Encoding**: All enquiry buttons open `https://wa.me/2347030391382?text=...` with URL-encoded pre-filled messages containing the product name, price, and unit (or *"Contact for Price"*).
- [x] **Admin Authentication & Protection**: Unauthenticated requests to `/admin` redirect to `/admin/login`, and unauthenticated calls to `/api/admin/data` or `/api/admin/upload` return `401 Unauthorised`.
- [x] **Product & Iron Rod CRUD**: Administrator can create, edit, price, hide/publish, and delete products and iron rod sizes, and changes persist after refresh and logout.
- [x] **Image Upload & Safe Cleanup**: Administrator can upload JPEG/PNG/WebP images up to 5 MB, preview them, and cannot accidentally delete an image from the library while it is still used by a product, property, or service.
- [x] **Property & Cleaning Management**: Property cards appear publicly only when published by the administrator; hidden listings remain invisible to public visitors.
- [x] **Honest Price Display**: Empty prices always display *"Contact for Price"* — never fabricated numbers.
- [x] **Production Build**: `npm run build` compiles cleanly with zero TypeScript or Next.js build errors.
