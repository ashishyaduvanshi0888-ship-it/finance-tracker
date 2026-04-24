# 💰 FinTrack — MERN Finance Tracker

A full-stack finance tracking web application built with the **MERN stack** (MongoDB, Express.js, React.js, Node.js). Features JWT authentication, OTP verification, server-side pagination, and a polished dark-theme UI.

---

## ✨ Features

### 🏠 Dashboard (Home)
- Add income/expense transactions with description, amount, and category
- Real-time stat cards: Total Income, Total Expenses, Net Balance
- Full filtering: by type, date range, amount range
- Sortable columns: date, amount, type (click table headers)
- Server-side pagination with configurable page size
- Filtered summary totals shown below the table

### 👤 Profile Page
- View username, email, password (masked), member since, last login
- Edit username inline
- Change password with OTP verification flow
- Sign out button

### 🔐 Authentication & Authorization
- Signup with email OTP verification
- Signin (redirects to OTP verify if account unverified)
- Forgot Password: email → OTP verify → reset password (3-step flow)
- Change Password: request OTP → verify → change (3-step flow)
- JWT access tokens (7d) + refresh tokens (30d) with auto-refresh
- Bcrypt password hashing (cost factor 12)
- Rate limiting on all auth endpoints
- All protected routes redirect to `/signin` if unauthenticated

---

## 🗂️ Project Structure

```
finance-tracker/
├── backend/
│   ├── config/
│   ├── controllers/
│   │   ├── auth.controller.js
│   │   ├── transaction.controller.js
│   │   └── user.controller.js
│   ├── middleware/
│   │   ├── auth.middleware.js
│   │   └── error.middleware.js
│   ├── models/
│   │   ├── User.model.js
│   │   └── Transaction.model.js
│   ├── routes/
│   │   ├── auth.routes.js
│   │   ├── transaction.routes.js
│   │   └── user.routes.js
│   ├── utils/
│   │   └── email.util.js
│   ├── server.js
│   ├── package.json
│   └── .env.example
│
├── frontend/
│   ├── src/
│   │   ├── api/          # Axios instance + API functions
│   │   ├── components/
│   │   │   ├── auth/     # ProtectedRoute, PublicRoute
│   │   │   ├── layout/   # AppLayout (sidebar + mobile header)
│   │   │   └── ui/       # OTPInput
│   │   ├── context/      # AuthContext (global auth state)
│   │   ├── pages/        # SignupPage, SigninPage, ForgotPasswordPage, HomePage, ProfilePage
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   └── index.css
│   ├── index.html
│   ├── vite.config.js
│   └── package.json
│
├── package.json          # Root – runs both servers concurrently
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites
- **Node.js** v18+
- **MongoDB** (local or [MongoDB Atlas](https://cloud.mongodb.com))
- **npm** v9+

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd finance-tracker
```

### 2. Configure Environment Variables

```bash
cp backend/.env.example backend/.env
```

Edit `backend/.env`:

```env
PORT=5000
MONGODB_URI=mongodb://localhost:27017/finance-tracker

JWT_SECRET=your_super_secret_jwt_key_change_this
JWT_EXPIRE=7d
JWT_REFRESH_SECRET=your_refresh_secret_key
JWT_REFRESH_EXPIRE=30d

# Gmail (use App Password, not your real password)
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your_email@gmail.com
EMAIL_PASS=your_16_char_app_password
EMAIL_FROM=Finance Tracker <your_email@gmail.com>

NODE_ENV=development
CLIENT_URL=http://localhost:5173
```

> **Gmail Setup**: Enable 2FA → Google Account → Security → App Passwords → generate for "Mail".

### 3. Install Dependencies

```bash
# Install all at once (root + backend + frontend)
npm run install:all
```

Or manually:
```bash
npm install              # root concurrently
cd backend && npm install
cd ../frontend && npm install
```

### 4. Run in Development

```bash
# From root — starts both backend (5000) and frontend (5173)
npm run dev
```

Or separately:
```bash
npm run dev:backend    # http://localhost:5000
npm run dev:frontend   # http://localhost:5173
```

### 5. Open the App

Visit **http://localhost:5173** in your browser.

> **Dev OTP fallback**: If email isn't configured, OTPs print to the backend terminal console.

---

## 📡 API Reference

### Auth Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/signup` | Register user |
| POST | `/api/auth/verify-signup` | Verify signup OTP |
| POST | `/api/auth/resend-otp` | Resend OTP |
| POST | `/api/auth/signin` | Sign in |
| POST | `/api/auth/forgot-password` | Request reset OTP |
| POST | `/api/auth/verify-forgot-otp` | Verify forgot-password OTP |
| POST | `/api/auth/reset-password` | Reset password with token |
| POST | `/api/auth/refresh` | Refresh access token |
| POST | `/api/auth/logout` | Logout (auth required) |

### Transaction Endpoints (all require auth)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/transactions` | Get transactions (filters, pagination) |
| POST | `/api/transactions` | Create transaction |
| PUT | `/api/transactions/:id` | Update transaction |
| DELETE | `/api/transactions/:id` | Delete transaction |
| GET | `/api/transactions/stats` | Get aggregate stats |

**GET `/api/transactions` query params:**
- `page`, `limit` — pagination
- `type` — `income` or `expense`
- `startDate`, `endDate` — date range (ISO strings)
- `minAmount`, `maxAmount` — amount range
- `sortBy` — `date` | `amount` | `type`
- `sortOrder` — `asc` | `desc`

### User Endpoints (all require auth)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/users/me` | Get profile |
| PUT | `/api/users/me` | Update username |
| POST | `/api/users/change-password/request` | Request change-password OTP |
| POST | `/api/users/change-password/verify` | Verify OTP + change password |

---

## 🛡️ Security Highlights

- Passwords hashed with **bcrypt** (cost factor 12)
- **JWT** access + refresh token rotation
- OTP stored as **bcrypt hash** in DB, expires in 10 minutes, max 5 attempts
- **Helmet.js** for HTTP security headers
- **Rate limiting**: 100 req/15min globally, 20 req/15min on auth routes
- **express-validator** on all input fields
- Forgot password never reveals if email exists (anti-enumeration)
- Users can only access/modify their own transactions (ownership check)

---

## 🏗️ Production Build

```bash
# Build the frontend
npm run build

# Serve static files from Express (add to server.js):
# app.use(express.static(path.join(__dirname, '../frontend/dist')))

# Start production backend
npm start
```

---

## 🧰 Tech Stack

| Layer | Tech |
|-------|------|
| Frontend | React 18, Vite, React Router v6 |
| Styling | Custom CSS (design system with CSS variables) |
| HTTP Client | Axios (with interceptors for token refresh) |
| Backend | Node.js, Express.js |
| Database | MongoDB, Mongoose |
| Auth | JWT (jsonwebtoken), bcryptjs |
| Email | Nodemailer |
| Validation | express-validator |
| Security | helmet, express-rate-limit |
| Dev | Nodemon, Concurrently |

---

## 📝 Notes

- The OTP is valid for **10 minutes** and allows **5 attempts**
- Access tokens expire in **7 days**, refresh tokens in **30 days**
- Transactions are paginated server-side; all filters hit MongoDB directly
- The frontend uses a Vite proxy during dev (`/api` → `localhost:5000`)
