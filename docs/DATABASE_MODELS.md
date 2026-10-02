# 🗄 Database Design & Mongoose Models

This document defines the schema designs, data types, validation rules, relationship references, and indexing strategies for the **MongoDB** database managed via **Mongoose**.

As per requirements, exactly **four models** are defined in this project:
1. `User`
2. `Category`
3. `Product`
4. `Order`

---

## 1. Entity-Relationship (ER) Overview

```text
┌─────────────────────────┐             ┌─────────────────────────┐
│          User           │             │        Category         │
├─────────────────────────┤             ├─────────────────────────┤
│ _id: ObjectId           │             │ _id: ObjectId           │
│ name: String            │             │ name: String (Unique)   │
│ email: String (Unique)  │             │ description: String     │
│ password: String (Hash) │             │ createdAt: Date         │
│ role: 'customer'|'admin'│             │ updatedAt: Date         │
│ createdAt: Date         │             └───────────┬─────────────┘
└───────────┬─────────────┘                         │
            │ 1                                     │ 1
            │                                       │
            │ has many                              │ categorizes
            │                                       │
            ▼ *                                     ▼ *
┌─────────────────────────┐             ┌─────────────────────────┐
│          Order          │             │         Product         │
├─────────────────────────┤             ├─────────────────────────┤
│ _id: ObjectId           │             │ _id: ObjectId           │
│ user: Ref -> User       │             │ name: String            │
│ products: [             │◄── embeds ──┤ description: String     │
│   {                     │    snapshot │ price: Number           │
│     product: Ref,       │    details  │ image: String           │
│     name: String,       │             │ category: Ref->Category │
│     price: Number,      │             │ stock: Number           │
│     quantity: Number    │             │ createdAt: Date         │
│   }                     │             │ updatedAt: Date         │
│ ]                       │             └─────────────────────────┘
│ totalAmount: Number     │
│ shippingAddress: Object │
│ status: Enum String     │
│ createdAt: Date         │
└─────────────────────────┘
```

---

## 2. Model Specifications

### 2.1 User Model (`User.js`)

Represents registered customers and administrative users.

#### Schema Definition
```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, 'Please provide a name'],
      trim: true,
      minlength: [2, 'Name must be at least 2 characters long'],
      maxlength: [50, 'Name cannot exceed 50 characters'],
    },
    email: {
      type: String,
      required: [true, 'Please provide an email'],
      unique: true,
      trim: true,
      lowercase: true,
      match: [
        /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/,
        'Please provide a valid email address',
      ],
      index: true,
    },
    password: {
      type: String,
      required: [true, 'Please provide a password'],
      minlength: [6, 'Password must be at least 6 characters long'],
      select: false, // Omit from default query projections
    },
    role: {
      type: String,
      enum: {
        values: ['customer', 'admin'],
        message: 'Role must be either customer or admin',
      },
      default: 'customer',
    },
  },
  {
    timestamps: true,
  }
);

// Pre-save hook: Hash password with bcrypt before persisting
userSchema.pre('save', async function (next) {
  if (!this.isModified('password')) return next();
  const salt = await bcrypt.genSalt(10);
  this.password = await bcrypt.hash(this.password, salt);
  next();
});

// Helper instance method: Compare plaintext with stored hash
userSchema.methods.matchPassword = async function (enteredPassword) {
  return await bcrypt.compare(enteredPassword, this.password);
};

module.exports = mongoose.model('User', userSchema);
```

#### Fields Reference
| Field | Type | Required | Constraints / Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `name` | String | Yes | Trimmed, 2-50 chars | Full name of the user |
| `email` | String | Yes | Unique, Lowercase, Regex | User's unique email address (login credential) |
| `password` | String | Yes | Min 6 chars, `select: false` | Bcrypt hashed string |
| `role` | String | No | Enum: `'customer'`, `'admin'`. Default: `'customer'` | Authorization access level |
| `createdAt` | Date | Auto | Current timestamp | Record creation date |
| `updatedAt` | Date | Auto | Current timestamp | Record modification date |

---

### 2.2 Category Model (`Category.js`)

Defines product groupings (e.g., Electronics, Fashion, Shoes).

#### Schema Definition
```javascript
const mongoose = require('mongoose');

const categorySchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, 'Category name is required'],
      unique: true,
      trim: true,
      minlength: [2, 'Category name must be at least 2 characters long'],
      maxlength: [50, 'Category name cannot exceed 50 characters'],
      index: true,
    },
    description: {
      type: String,
      trim: true,
      maxlength: [250, 'Description cannot exceed 250 characters'],
      default: '',
    },
  },
  {
    timestamps: true,
  }
);

module.exports = mongoose.model('Category', categorySchema);
```

#### Fields Reference
| Field | Type | Required | Constraints / Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `name` | String | Yes | Unique, Trimmed, 2-50 chars | Unique category display title |
| `description` | String | No | Max 250 chars, default: `""` | Description of items in this category |
| `createdAt` | Date | Auto | Current timestamp | Category creation date |
| `updatedAt` | Date | Auto | Current timestamp | Last update date |

---

### 2.3 Product Model (`Product.js`)

Stores item metadata, inventory stock levels, and category assignments.

