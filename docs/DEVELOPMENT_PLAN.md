# 🚀 Development Plan & Step-by-Step Implementation Roadmap

This document outlines the structured, phase-by-phase engineering plan for building the **Mini E-Commerce Demo Project (MERN Stack)** once documentation review is concluded.

---

## 📌 Phase Summary Matrix

| Phase | Module | Focus Area | Deliverables |
| :---: | :---: | :--- | :--- |
| **1** | Tooling | Workspace Setup | Initialize `/server` and `/client` directories with dependencies |
| **2** | Backend | Database & Schemas | MongoDB connection + 4 Mongoose models (`User`, `Category`, `Product`, `Order`) |
| **3** | Backend | Auth & Middlewares | bcrypt password hashing, JWT generation, `protect` & `admin` middlewares |
| **4** | Backend | RESTful Controllers | Category CRUD, Product CRUD with query filters, Order placement & status updates |
| **5** | Frontend | Foundation & Styling | React 18 + Vite, Tailwind CSS configuration, Axios client configuration |
| **6** | Frontend | Auth & Routing | `AuthContext`, Login/Register views, Protected Route guards |
| **7** | Frontend | Storefront & Discovery| Home view, responsive product catalog, category filters, and product details |
| **8** | Frontend | Cart & Checkout | `CartContext` with stock clamp, COD checkout form, order submission |
| **9** | Frontend | Admin Dashboard | Admin sidebar, Category table/modals, Product management, Order status management |
| **10**| Testing | End-to-End Demo Audit | Full user and admin workflow validation against acceptance criteria |

---

## 📋 Detailed Implementation Steps

### Phase 1: Workspace & Environment Setup
- [ ] Create `/server` and `/client` root directories.
- [ ] In `/server`:
  - Initialize Node project: `npm init -y`
  - Install dependencies: `express`, `mongoose`, `dotenv`, `cors`, `jsonwebtoken`, `bcryptjs`
  - Install dev dependencies: `nodemon`
  - Setup `.env.example` with `PORT`, `MONGO_URI`, `JWT_SECRET`, `CLIENT_URL`
- [ ] In `/client`:
  - Initialize Vite React project: `npm create vite@latest client -- --template react`
  - Install dependencies: `axios`, `react-router-dom`, `lucide-react`, `react-hot-toast`
  - Install & initialize Tailwind CSS: `npm install -D tailwindcss postcss autoprefixer && npx tailwindcss init -p`

---

### Phase 2: Database Configuration & Models
- [ ] Setup `server/src/config/db.js` using `mongoose.connect()`.
- [ ] Implement Mongoose Schemas:
  - `server/src/models/User.js`: User schema with email regex, unique index, bcrypt pre-save hash hook, `matchPassword` method.
  - `server/src/models/Category.js`: Category schema with unique trimmed name and description.
  - `server/src/models/Product.js`: Product schema referencing Category, price (> 0), stock (>= 0), image, text search indexes.
  - `server/src/models/Order.js`: Order schema with embedded products snapshot, `totalAmount`, structured `shippingAddress`, payment mode `COD`, and status enum.

---

### Phase 3: Authentication & Security Middlewares
- [ ] Implement `server/src/utils/generateToken.js` with signed JWT payload `{ id, role }`.
- [ ] Implement `server/src/middleware/authMiddleware.js` (`protect` function).
- [ ] Implement `server/src/middleware/adminMiddleware.js` (`admin` role check).
- [ ] Implement `server/src/middleware/errorHandler.js` for centralized error response formatting.
- [ ] Build Auth Controller & Routes (`/api/auth/register`, `/api/auth/login`).
- [ ] Seed an initial Admin account script for demo readiness.

---

### Phase 4: Backend REST APIs
- [ ] **Category Endpoints**:
  - `GET /api/categories`: Public list of all categories.
  - `POST /api/categories`: Admin create.
  - `PUT /api/categories/:id`: Admin update.
  - `DELETE /api/categories/:id`: Admin delete.
- [ ] **Product Endpoints**:
  - `GET /api/products`: Public query supporting `?category=...&search=...`.
  - `GET /api/products/:id`: Public single item retrieval.
  - `POST /api/products`: Admin create.
  - `PUT /api/products/:id`: Admin update.
  - `DELETE /api/products/:id`: Admin delete.
- [ ] **Order Endpoints**:
  - `POST /api/orders`: Secure order placement (authoritative DB price lookup, stock availability verification, atomic `$inc` stock reduction, order persistence).
  - `GET /api/orders/my-orders`: Logged-in customer order history.
  - `GET /api/admin/orders`: Admin list of all orders.
  - `PATCH /api/admin/orders/:id/status`: Admin status modifier (`Pending`, `Confirmed`, `Shipped`, `Delivered`, `Cancelled`).

