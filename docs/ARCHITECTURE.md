# 🏛 System Architecture & Design Document

This document outlines the architectural blueprints, technical topology, communication patterns, and state workflows for the **Mini E-Commerce Demo Project (MERN Stack)**.

---

## 1. High-Level Architecture Overview

The system is designed following a classic **Decoupled Client-Server Tiered Architecture**:

```text
┌──────────────────────────────────────────────────────────┐
│                      CLIENT TIER                         │
│   React 18 + Vite SPA | Tailwind CSS | React Router 6    │
│   Axios HTTP Client with Authorization Interceptors      │
└─────────────────────────────┬────────────────────────────┘
                              │
                    HTTPS / JSON REST API
                              │
┌─────────────────────────────▼────────────────────────────┐
│                      SERVER TIER                         │
│             Node.js + Express.js Web Server              │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Middlewares: CORS, Express JSON, Auth, Admin, Err  │  │
│  └──────────────────────────┬─────────────────────────┘  │
│  ┌──────────────────────────▼─────────────────────────┐  │
│  │ Controllers & Business Logic: Auth, Products, etc. │  │
│  └──────────────────────────┬─────────────────────────┘  │
│  ┌──────────────────────────▼─────────────────────────┐  │
│  │ Mongoose ODM Data Access Layer                     │  │
│  └────────────────────────────────────────────────────┘  │
└─────────────────────────────┬────────────────────────────┘
                              │
                     MongoDB Wire Protocol
                              │
┌─────────────────────────────▼────────────────────────────┐
│                     DATABASE TIER                        │
│         MongoDB (Users, Categories, Products, Orders)    │
└──────────────────────────────────────────────────────────┘
```

---

## 2. Component Breakdown

### 2.1 Frontend Client (`/client`)

The frontend is a single-page application built on React with Vite:
- **Build Tool**: Vite provides near-instant Hot Module Replacement (HMR) and fast build packaging.
- **Styling**: Tailwind CSS delivers modern, utility-first UI styling with custom design tokens.
- **State Management**:
  - `AuthContext`: Manages current authenticated user (`user`, `token`, `isAdmin`), login, logout, and token retention via `localStorage`.
  - `CartContext`: Manages shopping cart array, quantity modifications, price computation, and synchronization with `localStorage`.
- **Routing & Guards**:
  - **Public Routes**: Accessible by anyone (`/`, `/products`, `/products/:id`, `/cart`, `/login`, `/register`).
  - **Customer Protected Routes**: Requires active JWT token (`/checkout`, `/my-orders`).
  - **Admin Protected Routes**: Requires active JWT token and `role === 'admin'` (`/admin`, `/admin/categories`, `/admin/products`, `/admin/orders`).
- **Network Layer**: Centralized Axios instance (`src/api/axiosClient.js`) that automatically attaches `Authorization: Bearer <token>` to outbound requests and handles `401 Unauthorized` responses.

### 2.2 Backend Server (`/server`)

The backend is built as a RESTful web service using Express on Node.js:
- **Router Layer**: Routes map HTTP methods and URIs to controller functions (`/api/auth`, `/api/categories`, `/api/products`, `/api/orders`, `/api/admin/orders`).
- **Middleware Pipeline**:
  1. `cors()`: Cross-Origin Resource Sharing enablement for the Vite client.
  2. `express.json()`: Request payload parser.
  3. `protect`: JWT token validation middleware.
  4. `admin`: Role checking middleware ensuring only admins access restricted endpoints.
  5. `errorHandler`: Global centralized error interceptor for uniform JSON error responses.
- **Controller Layer**: Decoupled handlers responsible for request validation, invoking model queries, computing business rules, and returning consistent HTTP responses.
- **Model Layer**: Mongoose schemas defining schema rules, timestamps, constraints, and relationships.

---

## 3. Core Data Flow Diagrams

### 3.1 Authentication & Request Authorization Flow

