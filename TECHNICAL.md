# Technical Specification: SiapAja.id Platform

---

## 1. Executive Summary

SiapAja.id adalah platform gig economy real-time yang dibangun dengan arsitektur modern berbasis Rust workspace monorepo, Flutter Mobile App, dan SpacetimeDB. Dokumen ini menjelaskan secara komprehensif arsitektur sistem, komponen teknis, alur data, dan spesifikasi implementasi tanpa menyertakan kode sumber.

**Target Pembaca:**
- Software Architects evaluating the platform
- Backend Engineers (Rust)
- Frontend Engineers (Flutter/Dart)
- DevOps/Infrastructure Engineers
- Technical Product Managers

---

## 2. System Overview

### 2.1 Problem Domain
Platform gig economy konvensional memiliki masalah fundamental:
- **Latency tinggi:** Polling-based updates menciptakan delay 5-30 detik
- **Biaya infrastruktur mahal:** Arsitektur monolitik membutuhkan cluster server besar
- **Komisi tidak proporsional:** 20-30% dipotong dari pekerja
- **Regulasi berat:** Menampung dana user membutuhkan izin OJK sebagai e-money/payment gateway

### 2.2 Solution Approach
SiapAja.id menyelesaikan ini melalui:
1. **Third-party Escrow Integration** - Xendit/Flip/ Midtrans Escrow API menampung dana, bukan SiapAja
2. **Dual Database Architecture:**
   - **PostgreSQL:** REST API (Axum) dengan OpenAPI auto-generation untuk persistent data (user profiles, transactions, financial records)
   - **SpacetimeDB:** Binary protocol dengan community Dart SDK untuk real-time state (job feed, GPS tracking, live notifications)
3. **Ultra-efficient backend** dalam Rust workspace dengan clean architecture
4. **Native Mobile App** dengan personalized feed recommendation
5. **Dual-role architecture** - satu user bisa switch antara customer dan worker

### 2.3 Core Value Proposition
- **3% fee** untuk transaksi di bawah Rp500.000 (100% dari customer, worker 0%)
- **Tanpa izin OJK** - tidak menampung dana user
- **Sub-millisecond latency** untuk state updates
- **VPS $5/bulan** bisa handle 50.000+ concurrent users
- **Personalized feed** - tiap user lihat content berbeda berdasarkan preferensi dan lokasi

---

## 3. Architectural Philosophy

### 3.1 Design Principles

**Performance First**
- Setiap milidetik latency berarti kehilangan competitive advantage
- Memory efficiency lebih penting dari developer convenience
- Compile-time safety lebih baik dari runtime checks

**Zero Regulatory Burden**
- Tidak menampung uang user di rekening perusahaan
- Escrow di-handle oleh licensed payment gateway (Xendit/Flip)
- SiapAja hanya sebagai "platform introducer", bukan payment provider

**Zero External Cache Dependency**
- Tidak menggunakan Redis - SpacetimeDB cukup untuk real-time state
- Session management menggunakan JWT stateless + SpacetimeDB temporary storage
- Rate limiting menggunakan in-memory Axum middleware (tower-governor)

**Modularity & Separation of Concerns**
- Clean Architecture (Onion/Hexagonal) - domain logic di center, infrastructure di outer layer
- Workspace crates dengan dependencies yang jelas
- Trait-based abstractions untuk testability

### 3.2 Technology Selection Rationale

| Layer | Technology | Justification |
|-------|------------|---------------|
| Backend Runtime | Rust + Tokio | Zero-cost async, memory safety, no GC pauses |
| Web Framework | Axum | Built by Tokio team, Tower ecosystem, type-safe routing |
| Database (Persistent) | PostgreSQL | ACID compliance untuk financial records, SQLx untuk compile-time checked queries |
| Database (Real-time) | SpacetimeDB | In-memory speed dengan persistence, native WebSocket subscriptions, community Dart SDK (production ready), replaced Redis |
| AI Integration | OpenRouter API | Access 200+ LLM models (Claude, GPT, Llama) with single API key, text-only calls |
| API Documentation | Utoipa | Compile-time OpenAPI generation for REST API (Axum → Flutter) |
| Frontend Framework | Flutter 3+ | Native mobile app dengan performant rendering, Riverpod untuk state management |
| Build Tool | Flutter CLI + FVM | Flutter version management, fast compilation |
| State Management | Riverpod | Compile-time safe, testable, scalable state management |
| Styling | Flutter Widgets | Material Design 3, customizable theming |

### 3.3 Clean Architecture Layers (Onion Architecture)

```
┌─────────────────────────────────────────┐
│         Infrastructure Layer            │
│  - Axum handlers, middleware            │
│  - SQLx repositories                    │
│  - OpenRouter client                    │
│  - Xendit API client                    │
│  - SpacetimeDB client                   │
├─────────────────────────────────────────┤
│         Application Layer               │
│  - Use cases (services)                 │
│  - DTOs (Request/Response)              │
│  - Application errors                   │
│  - Transaction coordination             │
├─────────────────────────────────────────┤
│           Domain Layer                  │
│  - Entities (User, Job, Escrow)         │
│  - Repository traits                    │
│  - Domain errors                        │
│  - Business rules (Karma, Price Floor)  │
├─────────────────────────────────────────┤
│         Presentation Layer              │
│  - Axum routes (thin adapters)         │
│  - Request validation                   │
│  - Response mappers                     │
└─────────────────────────────────────────┘
```

---

## 4. Project Structure (Detailed File-Level)

### 4.1 Workspace Root Structure

```
siapaja-core/
├── Cargo.toml                    # Workspace root definition
├── Cargo.lock                    # Locked dependencies
├── rust-toolchain.toml           # Rust version pinning
├── .env.example                  # Environment template
├── docker-compose.yml            # Local development stack
├── Dockerfile                    # Production build
├── Makefile                      # Common commands
├── README.md                     # User-facing readme
├── TECHNICAL.md                  # This document
├── WHITEPAPER.md                 # Business logic
├── migrations/                   # SQLx migrations
│   ├── 001_initial_schema.sql
│   ├── 002_add_karma_system.sql
│   └── 003_add_escrow_tables.sql
├── docs/                         # Additional documentation
│   ├── ARCHITECTURE.md
│   ├── API_GUIDELINES.md
│   └── DEPLOYMENT.md
└── crates/                       # Workspace members
    ├── sa-schema/                # #1 Shared domain models & types
    ├── sa-domain/                # #2 Domain logic (entities, traits)
    ├── sa-application/           # #3 Use cases & services
    ├── sa-infrastructure/        # #4 External implementations
    ├── sa-api/                   # #5 Axum HTTP handlers
    ├── sa-spacetimedb/           # #6 SpacetimeDB module
    ├── sa-worker/                # #7 Background jobs
    └── sa-cli/                   # #8 CLI tools & admin
```

### 4.2 Crate Details

#### Crate: `sa-schema` (Shared Types)
**Purpose:** Domain models yang shared across semua crates. Tidak punya dependencies ke infrastructure.

