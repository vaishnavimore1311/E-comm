# 🛒 Mini E-Commerce Demo Project (MERN Stack)

A lightweight, modern, and production-ready **Mini E-Commerce Demo Project** built using the **MERN Stack** (MongoDB, Express.js, React.js, Node.js) styled with **Tailwind CSS**.

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Tech Stack](#-tech-stack)
- [Key Features](#-key-features)
  - [Customer Features](#customer-features)
  - [Admin Features](#admin-features)
- [Project Architecture & Directory Structure](#-project-architecture--directory-structure)
- [Demo Workflow](#-demo-workflow)
- [Security & Validation Rules](#-security--validation-rules)
- [Documentation Index](#-documentation-index)
- [Environment Variables](#-environment-variables)
- [Getting Started (Planned Setup)](#-getting-started-planned-setup)
- [Project Scope & Constraints](#-project-scope--constraints)

---

## 🚀 Overview

This repository contains the architecture, specifications, database design, and implementation blueprints for a clean, full-stack **Mini E-Commerce Demo Application**. 

The application is structured into two distinct decoupled modules:
1. **Frontend (`/client`)**: Single Page Application (SPA) powered by **React 18 + Vite** and styled with **Tailwind CSS**.
2. **Backend (`/server`)**: Robust REST API built on **Node.js + Express.js**, interfacing with **MongoDB** via **Mongoose**.

---

## 🛠 Tech Stack

| Layer | Technology | Details |
| :--- | :--- | :--- |
| **Frontend** | React.js (v18+) | Fast modern SPA initialized with Vite |
| **Styling** | Tailwind CSS | Utility-first, responsive, and clean design system |
| **Routing** | React Router DOM (v6+) | Client-side routing with Public & Protected guards |
| **HTTP Client** | Axios | Configured with interceptors for JWT auth |
| **Backend** | Node.js + Express.js | Modular RESTful API architecture |
| **Database** | MongoDB | Document database hosted locally or via MongoDB Atlas |
| **ODM** | Mongoose (v8+) | Schema validation, type casting, and relationship models |
| **Authentication** | JWT (JSON Web Tokens) | Stateless authorization stored securely in client storage |
| **Password Security**| bcrypt.js | Cryptographic hashing with 10 salt rounds |

---

## ✨ Key Features

### 👤 Customer Features
- **User Authentication**:
  - Secure Registration (Name, Email, Password, Confirm Password validation).
  - Secure Login with JWT token issuance.
  - One-click Logout with automatic state and token clearance.
- **Product Discovery**:
  - Product catalog browsing with responsive grid layouts.
  - Dynamic Category Filtering (e.g., *All | Electronics | Fashion | Shoes*).
  - Real-time Product Search by keyword.
  - Dedicated Product Details view showing images, category, price, and stock status.
- **Shopping Cart**:
  - Add to cart with live quantity selection.
  - Stock-bounded quantity increment and decrement (prevents ordering beyond available stock).
  - Item removal and dynamic subtotal / total calculation.
  - Persistent cart state in local storage.
- **Checkout & Orders**:
  - Simplified single-step Checkout form (Name, Phone, Address, City, Pincode).
  - **Cash on Delivery (COD)** payment mode.
  - Server-side stock verification and atomic stock decrement on order placement.
  - "My Orders" customer portal to track order history and fulfillment statuses.

### 🛡 Admin Features
- **Admin Dashboard**:
  - Dedicated administrative control panel with responsive sidebar and metrics.
  - Role-protected routes guarded by `adminMiddleware`.
- **Category Management (CRUD)**:
  - Create, view, edit, and delete categories (Name & Description).
- **Product Management (CRUD)**:
  - Add new products with image URL, category assignment, price, and inventory stock.
  - Update product details and inventory levels.
  - Delete products with modal confirmation.
- **Order Management**:
  - Master view of all customer orders across the platform.
  - Detailed modal inspection of customer shipping address, ordered items, and order totals.
  - Order status workflow updater: `Pending` → `Confirmed` → `Shipped` → `Delivered` (or `Cancelled`).

---

## 🏗 Project Architecture & Directory Structure

The project is split cleanly into `/client` and `/server`:

```text
E-comm/
├── docs/                               # Comprehensive project documentation
│   ├── ARCHITECTURE.md                 # System architecture and data flow
│   ├── API_SPECIFICATION.md            # REST API reference and contracts
│   ├── DATABASE_MODELS.md              # MongoDB / Mongoose schema specifications
│   ├── SECURITY_AND_VALIDATION.md      # Auth, bcrypt, JWT, and validation guards
│   ├── FRONTEND_UI_UX.md               # UI design system, pages, and components
│   └── DEVELOPMENT_PLAN.md             # Step-by-step development roadmap
│
├── client/                             # React + Vite Frontend
│   ├── public/                         # Public static assets
│   ├── src/
│   │   ├── api/                        # Axios instance and API service functions
│   │   ├── components/                 # Reusable UI components (Navbar, Footer, Modals, Cards)
│   │   │   ├── common/                 # Buttons, Inputs, Dialogs, Badges
│   │   │   ├── admin/                  # Sidebar, Admin Tables, Stats Cards
│   │   │   └── shop/                   # ProductCard, CategoryPill, CartDrawer
│   │   ├── context/                    # React Context (AuthContext, CartContext)
│   │   ├── layouts/                    # MainLayout, AdminLayout
│   │   ├── pages/                      # Public & Admin pages
│   │   │   ├── public/                 # Home, Products, ProductDetails, Cart, Checkout, MyOrders
│   │   │   ├── auth/                   # Login, Register
│   │   │   └── admin/                  # Dashboard, Categories, Products, Orders
│   │   ├── routes/                     # AppRoutes with ProtectedRoute guards
│   │   ├── App.jsx                     # Root application component
│   │   ├── main.jsx                    # React entrypoint
│   │   └── index.css                   # Tailwind directives and custom styles
│   ├── index.html                      # HTML root
│   ├── tailwind.config.js              # Tailwind CSS configuration
│   ├── vite.config.js                  # Vite configuration
│   └── package.json                    # Client dependencies
│
├── server/                             # Node.js + Express Backend
│   ├── src/
│   │   ├── config/                     # Database connection (db.js)
│   │   ├── controllers/                # Request handlers (auth, category, product, order)
│   │   ├── middleware/                 # authMiddleware.js, adminMiddleware.js, errorHandler.js
│   │   ├── models/                     # Mongoose models (User, Category, Product, Order)
│   │   ├── routes/                     # Express route definitions
│   │   ├── utils/                      # Token generator, validation helpers
│   │   ├── app.js                      # Express app initialization & middleware configuration
│   │   └── server.js                   # Server bootstrap and port listening
│   ├── .env.example                    # Environment variable template
│   └── package.json                    # Server dependencies
│
└── README.md                           # Master Project Overview
```

---

## 🔄 Demo Workflow

The complete end-to-end customer and administrative flow operates as follows:

```text
      ADMIN USER                                      CUSTOMER
          │                                              │
          ▼                                              │
    Admin Login                                          │
          │                                              │
          ▼                                              │
    Add Category (e.g., Electronics)                     │
          │                                              │
          ▼                                              │
    Add Product (e.g., Smartphone, $699, Stock: 15)      │
          │                                              │
          ▼                                              │
┌───────────────────────────┐                            │
│ Product published to DB   │                            │
└───────────────────────────┘                            │
          │                                              │
          │   Product instantly visible on storefront    ▼
          └───────────────────────────────────────► Browse Catalog
                                                         │
                                                         ▼
                                                    Register / Login
                                                         │
                                                         ▼
                                                    Filter by Category
                                                    or Search Products
                                                         │
                                                         ▼
                                                    Add to Cart
                                                    (Max qty bounded by stock)
                                                         │
                                                         ▼
                                                    Proceed to Checkout
                                                    (Fill Shipping Form - COD)
                                                         │
                                                         ▼
                                                    Place Order
                                                         │
          ┌──────────────────────────────────────────────┴──────────────┐
          │ 1. Backend fetches official prices from DB                  │
          │ 2. Backend validates available stock                        │
          │ 3. Order is created in MongoDB                              │
          │ 4. Product stock is reduced in DB                           │
          │ 5. Customer cart is cleared                                 │
          └──────────────────────────────────────────────┬──────────────┘
          │                                              │
          ▼                                              ▼
Admin views order in                             Customer sees order in
Admin Orders Dashboard                           "My Orders" (Status: Pending)
          │                                              │
          ▼                                              ▼
Admin updates Status                             Customer sees updated status
(Pending ➔ Confirmed ➔ Shipped ➔ Delivered)     (Real-time or on page refresh)
```

---

## 🔒 Security & Validation Rules

1. **Price Integrity (Zero Client Trust)**:
   - The server **never** trusts product prices sent from the client payload during checkout.
   - The backend queries each product's current database record and calculates the grand total authoritatively.
2. **Inventory Safety**:
   - Stock quantities cannot become negative.
   - Orders fail cleanly if any item exceeds real-time available stock.
   - Decrementing stock is executed with atomic DB operations (`$inc: { stock: -qty }`).
3. **Password Security**:
   - Passwords must be a minimum of 6 characters.
   - Hashes are computed using `bcryptjs` with salt rounds prior to persistence.
   - Plaintext passwords are never returned in API responses (`select: '-password'`).
4. **JWT Authentication & RBAC**:
   - Protected endpoints require an `Authorization: Bearer <token>` header.
   - Standard customers cannot access `/api/admin/*` or administrative CRUD routes.

---

## 📚 Documentation Index

Detailed specifications for each aspect of this project can be found in the [`/docs`](./docs) directory:

| Document | Description |
| :--- | :--- |
| **[Architecture & System Flow](./docs/ARCHITECTURE.md)** | Deep dive into system components, data flow, middleware pipelines, and state handling. |
| **[REST API Specifications](./docs/API_SPECIFICATION.md)** | Exhaustive documentation of all endpoints, parameters, request/response bodies, and HTTP status codes. |
| **[Database & Mongoose Models](./docs/DATABASE_MODELS.md)** | Complete Mongoose schemas for `User`, `Category`, `Product`, and `Order` models with index and validation rules. |
| **[Security & Validation Guide](./docs/SECURITY_AND_VALIDATION.md)** | Authentication flow, bcrypt hashing, JWT validation, and input sanitization policies. |
| **[Frontend UI/UX Specification](./docs/FRONTEND_UI_UX.md)** | Design guidelines, Tailwind CSS theme settings, page layouts, components, and interactive states. |
| **[Development Roadmap](./docs/DEVELOPMENT_PLAN.md)** | Step-by-step phased execution plan to build, test, and deliver the project. |

---

## ⚙️ Environment Variables

Create `.env` files in their respective folders when running the project:

### Backend (`/server/.env`)
```env
PORT=5000
NODE_ENV=development
MONGO_URI=mongodb://localhost:27017/mini_ecommerce
JWT_SECRET=your_jwt_super_secret_key_change_in_production
JWT_EXPIRE=7d
CLIENT_URL=http://localhost:5173
```

### Frontend (`/client/.env`)
```env
VITE_API_BASE_URL=http://localhost:5000/api
```

---

## 💻 Getting Started (Planned Setup)

When ready to start coding:

```bash
# 1. Clone the repository
git clone https://github.com/vaishnavimore1311/E-comm.git
cd E-comm

# 2. Setup Server
cd server
npm install
npm run dev

# 3. Setup Client (in a separate terminal)
cd ../client
npm install
npm run dev
```

---

## 🎯 Project Scope & Constraints

To ensure this project remains a **clean, lightweight, and focused MERN mini project**:
- ✅ **Included**: User/Admin Auth, Categories CRUD, Products CRUD, Stock Management, Cart, Cash on Delivery (COD) Checkout, Order Status Lifecycle, Responsive Tailwind UI.
- ❌ **Excluded (Intentional Out of Scope)**: Online Payment Gateways (Stripe/Razorpay), Product Reviews/Ratings, Coupons/Discounts, Wishlists, Multi-vendor capabilities, and Complex Analytics.