```text
Client (Browser)                 Backend (Express)                 Database (MongoDB)
      │                                  │                                  │
      ├─── POST /api/auth/login ────────►│                                  │
      │    { email, password }           ├──── Find User by Email ─────────►│
      │                                  │◄─── Returns User with Hash ──────┤
      │                                  │                                  │
      │                                  ├──── Compare bcrypt password      │
      │                                  ├──── Generate JWT Token           │
      │◄── 200 OK + JWT Token + User ────┤                                  │
      │                                  │                                  │
      ├─ Store token in localStorage ───┤                                  │
      │                                  │                                  │
      ├─── GET /api/orders/my-orders ───►│                                  │
      │    Header: Bearer <token>        ├──── Verify JWT Secret            │
      │                                  ├──── Attach req.user              │
      │                                  ├──── Query orders for user ──────►│
      │                                  │◄─── Return user's orders ────────┤
      │◄── 200 OK + [Order List] ────────┤                                  │
```

---

### 3.2 Secure Order Placement Flow (Zero Trust Pricing & Stock Validation)

```text
Client (Cart Page)                  Backend (/api/orders)              Database (MongoDB)
      │                                       │                                 │
      ├─── POST /api/orders ─────────────────►│                                 │
      │    Payload: items [{ productId, qty }]│                                 │
      │    (No trusted price from client)     │                                 │
      │                                       ├─ Query Products by IDs ────────►│
      │                                       │◄ Return Product Records ────────┤
      │                                       │  (with official price & stock)  │
      │                                       │                                 │
      │                                       ├─ 1. Verify Stock >= Requested   │
      │                                       │     (If not: 400 Bad Request)   │
      │                                       │                                 │
      │                                       ├─ 2. Calculate authoritative     │
      │                                       │     totalAmount using DB prices │
      │                                       │                                 │
      │                                       ├─ 3. Create Order Document ─────►│
      │                                       │                                 │
      │                                       ├─ 4. Decrement Stock for each ──►│
      │                                       │     Product in DB               │
      │                                       │                                 │
      │◄── 201 Created + Order Object ────────┤                                 │
      │                                       │                                 │
      ├─── Clear Local Cart ──────────────────┤                                 │
      └─── Redirect to "My Orders" ───────────┘                                 │
```

---

## 4. Middleware Architecture

```text
Incoming HTTP Request
         │
         ▼
┌──────────────────┐
│   CORS Handler   │ ──► Checks Allowed Origins (e.g. http://localhost:5173)
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  express.json()  │ ──► Parses incoming JSON body
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│   Router Match   │ ──► Selects target route (/api/...)
└────────┬─────────┘
         │
         ├──────────────────────────────────────────┐
         │ (If Public Route)                        │ (If Protected Route)
         │                                          ▼
         │                                 ┌──────────────────┐
         │                                 │   protect (JWT)  │ ──► Extracts token from Bearer
         │                                 └────────┬─────────┘     Validates signature & expiry
         │                                          │               Attaches req.user
         │                                          │
         │                                          ├─────────────────────────┐
         │                                          │ (If Admin Route)        │ (If Customer Route)
         │                                          ▼                         │
         │                                 ┌──────────────────┐               │
         │                                 │   admin Guard    │               │
         │                                 └────────┬─────────┘               │
         │                                          │ (If role === 'admin')   │
         │                                          ▼                         ▼
         └─────────────────────────────────► Controller Handler ◄─────────────┘
                                                    │
                                                    ▼
                                           ┌──────────────────┐
                                           │   errorHandler   │ ──► Formats errors into JSON:
                                           └──────────────────┘     { message: "...", stack: ... }
```

---

## 5. Security & Isolation Model

1. **Role-Based Isolation**: Standard customer users can never invoke administrative operations; routes checking `admin` middleware immediately short-circuit with `403 Forbidden`.
2. **Stateless Scalability**: The backend retains no session state in memory; any instance can verify incoming JWT tokens using the symmetric `JWT_SECRET`.
3. **Data Sanitization**: MongoDB queries are formulated using Mongoose type casting, preventing NoSQL injection.
4. **Environment Isolation**: Sensitive credentials (`MONGO_URI`, `JWT_SECRET`) are loaded strictly from environment variables and never checked into source control.