```
crates/sa-schema/
├── Cargo.toml
└── src/
    ├── lib.rs
    ├── models/
    │   ├── mod.rs
    │   ├── user.rs                 # User, UserRole, KarmaScore
    │   ├── job.rs                  # Job, JobStatus, Category
    │   ├── escrow.rs               # Escrow, EscrowStatus (Xendit integration)
    │   ├── payment.rs              # Payment, Withdrawal, Deposit
    │   ├── location.rs             # GeoPoint, Radius
    │   └── dispute.rs              # Dispute, Vote, Jury
    ├── dto/
    │   ├── mod.rs
    │   ├── request/                # API request payloads
    │   │   ├── mod.rs
    │   │   ├── auth.rs
    │   │   ├── job.rs
    │   │   └── payment.rs
    │   └── response/               # API response types
    │       ├── mod.rs
    │       ├── auth.rs
    │       ├── job.rs
    │       └── payment.rs
    ├── enums/
    │   ├── mod.rs
    │   ├── job_category.rs
    │   ├── payment_status.rs
    │   └── dispute_outcome.rs
    └── primitives/
        ├── mod.rs
        ├── phone_number.rs         # Validated phone type
        ├── money.rs                  # IDR amount type
        └── email.rs                # Validated email type
```

#### Crate: `sa-domain` (Domain Logic)
**Purpose:** Pure business logic. Tidak punya external dependencies (no SQLx, no HTTP clients).

```
crates/sa-domain/
├── Cargo.toml
└── src/
    ├── lib.rs
    ├── entities/
    │   ├── mod.rs
    │   ├── user.rs                 # User entity dengan behavior
    │   ├── job.rs                  # Job entity dengan state machine
    │   ├── escrow.rs               # Escrow domain logic
    │   └── karma.rs                # Karma calculation rules
    ├── repositories/
    │   ├── mod.rs
    │   ├── user_repo.rs            # Trait: UserRepository
    │   ├── job_repo.rs             # Trait: JobRepository
    │   ├── escrow_repo.rs          # Trait: EscrowRepository
    │   └── payment_repo.rs         # Trait: PaymentRepository
    ├── services/
    │   ├── mod.rs
    │   ├── pricing_engine.rs       # Price floor calculation
    │   ├── matching_engine.rs      # Job-worker matching logic
    │   ├── karma_engine.rs         # Karma decay & calculation
    │   └── dispute_resolver.rs     # Jury selection & voting
    ├── errors/
    │   ├── mod.rs
    │   └── domain_error.rs         # Domain-level errors
    └── traits/
        ├── mod.rs
        └── identifiable.rs         # Common entity traits
```

#### Crate: `sa-application` (Use Cases)
**Purpose:** Application services yang orchestrate domain logic. Dependencies: sa-domain, sa-schema.

```
crates/sa-application/
├── Cargo.toml
└── src/
    ├── lib.rs
    ├── services/
    │   ├── mod.rs
    │   ├── auth_service.rs         # Login, OTP, JWT generation
    │   ├── job_service.rs          # Create, claim, complete jobs
    │   ├── feed_service.rs         # Personalized feed generation
    │   ├── payment_service.rs      # Deposit, withdrawal, escrow ops
    │   ├── dispute_service.rs      # Dispute lifecycle management
    │   ├── user_service.rs         # Profile, karma, settings
    │   └── ai_service.rs           # OpenRouter integration
    ├── commands/
    │   ├── mod.rs
    │   ├── create_job.rs           # CreateJobCommand handler
    │   ├── claim_job.rs            # ClaimJobCommand handler
    │   ├── complete_job.rs         # CompleteJobCommand handler
    │   └── initiate_dispute.rs     # InitiateDisputeCommand handler
    ├── queries/
    │   ├── mod.rs
    │   ├── get_feed.rs             # GetFeedQuery handler
    │   ├── get_job_details.rs      # GetJobDetailsQuery handler
    │   └── get_user_profile.rs     # GetUserProfileQuery handler
    ├── dto/
    │   ├── mod.rs
    │   └── mappers/                # Domain → DTO conversions
    │       ├── mod.rs
    │       ├── job_mapper.rs
    │       └── user_mapper.rs
    └── errors/
        ├── mod.rs
        └── application_error.rs    # Application-level errors
```

#### Crate: `sa-infrastructure` (External Implementations)
**Purpose:** Implementasi konkret untuk external services. Dependencies: SQLx, reqwest, etc.

```
crates/sa-infrastructure/
├── Cargo.toml
└── src/
    ├── lib.rs
    ├── persistence/
    │   ├── mod.rs
    │   ├── postgres/
    │   │   ├── mod.rs
    │   │   ├── connection.rs       # Connection pooling
    │   │   ├── user_repository.rs  # Impl UserRepository untuk SQLx
    │   │   ├── job_repository.rs   # Impl JobRepository untuk SQLx
    │   │   ├── escrow_repository.rs
    │   │   └── payment_repository.rs
    │   └── spacetimedb/
    │       ├── mod.rs
    │       ├── client.rs           # SpacetimeDB client wrapper
    │       ├── job_cache.rs        # Real-time job state cache
    │       ├── location_cache.rs   # Worker GPS cache
    │       └── subscription.rs     # WebSocket subscription mgmt
    ├── external/
    │   ├── mod.rs
    │   ├── xendit/
    │   │   ├── mod.rs
    │   │   ├── client.rs           # Xendit Escrow API client
    │   │   ├── escrow_api.rs       # Create, release, refund escrow
    │   │   ├── payment_api.rs      # Virtual account, e-wallet
    │   │   └── webhooks.rs         # Xendit webhook handlers
    │   ├── openrouter/
    │   │   ├── mod.rs
    │   │   ├── client.rs           # OpenRouter API client
    │   │   ├── chat_completion.rs  # LLM text completion
    │   │   ├── price_estimation.rs # Price floor AI prompt
    │   │   └── manpower_estimate.rs # Manpower estimation AI
    │   ├── sms/
    │   │   ├── mod.rs
    │   │   ├── twilio_client.rs
    │   │   └── whatsapp_client.rs
    │   └── storage/
    │       ├── mod.rs
    │       └── s3_client.rs        # Buat upload evidence foto
    ├── security/
    │   ├── mod.rs
    │   ├── jwt.rs                  # JWT generation & validation
    │   ├── password.rs             # Argon2 hashing
    │   └── encryption.rs           # AES-256 untuk sensitive data
    └── config/
        ├── mod.rs
        └── app_config.rs           # Environment configuration
```

#### Crate: `sa-api` (HTTP Layer)
**Purpose:** Axum HTTP handlers dan middleware. Thin layer, hanya adapter.

```
crates/sa-api/
├── Cargo.toml
└── src/
    ├── lib.rs
    ├── routes/
    │   ├── mod.rs                  # Router aggregation
    │   ├── health.rs               # Health check endpoints
    │   ├── auth.rs                 # /auth/* routes
    │   ├── users.rs                # /users/* routes
    │   ├── jobs.rs                 # /jobs/* routes
    │   ├── feed.rs                 # /feed/* routes (personalized)
    │   ├── payments.rs             # /payments/* routes
    │   ├── disputes.rs             # /disputes/* routes
    │   └── webhooks.rs             # /webhooks/* (Xendit, GitHub)
    ├── handlers/
    │   ├── mod.rs
    │   ├── auth_handler.rs
    │   ├── job_handler.rs
    │   ├── feed_handler.rs         # Handler untuk personalized feed
    │   ├── payment_handler.rs
    │   └── dispute_handler.rs
    ├── middleware/
    │   ├── mod.rs
    │   ├── auth_middleware.rs      # JWT validation
    │   ├── rate_limit.rs           # Tower governor implementation
    │   ├── logging.rs              # Tracing middleware
    │   └── error_handler.rs        # Global error to response
    ├── extractors/
    │   ├── mod.rs
    │   ├── current_user.rs         # Extract User dari JWT
    │   └── validated_json.rs       # JSON + validation
    ├── openapi/
    │   ├── mod.rs
    │   └── api_doc.rs              # Utoipa OpenAPI spec definition
    └── state.rs                    # Axum shared state (AppState)
```

