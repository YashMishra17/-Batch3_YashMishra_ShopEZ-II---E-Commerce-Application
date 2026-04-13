# 🛒 ShopEZ – E-Commerce Backend API

**A fully functional, production-ready e-commerce backend API built with ASP.NET Core Web API, Entity Framework Core, and JWT Authentication.**

---

## 📋 Table of Contents

- [📖 Overview](#-overview)
- [✨ Features](#-features)
- [🏗️ Architecture](#️-architecture)
- [🗂️ Project Structure](#️-project-structure)
- [🗄️ Database Design](#️-database-design)
- [🔐 Authentication & Authorization](#-authentication--authorization)
- [📡 API Endpoints](#-api-endpoints)
- [⚙️ Prerequisites](#️-prerequisites)
- [🚀 Getting Started](#-getting-started)
- [🧪 Testing with Postman](#-testing-with-postman)
- [📦 NuGet Packages](#-nuget-packages)
- [🔧 Configuration](#-configuration)
- [🛡️ Security](#️-security)
- [📊 Role Access Matrix](#-role-access-matrix)

---

## 📖 Overview

ShopEZ is a **Phase 2 backend implementation** of a full-stack e-commerce application. This phase transforms the static frontend prototype (Phase 1) into a **data-driven, database-powered system** by implementing:

- 🔄 RESTful APIs for product and order management
- 🔐 JWT-based authentication with role-based authorization
- 💾 SQL Server database with Entity Framework Core
- 🔁 Transactional order processing with stock management
- ✅ Full input validation and structured exception handling

---

## ✨ Features

### 🛍️ Product Management
- ✅ Get all products
- ✅ Get product by ID
- ✅ Create new product *(Admin only)*
- ✅ Update existing product *(Admin only)*
- ✅ Delete product *(Admin only)*

### 📦 Order Processing
- ✅ Create order from cart items with full validation
- ✅ Automatic stock deduction on order placement
- ✅ Get all orders *(Admin only)*
- ✅ Get order by ID

### 🔐 Authentication
- ✅ User registration with BCrypt password hashing
- ✅ JWT token generation on login
- ✅ Role-based access control (Admin / Customer)
- ✅ Token expiry and validation

### 🏛️ Architecture
- ✅ Clean layered architecture: Controller → Service → Repository → DbContext
- ✅ Full async/await on every database operation
- ✅ DTOs — no direct entity exposure
- ✅ Database transactions with rollback on failure
- ✅ Structured exception handling with HTTP status codes
- ✅ Seed data for immediate testing

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                    Client / Postman                  │
└──────────────────────┬──────────────────────────────┘
                       │  HTTP Request + JWT Token
┌──────────────────────▼──────────────────────────────┐
│               Controllers Layer                      │
│   AuthController │ ProductsController │ OrdersController │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                Services Layer                        │
│      AuthService │ ProductService │ OrderService     │
│         (Business Logic + Validation)                │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│              Repositories Layer                      │
│        ProductRepository │ OrderRepository           │
│           (Data Access Logic)                        │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│            ApplicationDbContext (EF Core)            │
│                  SQL Server Database                 │
└─────────────────────────────────────────────────────┘
```

---

## 🗂️ Project Structure

```
ShopEZ.API/
│
├── 📁 Controllers/
│   ├── 🔐 AuthController.cs           # Register & Login
│   ├── 🛍️  ProductsController.cs       # Product CRUD
│   └── 📦 OrdersController.cs         # Order management
│
├── 📁 Models/
│   ├── 👤 User.cs
│   ├── 🛍️  Product.cs
│   ├── 📦 Order.cs
│   └── 📋 OrderItem.cs
│
├── 📁 DTOs/
│   ├── 🔐 RegisterDTO.cs
│   ├── 🔐 LoginDTO.cs
│   ├── 🔐 AuthResponseDTO.cs
│   ├── 🛍️  ProductDTO.cs
│   ├── 🛍️  CreateProductDTO.cs
│   ├── 🛍️  UpdateProductDTO.cs
│   ├── 📦 CreateOrderDTO.cs
│   ├── 📦 OrderDTO.cs
│   ├── 📋 OrderItemDTO.cs
│   └── 🛒 CartItemDTO.cs
│
├── 📁 Data/
│   └── 🗄️  ApplicationDbContext.cs    # EF Core DbContext + Seed Data
│
├── 📁 Repositories/
│   ├── 📁 Interfaces/
│   │   ├── IProductRepository.cs
│   │   └── IOrderRepository.cs
│   ├── ProductRepository.cs
│   └── OrderRepository.cs
│
├── 📁 Services/
│   ├── 📁 Interfaces/
│   │   ├── IAuthService.cs
│   │   ├── IProductService.cs
│   │   └── IOrderService.cs
│   ├── AuthService.cs
│   ├── ProductService.cs
│   └── OrderService.cs
│
├── 📁 Exceptions/
│   └── ⚠️  AppException.cs            # Custom exception with HTTP status
│
├── ⚙️  Program.cs                     # App configuration & DI
├── 📝 appsettings.json               # Connection string & JWT config
└── 📦 ShopEZ.API.csproj              # Project & NuGet packages
```

---

## 🗄️ Database Design

### 📊 Entity Relationship Diagram

```
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│    Users     │         │    Orders    │         │  OrderItems  │
├──────────────┤         ├──────────────┤         ├──────────────┤
│ UserId  (PK) │◄──┐     │ OrderId (PK) │◄──┐     │OrderItemId(PK│
│ Name         │   │     │ UserId  (FK) │   │     │ OrderId  (FK)│
│ Email        │   └─────│ OrderDate    │   └─────│ ProductId(FK)│
│ Password     │  1:Many │ TotalAmount  │  1:Many │ Quantity     │
│ Role         │         └──────────────┘         │ Price        │
└──────────────┘                                  └──────┬───────┘
                                                         │
                                                    Many:1│
                                                  ┌──────▼───────┐
                                                  │   Products   │
                                                  ├──────────────┤
                                                  │ProductId (PK)│
                                                  │ Name         │
                                                  │ Description  │
                                                  │ Price        │
                                                  │ ImageUrl     │
                                                  │ Stock        │
                                                  └──────────────┘
```

### 🔗 Relationships
| Relationship | Type | Delete Behavior |
|---|---|---|
| User → Orders | One to Many | Restrict |
| Order → OrderItems | One to Many | Cascade |
| Product → OrderItems | One to Many | Restrict |

### 🌱 Seed Data
The database is pre-loaded with:
- 👤 **2 Users** — Alice (Admin) and Bob (Customer)
- 🛍️ **3 Products** — Wireless Mouse, Mechanical Keyboard, USB-C Hub

---

## 🔐 Authentication & Authorization

ShopEZ uses **JWT Bearer Token** authentication with **BCrypt** password hashing.

### 🔑 How it works

```
1. 📝 Register  →  POST /api/auth/register  →  Returns JWT Token
2. 🔑 Login     →  POST /api/auth/login     →  Returns JWT Token
3. 🔒 Use Token →  Add header: Authorization: Bearer <token>
4. ✅ Access    →  Protected endpoints now accessible based on role
```

### 🎫 JWT Token Contents
Each token contains these **claims**:
```
sub         →  UserId
email       →  User email
jti         →  Unique token ID
name        →  User full name
role        →  Admin / Customer
```

### ⏰ Token Expiry
- Default: **24 hours**
- Configurable via `JwtSettings:ExpiryHours` in `appsettings.json`
- No clock skew — tokens expire exactly on time

---

## 📡 API Endpoints

### 🔐 Auth Endpoints
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| `POST` | `/api/auth/register` | Register new user | ❌ No |
| `POST` | `/api/auth/login` | Login & get token | ❌ No |

### 🛍️ Product Endpoints
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| `GET` | `/api/products` | Get all products | ❌ No |
| `GET` | `/api/products/{id}` | Get product by ID | ❌ No |
| `POST` | `/api/products` | Create product | ✅ Admin |
| `PUT` | `/api/products/{id}` | Update product | ✅ Admin |
| `DELETE` | `/api/products/{id}` | Delete product | ✅ Admin |

### 📦 Order Endpoints
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| `POST` | `/api/orders` | Create order | ✅ Admin / Customer |
| `GET` | `/api/orders` | Get all orders | ✅ Admin only |
| `GET` | `/api/orders/{id}` | Get order by ID | ✅ Admin / Customer |

---

## ⚙️ Prerequisites

Before running this project, make sure you have:

| Tool | Version | Download |
|------|---------|----------|
| 🖥️ Visual Studio | 2022 (17.8+) | [Download](https://visualstudio.microsoft.com/) |
| 🔷 .NET SDK | 10.0 | [Download](https://dotnet.microsoft.com/download) |
| 🗄️ SQL Server | LocalDB | Bundled with Visual Studio |
| 📮 Postman | Latest | [Download](https://www.postman.com/downloads/) |

---

## 🚀 Getting Started

### Step 1 — Clone the repository

```bash
git clone https://github.com/yourusername/ShopEZ-API.git
cd ShopEZ-API
```

### Step 2 — Open in Visual Studio

```
File → Open → Project/Solution → ShopEZ.API.sln
```

### Step 3 — Restore NuGet Packages

```
Right-click Solution → Restore NuGet Packages
```

Or via Package Manager Console:

```powershell
dotnet restore
```

### Step 4 — Configure the database connection

Open `appsettings.json` and update the connection string if needed:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=ShopEZDb;Trusted_Connection=True;TrustServerCertificate=True"
  }
}
```

> 💡 For **SQL Server Express** use:
> `"Server=.\\SQLEXPRESS;Database=ShopEZDb;Trusted_Connection=True;TrustServerCertificate=True"`

### Step 5 — Run database migrations

Open **Package Manager Console** (Tools → NuGet → Package Manager Console):

```powershell
Add-Migration InitialCreate
Update-Database
```

> ✅ This creates the `ShopEZDb` database with all 4 tables and seed data automatically.

### Step 6 — Run the application

Press **F5** or **Ctrl+F5** in Visual Studio.

The Swagger UI opens automatically at:
```
https://localhost:7110/
```

---

## 🧪 Testing with Postman

### 🔐 Step 1 — Register a new user

```http
POST https://localhost:7110/api/auth/register
Content-Type: application/json

{
  "Name": "John Doe",
  "Email": "john@example.com",
  "Password": "password123",
  "Role": "Customer"
}
```

✅ Response `201`:
```json
{
  "success": true,
  "data": {
    "UserId": 3,
    "Name": "John Doe",
    "Email": "john@example.com",
    "Role": "Customer",
    "Token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "ExpiresAt": "2025-08-16T10:30:00Z"
  }
}
```

### 🔑 Step 2 — Login as Admin

```http
POST https://localhost:7110/api/auth/login
Content-Type: application/json

{
  "Email": "alice@shopez.com",
  "Password": "hashed_password_alice"
}
```

> 📋 Copy the `Token` value from the response — you need it for all protected requests.

### 🔒 Step 3 — Add token to all requests

In Postman → Headers tab:
```
Key:   Authorization
Value: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### 🛍️ Step 4 — Create a product (Admin token)

```http
POST https://localhost:7110/api/products
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "Name": "Gaming Headset",
  "Description": "7.1 surround sound USB headset",
  "Price": 89.99,
  "ImageUrl": "https://via.placeholder.com/300",
  "Stock": 30
}
```

### 📦 Step 5 — Place an order (Customer token)

```http
POST https://localhost:7110/api/orders
Authorization: Bearer <customer_token>
Content-Type: application/json

{
  "UserId": 3,
  "CartItems": [
    { "ProductId": 1, "Quantity": 2 },
    { "ProductId": 2, "Quantity": 1 }
  ]
}
```

✅ Response `201`:
```json
{
  "success": true,
  "data": {
    "OrderId": 1,
    "UserId": 3,
    "UserName": "John Doe",
    "OrderDate": "2025-08-15T10:30:00Z",
    "TotalAmount": 139.97,
    "OrderItems": [
      {
        "OrderItemId": 1,
        "ProductId": 1,
        "ProductName": "Wireless Mouse",
        "Quantity": 2,
        "Price": 29.99,
        "Subtotal": 59.98
      },
      {
        "OrderItemId": 2,
        "ProductId": 2,
        "ProductName": "Mechanical Keyboard",
        "Quantity": 1,
        "Price": 79.99,
        "Subtotal": 79.99
      }
    ]
  }
}
```

---

## 📦 NuGet Packages

| Package | Version | Purpose |
|---------|---------|---------|
| `BCrypt.Net-Next` | 4.1.0 | 🔒 Password hashing |
| `Microsoft.AspNetCore.Authentication.JwtBearer` | 10.0.3 | 🎫 JWT middleware |
| `Microsoft.AspNetCore.OpenApi` | 10.0.3 | 📄 OpenAPI support |
| `Microsoft.EntityFrameworkCore.SqlServer` | 10.0.3 | 🗄️ SQL Server provider |
| `Microsoft.EntityFrameworkCore.Tools` | 10.0.3 | 🔧 EF migrations |
| `Microsoft.EntityFrameworkCore.Design` | 10.0.3 | 🎨 EF design tools |
| `Microsoft.IdentityModel.Tokens` | 8.17.0 | 🔑 Token validation |
| `Swashbuckle.AspNetCore` | 10.1.0 | 📚 Swagger UI |
| `System.IdentityModel.Tokens.Jwt` | 8.17.0 | 🎫 JWT generation |

---

## 🔧 Configuration

### `appsettings.json`

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=ShopEZDb;Trusted_Connection=True;MultipleActiveResultSets=true;TrustServerCertificate=True"
  },
  "JwtSettings": {
    "SecretKey": "ShopEZ@SuperSecret#Key!2025$Secure&Long@Enough32Chars",
    "Issuer": "ShopEZ.API",
    "Audience": "ShopEZ.Client",
    "ExpiryHours": "24"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

> ⚠️ **Production Warning:** Move `SecretKey` to environment variables or Azure Key Vault before deploying to production. Never commit real secrets to source control.

---

## 🛡️ Security

| Feature | Implementation |
|---------|---------------|
| 🔒 Password Storage | BCrypt with work factor 12 — never plain text |
| 🎫 Token Signing | HMAC-SHA256 with 32+ character secret key |
| ⏰ Token Expiry | 24 hours with zero clock skew tolerance |
| 🚫 User Enumeration | Generic "Invalid email or password" for both wrong email and wrong password |
| 📧 Duplicate Email | 409 Conflict returned on duplicate registration |
| 🔐 Role Enforcement | `[Authorize(Roles = "Admin")]` and `[Authorize(Roles = "Admin,Customer")]` |
| 💾 Transaction Safety | `BeginTransactionAsync` / `CommitAsync` / `RollbackAsync` on order creation |

---

## 📊 Role Access Matrix

| Endpoint | No Token | 👤 Customer | 👑 Admin |
|----------|----------|-------------|---------|
| `POST /api/auth/register` | ✅ | ✅ | ✅ |
| `POST /api/auth/login` | ✅ | ✅ | ✅ |
| `GET /api/products` | ✅ | ✅ | ✅ |
| `GET /api/products/{id}` | ✅ | ✅ | ✅ |
| `POST /api/products` | ❌ 401 | ❌ 403 | ✅ |
| `PUT /api/products/{id}` | ❌ 401 | ❌ 403 | ✅ |
| `DELETE /api/products/{id}` | ❌ 401 | ❌ 403 | ✅ |
| `POST /api/orders` | ❌ 401 | ✅ | ✅ |
| `GET /api/orders` | ❌ 401 | ❌ 403 | ✅ |
| `GET /api/orders/{id}` | ❌ 401 | ✅ | ✅ |

---

## 🔄 Order Processing Flow

```
📥 POST /api/orders
        │
        ▼
✅ Validate JWT Token
        │
        ▼
✅ Validate cart not empty
        │
        ▼
✅ Validate UserId exists in DB
        │
        ▼
✅ Validate each item Quantity > 0
        │
        ▼
✅ Load all products from DB (single query)
        │
        ▼
✅ Validate each product exists
        │
        ▼
✅ Validate stock availability
        │
        ▼
✅ Build OrderItems list
        │
        ▼
✅ Calculate TotalAmount using LINQ
        │
        ▼
🔄 BEGIN TRANSACTION
        │
        ├── Deduct stock for each product
        ├── Save Order + OrderItems
        │
        ├── ✅ SUCCESS → COMMIT
        └── ❌ FAILURE → ROLLBACK
        │
        ▼
📤 Return 201 Created with full order details
```

---

## 👨‍💻 Technology Stack

| Layer | Technology |
|-------|-----------|
| 🌐 Backend Framework | ASP.NET Core Web API (.NET 10) |
| 🗄️ ORM | Entity Framework Core 10 |
| 💾 Database | SQL Server / LocalDB |
| 🔐 Authentication | JWT Bearer + BCrypt |
| 📚 Documentation | Swagger / OpenAPI |
| 🏗️ Architecture | Layered (Controller → Service → Repository) |
| ⚡ Programming Model | Async / Await throughout |

---

## 📄 License

This project is built as a part  of the ShopEZ E-Commerce Application — Phase 2.

---

Built with  using **ASP.NET Core** | **Entity Framework Core** | **SQL Server** | **JWT**
