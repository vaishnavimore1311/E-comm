# 🔐 Security, Authentication & Data Validation Guide

This document defines the security architecture, authentication mechanisms, authorization middlewares, and data validation rules for the **Mini E-Commerce Demo Project**.

---

## 1. Authentication Architecture

The system uses stateless **JSON Web Tokens (JWT)** for customer and administrator authentication:

```text
User Submits Credentials (email, password)
                   │
                   ▼
       Verify User Existence in DB
                   │
                   ├── User Not Found ──► 401 Unauthorized ("Invalid credentials")
                   ▼
     Compare Password Hash via bcrypt
                   │
                   ├── Hash Mismatch ───► 401 Unauthorized ("Invalid credentials")
                   ▼
       Generate Signed JWT Token
   (Payload: { id: user._id, role: user.role })
                   │
                   ▼
Return Token + User Details (Excluding password hash)
```

### 1.1 JWT Token Generation Utility (`utils/generateToken.js`)

```javascript
const jwt = require('jsonwebtoken');

const generateToken = (id, role) => {
  return jwt.sign(
    { id, role },
    process.env.JWT_SECRET,
    { expiresIn: process.env.JWT_EXPIRE || '7d' }
  );
};

module.exports = generateToken;
```

---

## 2. Password Security with bcrypt

1. **Hashing on Registration and Password Update**:
   - Passwords must **never** be stored in plaintext.
   - We utilize `bcryptjs` with **10 salt rounds** within Mongoose's `pre('save')` middleware hook:
   ```javascript
   userSchema.pre('save', async function (next) {
     if (!this.isModified('password')) return next();
     const salt = await bcrypt.genSalt(10);
     this.password = await bcrypt.hash(this.password, salt);
     next();
   });
   ```
2. **Password Verification**:
   ```javascript
   userSchema.methods.matchPassword = async function (enteredPassword) {
     return await bcrypt.compare(enteredPassword, this.password);
   };
   ```
3. **Projection Protection**:
   - The password field has `select: false` configured on the schema to prevent accidental leaks in DB query results.

---

## 3. Route Protection Middlewares

### 3.1 Authentication Middleware (`middleware/authMiddleware.js`)

Verifies the Bearer token in the `Authorization` header, decodes the user ID, and attaches the active user to `req.user`.

```javascript
const jwt = require('jsonwebtoken');
const User = require('../models/User');

const protect = async (req, res, next) => {
  let token;

  if (
    req.headers.authorization &&
    req.headers.authorization.startsWith('Bearer')
  ) {
    try {
      token = req.headers.authorization.split(' ')[1];
      const decoded = jwt.verify(token, process.env.JWT_SECRET);

      req.user = await User.findById(decoded.id).select('-password');
      if (!req.user) {
        return res.status(401).json({ success: false, message: 'User no longer exists' });
      }

      next();
    } catch (error) {
      return res.status(401).json({ success: false, message: 'Invalid or expired authorization token' });
    }
  }

  if (!token) {
    return res.status(401).json({ success: false, message: 'Authorization token required' });
  }
};

module.exports = { protect };
```

### 3.2 Admin Authorization Middleware (`middleware/adminMiddleware.js`)

Guards admin endpoints, ensuring only users with `role === 'admin'` can execute privileged actions.

```javascript
const admin = (req, res, next) => {
  if (req.user && req.user.role === 'admin') {
    next();
  } else {
    res.status(403).json({ success: false, message: 'Forbidden: Admin access required' });
  }
};

module.exports = { admin };
```

---

## 4. Crucial Security Rules

### 4.1 Golden Rule: Never Trust Frontend Price 🛡️

> **Vulnerability Vector**: An attacker can modify client-side JavaScript or intercept network requests to send `price: 0.01` for a \$1,000 laptop.

**Our Mitigation**:
- The checkout endpoint (`POST /api/orders`) accepts **only** an array of `productId` and `quantity`.
- The server iterates through every ordered item, queries the real, current price from the MongoDB `Product` collection, and computes the authoritative `totalAmount`.
- Any price field sent in the request payload is strictly ignored.

```javascript
// Server-side Total Calculation
let totalAmount = 0;
const verifiedProducts = [];

for (const item of req.body.items) {
  const dbProduct = await Product.findById(item.productId);
  if (!dbProduct) {
    return res.status(404).json({ success: false, message: `Product not found: ${item.productId}` });
  }

  if (dbProduct.stock < item.quantity) {
    return res.status(400).json({
      success: false,
      message: `Insufficient stock for "${dbProduct.name}". Only ${dbProduct.stock} units left.`
    });
  }

  const itemTotal = dbProduct.price * item.quantity;
  totalAmount += itemTotal;

  verifiedProducts.push({
    product: dbProduct._id,
    name: dbProduct.name,
    price: dbProduct.price, // Real DB price snapshot
    quantity: item.quantity
  });
}
```

---

### 4.2 Inventory Concurrency & Atomic Stock Decrement

To prevent over-selling when multiple customers order the same item simultaneously:

```javascript
// Atomically deduct inventory
for (const item of verifiedProducts) {
  const result = await Product.findOneAndUpdate(
    { _id: item.product, stock: { $gte: item.quantity } },
    { $inc: { stock: -item.quantity } },
    { new: true }
  );

  if (!result) {
    // If another transaction claimed the stock in between, rollback / abort
    return res.status(400).json({
      success: false,
      message: `Stock conflict: "${item.name}" was just claimed by another order.`
    });
  }
}
```

---

## 5. Input Validation Rules Matrix

| Field | Location | Validation Rule | Error Message |
| :--- | :--- | :--- | :--- |
| `name` (User) | Frontend & Backend | Non-empty, trimmed, min 2 chars | "Name must be at least 2 characters" |
| `email` | Frontend & Backend | Valid email format, lowercase, unique | "Please provide a valid, unique email" |
| `password` | Frontend & Backend | Minimum 6 characters | "Password must be at least 6 characters" |
| `confirmPassword` | Frontend & Backend | Must strictly match `password` | "Passwords do not match" |
| `price` (Product) | Frontend & Backend | Number > 0 | "Product price must be greater than zero" |
| `stock` (Product) | Frontend & Backend | Integer >= 0 | "Stock cannot be negative" |
| `category` | Backend | Valid MongoDB ObjectId | "Valid category reference is required" |
| `items` (Order) | Backend | Non-empty array, valid product IDs | "Order must contain at least one item" |
| `quantity` (Cart/Order) | Frontend & Backend | Integer between 1 and available stock | "Quantity must be between 1 and stock limit" |
| `shippingAddress` | Frontend & Backend | All subfields (name, phone, address, city, pincode) non-empty | "All shipping address fields are required" |
| `status` (Order) | Backend | Enum: Pending, Confirmed, Shipped, Delivered, Cancelled | "Invalid order status value" |

---

## 6. Frontend Validation & Guard UX

1. **Client Form Validation**:
   - Forms use controlled inputs and trigger instant inline feedback before submitting HTTP requests.
   - Disable submission buttons while requests are in-flight to prevent duplicate posts.
2. **Protected Route Redirects**:
   - If an unauthenticated user accesses `/checkout` or `/my-orders`, they are seamlessly redirected to `/login` with an informative toast message.
   - If a non-admin attempts to access `/admin/*`, they are redirected to `/` with an access-denied notification.
