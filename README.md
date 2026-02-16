# 🛍️ DashboardQu - Sistem Manajemen Toko Baju

![Laravel](https://img.shields.io/badge/Laravel-11.x-FF2D20?style=for-the-badge&logo=laravel&logoColor=white)
![PHP](https://img.shields.io/badge/PHP-8.2+-777BB4?style=for-the-badge&logo=php&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-3.x-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white)

Dashboard manajemen toko baju lengkap dengan fitur inventori, penjualan, laporan, dan manajemen pelanggan.

---

## 📑 Daftar Isi

- [Fitur Utama](#-fitur-utama)
- [Arsitektur Sistem](#-arsitektur-sistem)
- [Struktur Folder](#-struktur-folder)
- [State Management](#-state-management)
- [API Design & Best Practices](#-api-design--best-practices)
- [Database Design](#-database-design)
- [Instalasi](#-instalasi)
- [Penggunaan](#-penggunaan)
- [Testing](#-testing)

---

## ✨ Fitur Utama

### 📦 Manajemen Produk

- CRUD produk (baju, celana, aksesoris)
- Kategori dan sub-kategori
- Varian produk (ukuran, warna)
- Upload multiple gambar
- Manajemen stok real-time

### 💰 Manajemen Penjualan

- Point of Sale (POS)
- Keranjang belanja
- Multiple payment method
- Diskon dan promo
- Invoice otomatis

### 👥 Manajemen Pelanggan

- Database pelanggan
- Riwayat pembelian
- Sistem membership/loyalty
- Notifikasi (email/SMS)

### 📊 Laporan & Analytics

- Dashboard overview
- Laporan penjualan harian/bulanan
- Best selling products
- Revenue analytics
- Export PDF/Excel

### 👤 Manajemen User

- Role-based access (Admin, Kasir, Supervisor)
- Audit log aktivitas
- Multi-tenant support

---

## 🏗️ Arsitektur Sistem

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                            │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Browser    │  │  Mobile App  │  │   POS App    │          │
│  │  (Blade +    │  │  (Optional)  │  │  (Optional)  │          │
│  │  Alpine.js)  │  │              │  │              │          │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘          │
└─────────┼─────────────────┼─────────────────┼──────────────────┘
          │                 │                 │
          ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                       APPLICATION LAYER                         │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Laravel 11.x                          │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │   │
│  │  │   Routes    │  │ Controllers │  │ Middleware  │      │   │
│  │  │  (Web/API)  │  │             │  │             │      │   │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘      │   │
│  │         │                │                │              │   │
│  │         ▼                ▼                ▼              │   │
│  │  ┌─────────────────────────────────────────────────┐    │   │
│  │  │              Form Requests (Validation)          │    │   │
│  │  └─────────────────────────────────────────────────┘    │   │
│  │                          │                               │   │
│  │                          ▼                               │   │
│  │  ┌─────────────────────────────────────────────────┐    │   │
│  │  │               Service Layer                      │    │   │
│  │  │  (Business Logic & State Management)             │    │   │
│  │  └─────────────────────────────────────────────────┘    │   │
│  │                          │                               │   │
│  │                          ▼                               │   │
│  │  ┌─────────────────────────────────────────────────┐    │   │
│  │  │           Repository Layer (Optional)            │    │   │
│  │  └─────────────────────────────────────────────────┘    │   │
│  │                          │                               │   │
│  │                          ▼                               │   │
│  │  ┌─────────────────────────────────────────────────┐    │   │
│  │  │              Eloquent Models                     │    │   │
│  │  └─────────────────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────────┐
│                         DATA LAYER                              │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │    MySQL     │  │    Redis     │  │   Storage    │          │
│  │  (Database)  │  │   (Cache)    │  │   (Files)    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

### Request Flow

```
Request → Route → Middleware → Controller → FormRequest → Service → Repository → Model → Database
                                                                                            │
Response ← Resource/View ← Controller ← Service ← Repository ← Model ←─────────────────────┘
```

---

## 🧠 Repository Pattern & Model - Penjelasan Mendalam

### 📚 Apa itu Repository?

**Repository** adalah pola desain yang **memisahkan logika akses data dari business logic**. Bayangkan Repository sebagai "perantara" antara aplikasi dengan database.

#### Tanpa Repository (Langsung ke Database)

```php
// BAD - Controller berkomunikasi langsung dengan database
$product = Product::where('id', $id)->first();
$product->price = 100000;
$product->save();
```

#### Dengan Repository (Best Practice)

```php
// GOOD - Controller berkomunikasi dengan Repository
$repository = new ProductRepository();
$product = $repository->updatePrice($id, 100000);
```

#### Fungsi Repository:

1. **Abstraksi Database** - Jika perlu ganti database, hanya edit Repository
2. **Reusable Logic** - Query kompleks disimpan di satu tempat
3. **Testing** - Mudah membuat mock untuk testing
4. **Clean Code** - Controller lebih bersih dan fokus pada bisnis logic

#### Contoh Repository:

```php
// app/Repositories/ProductRepository.php
<?php

namespace App\Repositories;

use App\Models\Product;

class ProductRepository
{
    /**
     * Ambil produk dengan filter harga
     */
    public function getByPriceRange(float $min, float $max)
    {
        return Product::where('price', '>=', $min)
            ->where('price', '<=', $max)
            ->active()
            ->get();
    }

    /**
     * Ambil top selling products
     */
    public function getTopSelling($limit = 10)
    {
        return Product::withCount('orderItems')
            ->orderByDesc('order_items_count')
            ->limit($limit)
            ->get();
    }

    /**
     * Update harga produk
     */
    public function updatePrice(int $id, float $newPrice): Product
    {
        $product = Product::findOrFail($id);
        $product->price = $newPrice;
        $product->save();

        return $product;
    }
}

// Penggunaan di Controller
class ProductController extends Controller
{
    public function __construct(private ProductRepository $repository) {}

    public function getAffordable()
    {
        $products = $this->repository->getByPriceRange(0, 500000);
        return view('products.index', compact('products'));
    }
}
```

---

### 🗂️ Apa itu Model?

**Model** adalah representasi dari **tabel database dalam bentuk class PHP**. Setiap Model mewakili satu tabel di database.

#### Fungsi Model:

| Aspek             | Penjelasan                                                           |
| ----------------- | -------------------------------------------------------------------- |
| **Data Mapper**   | Membuatkan data dari database jadi object PHP                        |
| **Relationships** | Mendefinisikan hubungan antar tabel (One-to-Many, Many-to-Many, dll) |
| **Validation**    | Mengatur casting type data dan mutator                               |
| **Scopes**        | Membuat query reusable dengan method helpers                         |
| **Attributes**    | Menambah computed properties yang tidak ada di database              |

#### Contoh Model:

```php
// app/Models/Product.php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Product extends Model
{
    // ==================
    // FILLABLE - Field yang boleh di-assign massal
    // ==================
    protected $fillable = ['name', 'price', 'category_id', 'stock'];

    // ==================
    // CASTS - Konversi tipe data otomatis
    // ==================
    protected $casts = [
        'price' => 'decimal:2',    // Selalu decimal 2 angka
        'stock' => 'integer',       // Selalu integer
        'created_at' => 'datetime', // Selalu datetime
    ];

    // ==================
    // RELATIONSHIPS - Hubungan dengan model lain
    // ==================

    // One-to-Many: Satu kategori banyak produk
    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }

    // One-to-Many: Satu produk banyak varian
    public function variants(): HasMany
    {
        return $this->hasMany(ProductVariant::class);
    }

    // ==================
    // SCOPES - Query helper yang reusable
    // ==================

    // Hanya ambil produk yang aktif
    public function scopeActive($query)
    {
        return $query->where('is_active', true);
    }

    // Ambil produk dengan harga antara min-max
    public function scopePriceRange($query, $min, $max)
    {
        return $query->whereBetween('price', [$min, $max]);
    }

    // ==================
    // ACCESSORS - Computed properties (read-only)
    // ==================

    // Format harga jadi "Rp 100.000"
    public function getFormattedPriceAttribute()
    {
        return 'Rp ' . number_format($this->price, 0, ',', '.');
    }

    // ==================
    // MUTATORS - Transform data sebelum disimpan
    // ==================

    // Ubah ke huruf besar sebelum disimpan
    public function setNameAttribute($value)
    {
        $this->attributes['name'] = strtoupper($value);
    }
}

// Contoh penggunaan:
$product = Product::active()              // scope: hanya active
    ->priceRange(10000, 500000)           // scope: range harga
    ->first();

echo $product->formatted_price;           // Accessor: "Rp 150.000"
```

#### Perbedaan Model vs Repository:

| Aspek              | Model                          | Repository                           |
| ------------------ | ------------------------------ | ------------------------------------ |
| **Tanggung Jawab** | Represent tabel database       | Akses data / Query kompleks          |
| **Lokasi**         | `app/Models/`                  | `app/Repositories/`                  |
| **Isi**            | Relationships, Scopes, Casting | Complex queries, Business logic      |
| **Contoh**         | `Product::find(1)`             | `productRepository->getTopSelling()` |
| **Diperlukan?**    | **YA, WAJIB**                  | **Opsional tapi best practice**      |

---

## 📁 Struktur Folder - Penjelasan Setiap File

```
DashboardQu_laravel/
├── app/                                        # Root aplikasi Laravel
│   ├── Console/                                # Artisan commands (task otomatis)
│   │   └── Commands/
│   │       ├── GenerateMonthlyReport.php      # Command: membuat laporan bulanan
│   │       │                                   # Gunakan: php artisan generate:monthly-report
│   │       └── CleanupExpiredCarts.php        # Command: hapus cart yang sudah kadaluarsa
│   │                                           # Gunakan: php artisan cleanup:expired-carts
│   │
│   ├── Enums/                                  # PHP 8.1+ Type-safe constants
│   │   ├── OrderStatus.php                    # Enum: PENDING, COMPLETED, CANCELLED
│   │   │                                       # Gunakan: OrderStatus::PENDING (bukan string)
│   │   ├── PaymentMethod.php                  # Enum: CASH, CARD, BANK_TRANSFER
│   │   ├── ProductSize.php                    # Enum: XS, S, M, L, XL, XXL
│   │   └── UserRole.php                       # Enum: ADMIN, SUPERVISOR, KASIR
│   │
│   ├── Events/                                 # Trigger-able events (Observer pattern)
│   │   ├── OrderPlaced.php                    # Event: dipicu saat order dibuat
│   │   │                                       # Hubung dengan listener untuk action
│   │   ├── ProductStockLow.php                # Event: dipicu saat stok produk rendah
│   │   └── PaymentReceived.php                # Event: dipicu saat pembayaran diterima
│   │
│   ├── Exceptions/                             # Custom exception classes
│   │   ├── Handler.php                        # Main exception handler (error page)
│   │   ├── InsufficientStockException.php     # Exception: stok tidak cukup
│   │   │                                       # throw new InsufficientStockException()
│   │   └── PaymentFailedException.php         # Exception: pembayaran gagal
│   │
│   ├── Http/                                   # HTTP requests handling
│   │   ├── Controllers/
│   │   │   ├── Api/
│   │   │   │   └── V1/                         # API versi 1 (untuk backward compatibility)
│   │   │   │       ├── AuthController.php     # API: login, register, logout
│   │   │   │       ├── ProductController.php  # API: list/create/edit/delete produk
│   │   │   │       ├── CategoryController.php # API: list/create/edit category
│   │   │   │       ├── OrderController.php    # API: list/create order
│   │   │   │       ├── CustomerController.php # API: list/create customer
│   │   │   │       └── ReportController.php   # API: dashboard stats, laporan
│   │   │   │
│   │   │   ├── Dashboard/                      # Web controllers (Blade views)
│   │   │   │   ├── DashboardController.php    # Page: dashboard home, welcome screen
│   │   │   │   ├── ProductController.php      # Page: list produk, form create/edit
│   │   │   │   ├── CategoryController.php     # Page: list kategori, form create/edit
│   │   │   │   ├── OrderController.php        # Page: list order, detail order
│   │   │   │   ├── CustomerController.php     # Page: list customer, detail customer
│   │   │   │   ├── ReportController.php       # Page: laporan penjualan, inventory
│   │   │   │   ├── SettingController.php      # Page: pengaturan toko, konfigurasi
│   │   │   │   └── UserController.php         # Page: list user, form create/edit
│   │   │   │
│   │   │   ├── Auth/
│   │   │   │   ├── LoginController.php        # Handle: login page & process
│   │   │   │   ├── RegisterController.php     # Handle: register page & process
│   │   │   │   └── PasswordController.php     # Handle: forgot password, reset
│   │   │   │
│   │   │   └── POS/
│   │   │       └── POSController.php          # Point of Sale: kasir interface
│   │   │
│   │   ├── Middleware/                         # Filter HTTP request (sebelum ke controller)
│   │   │   ├── CheckRole.php                  # Filter: hanya admin/supervisor bisa akses
│   │   │   │                                   # Gunakan: Route::middleware('role:admin')
│   │   │   ├── EnsureStoreIsActive.php        # Filter: pastikan toko aktif
│   │   │   └── ApiRateLimiter.php             # Filter: batasi request API (anti abuse)
│   │   │
│   │   ├── Requests/                           # Form validation (request validation)
│   │   │   ├── Product/
│   │   │   │   ├── StoreProductRequest.php    # Validasi: create product (required rules)
│   │   │   │   │                               # Contoh: name required, sku unique
│   │   │   │   └── UpdateProductRequest.php   # Validasi: update product
│   │   │   ├── Order/
│   │   │   │   ├── StoreOrderRequest.php      # Validasi: create order
│   │   │   │   └── UpdateOrderRequest.php     # Validasi: update order
│   │   │   ├── Customer/
│   │   │   │   └── StoreCustomerRequest.php   # Validasi: create customer
│   │   │   └── Auth/
│   │   │       ├── LoginRequest.php           # Validasi: login (email, password)
│   │   │       └── RegisterRequest.php        # Validasi: register (email unique, password)
│   │   │
│   │   └── Resources/                          # API response formatter (JSON structure)
│   │       ├── ProductResource.php            # Format: single product response
│   │       │                                   # Return: {id, name, price, stock}
│   │       ├── ProductCollection.php          # Format: collection products response
│   │       │                                   # Return: [product, product, ...] dengan meta
│   │       ├── OrderResource.php              # Format: single order response
│   │       ├── CustomerResource.php           # Format: single customer response
│   │       └── DashboardResource.php          # Format: dashboard stats response
│   │
│   ├── Listeners/                              # Event listeners (react to events)
│   │   ├── SendOrderConfirmation.php          # Listen: OrderPlaced → kirim email
│   │   ├── UpdateInventoryStock.php           # Listen: OrderPlaced → kurangi stok
│   │   └── NotifyLowStock.php                 # Listen: ProductStockLow → notif admin
│   │
│   ├── Models/                                 # Database models (wajib ada!)
│   │   ├── User.php                           # Model: untuk tabel users
│   │   │                                       # Isi: relationships, scopes, accessors
│   │   ├── Product.php                        # Model: untuk tabel products
│   │   ├── ProductVariant.php                 # Model: untuk tabel product_variants
│   │   ├── ProductImage.php                   # Model: untuk tabel product_images
│   │   ├── Category.php                       # Model: untuk tabel categories
│   │   ├── Order.php                          # Model: untuk tabel orders
│   │   ├── OrderItem.php                      # Model: untuk tabel order_items
│   │   ├── Customer.php                       # Model: untuk tabel customers
│   │   ├── Payment.php                        # Model: untuk tabel payments
│   │   ├── Cart.php                           # Model: untuk tabel carts
│   │   ├── CartItem.php                       # Model: untuk tabel cart_items
│   │   ├── Discount.php                       # Model: untuk tabel discounts
│   │   ├── Setting.php                        # Model: untuk tabel settings (config toko)
│   │   └── AuditLog.php                       # Model: untuk tabel audit_logs (history)
│   │
│   ├── Notifications/                          # Notification classes (email/SMS)
│   │   ├── OrderConfirmationNotification.php  # Notify: order confirmation email
│   │   │                                       # Gunakan: $user->notify(new OrderConfirmation())
│   │   ├── LowStockNotification.php           # Notify: low stock alert email
│   │   └── PaymentReceivedNotification.php    # Notify: payment confirmation email
│   │
│   ├── Observers/                              # Model observers (auto trigger saat model changed)
│   │   ├── ProductObserver.php                # Observer: saat Product created/updated/deleted
│   │   │                                       # Contoh: update slug otomatis dari name
│   │   ├── OrderObserver.php                  # Observer: saat Order created/updated
│   │   │                                       # Contoh: generate order number otomatis
│   │   └── UserObserver.php                   # Observer: saat User created/updated
│   │
│   ├── Policies/                               # Authorization policies (siapa boleh akses)
│   │   ├── ProductPolicy.php                  # Policy: siapa boleh view/create/edit/delete produk
│   │   │                                       # Gunakan: $this->authorize('update', $product)
│   │   ├── OrderPolicy.php                    # Policy: siapa boleh view/create/edit order
│   │   └── UserPolicy.php                     # Policy: siapa boleh view/edit user
│   │
│   ├── Providers/                              # Service providers (register services)
│   │   ├── AppServiceProvider.php             # Register: global services, config
│   │   ├── AuthServiceProvider.php            # Register: gates, policies
│   │   ├── EventServiceProvider.php           # Register: event-listener mappings
│   │   └── RouteServiceProvider.php           # Register: route namespaces, bindings
│   │
│   ├── Repositories/                           # Repository pattern (optional best practice)
│   │   ├── Contracts/
│   │   │   ├── ProductRepositoryInterface.php # Interface: abstract ProductRepository
│   │   │   │                                   # Manfaat: bisa ganti implementation nanti
│   │   │   ├── OrderRepositoryInterface.php   # Interface: abstract OrderRepository
│   │   │   └── CustomerRepositoryInterface.php # Interface: abstract CustomerRepository
│   │   ├── ProductRepository.php              # Impl: query kompleks produk (filter, search)
│   │   │                                       # Contoh: getByCategory(), getTopSelling()
│   │   ├── OrderRepository.php                # Impl: query kompleks order
│   │   └── CustomerRepository.php             # Impl: query kompleks customer
│   │
│   ├── Services/                               # Business logic layer (layak kompleks)
│   │   ├── ProductService.php                 # Logic: create/update/delete produk + validasi
│   │   ├── OrderService.php                   # Logic: create order, validate stock, trigger events
│   │   ├── CartService.php                    # Logic: add/remove item, calculate total
│   │   ├── PaymentService.php                 # Logic: process payment, integrate payment gateway
│   │   ├── ReportService.php                  # Logic: calculate stats, generate reports
│   │   ├── InventoryService.php               # Logic: manage stock, low stock alerts
│   │   └── ExportService.php                  # Logic: export to PDF/Excel
│   │
│   ├── Traits/                                 # Reusable code snippets (trait = mixin)
│   │   ├── HasUuid.php                        # Trait: auto generate UUID untuk primary key
│   │   ├── Searchable.php                     # Trait: add fulltext search ke model
│   │   ├── Filterable.php                     # Trait: standardize filter query
│   │   └── Auditable.php                      # Trait: track siapa edit apa kapan
│   │
│   └── Actions/                                # Single action classes (one method per action)
│       ├── Product/
│       │   ├── CreateProductAction.php        # Action: create product dengan validasi
│       │   │                                   # File kecil + focused + testable
│       │   └── UpdateStockAction.php          # Action: update stok produk
│       └── Order/
│           ├── CreateOrderAction.php          # Action: create order lengkap
│           └── ProcessPaymentAction.php       # Action: process pembayaran
│
├── bootstrap/
│   └── app.php                                 # Bootstrap config Laravel
│
├── config/                                      # Konfigurasi aplikasi
│   ├── app.php                                 # Config: name, timezone, providers
│   ├── auth.php                                # Config: auth driver, guard, password
│   ├── database.php                            # Config: database connection
│   ├── filesystems.php                         # Config: storage (local, S3, dll)
│   ├── queue.php                               # Config: queue driver (sync, database, redis)
│   ├── shop.php                                # Config: custom untuk inisialisasi toko
│   │                                           # Contoh: tax rate, default currency
│   └── payment.php                             # Config: payment gateway (Stripe, Midtrans)
│
├── database/
│   ├── factories/                              # Fake data generators (untuk testing)
│   │   ├── UserFactory.php                    # Generate: 100 fake user dalam 1 baris
│   │   ├── ProductFactory.php                 # Generate: 50 fake produk dengan relations
│   │   ├── CategoryFactory.php                # Generate: 10 fake kategori
│   │   ├── OrderFactory.php                   # Generate: 200 fake order
│   │   └── CustomerFactory.php                # Generate: 100 fake customer
│   │
│   ├── migrations/                             # Schema definition (table creation)
│   │   ├── 0001_01_01_000000_create_users_table.php    # Migration: buat tabel users
│   │   │                                               # Run: php artisan migrate
│   │   ├── 0001_01_01_000001_create_cache_table.php    # Migration: buat tabel cache
│   │   ├── 2024_01_01_000001_create_categories_table.php
│   │   ├── 2024_01_01_000002_create_products_table.php
│   │   ├── 2024_01_01_000003_create_product_variants_table.php
│   │   ├── 2024_01_01_000004_create_product_images_table.php
│   │   ├── 2024_01_01_000005_create_customers_table.php
│   │   ├── 2024_01_01_000006_create_orders_table.php
│   │   ├── 2024_01_01_000007_create_order_items_table.php
│   │   ├── 2024_01_01_000008_create_payments_table.php
│   │   ├── 2024_01_01_000009_create_carts_table.php
│   │   ├── 2024_01_01_000010_create_discounts_table.php
│   │   ├── 2024_01_01_000011_create_settings_table.php
│   │   └── 2024_01_01_000012_create_audit_logs_table.php
│   │
│   └── seeders/                                # Dummy data loaders (populate test data)
│       ├── DatabaseSeeder.php                 # Master seeder (panggil seeder lainnya)
│       ├── UserSeeder.php                     # Seed: insert test users
│       ├── CategorySeeder.php                 # Seed: insert kategori
│       ├── ProductSeeder.php                  # Seed: insert produk demo
│       └── SettingSeeder.php                  # Seed: insert app settings default
│
├── public/                                     # Public files (accessible dari browser)
│   ├── index.php                              # Entry point aplikasi (jangan di-edit)
│   ├── css/                                   # Compiled CSS files (dari Tailwind)
│   ├── js/                                    # Compiled JS files (dari build)
│   └── images/                                # Static images (logo, icon, dll)
│
├── resources/                                  # Source assets (sebelum compile)
│   ├── css/
│   │   └── app.css                            # Main CSS (import Tailwind CSS)
│   │
│   ├── js/
│   │   ├── app.js                             # Main JS entry point
│   │   ├── bootstrap.js                       # Bootstrap config (axios, csrf token)
│   │   └── components/                        # Reusable Alpine.js components
│   │       ├── cart.js                        # Component: shopping cart state
│   │       ├── product-filter.js              # Component: filter product UI
│   │       └── data-table.js                  # Component: reusable table dengan sorting
│   │
│   └── views/                                 # Blade templates (HTML templates)
│       ├── layouts/
│       │   ├── app.blade.php                  # Layout: main web layout
│       │   ├── dashboard.blade.php            # Layout: dashboard layout dengan sidebar
│       │   ├── auth.blade.php                 # Layout: auth (login, register) layout
│       │   └── partials/
│       │       ├── sidebar.blade.php          # Partial: sidebar navigation
│       │       ├── navbar.blade.php           # Partial: top navbar
│       │       ├── footer.blade.php           # Partial: footer
│       │       └── alerts.blade.php           # Partial: flash messages (success, error)
│       │
│       ├── components/                        # Blade components (reusable UI)
│       │   ├── button.blade.php               # Component: <x-button /> dengan styling
│       │   ├── input.blade.php                # Component: <x-input /> form field
│       │   ├── select.blade.php               # Component: <x-select /> dropdown
│       │   ├── modal.blade.php                # Component: <x-modal /> dialog popup
│       │   ├── card.blade.php                 # Component: <x-card /> container
│       │   ├── table.blade.php                # Component: <x-table /> responsive table
│       │   ├── pagination.blade.php           # Component: <x-pagination /> page navigation
│       │   ├── alert.blade.php                # Component: <x-alert /> message alert
│       │   └── stats-card.blade.php           # Component: <x-stats-card /> dashboard card
│       │
│       ├── auth/                              # Auth page templates
│       │   ├── login.blade.php                # Page: form login
│       │   ├── register.blade.php             # Page: form register
│       │   └── forgot-password.blade.php      # Page: form lupa password
│       │
│       ├── dashboard/                         # Dashboard page templates
│       │   ├── index.blade.php                # Page: dashboard home, welcome
│       │   ├── products/
│       │   │   ├── index.blade.php            # Page: list produk (table)
│       │   │   ├── create.blade.php           # Page: form create produk baru
│       │   │   ├── edit.blade.php             # Page: form edit produk
│       │   │   └── show.blade.php             # Page: detail produk
│       │   │
│       │   ├── categories/
│       │   │   ├── index.blade.php            # Page: list kategori
│       │   │   ├── create.blade.php           # Page: form create kategori
│       │   │   └── edit.blade.php             # Page: form edit kategori
│       │   │
│       │   ├── orders/
│       │   │   ├── index.blade.php            # Page: list order (table dengan filter)
│       │   │   ├── show.blade.php             # Page: detail order + items
│       │   │   └── invoice.blade.php          # Page: invoice untuk cetak
│       │   │
│       │   ├── customers/
│       │   │   ├── index.blade.php            # Page: list customer
│       │   │   ├── create.blade.php           # Page: form create customer
│       │   │   ├── edit.blade.php             # Page: form edit customer
│       │   │   └── show.blade.php             # Page: detail customer + order history
│       │   │
│       │   ├── reports/
│       │   │   ├── index.blade.php            # Page: laporan overview
│       │   │   ├── sales.blade.php            # Page: laporan penjualan (chart + table)
│       │   │   ├── inventory.blade.php        # Page: laporan stok
│       │   │   └── customers.blade.php        # Page: laporan customer
│       │   │
│       │   ├── settings/
│       │   │   ├── index.blade.php            # Page: pengaturan toko
│       │   │   ├── store.blade.php            # Page: form edit info toko
│       │   │   └── payment.blade.php          # Page: form edit payment gateway
│       │   │
│       │   └── users/
│       │       ├── index.blade.php            # Page: list user (admin bisa manage)
│       │       ├── create.blade.php           # Page: form create user baru
│       │       └── edit.blade.php             # Page: form edit user
│       │
│       ├── pos/
│       │   └── index.blade.php                # Page: POS interface (kasir UI)
│       │
│       └── emails/
│           ├── order-confirmation.blade.php   # Email: konfirmasi order ke customer
│           └── low-stock-alert.blade.php      # Email: notif stok rendah ke admin
│
├── routes/
│   ├── web.php                                 # Routes: web pages (return HTML/Blade)
│   │                                           # Contoh: Route::get('/products', [...])
│   ├── api.php                                 # Routes: API endpoints (return JSON)
│   │                                           # Contoh: Route::get('/api/products', [...])
│   ├── auth.php                                # Routes: auth (included in web.php)
│   └── dashboard.php                           # Routes: dashboard (included in web.php)
│
├── storage/
│   ├── app/
│   │   ├── public/
│   │   │   └── products/                       # Folder: produk image uploads
│   │   │                                       # Akses: /storage/products/image.jpg
│   │   └── exports/                            # Folder: generated export files
│   │                                           # Generated: PDF/Excel reports
│   └── logs/
│       └── laravel.log                         # File: aplikasi error logs
│
├── tests/
│   ├── Feature/                                # Integration tests (test features lengkap)
│   │   ├── Auth/
│   │   │   └── AuthenticationTest.php          # Test: login, register, logout flow
│   │   ├── Product/
│   │   │   └── ProductManagementTest.php       # Test: CRUD produk flow
│   │   ├── Order/
│   │   │   └── OrderProcessTest.php           # Test: order process, payment flow
│   │   └── Api/
│   │       └── ProductApiTest.php             # Test: API endpoints return correct
│   │
│   └── Unit/                                   # Unit tests (test single function)
│       ├── Services/
│       │   ├── ProductServiceTest.php          # Test: ProductService methods
│       │   └── OrderServiceTest.php            # Test: OrderService methods
│       └── Models/
│           └── ProductTest.php                 # Test: Product model relationships
│
├── .env.example                                # File: template environment variables
│                                               # Copy: ke .env untuk local config
├── .gitignore                                  # File: file yang di-ignore Git
├── artisan                                     # Executable: CLI tool untuk command
├── composer.json                               # File: PHP dependencies specification
├── package.json                                # File: Node.js dependencies (JS, CSS)
├── phpunit.xml                                 # File: Testing configuration
├── tailwind.config.js                          # File: Tailwind CSS configuration
├── vite.config.js                              # File: Vite build tool configuration
└── README.md                                   # File: project documentation (INI!)
```

---

## 🔄 State Management

### Server-Side State (Laravel)

```
┌─────────────────────────────────────────────────────────────────┐
│                    STATE MANAGEMENT FLOW                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────┐                                               │
│   │   Session   │ ◄── Authenticated User Data                   │
│   │   State     │ ◄── Flash Messages                            │
│   │             │ ◄── CSRF Token                                │
│   └─────────────┘                                               │
│          │                                                       │
│          ▼                                                       │
│   ┌─────────────┐                                               │
│   │   Cache     │ ◄── Dashboard Statistics                      │
│   │   (Redis)   │ ◄── Product Listings                          │
│   │             │ ◄── Category Tree                             │
│   └─────────────┘                                               │
│          │                                                       │
│          ▼                                                       │
│   ┌─────────────┐                                               │
│   │  Database   │ ◄── Persistent Data                           │
│   │  (MySQL)    │ ◄── Transactions                              │
│   │             │ ◄── Relationships                             │
│   └─────────────┘                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Client-Side State (Alpine.js + Livewire)

```javascript
// resources/js/components/cart.js
document.addEventListener("alpine:init", () => {
  Alpine.store("cart", {
    items: [],
    total: 0,

    init() {
      // Load cart from localStorage or session
      const savedCart = localStorage.getItem("cart");
      if (savedCart) {
        this.items = JSON.parse(savedCart);
        this.calculateTotal();
      }
    },

    addItem(product) {
      const existing = this.items.find((item) => item.id === product.id);
      if (existing) {
        existing.quantity++;
      } else {
        this.items.push({ ...product, quantity: 1 });
      }
      this.calculateTotal();
      this.persist();
    },

    removeItem(productId) {
      this.items = this.items.filter((item) => item.id !== productId);
      this.calculateTotal();
      this.persist();
    },

    calculateTotal() {
      this.total = this.items.reduce((sum, item) => {
        return sum + item.price * item.quantity;
      }, 0);
    },

    persist() {
      localStorage.setItem("cart", JSON.stringify(this.items));
    },

    clear() {
      this.items = [];
      this.total = 0;
      localStorage.removeItem("cart");
    },
  });
});
```

### State Flow dalam Aplikasi

```
┌──────────────────────────────────────────────────────────────────────────┐
│                           USER INTERACTION                                │
│                                                                          │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐               │
│  │   Browser    │    │  Alpine.js   │    │  Livewire    │               │
│  │   Events     │───▶│    Store     │◄──▶│  Components  │               │
│  └──────────────┘    └──────────────┘    └──────────────┘               │
│                             │                   │                        │
│                             │                   │                        │
│                      ┌──────▼───────────────────▼──────┐                │
│                      │         AJAX / Fetch            │                │
│                      │         API Requests            │                │
│                      └──────────────┬──────────────────┘                │
│                                     │                                    │
└─────────────────────────────────────┼────────────────────────────────────┘
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                           SERVER PROCESSING                              │
│                                                                          │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐               │
│  │  Controller  │───▶│   Service    │───▶│    Model     │               │
│  │              │    │              │    │              │               │
│  └──────────────┘    └──────┬───────┘    └──────┬───────┘               │
│                             │                   │                        │
│                      ┌──────▼───────┐    ┌──────▼───────┐               │
│                      │    Cache     │    │   Database   │               │
│                      │   (Redis)    │    │   (MySQL)    │               │
│                      └──────────────┘    └──────────────┘               │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

### Contoh Service Layer (State Management Server-Side)

```php
// app/Services/CartService.php
<?php

namespace App\Services;

use App\Models\Cart;
use App\Models\CartItem;
use App\Models\Product;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\DB;

class CartService
{
    /**
     * Get or create cart for current user/session
     */
    public function getCart(): Cart
    {
        $userId = Auth::id();
        $sessionId = session()->getId();

        $cacheKey = $userId ? "cart_user_{$userId}" : "cart_session_{$sessionId}";

        return Cache::remember($cacheKey, 3600, function () use ($userId, $sessionId) {
            return Cart::firstOrCreate([
                'user_id' => $userId,
                'session_id' => $userId ? null : $sessionId,
            ]);
        });
    }

    /**
     * Add item to cart with stock validation
     */
    public function addItem(int $productId, int $variantId = null, int $quantity = 1): CartItem
    {
        return DB::transaction(function () use ($productId, $variantId, $quantity) {
            $cart = $this->getCart();
            $product = Product::findOrFail($productId);

            // Check stock availability
            if (!$this->checkStock($product, $variantId, $quantity)) {
                throw new \App\Exceptions\InsufficientStockException(
                    "Insufficient stock for {$product->name}"
                );
            }

            // Add or update cart item
            $cartItem = CartItem::updateOrCreate(
                [
                    'cart_id' => $cart->id,
                    'product_id' => $productId,
                    'variant_id' => $variantId,
                ],
                [
                    'quantity' => DB::raw("quantity + {$quantity}"),
                    'price' => $product->price,
                ]
            );

            // Clear cart cache
            $this->clearCartCache();

            return $cartItem->fresh();
        });
    }

    /**
     * Calculate cart totals
     */
    public function calculateTotals(): array
    {
        $cart = $this->getCart();

        $subtotal = $cart->items->sum(function ($item) {
            return $item->price * $item->quantity;
        });

        $discount = $this->calculateDiscount($cart);
        $tax = ($subtotal - $discount) * 0.11; // PPN 11%
        $total = $subtotal - $discount + $tax;

        return [
            'subtotal' => $subtotal,
            'discount' => $discount,
            'tax' => $tax,
            'total' => $total,
            'items_count' => $cart->items->sum('quantity'),
        ];
    }

    /**
     * Clear cart cache
     */
    private function clearCartCache(): void
    {
        $userId = Auth::id();
        $sessionId = session()->getId();

        Cache::forget($userId ? "cart_user_{$userId}" : "cart_session_{$sessionId}");
    }

    private function checkStock(Product $product, ?int $variantId, int $quantity): bool
    {
        // Stock checking logic
        return true;
    }

    private function calculateDiscount(Cart $cart): float
    {
        // Discount calculation logic
        return 0;
    }
}
```

---

## 🚀 API Design & Best Practices

### API Versioning Structure

```
/api/v1/
├── /auth
│   ├── POST   /login              # Login
│   ├── POST   /register           # Register
│   ├── POST   /logout             # Logout
│   ├── POST   /refresh            # Refresh token
│   └── GET    /me                 # Current user
│
├── /products
│   ├── GET    /                   # List products (paginated)
│   ├── POST   /                   # Create product
│   ├── GET    /{id}               # Get product detail
│   ├── PUT    /{id}               # Update product
│   ├── DELETE /{id}               # Delete product
│   ├── POST   /{id}/images        # Upload images
│   ├── GET    /{id}/variants      # Get variants
│   └── POST   /{id}/variants      # Create variant
│
├── /categories
│   ├── GET    /                   # List categories
│   ├── POST   /                   # Create category
│   ├── GET    /{id}               # Get category
│   ├── PUT    /{id}               # Update category
│   ├── DELETE /{id}               # Delete category
│   └── GET    /{id}/products      # Category products
│
├── /orders
│   ├── GET    /                   # List orders
│   ├── POST   /                   # Create order
│   ├── GET    /{id}               # Get order detail
│   ├── PUT    /{id}/status        # Update status
│   └── GET    /{id}/invoice       # Get invoice
│
├── /customers
│   ├── GET    /                   # List customers
│   ├── POST   /                   # Create customer
│   ├── GET    /{id}               # Get customer
│   ├── PUT    /{id}               # Update customer
│   ├── DELETE /{id}               # Delete customer
│   └── GET    /{id}/orders        # Customer orders
│
├── /cart
│   ├── GET    /                   # Get cart
│   ├── POST   /items              # Add item
│   ├── PUT    /items/{id}         # Update item quantity
│   ├── DELETE /items/{id}         # Remove item
│   └── DELETE /                   # Clear cart
│
├── /payments
│   ├── POST   /                   # Process payment
│   ├── GET    /{id}               # Get payment status
│   └── POST   /webhook            # Payment webhook
│
└── /reports
    ├── GET    /dashboard          # Dashboard stats
    ├── GET    /sales              # Sales report
    ├── GET    /inventory          # Inventory report
    └── GET    /customers          # Customer report
```

### API Response Format (Best Practice)

```json
// Successful Response
{
    "success": true,
    "message": "Products retrieved successfully",
    "data": {
        "items": [...],
        "meta": {
            "current_page": 1,
            "per_page": 15,
            "total": 100,
            "last_page": 7
        }
    }
}

// Error Response
{
    "success": false,
    "message": "Validation failed",
    "errors": {
        "name": ["The name field is required."],
        "price": ["The price must be a number."]
    }
}
```

### Route File Example

```php
// routes/api.php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\V1;

Route::prefix('v1')->group(function () {

    // Public routes
    Route::post('auth/login', [V1\AuthController::class, 'login']);
    Route::post('auth/register', [V1\AuthController::class, 'register']);

    // Protected routes
    Route::middleware(['auth:sanctum', 'throttle:api'])->group(function () {

        // Auth
        Route::prefix('auth')->group(function () {
            Route::post('logout', [V1\AuthController::class, 'logout']);
            Route::post('refresh', [V1\AuthController::class, 'refresh']);
            Route::get('me', [V1\AuthController::class, 'me']);
        });

        // Products
        Route::apiResource('products', V1\ProductController::class);
        Route::post('products/{product}/images', [V1\ProductController::class, 'uploadImages']);
        Route::apiResource('products.variants', V1\ProductVariantController::class)->shallow();

        // Categories
        Route::apiResource('categories', V1\CategoryController::class);
        Route::get('categories/{category}/products', [V1\CategoryController::class, 'products']);

        // Orders
        Route::apiResource('orders', V1\OrderController::class)->except(['update', 'destroy']);
        Route::put('orders/{order}/status', [V1\OrderController::class, 'updateStatus']);
        Route::get('orders/{order}/invoice', [V1\OrderController::class, 'invoice']);

        // Customers
        Route::apiResource('customers', V1\CustomerController::class);
        Route::get('customers/{customer}/orders', [V1\CustomerController::class, 'orders']);

        // Cart
        Route::prefix('cart')->group(function () {
            Route::get('/', [V1\CartController::class, 'index']);
            Route::post('items', [V1\CartController::class, 'addItem']);
            Route::put('items/{item}', [V1\CartController::class, 'updateItem']);
            Route::delete('items/{item}', [V1\CartController::class, 'removeItem']);
            Route::delete('/', [V1\CartController::class, 'clear']);
        });

        // Reports (Admin only)
        Route::middleware('role:admin,supervisor')->prefix('reports')->group(function () {
            Route::get('dashboard', [V1\ReportController::class, 'dashboard']);
            Route::get('sales', [V1\ReportController::class, 'sales']);
            Route::get('inventory', [V1\ReportController::class, 'inventory']);
            Route::get('customers', [V1\ReportController::class, 'customers']);
        });
    });

    // Webhook (no auth required, verify signature)
    Route::post('payments/webhook', [V1\PaymentController::class, 'webhook']);
});
```

### Controller Example (Best Practice)

```php
// app/Http/Controllers/Api/V1/ProductController.php
<?php

namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use App\Http\Requests\Product\StoreProductRequest;
use App\Http\Requests\Product\UpdateProductRequest;
use App\Http\Resources\ProductResource;
use App\Http\Resources\ProductCollection;
use App\Models\Product;
use App\Services\ProductService;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    public function __construct(
        private ProductService $productService
    ) {}

    /**
     * Display a listing of products
     */
    public function index(Request $request): ProductCollection
    {
        $products = $this->productService->getFilteredProducts(
            filters: $request->only(['category', 'search', 'min_price', 'max_price', 'size']),
            sortBy: $request->get('sort_by', 'created_at'),
            sortOrder: $request->get('sort_order', 'desc'),
            perPage: $request->get('per_page', 15)
        );

        return new ProductCollection($products);
    }

    /**
     * Store a newly created product
     */
    public function store(StoreProductRequest $request): JsonResponse
    {
        $product = $this->productService->createProduct($request->validated());

        return response()->json([
            'success' => true,
            'message' => 'Product created successfully',
            'data' => new ProductResource($product),
        ], 201);
    }

    /**
     * Display the specified product
     */
    public function show(Product $product): JsonResponse
    {
        $product->load(['category', 'variants', 'images']);

        return response()->json([
            'success' => true,
            'data' => new ProductResource($product),
        ]);
    }

    /**
     * Update the specified product
     */
    public function update(UpdateProductRequest $request, Product $product): JsonResponse
    {
        $product = $this->productService->updateProduct($product, $request->validated());

        return response()->json([
            'success' => true,
            'message' => 'Product updated successfully',
            'data' => new ProductResource($product),
        ]);
    }

    /**
     * Remove the specified product
     */
    public function destroy(Product $product): JsonResponse
    {
        $this->authorize('delete', $product);

        $this->productService->deleteProduct($product);

        return response()->json([
            'success' => true,
            'message' => 'Product deleted successfully',
        ]);
    }

    /**
     * Upload product images
     */
    public function uploadImages(Request $request, Product $product): JsonResponse
    {
        $request->validate([
            'images' => 'required|array|max:5',
            'images.*' => 'image|mimes:jpeg,png,jpg,webp|max:2048',
        ]);

        $images = $this->productService->uploadImages($product, $request->file('images'));

        return response()->json([
            'success' => true,
            'message' => 'Images uploaded successfully',
            'data' => $images,
        ]);
    }
}
```

### Form Request Validation

```php
// app/Http/Requests/Product/StoreProductRequest.php
<?php

namespace App\Http\Requests\Product;

use App\Enums\ProductSize;
use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class StoreProductRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create', Product::class);
    }

    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'slug' => ['nullable', 'string', 'max:255', 'unique:products,slug'],
            'description' => ['nullable', 'string', 'max:5000'],
            'category_id' => ['required', 'exists:categories,id'],
            'price' => ['required', 'numeric', 'min:0', 'max:999999999'],
            'compare_price' => ['nullable', 'numeric', 'min:0', 'gt:price'],
            'cost_price' => ['nullable', 'numeric', 'min:0'],
            'sku' => ['required', 'string', 'unique:products,sku'],
            'barcode' => ['nullable', 'string', 'unique:products,barcode'],
            'stock' => ['required', 'integer', 'min:0'],
            'low_stock_threshold' => ['nullable', 'integer', 'min:1'],
            'weight' => ['nullable', 'numeric', 'min:0'],
            'is_active' => ['boolean'],
            'is_featured' => ['boolean'],

            // Variants
            'variants' => ['nullable', 'array'],
            'variants.*.size' => ['required_with:variants', Rule::enum(ProductSize::class)],
            'variants.*.color' => ['nullable', 'string', 'max:50'],
            'variants.*.sku' => ['required_with:variants', 'string', 'distinct'],
            'variants.*.price' => ['nullable', 'numeric', 'min:0'],
            'variants.*.stock' => ['required_with:variants', 'integer', 'min:0'],

            // Images
            'images' => ['nullable', 'array', 'max:5'],
            'images.*' => ['image', 'mimes:jpeg,png,jpg,webp', 'max:2048'],
        ];
    }

    public function messages(): array
    {
        return [
            'name.required' => 'Nama produk wajib diisi',
            'category_id.required' => 'Kategori wajib dipilih',
            'price.required' => 'Harga produk wajib diisi',
            'sku.unique' => 'SKU sudah digunakan',
            'stock.min' => 'Stok tidak boleh kurang dari 0',
        ];
    }

    protected function prepareForValidation(): void
    {
        if (!$this->slug && $this->name) {
            $this->merge([
                'slug' => \Str::slug($this->name),
            ]);
        }
    }
}
```

### API Resource (Response Transformer)

```php
// app/Http/Resources/ProductResource.php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class ProductResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'slug' => $this->slug,
            'description' => $this->description,
            'sku' => $this->sku,
            'barcode' => $this->barcode,

            // Pricing
            'price' => $this->price,
            'formatted_price' => $this->formatted_price,
            'compare_price' => $this->compare_price,
            'discount_percentage' => $this->discount_percentage,

            // Stock
            'stock' => $this->stock,
            'is_in_stock' => $this->is_in_stock,
            'is_low_stock' => $this->is_low_stock,

            // Status
            'is_active' => $this->is_active,
            'is_featured' => $this->is_featured,

            // Relationships (conditional loading)
            'category' => new CategoryResource($this->whenLoaded('category')),
            'variants' => ProductVariantResource::collection($this->whenLoaded('variants')),
            'images' => ProductImageResource::collection($this->whenLoaded('images')),

            // Computed
            'primary_image' => $this->primary_image_url,

            // Timestamps
            'created_at' => $this->created_at->toISOString(),
            'updated_at' => $this->updated_at->toISOString(),
        ];
    }
}
```

---

## 🗄️ Database Design

### Entity Relationship Diagram (ERD)

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│     users       │     │   categories    │     │    products     │
├─────────────────┤     ├─────────────────┤     ├─────────────────┤
│ id              │     │ id              │     │ id              │
│ name            │     │ name            │     │ category_id     │───┐
│ email           │     │ slug            │     │ name            │   │
│ password        │     │ description     │     │ slug            │   │
│ role            │     │ image           │     │ description     │   │
│ avatar          │     │ parent_id       │──┐  │ sku             │   │
│ is_active       │     │ is_active       │  │  │ barcode         │   │
│ created_at      │     │ created_at      │  │  │ price           │   │
│ updated_at      │     │ updated_at      │  │  │ compare_price   │   │
└────────┬────────┘     └────────┬────────┘  │  │ cost_price      │   │
         │                       │           │  │ stock           │   │
         │                       │           │  │ weight          │   │
         │                       └───────────┘  │ is_active       │   │
         │                                      │ is_featured     │   │
         │              ┌───────────────────────│ created_at      │   │
         │              │                       │ updated_at      │   │
         │              │                       └────────┬────────┘   │
         │              │                                │            │
         │              │                                │            │
         │              ▼                                ▼            │
         │     ┌─────────────────┐            ┌─────────────────┐    │
         │     │ product_variants│            │  product_images │    │
         │     ├─────────────────┤            ├─────────────────┤    │
         │     │ id              │            │ id              │    │
         │     │ product_id      │◄───────────│ product_id      │    │
         │     │ size            │            │ path            │    │
         │     │ color           │            │ is_primary      │    │
         │     │ sku             │            │ sort_order      │    │
         │     │ price           │            │ created_at      │    │
         │     │ stock           │            └─────────────────┘    │
         │     │ created_at      │                                   │
         │     └─────────────────┘                                   │
         │                                                           │
         │     ┌─────────────────┐            ┌─────────────────┐   │
         │     │    customers    │            │     orders      │   │
         │     ├─────────────────┤            ├─────────────────┤   │
         │     │ id              │◄───────────│ customer_id     │   │
         │     │ name            │            │ user_id         │◄──┤
         │     │ email           │            │ order_number    │   │
         │     │ phone           │            │ status          │   │
         │     │ address         │            │ subtotal        │   │
         │     │ city            │            │ discount        │   │
         │     │ postal_code     │            │ tax             │   │
         │     │ membership_type │            │ total           │   │
         │     │ points          │            │ notes           │   │
         │     │ created_at      │            │ created_at      │   │
         │     │ updated_at      │            │ updated_at      │   │
         │     └─────────────────┘            └────────┬────────┘   │
         │                                             │            │
         │                                             │            │
         │                                             ▼            │
         │                                    ┌─────────────────┐   │
         │                                    │   order_items   │   │
         │                                    ├─────────────────┤   │
         │                                    │ id              │   │
         │                                    │ order_id        │   │
         │                                    │ product_id      │───┘
         │                                    │ variant_id      │
         │                                    │ name            │
         │                                    │ price           │
         │                                    │ quantity        │
         │                                    │ subtotal        │
         │                                    │ created_at      │
         │                                    └─────────────────┘
         │
         │     ┌─────────────────┐            ┌─────────────────┐
         │     │    payments     │            │    discounts    │
         │     ├─────────────────┤            ├─────────────────┤
         │     │ id              │            │ id              │
         └────▶│ order_id        │            │ code            │
               │ method          │            │ type            │
               │ amount          │            │ value           │
               │ status          │            │ min_order       │
               │ transaction_id  │            │ max_uses        │
               │ paid_at         │            │ used_count      │
               │ created_at      │            │ starts_at       │
               └─────────────────┘            │ expires_at      │
                                              │ is_active       │
                                              │ created_at      │
                                              └─────────────────┘
```

### Migration Examples

```php
// database/migrations/2024_01_01_000002_create_products_table.php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->foreignId('category_id')->constrained()->cascadeOnDelete();
            $table->string('name');
            $table->string('slug')->unique();
            $table->text('description')->nullable();
            $table->string('sku')->unique();
            $table->string('barcode')->nullable()->unique();
            $table->decimal('price', 12, 2);
            $table->decimal('compare_price', 12, 2)->nullable();
            $table->decimal('cost_price', 12, 2)->nullable();
            $table->unsignedInteger('stock')->default(0);
            $table->unsignedInteger('low_stock_threshold')->default(10);
            $table->decimal('weight', 8, 2)->nullable()->comment('Weight in grams');
            $table->boolean('is_active')->default(true);
            $table->boolean('is_featured')->default(false);
            $table->timestamps();
            $table->softDeletes();

            // Indexes
            $table->index(['is_active', 'is_featured']);
            $table->index('created_at');
            $table->fullText(['name', 'description']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('products');
    }
};
```

```php
// database/migrations/2024_01_01_000006_create_orders_table.php
<?php

use App\Enums\OrderStatus;
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('orders', function (Blueprint $table) {
            $table->id();
            $table->string('order_number')->unique();
            $table->foreignId('customer_id')->nullable()->constrained()->nullOnDelete();
            $table->foreignId('user_id')->comment('Kasir/Admin yang memproses')->constrained();
            $table->string('status')->default(OrderStatus::PENDING->value);
            $table->decimal('subtotal', 12, 2);
            $table->decimal('discount', 12, 2)->default(0);
            $table->decimal('tax', 12, 2)->default(0);
            $table->decimal('total', 12, 2);
            $table->string('discount_code')->nullable();
            $table->text('notes')->nullable();
            $table->timestamps();
            $table->softDeletes();

            $table->index(['status', 'created_at']);
            $table->index('order_number');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('orders');
    }
};
```

### Model Example

```php
// app/Models/Product.php
<?php

namespace App\Models;

use App\Traits\HasUuid;
use App\Traits\Searchable;
use App\Traits\Filterable;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Casts\Attribute;

class Product extends Model
{
    use HasFactory, SoftDeletes, Searchable, Filterable;

    protected $fillable = [
        'category_id',
        'name',
        'slug',
        'description',
        'sku',
        'barcode',
        'price',
        'compare_price',
        'cost_price',
        'stock',
        'low_stock_threshold',
        'weight',
        'is_active',
        'is_featured',
    ];

    protected $casts = [
        'price' => 'decimal:2',
        'compare_price' => 'decimal:2',
        'cost_price' => 'decimal:2',
        'stock' => 'integer',
        'weight' => 'decimal:2',
        'is_active' => 'boolean',
        'is_featured' => 'boolean',
    ];

    // ==================
    // RELATIONSHIPS
    // ==================

    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }

    public function variants(): HasMany
    {
        return $this->hasMany(ProductVariant::class);
    }

    public function images(): HasMany
    {
        return $this->hasMany(ProductImage::class)->orderBy('sort_order');
    }

    public function orderItems(): HasMany
    {
        return $this->hasMany(OrderItem::class);
    }

    // ==================
    // ACCESSORS
    // ==================

    protected function formattedPrice(): Attribute
    {
        return Attribute::make(
            get: fn () => 'Rp ' . number_format($this->price, 0, ',', '.')
        );
    }

    protected function discountPercentage(): Attribute
    {
        return Attribute::make(
            get: function () {
                if (!$this->compare_price || $this->compare_price <= $this->price) {
                    return 0;
                }
                return round(($this->compare_price - $this->price) / $this->compare_price * 100);
            }
        );
    }

    protected function isInStock(): Attribute
    {
        return Attribute::make(
            get: fn () => $this->stock > 0
        );
    }

    protected function isLowStock(): Attribute
    {
        return Attribute::make(
            get: fn () => $this->stock > 0 && $this->stock <= $this->low_stock_threshold
        );
    }

    protected function primaryImageUrl(): Attribute
    {
        return Attribute::make(
            get: function () {
                $primary = $this->images->firstWhere('is_primary', true);
                return $primary
                    ? asset('storage/' . $primary->path)
                    : asset('images/no-image.png');
            }
        );
    }

    // ==================
    // SCOPES
    // ==================

    public function scopeActive($query)
    {
        return $query->where('is_active', true);
    }

    public function scopeFeatured($query)
    {
        return $query->where('is_featured', true);
    }

    public function scopeInStock($query)
    {
        return $query->where('stock', '>', 0);
    }

    public function scopeLowStock($query)
    {
        return $query->whereColumn('stock', '<=', 'low_stock_threshold')
                     ->where('stock', '>', 0);
    }

    public function scopeByCategory($query, int $categoryId)
    {
        return $query->where('category_id', $categoryId);
    }

    public function scopePriceRange($query, ?float $min, ?float $max)
    {
        return $query->when($min, fn ($q) => $q->where('price', '>=', $min))
                     ->when($max, fn ($q) => $q->where('price', '<=', $max));
    }

    // ==================
    // METHODS
    // ==================

    public function decreaseStock(int $quantity): bool
    {
        if ($this->stock < $quantity) {
            return false;
        }

        $this->decrement('stock', $quantity);

        if ($this->is_low_stock) {
            event(new \App\Events\ProductStockLow($this));
        }

        return true;
    }

    public function increaseStock(int $quantity): void
    {
        $this->increment('stock', $quantity);
    }
}
```

---

## 📦 Instalasi

### Prasyarat

- PHP >= 8.2
- Composer
- Node.js >= 18.x & NPM
- MySQL >= 8.0 atau MariaDB >= 10.3
- Redis (opsional, untuk cache & queue)

### Langkah Instalasi

```bash
# 1. Clone repository
git clone https://github.com/username/dashboardqu-laravel.git
cd dashboardqu-laravel

# 2. Install PHP dependencies
composer install

# 3. Install Node dependencies
npm install

# 4. Copy environment file
cp .env.example .env

# 5. Generate application key
php artisan key:generate

# 6. Configure database in .env
# DB_CONNECTION=mysql
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=dashboardqu
# DB_USERNAME=root
# DB_PASSWORD=

# 7. Run migrations & seeders
php artisan migrate --seed

# 8. Create storage link
php artisan storage:link

# 9. Build assets
npm run build
# atau untuk development
npm run dev

# 10. Start development server
php artisan serve
```

### Konfigurasi Tambahan

```bash
# Cache configuration (production)
php artisan config:cache
php artisan route:cache
php artisan view:cache

# Queue worker (jika menggunakan queue)
php artisan queue:work

# Scheduler (tambahkan ke crontab)
* * * * * cd /path-to-project && php artisan schedule:run >> /dev/null 2>&1
```

---

## 🔧 Penggunaan

### Menjalankan Development Server

```bash
# Terminal 1 - Laravel server
php artisan serve

# Terminal 2 - Vite dev server
npm run dev

# Terminal 3 - Queue worker (opsional)
php artisan queue:work
```

### Akses Aplikasi

- **Dashboard**: http://localhost:8000/dashboard
- **POS**: http://localhost:8000/pos
- **API**: http://localhost:8000/api/v1

### Default Credentials

| Role       | Email                  | Password |
| ---------- | ---------------------- | -------- |
| Admin      | admin@example.com      | password |
| Supervisor | supervisor@example.com | password |
| Kasir      | kasir@example.com      | password |

---

## 🧪 Testing

### Menjalankan Tests

```bash
# Run all tests
php artisan test

# Run specific test suite
php artisan test --testsuite=Feature
php artisan test --testsuite=Unit

# Run specific test file
php artisan test tests/Feature/Product/ProductManagementTest.php

# Run with coverage
php artisan test --coverage

# Run in parallel
php artisan test --parallel
```

### Test Example

```php
// tests/Feature/Product/ProductManagementTest.php
<?php

namespace Tests\Feature\Product;

use App\Models\Product;
use App\Models\User;
use App\Models\Category;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;
use Tests\TestCase;

class ProductManagementTest extends TestCase
{
    use RefreshDatabase;

    private User $admin;
    private Category $category;

    protected function setUp(): void
    {
        parent::setUp();

        $this->admin = User::factory()->admin()->create();
        $this->category = Category::factory()->create();
    }

    public function test_admin_can_view_products_list(): void
    {
        Product::factory()->count(15)->create();

        $response = $this->actingAs($this->admin)
            ->get(route('dashboard.products.index'));

        $response->assertOk()
            ->assertViewIs('dashboard.products.index')
            ->assertViewHas('products');
    }

    public function test_admin_can_create_product(): void
    {
        Storage::fake('public');

        $productData = [
            'name' => 'Kemeja Batik Premium',
            'category_id' => $this->category->id,
            'price' => 250000,
            'sku' => 'KBP-001',
            'stock' => 50,
            'description' => 'Kemeja batik premium dengan motif modern',
            'images' => [
                UploadedFile::fake()->image('product1.jpg'),
            ],
        ];

        $response = $this->actingAs($this->admin)
            ->post(route('dashboard.products.store'), $productData);

        $response->assertRedirect(route('dashboard.products.index'))
            ->assertSessionHas('success');

        $this->assertDatabaseHas('products', [
            'name' => 'Kemeja Batik Premium',
            'sku' => 'KBP-001',
            'price' => 250000,
        ]);
    }

    public function test_admin_can_update_product(): void
    {
        $product = Product::factory()->create();

        $response = $this->actingAs($this->admin)
            ->put(route('dashboard.products.update', $product), [
                'name' => 'Updated Product Name',
                'category_id' => $this->category->id,
                'price' => 300000,
                'sku' => $product->sku,
                'stock' => 100,
            ]);

        $response->assertRedirect()
            ->assertSessionHas('success');

        $this->assertDatabaseHas('products', [
            'id' => $product->id,
            'name' => 'Updated Product Name',
            'price' => 300000,
        ]);
    }

    public function test_api_returns_paginated_products(): void
    {
        Product::factory()->count(20)->create();

        $response = $this->actingAs($this->admin, 'sanctum')
            ->getJson('/api/v1/products');

        $response->assertOk()
            ->assertJsonStructure([
                'data' => [
                    '*' => ['id', 'name', 'slug', 'price', 'stock']
                ],
                'meta' => ['current_page', 'per_page', 'total'],
            ]);
    }
}
```

---

## 📚 Best Practices yang Diterapkan

### 1. **SOLID Principles**

- Single Responsibility: Setiap class memiliki satu tanggung jawab
- Open/Closed: Menggunakan interface dan abstraction
- Liskov Substitution: Child class dapat menggantikan parent
- Interface Segregation: Interface yang spesifik
- Dependency Inversion: Dependency injection via constructor

### 2. **Laravel Best Practices**

- ✅ Form Request untuk validasi
- ✅ API Resources untuk response transformation
- ✅ Service Layer untuk business logic
- ✅ Repository Pattern (opsional) untuk data access
- ✅ Observer untuk model events
- ✅ Policy untuk authorization
- ✅ Events & Listeners untuk decoupling
- ✅ Blade Components untuk reusable UI
- ✅ Route Model Binding
- ✅ Eloquent Relationships & Eager Loading
- ✅ Database Transactions
- ✅ Caching Strategy
- ✅ Queue for heavy tasks

### 3. **Security**

- ✅ CSRF Protection
- ✅ XSS Prevention (Blade escaping)
- ✅ SQL Injection Prevention (Eloquent)
- ✅ Authentication dengan Sanctum
- ✅ Authorization dengan Policies
- ✅ Rate Limiting
- ✅ Input Validation
- ✅ Password Hashing

### 4. **Performance**

- ✅ Eager Loading (N+1 prevention)
- ✅ Database Indexing
- ✅ Query Caching
- ✅ Response Caching
- ✅ Asset Optimization (Vite)
- ✅ Lazy Loading Images

---

## 🤝 Contributing

1. Fork repository
2. Create feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open Pull Request

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for more information.

---

## 📞 Contact

Your Name - [@yourtwitter](https://twitter.com/yourtwitter) - email@example.com

Project Link: [https://github.com/username/dashboardqu-laravel](https://github.com/username/dashboardqu-laravel)
