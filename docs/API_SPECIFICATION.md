# 📡 REST API Specification

This document provides complete documentation for all REST endpoints exposed by the **Mini E-Commerce Demo Backend**.

---

## 1. Global Standards & Conventions

- **Base URL**: `http://localhost:5000/api`
- **Content-Type**: `application/json`
- **Authentication**: JWT Bearer token passed in the HTTP Authorization header:
  ```http
  Authorization: Bearer <your_jwt_token_here>
  ```
- **Standard Error Response Format**:
  ```json
  {
    "success": false,
    "message": "Human readable error description",
    "stack": "Stack trace (included in development mode only)"
  }
  ```

---

## 2. Authentication Endpoints (`/api/auth`)

### 2.1 Customer Registration
- **Method**: `POST`
- **Endpoint**: `/api/auth/register`
- **Access**: Public
- **Description**: Registers a new customer and returns user info with a JWT token.

#### Request Body
```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "secretpassword",
  "confirmPassword": "secretpassword"
}
```

#### Validation Rules
- `name`: Required, trimmed.
- `email`: Required, valid email format, must be unique.
- `password`: Required, min 6 characters.
- `confirmPassword`: Required, must exactly match `password`.

#### Response `201 Created`
```json
{
  "success": true,
  "data": {
    "_id": "651a1b2c3d4e5f6a7b8c9d0e",
    "name": "Jane Doe",
    "email": "jane@example.com",
    "role": "customer",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

---

### 2.2 Customer & Admin Login
- **Method**: `POST`
- **Endpoint**: `/api/auth/login`
- **Access**: Public
- **Description**: Authenticates user credentials (both customer and admin) and returns user profile with JWT.

#### Request Body
```json
{
  "email": "jane@example.com",
  "password": "secretpassword"
}
```

#### Response `200 OK`
```json
{
  "success": true,
  "data": {
    "_id": "651a1b2c3d4e5f6a7b8c9d0e",
    "name": "Jane Doe",
    "email": "jane@example.com",
    "role": "customer",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

#### Error Responses
- `400 Bad Request`: Missing email or password.
- `401 Unauthorized`: Invalid credentials.

---

## 3. Categories Endpoints (`/api/categories`)

### 3.1 Get All Categories
- **Method**: `GET`
- **Endpoint**: `/api/categories`
- **Access**: Public
- **Description**: Retrieves all product categories sorted by name.

#### Response `200 OK`
```json
{
  "success": true,
  "count": 3,
  "data": [
    {
      "_id": "651a2c3d4e5f6a7b8c9d0e01",
      "name": "Electronics",
      "description": "Gadgets, smartphones, and accessories",
      "createdAt": "2026-10-02T10:00:00.000Z"
    },
    {
      "_id": "651a2c3d4e5f6a7b8c9d0e02",
      "name": "Fashion",
      "description": "Apparel, clothing, and streetwear",
      "createdAt": "2026-10-02T10:05:00.000Z"
    }
  ]
}
```

---

### 3.2 Create Category (Admin Only)
- **Method**: `POST`
- **Endpoint**: `/api/categories`
- **Access**: Private (Admin)
- **Headers**: `Authorization: Bearer <token>`
- **Description**: Creates a new product category.

#### Request Body
```json
{
  "name": "Shoes",
  "description": "Sneakers, formal footwear, and athletic shoes"
}
```

#### Response `201 Created`
```json
{
  "success": true,
  "message": "Category created successfully",
  "data": {
    "_id": "651a2c3d4e5f6a7b8c9d0e03",
    "name": "Shoes",
    "description": "Sneakers, formal footwear, and athletic shoes",
    "createdAt": "2026-10-02T10:15:00.000Z"
  }
}
```

---

### 3.3 Update Category (Admin Only)
- **Method**: `PUT`
- **Endpoint**: `/api/categories/:id`
- **Access**: Private (Admin)
- **Headers**: `Authorization: Bearer <token>`
- **Description**: Updates an existing category's name or description.

#### Request Body
```json
{
  "name": "Footwear & Shoes",
  "description": "All seasonal athletic and casual footwear"
}
```

#### Response `200 OK`
```json
{
  "success": true,
  "message": "Category updated successfully",
  "data": {
    "_id": "651a2c3d4e5f6a7b8c9d0e03",
    "name": "Footwear & Shoes",
    "description": "All seasonal athletic and casual footwear",
    "updatedAt": "2026-10-02T10:20:00.000Z"
  }
}
```

---

### 3.4 Delete Category (Admin Only)
- **Method**: `DELETE`
- **Endpoint**: `/api/categories/:id`
- **Access**: Private (Admin)
- **Headers**: `Authorization: Bearer <token>`
- **Description**: Permanently deletes a category by its ID.

#### Response `200 OK`
```json
{
  "success": true,
  "message": "Category deleted successfully"
}
```

---

## 4. Products Endpoints (`/api/products`)

### 4.1 Get All Products (Filter & Search)
- **Method**: `GET`
- **Endpoint**: `/api/products`
- **Access**: Public
- **Query Parameters**:
  - `category` *(optional)*: Category ID or name slug to filter by.
  - `search` *(optional)*: Keyword matching product title or description.

#### Example Request
```http
GET /api/products?category=electronics&search=phone
```

#### Response `200 OK`
```json
{
  "success": true,
  "count": 1,
  "data": [
    {
      "_id": "651a3d4e5f6a7b8c9d0e0001",
      "name": "Pro Smartphone X",
      "description": "Flagship 5G smartphone with 128GB storage",
      "price": 699.99,
      "image": "https://images.unsplash.com/photo-1511707171634-5f897ff02aa9",
      "category": {
        "_id": "651a2c3d4e5f6a7b8c9d0e01",
        "name": "Electronics"
      },
      "stock": 14,
      "createdAt": "2026-10-02T10:30:00.000Z"
    }
  ]
}
```

---

### 4.2 Get Single Product by ID
- **Method**: `GET`
- **Endpoint**: `/api/products/:id`
- **Access**: Public
- **Description**: Returns detailed information for a single product.

#### Response `200 OK`
```json
{
  "success": true,
  "data": {
    "_id": "651a3d4e5f6a7b8c9d0e0001",
    "name": "Pro Smartphone X",
    "description": "Flagship 5G smartphone with 128GB storage",
    "price": 699.99,
    "image": "https://images.unsplash.com/photo-1511707171634-5f897ff02aa9",
    "category": {
      "_id": "651a2c3d4e5f6a7b8c9d0e01",
      "name": "Electronics"
    },
    "stock": 14,
    "createdAt": "2026-10-02T10:30:00.000Z"
  }
}
```

---

### 4.3 Create Product (Admin Only)
- **Method**: `POST`
- **Endpoint**: `/api/products`
- **Access**: Private (Admin)
- **Headers**: `Authorization: Bearer <token>`
- **Description**: Creates a new product catalog entry.

#### Request Body
```json
{
  "name": "Pro Smartphone X",
  "description": "Flagship 5G smartphone with 128GB storage",
  "price": 699.99,
  "image": "https://images.unsplash.com/photo-1511707171634-5f897ff02aa9",
  "category": "651a2c3d4e5f6a7b8c9d0e01",
  "stock": 25
}
```

#### Response `201 Created`
```json
{
  "success": true,
  "message": "Product created successfully",
  "data": {
    "_id": "651a3d4e5f6a7b8c9d0e0001",
    "name": "Pro Smartphone X",
    "description": "Flagship 5G smartphone with 128GB storage",
    "price": 699.99,
    "image": "https://images.unsplash.com/photo-1511707171634-5f897ff02aa9",
    "category": "651a2c3d4e5f6a7b8c9d0e01",
    "stock": 25,
    "createdAt": "2026-10-02T10:30:00.000Z"
  }
}
```

---

### 4.4 Update Product (Admin Only)
- **Method**: `PUT`
- **Endpoint**: `/api/products/:id`
- **Access**: Private (Admin)
- **Headers**: `Authorization: Bearer <token>`
- **Description**: Modifies product fields (e.g. price, stock, details).

#### Request Body
```json
{
  "price": 649.99,
  "stock": 30
}
```

#### Response `200 OK`
```json
{
  "success": true,
  "message": "Product updated successfully",
  "data": {
    "_id": "651a3d4e5f6a7b8c9d0e0001",
    "name": "Pro Smartphone X",
    "price": 649.99,
    "stock": 30,
    "updatedAt": "2026-10-02T10:45:00.000Z"
  }
}
```

---

### 4.5 Delete Product (Admin Only)
- **Method**: `DELETE`
- **Endpoint**: `/api/products/:id`
- **Access**: Private (Admin)
- **Headers**: `Authorization: Bearer <token>`
- **Description**: Deletes a product from the database.

#### Response `200 OK`
```json
{
  "success": true,
  "message": "Product deleted successfully"
}
```

---

## 5. Orders Endpoints (`/api/orders` & `/api/admin/orders`)

### 5.1 Place Order (Checkout)
- **Method**: `POST`
- **Endpoint**: `/api/orders`
- **Access**: Private (Customer / Auth Required)
- **Headers**: `Authorization: Bearer <token>`
- **Description**: Places a new Cash on Delivery order. Validates stock, fetches prices securely from MongoDB, atomically decrements stock, and saves order record.

#### Request Body
*(Notice: Unit prices are NOT provided by the client to prevent price tampering)*
```json
{
  "items": [
    {
      "productId": "651a3d4e5f6a7b8c9d0e0001",
      "quantity": 2
    }
  ],
  "shippingAddress": {
    "name": "Jane Doe",
    "phone": "+1 555-0199",
    "address": "456 Market St, Suite 200",
    "city": "Metropolis",
    "pincode": "10001"
  }
}
```

#### Response `201 Created`
```json
{
  "success": true,
  "message": "Order placed successfully",
  "data": {
    "_id": "651a4e5f6a7b8c9d0e000099",
    "user": "651a1b2c3d4e5f6a7b8c9d0e",
    "products": [
      {
        "product": "651a3d4e5f6a7b8c9d0e0001",
        "name": "Pro Smartphone X",
        "price": 649.99,
        "quantity": 2
      }
    ],
    "totalAmount": 1299.98,
    "shippingAddress": {
      "name": "Jane Doe",
      "phone": "+1 555-0199",
      "address": "456 Market St, Suite 200",
      "city": "Metropolis",
      "pincode": "10001"
    },
    "paymentMethod": "COD",
    "status": "Pending",
    "createdAt": "2026-10-02T11:00:00.000Z"
  }
}
```

#### Error Cases
- `400 Bad Request`: Empty items list or missing address fields.
- `400 Bad Request`: `Insufficient stock for product "Pro Smartphone X". Available: 1, Requested: 2`.

---

### 5.2 Get Logged-in Customer's Orders
- **Method**: `GET`
- **Endpoint**: `/api/orders/my-orders`
- **Access**: Private (Customer / Auth Required)
- **Headers**: `Authorization: Bearer <token>`
- **Description**: Returns all orders placed by the requesting authenticated user.

#### Response `200 OK`
```json
{
  "success": true,
  "count": 1,
  "data": [
    {
      "_id": "651a4e5f6a7b8c9d0e000099",
      "products": [
        {
          "product": "651a3d4e5f6a7b8c9d0e0001",
          "name": "Pro Smartphone X",
          "price": 649.99,
          "quantity": 2
        }
      ],
      "totalAmount": 1299.98,
      "status": "Pending",
      "createdAt": "2026-10-02T11:00:00.000Z"
    }
  ]
}
```

---

### 5.3 Get All Orders (Admin Only)
- **Method**: `GET`
- **Endpoint**: `/api/admin/orders`
- **Access**: Private (Admin)
- **Headers**: `Authorization: Bearer <token>`
- **Description**: Retrieves all platform orders, populated with customer user info.

#### Response `200 OK`
```json
{
  "success": true,
  "count": 25,
  "data": [
    {
      "_id": "651a4e5f6a7b8c9d0e000099",
      "user": {
        "_id": "651a1b2c3d4e5f6a7b8c9d0e",
        "name": "Jane Doe",
        "email": "jane@example.com"
      },
      "products": [
        {
          "product": "651a3d4e5f6a7b8c9d0e0001",
          "name": "Pro Smartphone X",
          "price": 649.99,
          "quantity": 2
        }
      ],
      "totalAmount": 1299.98,
      "shippingAddress": {
        "name": "Jane Doe",
        "phone": "+1 555-0199",
        "address": "456 Market St, Suite 200",
        "city": "Metropolis",
        "pincode": "10001"
      },
      "paymentMethod": "COD",
      "status": "Pending",
      "createdAt": "2026-10-02T11:00:00.000Z"
    }
  ]
}
```

---

### 5.4 Update Order Status (Admin Only)
- **Method**: `PATCH`
- **Endpoint**: `/api/admin/orders/:id/status`
- **Access**: Private (Admin)
- **Headers**: `Authorization: Bearer <token>`
- **Description**: Updates order fulfillment progress.

#### Request Body
```json
{
  "status": "Shipped"
}
```

#### Allowed Status Values
`Pending` | `Confirmed` | `Shipped` | `Delivered` | `Cancelled`

#### Response `200 OK`
```json
{
  "success": true,
  "message": "Order status updated to Shipped",
  "data": {
    "_id": "651a4e5f6a7b8c9d0e000099",
    "status": "Shipped",
    "updatedAt": "2026-10-02T11:15:00.000Z"
  }
}
```

---

## 6. HTTP Status Summary Table

| Code | Status | Meaning in this Project |
| :--- | :--- | :--- |
| `200` | OK | Query or update executed successfully. |
| `201` | Created | New resource created (User, Category, Product, Order). |
| `400` | Bad Request | Validation failure, missing fields, or insufficient stock. |
| `401` | Unauthorized | Missing, invalid, or expired JWT token. |
| `403` | Forbidden | Authenticated user is not an administrator. |
| `404` | Not Found | Requested Category, Product, or Order not found. |
| `500` | Internal Server Error | Uncaught server exception (handled by global error handler). |
