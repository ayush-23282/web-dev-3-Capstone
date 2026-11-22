# NexCart — E-Commerce & Inventory Management System

NexCart is a full-stack **E-Commerce and Inventory Management System** built as a **3rd-Semester B.Tech CSE Backend Development Capstone Project**. It pairs the React + Vite frontend with a modular **Node.js, Express, MongoDB Community Server, and Mongoose** backend using beginner-friendly CommonJS JavaScript.

---

## Tech Stack

- **Backend**: Node.js, Express.js (CommonJS), MongoDB Community Server, Mongoose
- **Security & Auth**: `bcrypt` password hashing, JWT Access Tokens & Refresh Tokens, Role-Based Access Control (`customer`, `seller`, `admin`)
- **Real-Time & Events**: Node.js `EventEmitter`, Socket.IO (JWT-authenticated private user rooms)
- **Frontend**: React 19, React Router DOM, Vite, Tailwind CSS v4, `socket.io-client`

---

## Folder Structure

```text
.
├── backend/
│   ├── config/
│   │   ├── db.js                  # MongoDB connection helper
│   │   └── socket.js              # Socket.IO server + JWT handshake auth
│   ├── controllers/               # Route handlers (MVC pattern)
│   │   ├── analyticsController.js
│   │   ├── authController.js
│   │   ├── cartController.js
│   │   ├── categoryController.js
│   │   ├── inventoryController.js
│   │   ├── notificationController.js
│   │   ├── orderController.js
│   │   └── productController.js
│   ├── events/
│   │   └── appEvents.js           # EventEmitter listeners + DB notifications + Socket emit
│   ├── middleware/
│   │   ├── authMiddleware.js      # JWT Bearer verification
│   │   └── roleMiddleware.js      # Role authorization guard
│   ├── models/                    # 7 Mongoose schemas + indexes
│   │   ├── Cart.js
│   │   ├── Category.js
│   │   ├── InventoryLog.js
│   │   ├── Notification.js
│   │   ├── Order.js
│   │   ├── Product.js
│   │   └── User.js
│   ├── postman/
│   │   └── NexCart.postman_collection.json
│   ├── routes/                    # Express routers mounted in server.js
│   ├── seed/
│   │   └── seed.js                # Idempotent database seeder
│   ├── utils/
│   │   └── tokenUtils.js
│   ├── .env.example
│   ├── package.json
│   ├── README.md                  # Detailed DB schema, ER diagram, & API docs
│   └── server.js                  # Express + Socket.IO entry point
├── src/
│   ├── components/                # Navbar, ProductCard, SearchBar, Pagination
│   ├── context/                   # AuthContext (JWT + Socket.IO) & CartContext
│   ├── pages/                     # Home, ProductDetail, Cart, Wishlist, Orders, Dashboard, Login
│   ├── utils/
│   │   └── api.js                 # REST client with automatic JWT refresh retry
│   ├── App.jsx
│   └── main.jsx
├── .env.example
├── package.json
└── vite.config.js                 # Proxies /api and /socket.io to localhost:5000
```

---

## Core Backend Features

1. **Authentication & RBAC**: Public registration for `customer` and `seller` (admin self-registration blocked), bcrypt password hashing, JWT access/refresh tokens, refresh token rotation check on logout.
2. **Product & Category Management**: Search, category filter, price range filter, availability filter, sorting, pagination, seller ownership enforcement, and safe category deletion checks.
3. **Persistent Cart**: Customer-only cart endpoints where prices and totals are always computed from MongoDB records.
4. **Checkout & Order Lifecycle**:
   - Attempts a **MongoDB transaction** first (for replica sets).
   - Automatically falls back on **standalone MongoDB Community Server** to atomic `findOneAndUpdate` (`stock >= quantity`) stock deduction with compensating rollback on failure.
   - Records `InventoryLog` entries for every checkout, manual adjustment, and order cancellation.
   - Simulates payment (`paid` for `card`/`upi`, `pending` for `cash`, `refunded` when paid orders are cancelled).