---

### Phase 5: Client Base Setup & Design Tokens
- [ ] Configure `tailwind.config.js` with primary indigo palette and custom container widths.
- [ ] Set up `client/src/index.css` with `@tailwind base; @tailwind components; @tailwind utilities;`.
- [ ] Configure `client/src/api/axiosClient.js` with `baseURL: import.meta.env.VITE_API_BASE_URL` and auto-attaching Bearer token interceptor.
- [ ] Set up layout shells: `MainLayout.jsx` (with Header and Footer) and `AdminLayout.jsx` (with Sidebar).

---

### Phase 6: Authentication State & Routing Guards
- [ ] Build `AuthContext.jsx`:
  - State: `user`, `token`, `isAuthenticated`, `isAdmin`, `loading`.
  - Actions: `login(email, password)`, `register(userData)`, `logout()`.
  - Persistence via `localStorage`.
- [ ] Implement route guards:
  - `ProtectedRoute.jsx`: Redirects unauthenticated guests to `/login`.
  - `AdminRoute.jsx`: Redirects non-admin users to `/`.
- [ ] Build `Login.jsx` and `Register.jsx` pages with form validations and toast alerts.

---

### Phase 7: Storefront UI & Product Discovery
- [ ] Build `Home.jsx` with Hero banner, Category highlight cards, and latest products.
- [ ] Build `Products.jsx`:
  - Category filter pills (`All | Electronics | Fashion | Shoes`).
  - Search input with query string synchronization.
  - Responsive `ProductCard.jsx` grid.
- [ ] Build `ProductDetails.jsx`:
  - High-resolution image preview.
  - Category badge, description, and pricing.
  - Live stock badge (`In Stock` / `Out of Stock`).
  - Quantity picker with maximum bound enforcement.
  - `"Add to Cart"` action with toast feedback.

---

### Phase 8: Cart Context, Stock Limits & COD Checkout
- [ ] Build `CartContext.jsx`:
  - State: `cartItems` array synced with `localStorage`.
  - Actions: `addToCart(product, qty)`, `updateQuantity(productId, newQty)` (strictly clamped to `product.stock`), `removeFromCart(productId)`, `clearCart()`.
  - Computed: `subtotal`, `itemCount`.
- [ ] Build `Cart.jsx`:
  - Item listing with quantity controllers and delete triggers.
  - Order summary card with checkout redirect button.
- [ ] Build `Checkout.jsx`:
  - Shipping address inputs (Name, Phone, Address, City, Pincode).
  - COD payment badge.
  - Order submission handling: Invokes `POST /api/orders`, clears cart on 201 response, redirects to `/my-orders`.
- [ ] Build `MyOrders.jsx`:
  - List customer order cards, items, total amounts, and colored status badges.

---

### Phase 9: Admin Dashboard
- [ ] Build `AdminDashboard.jsx` summary view with quick stats.
- [ ] Build `AdminCategories.jsx`:
  - Table of categories.
  - Modal for Add/Edit category.
  - Confirmation dialog for category deletion.
- [ ] Build `AdminProducts.jsx`:
  - Table showing thumbnail, name, category, price, and stock.
  - Modal with form for creating and updating products.
  - Confirmation dialog for product deletion.
- [ ] Build `AdminOrders.jsx`:
  - Master table of all customer orders.
  - Order Details modal showing delivery address and ordered items.
  - Status change dropdown with instant PATCH trigger.

---

### Phase 10: End-to-End Demo Verification Checklist

Before final project sign-off, verify the complete execution of the 10-step demo flow:
- [ ] **1. Admin Login**: Log in as admin user and verify access to `/admin`.
- [ ] **2. Add Category**: Successfully create categories (e.g. `Electronics`, `Fashion`, `Shoes`).
- [ ] **3. Add Product**: Create a product with image, category, price, and stock (e.g., Stock: 5).
- [ ] **4. Catalog Sync**: Confirm product appears immediately on the public website.
- [ ] **5. Customer Registration**: Register a new customer user and log in.
- [ ] **6. Search & Filter**: Test category filter pills and search bar.
- [ ] **7. Cart Limits**: Add item to cart; ensure quantity cannot be incremented past available stock (5).
- [ ] **8. Checkout**: Enter shipping details and submit order via Cash on Delivery.
- [ ] **9. Inventory Decrement**: Verify database product stock drops from 5 to 4.
- [ ] **10. Status Lifecycle**: Admin views the order in dashboard and transitions status: `Pending` ➔ `Confirmed` ➔ `Shipped` ➔ `Delivered`.