#### Crate: `sa-spacetimedb` (Real-time Module)
**Purpose:** SpacetimeDB module definition untuk real-time state.

```
crates/sa-spacetimedb/
├── Cargo.toml
└── src/
    ├── lib.rs
    ├── tables/
    │   ├── mod.rs
    │   ├── active_jobs.rs          # #[table] definition untuk jobs
    │   ├── worker_locations.rs     # #[table] untuk GPS tracking
    │   ├── user_sessions.rs        # #[table] untuk online status
    │   └── escrow_states.rs        # #[table] untuk escrow cache
    ├── reducers/
    │   ├── mod.rs
    │   ├── job_reducers.rs         # #[reducer] untuk job ops
    │   ├── location_reducers.rs    # #[reducer] untuk GPS updates
    │   └── escrow_reducers.rs      # #[reducer] untuk escrow state
    ├── subscriptions/
    │   ├── mod.rs
    │   ├── feed_subscription.rs    # Logic untuk feed subscriptions
    │   └── job_subscription.rs     # Logic untuk job-specific subs
    └── client_bindings/
        └── mod.rs                  # Auto-generated client code
```

#### Crate: `sa-worker` (Billing & SA-TEV Module)
**Purpose:** Menghitung penggunaan sumber daya server untuk invoice teknologi.

- **CPU Cycles Tracker:** Menghitung waktu eksekusi setiap `Reducer` (SpacetimeDB) dan `Handler` (Axum).
- **I/O Metering:** Mencatat volume data yang ditulis/dibaca dari PostgreSQL dan SpacetimeDB.
- **SA-TEV Integrator:** Mengonversi metrik teknis menjadi unit SA-TEV berdasarkan koefisien yang ditetapkan Solidarity-ID.
- **Billing Ledger:** Mencatat tagihan harian ke tabel `solidarity_usage_logs` untuk transparansi audit Kopi.

```
crates/sa-worker/
├── Cargo.toml
└── src/
    ├── lib.rs
    ├── billing/
    │   ├── mod.rs
    │   ├── cpu_cycles.rs              # CPU execution time tracking
    │   ├── io_metering.rs             # Data I/O volume tracking
    │   ├── sa_tev_calculator.rs       # SA-TEV unit conversion
    │   └── billing_ledger.rs         # Daily invoice generation
    ├── jobs/
    │   ├── mod.rs
    │   ├── karma_decay.rs          # Monthly karma decay job
    │   ├── escrow_cleanup.rs       # Expired escrow refund
    │   ├── treasury_yield.rs       # Monthly yield calculation
    │   ├── feed_refresh.rs         # Feed personalization refresh
    │   └── dispute_timeout.rs      # Auto-resolve disputes
    ├── scheduler.rs                # Job scheduling (tokio-cron)
    └── worker_pool.rs              # Concurrent job execution
```

#### Crate: `sa-cli` (CLI Tools)
**Purpose:** Admin CLI dan development tools.

```
crates/sa-cli/
├── Cargo.toml
└── src/
    ├── main.rs
    ├── commands/
    │   ├── mod.rs
    │   ├── db.rs                   # Database migrations, reset
    │   ├── user.rs                 # User management (create, ban)
    │   ├── job.rs                  # Job inspection, manual fix
    │   ├── escrow.rs               # Escrow inspection
    │   └── generate.rs             # Code generation (API client)
    └── config.rs                   # CLI configuration
```

### 4.3 Frontend Structure (Flutter Mobile App)

```
frontend/
├── pubspec.yaml
├── analysis_options.yaml
├── lib/
│   ├── main.dart                    # Entry point
│   ├── app.dart                     # Root widget with MaterialApp
│   ├── core/
│   │   ├── config/
│   │   │   ├── app_config.dart      # Environment configuration
│   │   │   └── api_config.dart      # API endpoints
│   │   ├── theme/
│   │   │   ├── app_theme.dart       # Material 3 theme
│   │   │   └── colors.dart          # Color palette
│   │   ├── constants/
│   │   │   └── app_constants.dart   # App-wide constants
│   │   └── utils/
│   │       ├── formatters.dart      # Currency, date formatting
│   │       ├── validators.dart      # Input validation
│   │       └── geo.dart             # Geolocation helpers
│   ├── data/
│   │   ├── models/                  # Data models
│   │   │   ├── user_model.dart
│   │   │   ├── job_model.dart
│   │   │   ├── escrow_model.dart
│   │   │   └── location_model.dart
│   │   ├── repositories/             # Data repositories
│   │   │   ├── auth_repository.dart
│   │   │   ├── job_repository.dart
│   │   │   └── payment_repository.dart
│   │   └── services/
│   │       ├── api_service.dart     # HTTP client
│   │       ├── auth_service.dart
│   │       ├── job_service.dart
│   │       ├── feed_service.dart
│   │       ├── payments_service.dart
│   │       └── spacetimedb_service.dart
│   ├── domain/
│   │   ├── entities/                # Domain entities
│   │   │   ├── user.dart
│   │   │   ├── job.dart
│   │   │   └── escrow.dart
│   │   └── repositories/            # Repository interfaces
│   │       ├── user_repository.dart
│   │       └── job_repository.dart
│   ├── presentation/
│   │   ├── providers/               # Riverpod providers
│   │   │   ├── auth_provider.dart
│   │   │   ├── feed_provider.dart
│   │   │   ├── job_provider.dart
│   │   │   ├── location_provider.dart
│   │   │   └── payments_provider.dart
│   │   ├── screens/
│   │   │   ├── splash/
│   │   │   │   └── splash_screen.dart
│   │   │   ├── auth/
│   │   │   │   ├── login_screen.dart
│   │   │   │   └── otp_screen.dart
│   │   │   ├── home/
│   │   │   │   └── home_screen.dart
│   │   │   ├── feed/
│   │   │   │   ├── feed_screen.dart
│   │   │   │   └── feed_card.dart
│   │   │   ├── job/
│   │   │   │   ├── job_detail_screen.dart
│   │   │   │   ├── create_job_screen.dart
│   │   │   │   └── job_form.dart
│   │   │   ├── profile/
│   │   │   │   ├── profile_screen.dart
│   │   │   │   └── edit_profile_screen.dart
│   │   │   ├── wallet/
│   │   │   │   └── wallet_screen.dart
│   │   │   ├── disputes/
│   │   │   │   └── disputes_screen.dart
│   │   │   └── settings/
│   │   │       └── settings_screen.dart
│   │   └── widgets/
│   │       ├── common/              # Reusable widgets
│   │       │   ├── app_button.dart
│   │       │   ├── app_text_field.dart
│   │       │   ├── app_card.dart
│   │       │   └── loading_indicator.dart
│   │       ├── feed/                # Feed widgets
│   │       │   ├── feed_card.dart
│   │       │   ├── feed_list.dart
│   │       │   └── feed_filter.dart
│   │       ├── job/                 # Job widgets
│   │       │   ├── job_card.dart
│   │       │   ├── job_detail.dart
│   │       │   └── job_form.dart
│   │       └── map/                 # Map widgets
│   │           ├── map_view.dart
│   │           └── worker_marker.dart
│   └── generated/                   # OpenAPI generated code
│       └── api/
│           ├── api_client.dart
│           └── api_models.dart
├── test/
│   ├── unit/                        # Unit tests
│   ├── widget/                      # Widget tests
│   └── integration/                 # Integration tests
├── assets/
│   ├── images/
│   ├── icons/
│   └── fonts/
└── android/                         # Android native config
ios/                                # iOS native config
```

---