#### Schema Definition
```javascript
const mongoose = require('mongoose');

const productSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, 'Product name is required'],
      trim: true,
      minlength: [2, 'Product name must be at least 2 characters long'],
      maxlength: [100, 'Product name cannot exceed 100 characters'],
      index: true,
    },
    description: {
      type: String,
      required: [true, 'Product description is required'],
      trim: true,
      maxlength: [2000, 'Description cannot exceed 2000 characters'],
    },
    price: {
      type: Number,
      required: [true, 'Product price is required'],
      min: [0.01, 'Price must be greater than zero'],
    },
    image: {
      type: String,
      required: [true, 'Product image URL is required'],
      trim: true,
    },
    category: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Category',
      required: [true, 'Product must belong to a category'],
      index: true,
    },
    stock: {
      type: Number,
      required: [true, 'Stock count is required'],
      min: [0, 'Stock cannot be negative'],
      default: 0,
    },
  },
  {
    timestamps: true,
  }
);

// Compound text index for search queries
productSchema.index({ name: 'text', description: 'text' });

module.exports = mongoose.model('Product', productSchema);
```

#### Fields Reference
| Field | Type | Required | Constraints / Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `name` | String | Yes | Trimmed, 2-100 chars | Display title of the product |
| `description` | String | Yes | Trimmed, Max 2000 chars | Detailed product specifications |
| `price` | Number | Yes | Positive number (> 0) | Authoritative unit price |
| `image` | String | Yes | Trimmed URL string | Direct image URL or CDN asset link |
| `category` | ObjectId | Yes | References `Category` model | Foreign reference to associated category |
| `stock` | Number | Yes | Integer >= 0, default: `0` | Available warehouse inventory |
| `createdAt` | Date | Auto | Current timestamp | Product publication date |
| `updatedAt` | Date | Auto | Current timestamp | Last update date |

---

### 2.4 Order Model (`Order.js`)

Encapsulates placed customer orders, snapshot pricing, delivery address, and tracking status.

#### Schema Definition
```javascript
const mongoose = require('mongoose');

const orderSchema = new mongoose.Schema(
  {
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: [true, 'Order must belong to a user'],
      index: true,
    },
    products: [
      {
        product: {
          type: mongoose.Schema.Types.ObjectId,
          ref: 'Product',
          required: true,
        },
        name: {
          type: String,
          required: true,
        },
        price: {
          type: Number,
          required: true,
          min: [0, 'Price cannot be negative'],
        },
        quantity: {
          type: Number,
          required: true,
          min: [1, 'Quantity must be at least 1'],
        },
      },
    ],
    totalAmount: {
      type: Number,
      required: [true, 'Total amount is required'],
      min: [0, 'Total amount cannot be negative'],
    },
    shippingAddress: {
      name: {
        type: String,
        required: [true, 'Recipient name is required'],
        trim: true,
      },
      phone: {
        type: String,
        required: [true, 'Recipient phone number is required'],
        trim: true,
      },
      address: {
        type: String,
        required: [true, 'Street address is required'],
        trim: true,
      },
      city: {
        type: String,
        required: [true, 'City is required'],
        trim: true,
      },
      pincode: {
        type: String,
        required: [true, 'Pincode is required'],
        trim: true,
      },
    },
    paymentMethod: {
      type: String,
      default: 'COD',
      enum: ['COD'],
    },
    status: {
      type: String,
      enum: {
        values: ['Pending', 'Confirmed', 'Shipped', 'Delivered', 'Cancelled'],
        message: '{VALUE} is not a supported status',
      },
      default: 'Pending',
      index: true,
    },
  },
  {
    timestamps: true,
  }
);

module.exports = mongoose.model('Order', orderSchema);
```

#### Fields Reference
| Field | Type | Required | Constraints / Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `user` | ObjectId | Yes | References `User` | Customer who initiated the purchase |
| `products` | Array of Objects | Yes | Non-empty array | Embedded snapshots of purchased products |
| `products[].product`| ObjectId | Yes | References `Product` | Product foreign key |
| `products[].name` | String | Yes | Preserved product name | Snapshot of product title at time of order |
| `products[].price`| Number | Yes | >= 0 | Authoritative price at purchase time |
| `products[].quantity`| Number | Yes | >= 1 | Units purchased |
| `totalAmount` | Number | Yes | >= 0 | Calculated sum of `(price * quantity)` |
| `shippingAddress`| Sub-document | Yes | All sub-fields required | Complete physical delivery address |
| `paymentMethod` | String | No | Default: `'COD'` (Cash on Delivery) | Payment mechanism |
| `status` | String | No | Enum: `['Pending', 'Confirmed', 'Shipped', 'Delivered', 'Cancelled']`. Default: `'Pending'` | Order fulfillment lifecycle stage |
| `createdAt` | Date | Auto | Current timestamp | Timestamp when order was placed |

---

## 3. Atomic Inventory Deduction Pattern

To prevent race conditions during high-concurrency checkout events:

```javascript
// Step 1: For each item in cart, atomically deduct stock only if current stock >= quantity
for (const item of orderItems) {
  const updatedProduct = await Product.findOneAndUpdate(
    { _id: item.productId, stock: { $gte: item.quantity } },
    { $inc: { stock: -item.quantity } },
    { new: true }
  );

  if (!updatedProduct) {
    throw new Error(`Insufficient stock for product ID: ${item.productId}`);
  }
}
```
This guarantees inventory never drops below zero and orders cannot be double-sold.
