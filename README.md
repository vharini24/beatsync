# 🥁 BeatSync

**BeatSync** is a route-planning and collection-management platform for FMCG distributors and their delivery teams. Owners generate a daily **beat** (the list of stores each delivery partner should visit, in priority order), delivery partners complete visits from a mobile app, and every delivery, return, payment and credit promise flows into a **store ledger** so outstanding balances stay accurate.

![Node.js](https://img.shields.io/badge/Node.js-Express%205-339933)
![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL-336791)
![Prisma](https://img.shields.io/badge/ORM-Prisma%207-2D3748)
![React](https://img.shields.io/badge/Web-React%2019%20%2B%20Vite-61DAFB)
![Expo](https://img.shields.io/badge/Mobile-Expo%20%2F%20React%20Native-000020)
![Socket.IO](https://img.shields.io/badge/Realtime-Socket.IO-010101)

---

## 📑 Table of Contents

- [The Problem](#-the-problem)
- [Key Features](#-key-features)
- [How a Beat Works](#-how-a-beat-works)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
- [Environment Variables](#-environment-variables)
- [API Overview](#-api-overview)
- [Data Model](#-data-model)
- [Testing](#-testing)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)

---

## 🎯 The Problem

Distributors serve hundreds of small retail stores. Deciding **which stores to visit today**, **who visits them**, and **how much money is still owed** is usually handled on paper or in memory. Stores with large or long-overdue balances get missed, collections are hard to reconcile, and owners have no live view of the day.

BeatSync gives owners an automatic, priority-based daily plan and gives delivery partners a simple mobile workflow, with a ledger that records everything.

---

## ✨ Key Features

### For distributor owners (web dashboard)
- **Automatic beat generation**: stores are scored and split across delivery partners for the day
- **Review and adjust**: manually move a store to a different delivery partner before publishing
- **Publish beats** so delivery partners receive them
- **Dashboard** with live operational metrics and collection charts
- **Store ledger**: full history of sales, returns, payments and credit for each store
- **Reconciliation** of a beat's expected collection vs. what was actually collected
- **Store and SKU management** (full CRUD)
- **Team management** using a distributor code that delivery partners use to join
- Dark mode and multi-language support (i18n)

### For delivery partners (mobile app)
- See **today's beat** and the day summary
- **Start a visit**, record delivered items, returns, payments and credit promises, then complete it
- **Cash and UPI payments**, with **Razorpay** order creation and signature verification for UPI
- Work with **multiple distributors**, joining each with a team code
- **Google Sign-In** or phone and password login
- Contact the owner straight from the app

### Platform
- JWT authentication with role-based access (`owner`, `delivery_boy`)
- Real-time updates over **Socket.IO**
- Risk flagging of stores (green, yellow, red) based on outstanding balance and overdue days

---

## 🧭 How a Beat Works

1. **Generate**: the owner generates a beat for the day. Each active store gets a priority score:

   | Signal | Weight |
   |---|---|
   | Days since last visit (capped at 7 days) | 30% |
   | Outstanding balance (capped at ₹10,000) | 35% |
   | Overdue days (capped at 30 days) | 15% |
   | Not visited for 5+ days | 20% |

2. **Risk level**: every store is tagged for information only. It never blocks delivery.

   | Level | Condition |
   |---|---|
   | 🔴 Red | Outstanding above ₹15,000, or overdue more than 60 days |
   | 🟡 Yellow | Outstanding ₹5,000 to ₹15,000, or overdue 30 to 60 days |
   | 🟢 Green | Everything else |

3. **Assign**: stores are distributed among delivery partners with a visit order. The owner can reassign any store manually.
4. **Publish**: the beat goes live for delivery partners.
5. **Visit**: the delivery partner starts a visit, records items delivered, returns, payments (cash/UPI) and any credit promise, then completes it.
6. **Ledger**: completing a visit writes a ledger entry (previous balance, sales, returns, payments, credit, new balance) and updates the store's outstanding balance.
7. **Reconcile**: the owner compares expected collection against actual collection for the beat.

---

## 🏗 Architecture

```
┌──────────────────────┐        ┌──────────────────────┐
│  Web dashboard       │        │  Mobile app          │
│  React + Vite        │        │  Expo / React Native │
│  (owners)            │        │  (delivery partners) │
└──────────┬───────────┘        └──────────┬───────────┘
           │        REST (JWT)  +  Socket.IO        │
           └──────────────────┬─────────────────────┘
                              ▼
                 ┌─────────────────────────┐
                 │  Backend API            │
                 │  Node.js + Express 5    │
                 │  Prisma ORM             │
                 └───────┬─────────┬───────┘
                         │         │
                         ▼         ▼
                 ┌───────────┐ ┌──────────────┐
                 │ PostgreSQL│ │  Razorpay    │
                 └───────────┘ └──────────────┘
```

---

## 🛠 Tech Stack

| Area | Technology |
|---|---|
| Backend | Node.js, Express 5, Socket.IO |
| Database and ORM | PostgreSQL, Prisma 7 |
| Auth | JWT, bcrypt, Google OAuth (`google-auth-library`) |
| Payments | Razorpay |
| Web dashboard | React 19, Vite, Tailwind CSS, React Router, Zustand, Recharts, Lucide icons |
| Mobile app | Expo SDK 54, React Native 0.81, Expo Router, TypeScript, Reanimated, Lottie |

---

## 📁 Project Structure

```
beatsync/
├── backend/
│   ├── prisma/                 # schema.prisma and migrations
│   ├── src/
│   │   ├── controllers/        # auth, beat, dashboard, delivery, ledger,
│   │   │                       # reconciliation, sku, store, team
│   │   ├── routes/             # Express routers
│   │   ├── middleware/         # auth and distributor-context middleware
│   │   ├── lib/                # Prisma and Razorpay clients
│   │   ├── utils/              # JWT, ledger maths, sanitizers
│   │   ├── app.js              # Express app
│   │   ├── server.js           # HTTP server entry point
│   │   └── socket.js           # Socket.IO setup
│   ├── test/                   # Node test runner tests
│   └── .env.example
│
└── frontend/
    ├── src/                    # Owner web dashboard (React + Vite)
    │   ├── pages/              # BeatGenerator, Dashboard, Ledger
    │   ├── components/
    │   ├── store/              # Zustand store
    │   └── api/client.js
    └── BeatSync/               # Delivery partner mobile app (Expo)
        ├── app/                # Expo Router screens: (auth), (tabs), store, settings
        ├── context/            # Auth, Beat, Distributor, Language, Theme
        └── api/client.ts
```

---

## 🚀 Getting Started

### Prerequisites

- Node.js 20 or later and npm
- PostgreSQL 14 or later
- Expo Go app or an Android/iOS emulator (for the mobile app)
- Optional: a Razorpay test account and Google OAuth client IDs

### 1. Clone the repository

```bash
git clone https://github.com/vharini24/beatsync.git
cd beatsync
```

### 2. Backend

```bash
cd backend
npm install
cp .env.example .env        # then fill in the values (see Environment Variables)
npx prisma migrate deploy   # apply database migrations
npx prisma generate
npm run dev                 # http://localhost:5000
```

Check that it is running: `GET http://localhost:5000/api/health`

### 3. Web dashboard (owners)

```bash
cd frontend
npm install
npm run dev                 # http://localhost:5173
```

### 4. Mobile app (delivery partners)

```bash
cd frontend/BeatSync
npm install
cp .env.example .env        # add your Google client IDs
npx expo start
```

Then press `a` for Android, `i` for iOS, or scan the QR code with Expo Go.

> **Note:** The API base URL is currently set in `frontend/src/api/client.js` and `frontend/BeatSync/api/client.ts` as `http://localhost:5000/api`. The Android emulator uses `10.0.2.2`. On a physical device, change it to your computer's local IP address.

---

## 🔐 Environment Variables

### `backend/.env`

| Variable | Description |
|---|---|
| `PORT` | API port (default `5000`) |
| `DATABASE_URL` | PostgreSQL connection string, e.g. `postgresql://user:password@localhost:5432/beatsync` |
| `JWT_SECRET` | Secret used to sign JWTs. Use a long random value |
| `RAZORPAY_KEY_ID` | Razorpay key ID (test mode) |
| `RAZORPAY_KEY_SECRET` | Razorpay key secret |
| `GOOGLE_CLIENT_ID_WEB` | Google OAuth client ID (web) |
| `GOOGLE_CLIENT_ID_ANDROID` | Google OAuth client ID (Android) |
| `GOOGLE_CLIENT_ID_IOS` | Google OAuth client ID (iOS) |

### `frontend/BeatSync/.env`

| Variable | Description |
|---|---|
| `EXPO_PUBLIC_GOOGLE_CLIENT_ID_WEB` | Google OAuth client ID (web) |
| `EXPO_PUBLIC_GOOGLE_CLIENT_ID_ANDROID` | Google OAuth client ID (Android) |
| `EXPO_PUBLIC_GOOGLE_CLIENT_ID_IOS` | Google OAuth client ID (iOS) |

> ⚠️ Never commit real secrets. Only `.env.example` files should be tracked.

---

## 📡 API Overview

Base URL: `http://localhost:5000/api`. All routes except auth and health need an `Authorization: Bearer <token>` header.

| Area | Method and path | Role | Description |
|---|---|---|---|
| Health | `GET /health` | Public | Service health check |
| Auth | `POST /auth/register-owner` | Public | Register a distributor owner |
| Auth | `POST /auth/register-delivery-boy` | Public | Register a delivery partner |
| Auth | `POST /auth/login` | Public | Login and receive a JWT |
| Auth | `POST /auth/google` | Public | Sign in with Google |
| Auth | `GET /auth/me` | Any | Current user |
| Stores | `GET/POST /stores`, `GET/PUT/DELETE /stores/:id` | Owner (write), any (read) | Store management |
| SKUs | `GET/POST /skus`, `GET/PUT/DELETE /skus/:id` | Owner (write), any (read) | Product catalogue |
| Beats | `POST /beats/generate` | Owner | Generate today's beat |
| Beats | `GET /beats/:id` | Owner | View a complete beat |
| Beats | `PUT /beats/:beatId/stores/:assignmentStoreId/assign` | Owner | Move a store to another delivery partner |
| Beats | `POST /beats/:id/publish` | Owner | Publish a beat |
| Delivery | `GET /delivery/my-beat`, `GET /delivery/my-beat/summary` | Delivery | My beat and day summary |
| Delivery | `GET /delivery/stores/:assignmentStoreId` | Delivery | Visit details |
| Delivery | `POST /delivery/visits/:assignmentStoreId/start` | Delivery | Start a visit |
| Delivery | `POST/PUT/DELETE /delivery/visits/:visitId/items` | Delivery | Add, update or remove delivered items |
| Delivery | `POST /delivery/visits/:visitId/returns` | Delivery | Record returns |
| Delivery | `POST /delivery/visits/:visitId/payment` | Delivery | Record a cash or UPI payment |
| Delivery | `POST /delivery/visits/:visitId/payment/razorpay/order` and `/verify` | Delivery | Razorpay order and verification |
| Delivery | `POST /delivery/visits/:visitId/credit` | Delivery | Record a credit promise |
| Delivery | `POST /delivery/visits/:visitId/complete` | Delivery | Complete the visit |
| Dashboard | `GET /dashboard/owner` | Owner | Owner dashboard data |
| Ledger | `GET /ledger/stores/:storeId` | Owner | Store ledger history |
| Reconciliation | `GET /reconciliation/beats/:beatId` | Owner | Expected vs actual collection |
| Team | `GET /team` | Any | Distributor team |
| Team | `GET /team/distributor/mine` | Any | Distributors I belong to |
| Team | `POST /team/distributor/preview/:code` | Any | Preview a distributor by team code |
| Team | `POST /team/distributor/join` | Any | Join a distributor |

**Socket.IO:** clients connect with `auth: { token }`. Owners join a `distributor:<id>` room, and clients can switch distributor context with the `distributor:select` event.

---

## 🗄 Data Model

Main entities (see `backend/prisma/schema.prisma`):

- **Distributor**, **User** (`owner` or `delivery_boy`), **DistributorMember**
- **Store**: outstanding balance, last visited date, overdue days
- **SKU**: product catalogue
- **Beat**, **BeatAssignment**, **BeatAssignmentStore**: the daily plan, with visit order and risk snapshots
- **StoreVisit**, **DeliveryItem**, **ReturnItem**: what happened at each store
- **Payment**: cash or UPI, with Razorpay order/payment/signature fields
- **CreditPromise**: amount promised and promised date
- **StoreLedgerEntry**: previous balance → sales, returns, payments, credit → new balance

---

## 🧪 Testing

```bash
cd backend
node --test test/
```

The ledger test suite checks the sales, return, payment and credit calculations (for example, only captured payments are counted).

---

## 🗺 Roadmap

- Replace the mock "Ask BeatSync AI" assistant with a real data-backed one
- Move API URLs to environment variables for the web and mobile apps
- Route optimisation using store coordinates
- Offline mode for delivery partners
- Push notifications for new or changed beats
- Docker Compose setup for one-command local start
- Full automated test coverage and CI

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes and push the branch
4. Open a pull request

---

Built by [@vharini24](https://github.com/vharini24) · ⭐ Star the repo if you find it useful!