## 5. System Architecture

### 5.1 C4 Model: Context Diagram

**External Actors:**
- **Pembuat Job/Jagoan:** User yang bisa switch role (dual-role)
- **Xendit:** Payment gateway + Escrow provider (menampung dana)
- **OpenRouter:** AI/LLM API provider untuk price estimation
- **GitHub:** Webhook provider untuk Build in Public
- **Twilio/WhatsApp:** OTP dan notifikasi SMS

**External Systems:**
- **Xendit Escrow API:** Menampung dana escrow, release ke worker
- **OpenRouter API:** LLM calls untuk price floor dan manpower estimation
- **SpacetimeDB Cloud:** Real-time database hosting (atau self-hosted)
- **PostgreSQL Cloud:** Persistent database (AWS RDS/DO Managed DB)
- **Object Storage:** S3/Spaces untuk evidence foto

### 5.2 C4 Model: Container Diagram

**API Gateway Container (Axum):**
- Stateless, bisa scale horizontal
- JWT validation stateless (tanpa Redis)
- Rate limiting menggunakan tower-governor (in-memory)
- OpenAPI generation dengan Utoipa

**Application Services Container:**
- Business logic orchestration
- Transaction coordination (Saga pattern untuk distributed transactions)
- Event publishing ke SpacetimeDB

**Domain Logic Container:**
- Pure business rules
- Tidak punya external dependencies
- 100% unit testable

**Infrastructure Container:**
- SQLx PostgreSQL repositories
- Xendit API client
- OpenRouter API client
- SpacetimeDB client

**SpacetimeDB Container:**
- Real-time state: active jobs, worker locations, user sessions
- In-memory dengan persistence
- Binary protocol (bukan REST/OpenAPI) dengan community Dart SDK untuk Flutter

**Background Worker Container:**
- Cron jobs: karma decay, escrow cleanup, treasury yield
- Queue processing: job matching notifications

### 5.3 Escrow Flow dengan Pihak Ketiga (Xendit)

**Problem:** v1.0 menggunakan Virtual Ledger internal yang menampung dana. Ini membutuhkan izin OJK sebagai e-money provider.

**Solution v2.0:** Xendit Escrow API menampung dana.

**Flow Create Job:**
1. Pembuat Job create job dengan budget Rp150.000
2. Backend call Xendit Escrow API: `create_escrow(amount: 150000, customer_id, job_id)`
3. Xendit return escrow_id dan payment link/virtual account
4. Pembuat Job bayar ke Xendit (VA/e-wallet/QRIS)
5. Xendit webhook ke backend: payment received, escrow locked
6. Backend update job status: "ESCROW_LOCKED" dan broadcast ke feed

**Flow Complete Job:**
1. Jagoan tap "Selesai"
2. Pembuat Job tap "Konfirmasi"
3. Backend call Xendit: `release_escrow(escrow_id, taskee_id, amount: 148500)`
4. Xendit transfer ke rekening Jagoan (minus 1% solidarity pool ke rekening koperasi)
5. Xendit webhook: release completed
6. Backend update job: "COMPLETED"

**Flow Dispute:**
1. Pembuat Job tap "Dispute" dalam 24 jam
2. Backend call Xendit: `freeze_escrow(escrow_id)`
3. Xendit hold dana (tidak bisa di-release)
4. Jury voting (7 jurors)
5. Setelah voting selesai:
   - Jika Pembuat Job menang: Xendit `refund_escrow(ke Pembuat Job)`
   - Jika Jagoan menang: Xendit `release_escrow(ke Jagoan)`

**Keuntungan:**
- SiapAja tidak pernah menampung dana user
- Tidak perlu izin OJK
- Fraud risk ditanggung Xendit (licensed payment gateway)
- Compliance: Xendit sudah licensed sebagai Payment Gateway oleh BI

### 5.4 Personalized Feed Architecture

**Problem:** Tiap user harus lihat feed yang berbeda berdasarkan:
- Lokasi GPS (radius 5km default)
- Preferensi kategori (history job yang pernah di-claim)
- Pamor score (job premium untuk pamor tinggi)
- Availability (worker hanya lihat job saat online dan available)

**Solution:**

**Feed Generation Pipeline:**
1. **SpacetimeDB Subscription:** Client subscribe ke `jobs:nearby:{lat}:{lng}:{radius}`
2. **Server-side Filtering:** SpacetimeDB reducer filter berdasarkan:
   - Distance from user GPS
   - Job status (POSTED saja)
   - User blocklist (jika pernah dispute)
3. **Personalization Layer:** Application service tambah ranking berdasarkan:
   - Category match dengan history user
   - Price preference (user suka job berapa harga)
   - Pamor compatibility (job premium untuk pamor tinggi)
4. **Client-side Cache:** React cache dengan React Query/SWR

**Feed Algorithm (simplified):**
```
score = (distance_weight * (1 / distance_km)) +
        (category_match * 10) +
        (price_preference_match * 5) +
        (karma_tier_bonus)

Sort by score descending, limit 50
```

**Real-time Updates:**
- WebSocket subscription ke SpacetimeDB
- Push notification untuk high-match jobs (Firebase Cloud Messaging)
- Background refresh setiap 30 detik

### 5.5 Dual-Role Support (Pembuat Job = Jagoan)

**Database Schema:**
- Table `users` tidak punya role fixed
- Field `can_work_as_worker: boolean` (verified via KTP)
- Field `can_post_as_customer: boolean` (default true)

**Switching Logic:**
- User bisa switch role dalam satu session
- State disimpan di frontend (React Context)
- Backend tidak peduli role, cek permission per-action:
  - `create_job`: cek `can_post_as_customer`
  - `claim_job`: cek `can_work_as_worker` dan `karma >= threshold`

**UI Pattern:**
- Toggle switch di header: "Mode Pembuat Job" / "Mode Jagoan"
  - Pembuat Job mode: lihat "My Posted Jobs" + "Nearby Jagoans"
  - Jagoan mode: lihat "Available Jobs" + "My Active Jobs"

---

## 6. Data Architecture

### 6.1 Database Schema (PostgreSQL)

**Core Tables:**

