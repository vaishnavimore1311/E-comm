# 🎨 Frontend UI / UX & Design System Specification

This document details the frontend architecture, Tailwind CSS design system, public storefront views, administrative dashboard layouts, interactive components, and UX micro-interactions.

---

## 1. Design Principles & Style Guide

- **Modern & Clean**: Minimalist whitespace, crisp typography, and uncluttered layouts.
- **Accessible & Responsive**: Full responsiveness on mobile (360px+), tablet (768px+), laptop (1024px+), and desktop (1280px+).
- **Feedback-Rich**: Clear loading skeletons, empty state illustrations, toast alerts, and confirmation dialogs.

### 1.1 Color Palette

| Token | Class / Value | Purpose |
| :--- | :--- | :--- |
| **Primary** | `indigo-600` (`#4F46E5`) | Brand accent, primary action buttons, active tabs |
| **Primary Hover** | `indigo-700` (`#4338CA`) | Hover state for buttons and links |
| **Background** | `gray-50` (`#F9FAFB`) | Application background |
| **Surface/Card** | `white` (`#FFFFFF`) | Cards, tables, modal containers, and sidebars |
| **Text Primary** | `gray-900` (`#111827`) | Main headers and primary text |
| **Text Muted** | `gray-500` (`#6B7280`) | Labels, secondary descriptions, timestamps |
| **Success** | `emerald-600` (`#059669`) | In-stock badges, Delivered status, order success |
| **Warning** | `amber-500` (`#F59E0B`) | Low stock alerts, Pending status, warnings |
| **Danger** | `rose-600` (`#E11D48`) | Delete actions, Cancelled status, error notifications |

---

## 2. Public Storefront Pages

### 2.1 Navigation Bar (`Navbar.jsx`)
- **Logo**: Bold brand text (`MiniStore`).
- **Navigation Links**:
  - `Home` (`/`)
  - `Products` (`/products`)
  - `My Orders` (`/my-orders`, visible when logged in)
- **Actions Bar**:
  - Search trigger icon.
  - Cart Icon with live badge count (e.g. `3`).
  - Auth Buttons: `Login` / `Register` if guest; User avatar dropdown with `Logout` & `Admin Dashboard` link (if `role === 'admin'`).
  - Responsive mobile drawer toggle.

---

### 2.2 Home Page (`pages/public/Home.jsx`)
- **Hero Section**:
  - Catchy title, subtitle, and primary call-to-action button: `"Explore Catalog"` pointing to `/products`.
- **Category Quick-Filter**:
  - Visual cards representing top categories (`Electronics`, `Fashion`, `Shoes`).
  - Clicking a category card navigates directly to `/products?category=...`.
- **Featured Products Section**:
  - Responsive 4-column product grid showcasing top in-stock products with `"Add to Cart"` buttons.
- **Value Propositions Banner**:
  - Fast Delivery, 100% Quality Guaranteed, Cash on Delivery available.

---

### 2.3 Products Catalog Page (`pages/public/Products.jsx`)
- **Search Bar**:
  - Centered search input with instant debounce or submit trigger.
- **Category Filter Pills**:
  - Horizontal pill bar: `All` | `Electronics` | `Fashion` | `Shoes`.
  - Active pill is highlighted in `bg-indigo-600 text-white`.
- **Product Grid**:
  - 1 column on mobile, 2 columns on tablet, 3-4 columns on desktop.
  - **Product Card (`ProductCard.jsx`)**:
    - Product Image with smooth hover zoom.
    - Category badge.
    - Product title and truncated description.
    - Price formatted as `$XX.XX`.
    - Stock status: In Stock (`text-emerald-600`) vs Out of Stock (`text-rose-500`).
    - `"Add to Cart"` button (disabled if `stock === 0`).
- **Empty State**:
  - Friendly icon with `"No products match your search or filter"` message.

---

### 2.4 Product Details Page (`pages/public/ProductDetails.jsx`)
- **Layout**: 2-column split (Left: High-res product image; Right: Product info & purchase actions).
- **Details Displayed**:
  - Category breadcrumb.
  - Product Name and Price.
  - Inventory Status: `"In Stock (X available)"` or `"Out of Stock"`.
  - Full product description.
  - Quantity selector: `[-] [ 1 ] [+]` (clamped between 1 and `product.stock`).
  - Primary `"Add to Cart"` button.

---

### 2.5 Shopping Cart Page (`pages/public/Cart.jsx`)
- **Item List**:
  - Thumbnail image, product name, and unit price.
  - Quantity controller with live bounds check against product stock.
  - Total line price (`qty * price`).
  - Trash can icon to remove item with toast notification.
- **Order Summary Card**:
  - Subtotal calculation.
  - Shipping fee (Free).
  - Total Amount.
  - Primary button: `"Proceed to Checkout"` (redirects to `/checkout`).
