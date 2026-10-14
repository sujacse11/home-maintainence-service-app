# HMS Home Services - Real-Time Home Maintenance & Repair Mobile App

Production-quality on-demand Home Maintenance & Repair platform  built with **React Native (Expo)** frontend and **Django REST Framework** backend.

---

## 🌟 Key Features

1. **Authentication & Role-Based Access Control**:
   - SimpleJWT Authentication (`access` & `refresh` tokens).
   - Roles: `customer`, `technician`, `admin`.
   - Admin Login Portal with real-time financial analytics (`₹`).

2. **Full Service Catalog & Sub-Services (Urban Company Style)**:
   - 7 Categories: Electrical, Plumbing, AC Services, Washing Machine Repair, Refrigerator Repair, Water Heater Service, General Home Maintenance, each with its own sub categories (e.g. AC Services > Cleaning & Servicing / Gas Charging & Repair / Installation).
   - Itemized Sub-Services / Inclusions (e.g. `✓ High-Pressure Foam Jet Coil Wash`, `✓ Gas Pressure Check`).
   - Prices in Indian Rupees (`₹`) with automated **18% GST** breakdown.

3. **Multi-Step Booking Wizard**:
   - Step 1: Issue description & photo attachment (Camera / Gallery picker).
   - Step 2: 14-Day Calendar Date Strip (past dates disabled) + Morning/Afternoon/Evening time slots.
   - Step 3: Address selector + **`🎯 Detect Current Delivery Location (GPS)`** pinpoint button.
   - Step 4: Review summary & 18% GST calculation.

4. **Razorpay & GPay Payment Integration**:
   - Integrated **Razorpay Secure Payment Gateway** & **Google Pay (GPay) / UPI** checkout modal sheet.
   - Generates authentic Razorpay transaction IDs (`pay_Rzp...`).

5. **Live Status Timeline & Notifications**:
   - Real-time status progress timeline (`Requested → Confirmed → Technician Assigned → In Progress → Completed`).
   - In-app notification center.

6. **HMS Admin Portal**:
   - Financials Overview: Live Total Revenue (`₹`), Pending Requests, Active Jobs, Completed Jobs.
   - Assign Chennai Technicians (Alex Kumar, Sarah Raman) to customer bookings with 1 tap.
   - Status updates & Service pricing manager.

7. **Mobile Smartphone Viewport Frame for Web**:
   - Centered smartphone container frame on web browsers (`http://localhost:8089`) with top status bar & notch.

---

## 🚀 Quick Setup & Run Instructions

### 1. Backend Setup (Django REST Framework)
```bash
cd backend
python -m venv venv
.\venv\Scripts\activate          # Windows   (macOS/Linux: source venv/bin/activate)
pip install -r requirements.txt
cp .env.example .env              # optional - all settings have dev defaults
python manage.py migrate
python manage.py seed_data        # demo users + 7 categories + services + technicians
python manage.py runserver 0.0.0.0:8000
```
- **Backend Base URL**: `http://127.0.0.1:8000/api/`
- **Django Admin**: `http://127.0.0.1:8000/admin/`

### 2. Frontend Setup (React Native Expo Web & Mobile)
```bash
cd frontend
npm install
npx expo start --web --port 8089
```
- **Live Web App**: `http://localhost:8089`

---

## 🔑 Demo Account Credentials

| Role | Username | Password | Features |
| :--- | :--- | :--- | :--- |
| **Customer** | `john_doe` | `password123` | Service booking flow, calendar date picker, GPS location detector, Razorpay checkout |
| **Admin** | `admin` | `password123` | **HMS Admin Portal**: Financials (₹), Booking status manager & Technician assignment |
| **Technician 1** | `alex_tech` | `password123` | Assigned jobs view, job completion |
| **Technician 2** | `sarah_tech` | `password123` | Assigned jobs view, job completion |

---

## 🛠️ API Endpoints Summary (`/api/`)

| Area | Endpoint | Who |
| :--- | :--- | :--- |
| Auth | `POST /token/` (username **or** email), `POST /token/refresh/` | public (rate-limited) |
| | `POST /auth/register/` (always creates a *customer*) | public (rate-limited) |
| | `GET/PATCH/DELETE /auth/profile/` (DELETE deactivates the account) | signed in |
| | `POST /auth/logout/` `{refresh}` (blacklists the token), `POST /auth/change-password/` | signed in |
| Catalog | `GET /categories/` (each includes its `subcategories`), `GET /subcategories/?category=<id>`, `GET /services/`, `GET /services/<id>/` | public |
| | `GET /services/?search=&category=&subcategory=&category__slug=&subcategory__slug=&min_price=&max_price=&min_rating=&ordering=` (`starting_price`, `average_rating`, `review_count`, `name`) | public |
| | `GET /services/<id>/slots/?date=YYYY-MM-DD` - **hourly** slots (06:00-17:00, i.e. 6 AM - 6 PM) with live availability (`available`, `reason`, `technicians_free`) | public |
| Addresses | `/addresses/` CRUD (own only) | signed in |
| Bookings | `GET/POST /bookings/` (`?status=`; multipart `uploaded_images`, max 3) | customer / technician (own jobs) / admin (all) |
| | `POST /bookings/<id>/cancel/` `{reason}` | owner (until work starts) or admin |
| | `POST /bookings/<id>/update_status/` `{status}` | assigned technician (`IN_PROGRESS`, `COMPLETED`) or admin |
| | `POST /bookings/<id>/assign_technician/` `{technician_id}` | admin |
| | `GET /bookings/admin_stats/`, `GET /technicians/` | admin |
| Reviews | `POST /reviews/` `{booking, rating, comment}` (own, completed, once), `GET /reviews/?service=` | signed in |
| Notifications | `GET /notifications/`, `POST /notifications/<id>/mark_read/`, `POST /notifications/mark_all_read/`, `GET /notifications/unread_count/` | signed in |

**Hourly time slots.** `time_slot` is the slot's start time as `HH:MM` (e.g. `"14:00"` = 2:00-3:00 PM). Opening hours come from
`BOOKING_OPEN_HOUR` / `BOOKING_CLOSE_HOUR` (default 6 and 18). Availability is computed against the live clock in `DJANGO_TIME_ZONE`:
slots that have started (or start within `BOOKING_MIN_LEAD_MINUTES`, default 60) are rejected, a service longer than one hour
reserves the technician for every hour it spans, and it can't be started so late that it would finish after closing.

**Booking status flow** (enforced server-side, every change is logged in `status_logs`):
`REQUESTED -> CONFIRMED -> TECHNICIAN_ASSIGNED -> IN_PROGRESS -> COMPLETED`, with `CANCELLED` possible until `IN_PROGRESS`.
Each booking returns `allowed_transitions` and `can_cancel` so clients never have to hard-code the rules.

**Errors** keep DRF's per-field shape and add a top-level `error` string, e.g.
`{"booking_date": ["You cannot book a date in the past."], "error": "Booking date: You cannot book a date in the past."}`.

---

## 🧪 Running Unit Tests

```bash
cd backend
python manage.py test api      # 72 tests: auth, permissions, booking rules, hourly slots, subcategories, reviews, seed
```

---

## ⚙️ Configuration

All backend settings come from environment variables (see `backend/.env.example`).
For production set `DJANGO_DEBUG=False`, a real `DJANGO_SECRET_KEY`, `DJANGO_ALLOWED_HOSTS`,
`CORS_ALLOWED_ORIGINS` and the `POSTGRES_*` variables. `seed_data` refuses to run with `DEBUG` off
(it creates demo users with a known password) unless you pass `--force`.