```sql
-- Users (dual-role support)
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    phone_number VARCHAR(20) UNIQUE NOT NULL,
    phone_hash VARCHAR(64) NOT NULL, -- hashed untuk privacy
    email VARCHAR(255),
    full_name VARCHAR(255) NOT NULL,
    ktp_hash VARCHAR(64), -- hash saja, bukan nomor asli
    can_work_as_worker BOOLEAN DEFAULT FALSE,
    can_post_as_customer BOOLEAN DEFAULT TRUE,
    karma_score INTEGER DEFAULT 100,
    voting_power INTEGER DEFAULT 1,
    blue_check_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Jobs (demand feed)
CREATE TABLE jobs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id UUID NOT NULL REFERENCES users(id),
    worker_id UUID REFERENCES users(id), -- nullable sampai di-claim
    title VARCHAR(255) NOT NULL,
    description TEXT,
    category job_category NOT NULL,
    budget_idr INTEGER NOT NULL CHECK (budget_idr > 0),
    gps_lat DECIMAL(10, 8) NOT NULL,
    gps_lng DECIMAL(11, 8) NOT NULL,
    location_geog GEOGRAPHY(POINT), -- PostGIS untuk spatial queries
    status job_status DEFAULT 'POSTED',
    escrow_id VARCHAR(255), -- Xendit escrow ID (external)
    escrow_status escrow_status,
    ai_price_floor INTEGER, -- minimum harga dari AI
    ai_workers_required INTEGER DEFAULT 1,
    ai_risk_level risk_level,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    claimed_at TIMESTAMPTZ,
    completed_at TIMESTAMPTZ,
    expires_at TIMESTAMPTZ DEFAULT (NOW() + INTERVAL '24 hours')
);

-- Escrow References (Xendit integration)
CREATE TABLE escrow_refs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_id UUID UNIQUE NOT NULL REFERENCES jobs(id),
    xendit_escrow_id VARCHAR(255) NOT NULL, -- ID dari Xendit
    amount_idr INTEGER NOT NULL,
    customer_id UUID NOT NULL REFERENCES users(id),
    worker_id UUID REFERENCES users(id),
    status escrow_status DEFAULT 'PENDING_PAYMENT',
    created_at TIMESTAMPTZ DEFAULT NOW(),
    locked_at TIMESTAMPTZ,
    released_at TIMESTAMPTZ,
    refunded_at TIMESTAMPTZ,
    xendit_payload JSONB -- raw response dari Xendit untuk audit
);

-- Immutable Ledger (Merkle-Chained)
CREATE TABLE ledger_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    previous_hash VARCHAR(64), -- Link ke entry sebelumnya (NULL untuk genesis)
    entry_hash VARCHAR(64) NOT NULL, -- SHA-256(data + previous_hash)
    payload JSONB NOT NULL, -- Detail mutasi (uang atau pamor)
    entry_type VARCHAR(32) NOT NULL, -- 'financial' | 'pamor'
    created_at TIMESTAMPTZ DEFAULT NOW(),
    created_by UUID, -- User yang trigger event (nullable untuk system events)
    CONSTRAINT valid_hash_chain CHECK (
        (previous_hash IS NULL) OR 
        (previous_hash ~ '^[a-f0-9]{64}$')
    )
);

-- Index untuk audit trail queries
CREATE INDEX idx_ledger_entries_type ON ledger_entries(entry_type);
CREATE INDEX idx_ledger_entries_created_at ON ledger_entries(created_at DESC);
CREATE INDEX idx_ledger_entries_created_by ON ledger_entries(created_by);

-- Transactions (financial records, deprecated - use ledger_entries instead)
CREATE TABLE transactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    escrow_ref_id UUID REFERENCES escrow_refs(id),
    type transaction_type NOT NULL,
    amount_idr INTEGER NOT NULL, -- positive untuk credit, negative untuk debit
    balance_after_idr INTEGER NOT NULL,
    metadata JSONB,
    xendit_reference VARCHAR(255), -- reference ID dari Xendit
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Job Categories (untuk personalization)
CREATE TABLE user_category_preferences (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    category job_category NOT NULL,
    preference_score INTEGER DEFAULT 0, -- naik tiap kali user claim job kategori ini
    created_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(user_id, category)
);

-- Disputes
CREATE TABLE disputes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_id UUID UNIQUE NOT NULL REFERENCES jobs(id),
    escrow_ref_id UUID NOT NULL REFERENCES escrow_refs(id),
    initiator_id UUID NOT NULL REFERENCES users(id), -- yang buka dispute
    reason TEXT NOT NULL,
    evidence_urls TEXT[], -- array of S3 URLs
    status dispute_status DEFAULT 'OPEN',
    outcome dispute_outcome,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    resolved_at TIMESTAMPTZ,
    expires_at TIMESTAMPTZ DEFAULT (NOW() + INTERVAL '7 days')
);

-- Jury Votes
CREATE TABLE jury_votes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    dispute_id UUID NOT NULL REFERENCES disputes(id),
    juror_id UUID NOT NULL REFERENCES users(id),
    vote dispute_outcome NOT NULL, -- CUSTOMER_WIN atau WORKER_WIN
    voted_at TIMESTAMPTZ DEFAULT NOW(),
    confidence INTEGER CHECK (confidence BETWEEN 1 AND 5),
    UNIQUE(dispute_id, juror_id)
);

-- Pamor History
CREATE TABLE karma_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    delta INTEGER NOT NULL,
    reason karma_reason NOT NULL,
    reference_type VARCHAR(50),
    reference_id UUID,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Koperasi Open Ledger (Transparency Table)
CREATE TABLE community_treasury_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    entry_type VARCHAR(50) NOT NULL, 
    amount_idr BIGINT NOT NULL,
    founder_cut_idr BIGINT NOT NULL, -- Jatah lu & tim inti
    community_cut_idr BIGINT NOT NULL, -- Jatah kas koperasi
    description TEXT,
    balance_snapshot BIGINT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Solidarity-ID / Technology Service Wallet (Private Tracking)
CREATE TABLE technology_service_wallets (
    entity_name VARCHAR(50) DEFAULT 'SOLIDARITY_ID',
    total_earned_idr BIGINT DEFAULT 0,
    withdrawable_balance_idr BIGINT DEFAULT 0,
    allocation_category VARCHAR(20) DEFAULT 'INFRASTRUCTURE' -- INFRASTRUCTURE, R&D, COMPLIANCE
);

-- Future Web3 Settlement Table
CREATE TABLE blockchain_settlement_batches (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    batch_merkle_root VARCHAR(64) NOT NULL,
    solana_tx_id VARCHAR(128), -- Nullable sampai roadmap aktif
    status VARCHAR(20) DEFAULT 'OFF_CHAIN_ONLY',
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Solidarity Pool (Asuransi Komunitas)
CREATE TABLE solidarity_claims (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    amount_requested INTEGER NOT NULL,
    status VARCHAR(20) DEFAULT 'VOTING_PENDING', -- Juri vote buat acc bantuan
    evidence_url TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 6.10 Immutable Ledger Implementation

### 6.10.1 Hash Chain Architecture

Ledger entries menggunakan **Merkle Log** technique (mirip Git atau Certificate Transparency) untuk memastikan integritas data tanpa perlu full blockchain yang lambat.

**Prinsip Dasar:**
```
Entry 0 (Genesis): hash₀ = SHA-256(payload₀)
Entry 1: hash₁ = SHA-256(payload₁ + hash₀)
Entry 2: hash₂ = SHA-256(payload₂ + hash₁)
...
Entry N: hashₙ = SHA-256(payloadₙ + hashₙ₋₁)
```

**Properties:**
- **Append-Only**: Tidak ada UPDATE atau DELETE. Semua mutasi adalah INSERT baru.
- **Cryptographic Link**: Setiap entry terikat dengan entry sebelumnya melalui hash.
- **Tamper-Evident**: Mengubah 1 entry akan memutus rantai hash semua entry setelahnya.
- **Efficient Audit**: Verifikasi integritas cukup O(log n) dengan Merkle proof.

### 6.10.2 Rust Implementation (sa-infrastructure)

**Module:** `crates/sa-infrastructure/src/persistence/postgres/ledger_repository.rs`

**Struktur Data:**
```rust
pub struct LedgerEntry {
    pub id: Uuid,
    pub previous_hash: Option<String>, // None untuk genesis
    pub entry_hash: String, // SHA-256 hex
    pub payload: LedgerPayload,
    pub entry_type: LedgerEntryType,
    pub created_at: chrono::DateTime<Utc>,
    pub created_by: Option<Uuid>,
}

pub enum LedgerEntryType {
    Financial, // Mutasi Rupiah
    Pamor,     // Mutasi reputasi
}

pub struct LedgerPayload {
    // Untuk financial
    pub amount_idr: i64,
    pub balance_before: i64,
    pub balance_after: i64,
    pub reference_id: Uuid, // job_id, escrow_id, dll
    pub description: String,
    