- **Empty Cart View**:
  - Empty cart icon with `"Your cart is currently empty"` and button `"Continue Shopping"`.

---

### 2.6 Checkout Page (`pages/public/Checkout.jsx`)
- **Access**: Customer authentication required.
- **Shipping Address Form**:
  - Full Name (`input`)
  - Phone Number (`input`)
  - Street Address (`textarea`)
  - City (`input`)
  - Pincode (`input`)
- **Payment Method**:
  - Selected by default: **Cash on Delivery (COD)** with info icon: *"Pay with cash when your shipment arrives at your doorstep"*.
- **Review Items & Total**:
  - Compact summary of items being purchased.
  - Grand total display.
- **Submit Action**:
  - Button: `"Place Order (Cash on Delivery)"` with loading spinner during submission.
  - On success: Empties cart, presents success toast, and redirects to `/my-orders`.

---

### 2.7 My Orders Page (`pages/public/MyOrders.jsx`)
- **Access**: Customer authentication required.
- **Order History Cards**:
  - Order ID & Placed Date.
  - Status Badge with dynamic colors:
    - `Pending` (Yellow)
    - `Confirmed` (Blue)
    - `Shipped` (Purple)
    - `Delivered` (Green)
    - `Cancelled` (Red)
  - List of items with thumbnail, title, quantity, and unit price snapshot.
  - Grand total and shipping address summary.
- **Empty State**:
  - `"You haven't placed any orders yet"` with a link to the store.

---

## 3. Admin Dashboard (`/admin`)

### 3.1 Admin Layout (`layouts/AdminLayout.jsx`)
- **Sidebar**:
  - Platform Brand & `"Admin Dashboard"` indicator.
  - Navigation links:
    - 📦 `Products` (`/admin/products`)
    - 🗂 `Categories` (`/admin/categories`)
    - 📑 `Orders` (`/admin/orders`)
  - Back to Storefront link (`/`).
  - Logout button.
- **Topbar**:
  - Admin user greeting and current role badge.

---

### 3.2 Category Management (`pages/admin/AdminCategories.jsx`)
- **Header**: Title + `"Add New Category"` button (opens modal).
- **Categories Table**:
  - Columns: Name, Description, Created Date, Actions.
  - Actions: Edit (opens modal pre-filled), Delete (triggers confirmation dialog).
- **Add/Edit Modal**:
  - Name (required input).
  - Description (textarea).
  - Save / Cancel buttons.

---

### 3.3 Product Management (`pages/admin/AdminProducts.jsx`)
- **Header**: Title + `"Add New Product"` button (opens modal).
- **Products Table**:
  - Columns: Image preview, Product Name, Category, Price, Stock, Actions.
  - Actions: Edit, Delete.
- **Add/Edit Modal**:
  - Product Name (input).
  - Description (textarea).
  - Price (number input with step `0.01`).
  - Image URL (input with live thumbnail preview).
  - Category (dropdown select populated dynamically from `/api/categories`).
  - Stock (number input with min `0`).

---

### 3.4 Order Management (`pages/admin/AdminOrders.jsx`)
- **Orders Table**:
  - Columns: Order ID, Customer Name & Email, Date, Total Amount, Current Status, Actions.
- **Order Actions**:
  - **Quick Status Changer**: Dropdown to switch status directly between `Pending`, `Confirmed`, `Shipped`, `Delivered`, `Cancelled`.
  - **View Details Modal**:
    - Complete customer shipping address.
    - Itemized order breakdown (product name, unit price, quantity, line subtotal).
    - Payment mode (Cash on Delivery).

---

## 4. UI/UX Interaction Standards

### 4.1 Toast Notifications
- Powered by `react-hot-toast` or custom Tailwind toast dispatch:
  - **Success**: `"Product added to cart"`, `"Category created successfully"`, `"Order placed!"`.
  - **Error**: `"Invalid email or password"`, `"Requested quantity exceeds available stock"`.
  - **Warning**: `"Please sign in to complete checkout"`.

### 4.2 Modal Confirmation Dialogs
- Any destructive operation (such as Deleting a Product or Category) **must** prompt the administrator with a modal confirmation dialog:
  ```text
  ┌────────────────────────────────────────────────────────┐
  │ Confirm Deletion                                       │
  │ Are you sure you want to delete "Pro Smartphone X"?    │
  │ This action cannot be undone.                          │
  │                                                        │
  │                 [ Cancel ]  [ Delete Product (Red) ]   │
  └────────────────────────────────────────────────────────┘
  ```

### 4.3 Loading & Empty States
- **Loading**: Clean CSS skeleton loaders for product cards and admin table rows.
- **Empty States**: Meaningful SVG icons with friendly descriptions whenever tables or catalog filters return 0 results.