5. **Inventory & Analytics**: Low-stock detection (`stock <= lowStockThreshold`), stock update/adjustment endpoints, audit logs, and MongoDB aggregation pipelines for sales reports, best-sellers, and inventory summaries.
6. **Real-Time Notifications**: Node `EventEmitter` triggers persistent `Notification` documents and pushes live Socket.IO events (`notification`, `order:statusUpdated`, `stock:low`) to the exact authenticated user room (`user:<id>`).

---

## Local Setup Guide (Windows)

### Prerequisites

- **Node.js** (v18 or newer)
- **MongoDB Community Server** (running locally on `mongodb://127.0.0.1:27017`)

### 1. Start MongoDB Community Server on Windows

Open Command Prompt or PowerShell (as Administrator if needed) and verify MongoDB is running:

```powershell
net start MongoDB
```

Or launch `mongod` manually if installed without a Windows service:

```powershell
mongod --dbpath C:\data\db
```

### 2. Configure Environment Variables

Create `backend\.env` by copying `backend\.env.example`:

```powershell
copy backend\.env.example backend\.env
```

Edit `backend\.env` and set strong random secrets for `JWT_ACCESS_SECRET` and `JWT_REFRESH_SECRET`:

```env
PORT=5000
MONGO_URI=mongodb://127.0.0.1:27017/nexcart
JWT_ACCESS_SECRET=your_strong_local_access_secret
JWT_REFRESH_SECRET=your_strong_local_refresh_secret
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
CLIENT_URL=http://localhost:3000
```

### 3. Install Dependencies

Install both frontend and backend dependencies:

```powershell
npm install
npm --prefix backend install
```

### 4. Seed Initial Data (Development Only)

Run the idempotent seed script to create sample users, categories, products, and an initial order/log/notification:

```powershell
npm --prefix backend run seed
```

#### Development-Only Test Accounts

> **Warning**: These credentials are for local development and evaluation only. Never use them in production.

| Role | Email | Password |
| :--- | :--- | :--- |
| **Admin** | `admin@nexcart.local` | `Admin@123` |
| **Seller 1** | `seller1@nexcart.local` | `Seller@123` |
| **Seller 2** | `seller2@nexcart.local` | `Seller@123` |
| **Customer 1** | `customer1@nexcart.local` | `Customer@123` |
| **Customer 2** | `customer2@nexcart.local` | `Customer@123` |

### 5. Run the Application

Start the backend in Terminal 1:

```powershell
npm --prefix backend run dev
```

Start the frontend in Terminal 2:

```powershell
npm run dev
```

Open `http://localhost:3000` in your browser. Vite automatically proxies `/api/*` and `/socket.io/*` requests to `http://localhost:5000`.

---

## Testing the Application

1. **API Testing with Postman / Thunder Client**:
   - Import `backend/postman/NexCart.postman_collection.json` into Postman.
   - Call `GET /api/health` to confirm Express and MongoDB status (`Connected`).
   - Log in with any seeded account; save the returned `accessToken` into the collection `accessToken` variable to test protected routes.
2. **End-to-End Browser Flow**:
   - Log in as `customer1@nexcart.local` (`Customer@123`), browse products, add items to cart, place an order, and view `/orders` (try cancelling an order to verify stock restoration).
   - Log in as `seller1@nexcart.local` (`Seller@123`) in another browser/incognito window, open `/dashboard`, inspect analytics, adjust inventory stock (observe `Alerts` badge update live via Socket.IO), and update order status on `/orders`.
   - Log in as `admin@nexcart.local` (`Admin@123`) to manage categories and view storewide analytics and orders.

---

## Known Limitations & Future Improvements

- **Simulated Payments**: Checkout marks `card`/`upi` orders as `paid` and `cash` orders as `pending` without connecting to an external payment gateway (by capstone requirement).
- **Standalone MongoDB Fallback**: Local single-node `mongod` instances do not support multi-document transactions; `orderController.js` automatically falls back to atomic item stock deduction with compensating rollback. Using a replica set enables full ACID transactions automatically.
- **Wishlist Storage**: Wishlist items remain stored in browser `localStorage` to preserve the original frontend behavior, while Cart, Orders, Inventory, Analytics, and Notifications are persisted in MongoDB.
- **Future Enhancements**: Refresh token storage in `HttpOnly` cookies, image uploads via cloud object storage, automated Jest + Supertest integration tests, and email receipts.