    // Untuk pamor
    pub pamor_delta: i32,
    pub pamor_before: i32,
    pub pamor_after: i32,
    pub reason: String,
}
```

**Repository Interface:**
```rust
pub trait LedgerRepository: Send + Sync {
    /// Append entry ke ledger dengan hash chain validation
    async fn append(&self, payload: LedgerPayload, entry_type: LedgerEntryType, created_by: Option<Uuid>) -> Result<LedgerEntry, LedgerError>;
    
    /// Verifikasi integritas chain dari entry tertentu ke belakang
    async fn verify_chain_from(&self, entry_id: Uuid) -> Result<bool, LedgerError>;
    
    /// Ambil entry berdasarkan hash (untuk audit publik)
    async fn find_by_hash(&self, hash: &str) -> Result<Option<LedgerEntry>, LedgerError>;
    
    /// Export ledger entries dengan Merkle proof (untuk eksternal audit)
    async fn export_with_proof(&self, from: chrono::DateTime<Utc>, to: chrono::DateTime<Utc>) -> Result<LedgerExport, LedgerError>;
}
```

**Hash Calculation Logic:**
```rust
impl LedgerRepository {
    fn calculate_entry_hash(previous_hash: Option<&str>, payload: &LedgerPayload) -> String {
        let mut hasher = Sha256::new();
        
        // Hash previous entry (jika ada)
        if let Some(prev) = previous_hash {
            hasher.update(prev.as_bytes());
        }
        
        // Hash payload dengan canonical JSON serialization
        let payload_json = serde_json::to_string(payload)
            .expect("Payload must be serializable");
        hasher.update(payload_json.as_bytes());
        
        let result = hasher.finalize();
        hex::encode(result) // Return 64-char hex string
    }
    
    async fn get_latest_hash(&self) -> Result<Option<String>, LedgerError> {
        // Query: SELECT entry_hash FROM ledger_entries ORDER BY created_at DESC LIMIT 1
        // Return hash dari entry terakhir untuk jadi previous_hash entry baru
    }
}
```

**Transaction Integration (Saga Pattern):**
```rust
// Di sa-application/src/services/payment_service.rs
pub async fn release_escrow_payment(
    &self,
    escrow_id: Uuid,
    taskee_id: Uuid,
    amount: i64,
) -> Result<(), PaymentError> {
    // 1. Call Xendit API untuk release escrow
    let xendit_result = self.xendit_client.release_escrow(escrow_id).await?;
    
    // 2. Mulai database transaction (SQLx)
    let mut tx = self.db.begin().await?;
    
    // 3. Get latest hash dari ledger
    let latest_hash = self.ledger_repo.get_latest_hash().await?;
    
    // 4. Append financial entry ke ledger
    let payload = LedgerPayload {
        amount_idr: amount,
        balance_before: current_balance,
        balance_after: current_balance + amount,
        reference_id: escrow_id,
        description: format!("Escrow release for job {}", escrow_id),
        pamor_delta: 0,
        pamor_before: 0,
        pamor_after: 0,
        reason: String::new(),
    };
    
    let entry = self.ledger_repo.append_with_tx(&mut tx, payload, LedgerEntryType::Financial, Some(taskee_id)).await?;
    
    // 5. Update user balance (harus konsisten dengan ledger)
    self.user_repo.update_balance_with_tx(&mut tx, taskee_id, amount).await?;
    
    // 6. Commit transaction
    tx.commit().await?;
    
    // 7. Return hash untuk UI display
    Ok(entry.entry_hash)
}
```

### 6.10.3 Pamor Ledger Integration

**Module:** `crates/sa-application/src/services/karma_engine.rs`

```rust
pub async fn award_pamor(
    &self,
    user_id: Uuid,
    delta: i32,
    reason: PamorReason,
    reference_id: Uuid,
) -> Result<(), KarmaError> {
    // 1. Get current pamor score
    let current_pamor = self.user_repo.get_pamor(user_id).await?;
    let new_pamor = current_pamor.saturating_add(delta);
    
    // 2. Append to immutable ledger
    let payload = LedgerPayload {
        amount_idr: 0,
        balance_before: 0,
        balance_after: 0,
        reference_id,
        description: format!("Pamor {} from {}", delta, reason),
        pamor_delta: delta,
        pamor_before: current_pamor,
        pamor_after: new_pamor,
        reason: reason.to_string(),
    };
    
    let entry = self.ledger_repo.append(payload, LedgerEntryType::Pamor, Some(user_id)).await?;
    
    // 3. Update user pamor (read-only di repo, update via reducer)
    self.user_repo.update_pamor(user_id, new_pamor).await?;
    
    // 4. Log hash untuk audit trail
    tracing::info!(
        user_id = %user_id,
        pamor_delta = delta,
        entry_hash = %entry.entry_hash,
        "Pamor updated with immutable ledger entry"
    );
    
    Ok(())
}
```

### 6.10.4 Audit & Verification API

**Endpoint:** `GET /api/v1/ledger/verify/{hash}`

```rust
// Di crates/sa-api/src/routes/ledger.rs
pub async fn verify_entry(
    Path(hash): Path<String>,
    State(state): State<AppState>,
) -> Result<Json<LedgerVerificationResponse>, ApiError> {
    // 1. Find entry by hash
    let entry = state.ledger_repo.find_by_hash(&hash).await?
        .ok_or(ApiError::NotFound)?;
    
    // 2. Verify chain integrity dari entry ini ke genesis
    let is_valid = state.ledger_repo.verify_chain_from(entry.id).await?;
    
    // 3. Build response dengan Merkle proof
    let proof = state.ledger_repo.build_merkle_proof(entry.id).await?;
    
    Ok(Json(LedgerVerificationResponse {
        entry,
        chain_valid: is_valid,
        merkle_proof: proof,
        verified_at: Utc::now(),
    }))
}
```

**Response Example:**
```json
{
  "entry": {
    "id": "uuid",
    "previous_hash": "0xabc...",
    "entry_hash": "0x7f83b1657ff1fc53b92dc18148a1d65dfa1350f...",
    "payload": {
      "amount_idr": 148500,
      "balance_before": 0,
      "balance_after": 148500,
      "reference_id": "job-uuid",
      "description": "Escrow release for job #123"
    },
    "entry_type": "financial",
    "created_at": "2026-03-16T10:30:00Z"
  },
  "chain_valid": true,
  "merkle_proof": {
    "leaf_hash": "0x7f83...",
    "root_hash": "0xabc...",
    "proof_path": ["0x123...", "0x456..."]
  },
  "verified_at": "2026-03-16T12:00:00Z"
}
```

### 6.10.5 Performance Optimization

**Problem:** Hash chain calculation bisa jadi bottleneck kalau setiap insert harus query entry terakhir.

**Solution:**
1. **In-Memory Latest Hash Cache**: Simpan latest hash di SpacetimeDB singleton state.
2. **Batch Inserts**: Untuk operasi bulk (misal: monthly karma decay), gunakan batch insert dengan pre-calculated hashes.
3. **Snapshot Checkpoints**: Setiap 10,000 entries, buat snapshot dengan Merkle root untuk faster verification.

**SpacetimeDB Integration:**
```rust
// Di crates/sa-spacetimedb/src/tables/ledger_state.rs
#[table]
pub struct LedgerState {
    #[primarykey]
    pub id: u64, // Always 1 (singleton)
    pub latest_hash: String,
    pub latest_entry_id: Uuid,
    pub entry_count: u64,
    pub last_checkpoint_merkle_root: Option<String>,
    pub last_checkpoint_at: Timestamp,
}

