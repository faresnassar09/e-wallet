# 💳 E-Wallet

A production-ready, multi-actor digital wallet platform built with **Laravel 11** using a clean **modular architecture**. Supports seamless fund transfers, wallet management, and transaction tracking — with fully isolated authentication for Users, Admins, and Super Admins.

---

## ✨ Features

### 👤 Users
- Register, login & logout with Sanctum tokens
- OTP / Two-Factor Authentication (2FA)
- Personal wallet creation & balance management
- Send & receive money between wallets
- Full transaction history with filtering & pagination
- Profile & KYC management

### 🛡 Admins
- Manage users (view, suspend, activate)
- Monitor all platform transactions
- Flag suspicious activity
- Dashboard with real-time statistics

### 👑 Super Admins
- Full admin management (create, suspend admins)
- System configuration & settings
- Platform-wide reporting & analytics

---

## 🏗 Architecture

This project is structured as a **modular monolith** using [`nwidart/laravel-modules`](https://nwidart.com/laravel-modules). Each business domain lives in its own isolated module with its own routes, controllers, services, models, and migrations.

### Module Dependency Graph

```
AuthModule ──► UserModule
AuthModule ──► AdminModule
WalletModule ──► UserModule
TransactionModule ──► WalletModule ──► UserModule
```

> `UserModule` is the foundation — it has zero dependencies on other modules.

### Key Design Decisions

- **AuthModule** handles authentication for all actors but owns **no models** — it delegates to `UserModule` and `AdminModule`
- A single shared `AuthService` accepts a `guard` parameter, eliminating duplicated login logic across actor types
- Modules communicate only through **Service classes** — never directly between controllers or models

---

## 📦 Modules

| Module | Responsibility |
|---|---|
| **Auth** | Login, logout, registration, OTP, token management for all actors |
| **User** | User model, profile, KYC, preferences |
| **Admin** | Admin & SuperAdmin models, user management, dashboard |
| **Wallet** | Wallet creation, balance, locking/unlocking |
| **Transaction** | Transfers, deposits, withdrawals, history |

---

## 📁 Project Structure

```
Modules/
├── Auth/
│   ├── Controllers/
│   │   ├── User/UserAuthController.php
│   │   ├── Admin/AdminAuthController.php
│   │   └── SuperAdmin/SuperAdminAuthController.php
│   ├── Services/
│   │   ├── AuthService.php
│   │   ├── TokenService.php
│   │   └── OtpService.php
│   ├── Requests/
│   │   ├── User/LoginRequest.php
│   │   └── Admin/LoginRequest.php
│   └── Routes/
│       ├── user.php
│       ├── admin.php
│       └── super-admin.php
│
├── User/
│   ├── Models/User.php
│   ├── Controllers/ProfileController.php
│   ├── Services/UserService.php
│   └── Routes/api.php
│
├── Admin/
│   ├── Models/
│   │   ├── Admin.php
│   │   └── SuperAdmin.php
│   ├── Controllers/
│   │   ├── UserManagementController.php
│   │   └── DashboardController.php
│   ├── Services/AdminService.php
│   └── Routes/api.php
│
├── Wallet/
│   ├── Models/Wallet.php
│   ├── Controllers/WalletController.php
│   ├── Services/WalletService.php
│   └── Routes/api.php
│
└── Transaction/
    ├── Models/Transaction.php
    ├── Controllers/TransactionController.php
    ├── Services/TransactionService.php
    └── Routes/api.php
```

---

## 🔑 Authentication & Guards

All actors authenticate via **Laravel Sanctum** with separate isolated guards:

```php
// config/auth.php
'guards' => [
    'user'        => ['driver' => 'sanctum', 'provider' => 'users'],
    'admin'       => ['driver' => 'sanctum', 'provider' => 'admins'],
    'super_admin' => ['driver' => 'sanctum', 'provider' => 'super_admins'],
],

'providers' => [
    'users'        => ['driver' => 'eloquent', 'model' => \Modules\User\Models\User::class],
    'admins'       => ['driver' => 'eloquent', 'model' => \Modules\Admin\Models\Admin::class],
    'super_admins' => ['driver' => 'eloquent', 'model' => \Modules\Admin\Models\SuperAdmin::class],
],
```

Routes are protected per actor:

```php
// User routes
Route::middleware('auth:user')->group(function () {
    Route::get('/wallet', [WalletController::class, 'index']);
    Route::post('/transactions/transfer', [TransactionController::class, 'transfer']);
});

// Admin routes
Route::middleware('auth:admin')->group(function () {
    Route::get('/users', [UserManagementController::class, 'index']);
});

// Super Admin routes
Route::middleware('auth:super_admin')->group(function () {
    Route::get('/admins', [AdminManagementController::class, 'index']);
});
```

---

## 🌐 API Reference

### Auth

| Method | Endpoint | Actor |
|---|---|---|
| POST | `/api/auth/user/register` | User |
| POST | `/api/auth/user/login` | User |
| POST | `/api/auth/user/logout` | User |
| POST | `/api/auth/admin/login` | Admin |
| POST | `/api/auth/admin/logout` | Admin |
| POST | `/api/auth/super-admin/login` | Super Admin |

### User

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/user/profile` | Get authenticated user profile |
| PUT | `/api/user/profile` | Update profile |

### Wallet

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/wallet` | Get wallet balance |
| POST | `/api/wallet/lock` | Lock wallet |
| POST | `/api/wallet/unlock` | Unlock wallet |

### Transactions

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/transactions/transfer` | Transfer funds |
| POST | `/api/transactions/deposit` | Deposit funds |
| GET | `/api/transactions` | Transaction history |
| GET | `/api/transactions/{id}` | Single transaction details |

### Admin

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/admin/users` | List all users |
| PUT | `/api/admin/users/{id}/suspend` | Suspend a user |
| PUT | `/api/admin/users/{id}/activate` | Activate a user |
| GET | `/api/admin/transactions` | All platform transactions |

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| Framework | Laravel 11 |
| Modular System | nwidart/laravel-modules |
| Authentication | Laravel Sanctum |
| Database | MySQL |
| Cache | Redis |
| Queue | Laravel Queues (Redis driver) |
| Testing | PestPHP |

---

## ⚙️ Installation

### Requirements

- PHP >= 8.2
- Composer
- MySQL
- Redis

### Steps

```bash
# 1. Clone the repository
git clone https://github.com/your-org/e-wallet.git
cd e-wallet

# 2. Install dependencies
composer install

# 3. Set up environment
cp .env.example .env
php artisan key:generate

# 4. Configure your database and Redis in .env, then run:
php artisan migrate --seed

# 5. Start the server
php artisan serve
```

### Environment Variables

```env
APP_NAME="E-Wallet"
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_DATABASE=ewallet
DB_USERNAME=root
DB_PASSWORD=

REDIS_HOST=127.0.0.1
REDIS_PORT=6379

SANCTUM_STATEFUL_DOMAINS=localhost:3000
```

---

## 🧪 Testing

Tests are written with **PestPHP** and organized per module:

```bash
# Run all tests
php artisan test

# Run a specific module's tests
php artisan test --filter=AuthTest
php artisan test --filter=WalletTest
```

---

## 👥 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Follow the module structure — new features go inside the appropriate module
4. Modules must communicate only through Service classes
5. Cover new logic with PestPHP tests
6. Submit a pull request with a clear description

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).