#[reducer]
pub fn update_latest_hash(ctx: &ReducerContext, new_hash: String, entry_id: Uuid) {
    // Update singleton state
    let mut state = ctx.db.singleton_ledger_state();
    state.latest_hash = new_hash;
    state.latest_entry_id = entry_id;
    state.entry_count += 1;
}
```

### 6.10.6 Migration Strategy

**Phase 1 (Current):** Hash chain di PostgreSQL (`ledger_entries` table)
- Cepat, efficient, cukup untuk audit internal
- Hash calculation di Rust backend

**Phase 4 (Roadmap):** Blockchain Settlement
- Periodic batch export ke Solana (setiap 24 jam atau setiap 1000 entries)
- Merkle root di-commit ke Solana transaction
- Jagoan bisa verify data mereka on-chain

**Future-Proof Design:**
```rust
// Table sudah siap untuk blockchain integration
CREATE TABLE blockchain_settlement_batches (
    id UUID PRIMARY KEY,
    batch_merkle_root VARCHAR(64) NOT NULL,
    solana_tx_id VARCHAR(128), -- Nullable sampai Phase 4
    status VARCHAR(20) DEFAULT 'OFF_CHAIN_ONLY',
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 6.11 SpacetimeDB Schema (Real-time State)

```rust
// Active Jobs (real-time feed)
#[table(name = active_jobs, public)]
pub struct ActiveJob {
    #[primary_key]
    pub job_id: Uuid,
    pub customer_id: Uuid,
    pub worker_id: Option<Uuid>,
    pub title: String,
    pub budget_idr: i32,
    pub gps_lat: f64,
    pub gps_lng: f64,
    pub category: String,
    pub status: String, // POSTED, CLAIMED, IN_PROGRESS
    pub created_at: u64, // timestamp ms
    pub expires_at: u64,
    pub required_workers: i32,
    pub joined_workers: Vec<Uuid>, // untuk squad jobs
    pub karma_tier: i32, // minimum karma untuk lihat job ini
}

// Worker Locations (GPS tracking)
#[table(name = worker_locations, public)]
pub struct WorkerLocation {
    #[primary_key]
    pub worker_id: Uuid,
    pub gps_lat: f64,
    pub gps_lng: f64,
    pub accuracy_meters: f32,
    pub last_updated: u64,
    pub karma_score: i32,
    pub current_job_id: Option<Uuid>,
    pub is_online: bool,
    pub available_until: u64, // timestamp sampai kapan available
}

// User Sessions (online status)
#[table(name = user_sessions, public)]
pub struct UserSession {
    #[primary_key]
    pub user_id: Uuid,
    pub connection_id: String,
    pub is_worker_mode: bool, // dual-role: lagi mode apa?
    pub subscribed_radius_km: i32,
    pub subscribed_categories: Vec<String>,
    pub last_activity: u64,
}

// Feed Personalization Cache
#[table(name = feed_personalization, public)]
pub struct FeedPersonalization {
    #[primary_key]
    pub user_id: Uuid,
    pub preferred_categories: Vec<String>,
    pub avg_job_budget: i32, -- rata-rata budget job yang pernah di-claim
    pub price_sensitivity: f32, -- 0-1, lower = suka job mahal
    pub last_calculated: u64,
}
```

---

## 7. API Design Specification

### 7.1 OpenAPI Generation dengan Utoipa

**Setup:**
- Menggunakan `utoipa` dengan feature `axum_extras`
- Menggunakan `utoipa-swagger-ui` untuk Swagger UI
- Compile-time generation - tidak ada runtime overhead

**Structure:**
- Setiap handler di-annotate dengan `#[utoipa::path(...)]`
- Request/Response DTOs implement `ToSchema`
- Central `ApiDoc` struct mengaggregate semua routes dan schemas

### 7.2 Endpoint Categories

**Auth Endpoints:**
- `POST /auth/otp/request` - Request OTP
- `POST /auth/otp/verify` - Verifikasi OTP, return JWT
- `POST /auth/refresh` - Refresh access token
- `POST /auth/logout` - Revoke token

**User Endpoints:**
- `GET /users/me` - Get current user profile
- `PATCH /users/me` - Update profile
- `GET /users/me/balance` - Get balance dari Xendit
- `POST /users/me/switch-mode` - Switch customer/worker mode

**Feed Endpoints (Personalized):**
- `GET /feed` - Get personalized feed (query: lat, lng, radius, category)
- `GET /feed/nearby-workers` - Lihat worker online di sekitar (customer mode)
- `POST /feed/subscribe` - Subscribe ke real-time updates

**Job Endpoints:**
- `POST /jobs` - Create job (initiates Xendit escrow)
- `GET /jobs/:id` - Get job detail
- `POST /jobs/:id/claim` - Claim job sebagai worker
- `POST /jobs/:id/complete` - Mark complete (worker)
- `POST /jobs/:id/confirm` - Confirm completion (customer)
- `POST /jobs/:id/cancel` - Cancel job (customer, sebelum di-claim)

**Payment Endpoints:**
- `POST /payments/deposit` - Create deposit request (Xendit VA/e-wallet)
- `POST /payments/withdraw` - Request withdrawal ke rekening bank
- `GET /payments/history` - Transaction history
- `GET /payments/escrow/:job_id` - Escrow status

**Dispute Endpoints:**
- `POST /disputes` - Initiate dispute
- `GET /disputes/:id` - Get dispute detail
- `POST /disputes/:id/vote` - Jury submit vote
- `GET /disputes/:id/evidence` - Get evidence URLs

**Webhook Endpoints (Public):**
- `POST /webhooks/xendit` - Xendit payment callbacks
- `POST /webhooks/github` - GitHub issue/PR events

### 7.3 Error Handling

**Error Response Format:**
```json
{
  "error": {
    "code": "INSUFFICIENT_ESCROW_BALANCE",
    "message": "Saldo escrow tidak mencukupi",
    "details": {
      "required": 150000,
      "current": 75000,
      "xendit_escrow_id": "escrow_abc123"
    },
    "request_id": "req_uuid",
    "timestamp": "2026-03-15T10:30:00Z"
  }
}
```

**Error Categories:**
- `400`: Validation error, business logic violation
- `401`: Unauthorized (JWT invalid/expired)
- `403`: Forbidden (karma too low, not verified, wrong role)
- `404`: Resource not found
- `409`: Conflict (job sudah di-claim, race condition)
- `422`: Xendit API error (escrow creation failed)
- `429`: Rate limit exceeded
- `500`: Internal server error

### 7.4 Rate Limiting (Tower Governor)

**Implementation:**
- Menggunakan `tower-governor` crate (in-memory, no Redis)
- Configurable per-endpoint
- Key: user_id (authenticated) atau IP (anonymous)

**Limits:**
- Anonymous: 10 req/minute
- Authenticated: 60 req/minute
- High Pamor: 300 req/minute
- Blue Check: 600 req/minute

**Specific Endpoints:**
- `POST /jobs`: 5 req/minute (anti-spam)
- `POST /jobs/:id/claim`: 10 req/minute (anti-bot)
- `POST /auth/otp/request`: 3 req/minute per phone

---

## 8. AI Integration (OpenRouter)

> **📋 Source of Truth:** Untuk detail lengkap AI pipeline, LLM models, prompt templates, dan JSON schemas, lihat [AI-SPECS.md](./docs/AI-SPECS.md)

### 8.1 OpenRouter Architecture

**Why OpenRouter:**
- Single API key untuk access 200+ LLM models
- Fallback otomatis jika satu model down
- Cost optimization (pilih model berdasarkan price/performance)
- Tidak perlu manage multiple API keys (OpenAI, Anthropic, etc)

**Implementation:**
- HTTP client menggunakan `reqwest` dengan connection pooling
- Async calls dengan timeout 10 detik
- Retry logic: 3 attempts dengan exponential backoff
- Caching: identical prompts di-cache 1 jam (SpacetimeDB)

### 8.2 AI Use Cases

**1. Price Floor Estimation**
- **Input:** Job title, description, category, location
- **Prompt:** "Estimate fair price for [job] in [location]. Consider: labor cost, difficulty, market rate. Return JSON: {min_price, recommended_price, confidence}"
- **Model:** Claude 3 Haiku (fast, cheap, good enough)
- **Fallback:** GPT-3.5 Turbo

**2. Manpower Estimation**
- **Input:** Job description + image (optional, URL ke S3)
- **Prompt:** "Analyze if this job needs multiple workers. Consider: weight, stairs, tools. Return: {workers_needed: 1-5, reasoning}"
- **Note:** Image analysis via text description (worker upload foto, AI analyze via vision model jika available, atau manual review)

**3. Content Moderation**
- **Input:** Job title, description
- **Prompt:** "Check if this content violates safety guidelines (illegal, dangerous, scam). Return: {approved: bool, reason: string?}"
- **Model:** Claude 3 Haiku

**4. Feed Personalization (Batch)**
- **Input:** User history (categories, budgets, locations)
- **Prompt:** "Given user history, predict preferred job categories and price range. Return: {categories: [], avg_budget: number}"
- **Frequency:** Daily batch untuk semua active users
- **Cache:** Store di SpacetimeDB `feed_personalization` table

### 8.3 Error Handling AI

- Timeout: fallback ke rule-based heuristic
- API error: log dan retry, jika gagal gunakan default values
- Invalid JSON response: retry dengan stricter prompt

---

## 9. Real-Time System (SpacetimeDB)

### 9.1 SpacetimeDB Protocol Architecture

**Note Penting:** SpacetimeDB menggunakan binary protocol sendiri (bukan REST/OpenAPI). Flutter frontend terhubung ke SpacetimeDB menggunakan community SpacetimeDB Dart SDK yang production-ready.

**Connection Model:**
- Persistent WebSocket per user session menggunakan SpacetimeDB protocol
- Auto-reconnect dengan exponential backoff (max 30s)
- Heartbeat 30 detik
- Binary serialization untuk efisiensi bandwidth

**Subscriptions:**
- `jobs:nearby:{lat}:{lng}:{radius}` - Job feed personalized
- `user:{user_id}:notifications` - Personal notifications
- `job:{job_id}` - Specific job updates (setelah claim)

### 9.2 State Synchronization

**Optimistic Updates:**
- Frontend update UI sebelum server confirm
- Rollback jika server reject
- Conflict resolution: last-write-wins

**Eventual Consistency:**
- PostgreSQL: source of truth untuk financial data
- SpacetimeDB: cache untuk real-time queries
- Sync: 5 detik untuk non-critical, immediate untuk financial

---

## 10. Security Architecture

### 10.1 Defense Layers

**Layer 1: Network**
- TLS 1.3 mandatory
- Certificate pinning di Flutter App

**Layer 2: Application**
- Input validation: strict JSON schema
- SQL injection: SQLx compile-time checked (zero runtime SQL strings)
- XSS: CSP headers, output encoding

**Layer 3: Authentication**
- JWT RS256 (asymmetric signing)
- Token binding ke device fingerprint
- OTP brute force: 5 attempts lockout 15 menit

**Layer 4: Data Protection**
- Encryption at rest: PostgreSQL LUKS
- Sensitive fields: AES-256-GCM (phone, KTP hash)
- Key management: HashiCorp Vault

**Layer 5: Financial Security**
- Xendit escrow: dana tidak pernah touch SiapAja systems
- Webhook verification: Xendit signature validation
- Idempotency keys: prevent double-charge

### 10.2 Privacy Design

- KTP: hanya hash, bukan nomor asli
- GPS history: tidak disimpan (hanya last location)
- Dispute evidence: auto-delete 90 hari
- User identities: masked di jury voting

---

## 11. Infrastructure & Deployment

### 11.1 Resource Requirements

**Minimal Production:**
- 1x VPS 2 vCPU, 4GB RAM: Axum + PostgreSQL + SpacetimeDB
- Cost: ~$20-40/bulan
- Handle: 10.000 concurrent users

**Recommended Production:**
- 2x VPS 4 vCPU, 8GB RAM: Axum API Gateway (load balanced)
- 1x Managed PostgreSQL: 2 vCPU, 4GB RAM
- SpacetimeDB Cloud: sesuai usage
- Cost: ~$100-150/bulan
- Handle: 100.000+ concurrent users

### 11.2 Monitoring

**Metrics (Prometheus):**
- Request latency (p50, p95, p99)
- Error rates
- Active WebSocket connections
- Xendit API latency
- OpenRouter API latency

**Logging (Structured JSON):**
- Request ID correlation
- User ID, IP, user agent
- Performance timings
- Security events

### 11.3 Backup & DR

- PostgreSQL: Continuous WAL archiving
- Daily full backups, retained 30 hari
- SpacetimeDB: snapshot 6 jam, retained 7 hari
- RPO: < 5 menit, RTO: < 30 menit

---

## 12. Glossary

| Term | Definition |
|------|------------|
| **Xendit Escrow** | Third-party escrow service yang menampung dana. SiapAja tidak perlu izin OJK. |
| **OpenRouter** | Unified API untuk 200+ LLM models. |
| **SpacetimeDB** | In-memory database dengan persistence, real-time subscriptions, dan binary protocol. Flutter client menggunakan community Dart SDK. |
| **OpenAPI/Swagger** | API specification untuk REST endpoints (Axum → Flutter). Bukan untuk SpacetimeDB. |
| **Dual-Role** | User bisa switch antara customer dan worker dalam satu akun. |
| **Personalized Feed** | Feed job yang berbeda untuk setiap user berdasarkan preferensi. |
| **Pamor** | Reputation score user. |
| **Price Floor** | Harga minimum yang ditentukan AI. |
| **Squad Formation** | Job yang butuh multiple workers. |
| **Clean Architecture** | Onion/Hexagonal architecture dengan domain di center. |

---

## 13. Appendices

### Appendix A: Technology Versions
- Rust: 1.75+ (Edition 2021)
- Axum: 0.7+
- SQLx: 0.7+
- Utoipa: 4.2+
- PostgreSQL: 15+
- SpacetimeDB: 1.0+
- Flutter: 3.0+
- Dart: 3.0+
- Riverpod: 2.0+

### Appendix B: External APIs
- Xendit: Escrow API v2
- OpenRouter: Chat Completions API
- Twilio: SMS API

### Appendix C: Related Documents
- `README.md` - Quick start
- `WHITEPAPER.md` - Business logic
- `API_REFERENCE.md` - Auto-generated OpenAPI
- `DEPLOYMENT.md` - Infrastructure setup

---

**Document Control:**
- **Author:** Core Technical Team
- **Version:** 2.1
- **Date:** 2026-03-16
- **Next Review:** 2026-06-16

**Changelog:**
| Version | Date | Changes |
|---------|------|---------|
| 2.1 | 2026-03-16 | Added Immutable Ledger implementation with Merkle-chained hash chain, audit API, and SpacetimeDB integration |
| 2.0 | 2026-03-15 | Third-party escrow, OpenRouter AI, personalized feed, dual-role, workspace structure |
| 1.0 | 2026-03-01 | Initial specification |

---

*End of Document*
