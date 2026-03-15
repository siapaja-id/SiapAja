# Technical Specification: SiapAja.id Platform

---

## 1. Executive Summary

SiapAja.id adalah platform gig economy real-time yang dibangun dengan arsitektur modern berbasis Rust workspace monorepo, React PWA, dan SpacetimeDB. Dokumen ini menjelaskan secara komprehensif arsitektur sistem, komponen teknis, alur data, dan spesifikasi implementasi tanpa menyertakan kode sumber.

**Target Pembaca:**
- Software Architects evaluating the platform
- Backend Engineers (Rust)
- Frontend Engineers (React/TypeScript)
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
2. **Real-time state synchronization** menggunakan SpacetimeDB (in-memory dengan persistence)
3. **Ultra-efficient backend** dalam Rust workspace dengan clean architecture
4. **Progressive Web App** dengan personalized feed recommendation
5. **Dual-role architecture** - satu user bisa switch antara customer dan worker

### 2.3 Core Value Proposition
- **0% komisi** untuk transaksi di bawah Rp500.000
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
| Database (Real-time) | SpacetimeDB | In-memory speed dengan persistence, native WebSocket subscriptions, menggantikan Redis |
| AI Integration | OpenRouter API | Access 200+ LLM models (Claude, GPT, Llama) dengan single API key, text-only calls |
| API Documentation | Utoipa | Compile-time OpenAPI generation, Swagger UI integration dengan Axum |
| Frontend Framework | React 18+ | Concurrent features, Suspense untuk async state |
| Build Tool | Vite + SWC | Rust-based compiler, instant HMR |
| Styling | Tailwind CSS | Utility-first, minimal CSS bundle size |

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

#### Crate: `sa-worker` (Background Jobs)
**Purpose:** Cron jobs dan background processing.

```
crates/sa-worker/
├── Cargo.toml
└── src/
    ├── lib.rs
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

### 4.3 Frontend Structure (React PWA)

```
frontend/
├── package.json
├── vite.config.ts
├── tsconfig.json
├── tailwind.config.js
├── index.html
├── public/
│   ├── manifest.json             # PWA manifest
│   ├── sw.js                     # Service worker
│   └── icons/
└── src/
    ├── main.tsx                  # Entry point
    ├── App.tsx                   # Root component
    ├── routes.tsx                # React Router config
    ├── components/
    │   ├── common/               # Reusable UI components
    │   │   ├── Button.tsx
    │   │   ├── Input.tsx
    │   │   └── Card.tsx
    │   ├── feed/                 # Feed-specific components
    │   │   ├── FeedCard.tsx
    │   │   ├── FeedList.tsx
    │   │   └── FeedFilter.tsx
    │   ├── job/                  # Job components
    │   │   ├── JobCard.tsx
    │   │   ├── JobDetail.tsx
    │   │   └── JobForm.tsx
    │   ├── map/                  # Map components
    │   │   ├── MapView.tsx
    │   │   └── WorkerMarker.tsx
    │   └── layout/
    │       ├── Header.tsx
    │       ├── BottomNav.tsx
    │       └── Sidebar.tsx
    ├── pages/
    │   ├── Home.tsx              # Personalized feed page
    │   ├── JobDetail.tsx
    │   ├── CreateJob.tsx
    │   ├── Profile.tsx
    │   ├── Wallet.tsx
    │   ├── Disputes.tsx
    │   └── Settings.tsx
    ├── hooks/
    │   ├── useAuth.ts            # Authentication hook
    │   ├── useFeed.ts            # Personalized feed hook
    │   ├── useJobs.ts            # Job management hook
    │   ├── useLocation.ts        # GPS tracking hook
    │   ├── useSpacetimeDB.ts     # Real-time subscription hook
    │   └── usePayments.ts        # Payment operations hook
    ├── services/
    │   ├── api.ts                # Generated API client (OpenAPI)
    │   ├── auth.ts
    │   ├── jobs.ts
    │   ├── feed.ts               # Feed personalization API
    │   ├── payments.ts
    │   └── spacetimedb.ts        # SpacetimeDB SDK wrapper
    ├── stores/
    │   ├── authStore.ts          # Zustand auth state
    │   ├── feedStore.ts          # Feed personalization state
    │   ├── jobStore.ts           # Active jobs cache
    │   └── userStore.ts          # User profile state
    ├── types/
    │   └── generated/            # Auto-generated dari OpenAPI
    │       └── api.ts
    ├── utils/
    │   ├── formatters.ts         # Currency, date formatting
    │   ├── validators.ts         # Input validation
    │   └── geo.ts                # Geolocation helpers
    └── styles/
        └── globals.css
```

---

## 5. System Architecture

### 5.1 C4 Model: Context Diagram

**External Actors:**
- **Customer/Worker:** User yang bisa switch role (dual-role)
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
- WebSocket subscriptions untuk live updates

**Background Worker Container:**
- Cron jobs: karma decay, escrow cleanup, treasury yield
- Queue processing: job matching notifications

### 5.3 Escrow Flow dengan Pihak Ketiga (Xendit)

**Problem:** v1.0 menggunakan Virtual Ledger internal yang menampung dana. Ini membutuhkan izin OJK sebagai e-money provider.

**Solution v2.0:** Xendit Escrow API menampung dana.

**Flow Create Job:**
1. Customer create job dengan budget Rp150.000
2. Backend call Xendit Escrow API: `create_escrow(amount: 150000, customer_id, job_id)`
3. Xendit return escrow_id dan payment link/virtual account
4. Customer bayar ke Xendit (VA/e-wallet/QRIS)
5. Xendit webhook ke backend: payment received, escrow locked
6. Backend update job status: "ESCROW_LOCKED" dan broadcast ke feed

**Flow Complete Job:**
1. Worker tap "Selesai"
2. Customer tap "Konfirmasi"
3. Backend call Xendit: `release_escrow(escrow_id, worker_id, amount: 148500)`
4. Xendit transfer ke rekening worker (minus 1% solidarity pool ke rekening koperasi)
5. Xendit webhook: release completed
6. Backend update job: "COMPLETED"

**Flow Dispute:**
1. Customer tap "Dispute" dalam 24 jam
2. Backend call Xendit: `freeze_escrow(escrow_id)`
3. Xendit hold dana (tidak bisa di-release)
4. Jury voting (7 jurors)
5. Setelah voting selesai:
   - Jika customer menang: Xendit `refund_escrow(ke customer)`
   - Jika worker menang: Xendit `release_escrow(ke worker)`

**Keuntungan:**
- SiapAja tidak pernah menampung dana user
- Tidak perlu izin OJK
- Fraud risk ditanggung Xendit (licensed payment gateway)
- Compliance: Xendit sudah licensed sebagai Payment Gateway oleh BI

### 5.4 Personalized Feed Architecture

**Problem:** Tiap user harus lihat feed yang berbeda berdasarkan:
- Lokasi GPS (radius 5km default)
- Preferensi kategori (history job yang pernah di-claim)
- Karma score (job premium untuk karma tinggi)
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
   - Karma compatibility (job premium untuk karma tinggi)
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

### 5.5 Dual-Role Support (Customer = Worker)

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
- Toggle switch di header: "Mode Customer" / "Mode Worker"
- Feed berubah berdasarkan mode:
  - Customer mode: lihat "My Posted Jobs" + "Nearby Workers"
  - Worker mode: lihat "Available Jobs" + "My Active Jobs"

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

-- Transactions (financial records, immutable)
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

-- Karma History
CREATE TABLE karma_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    delta INTEGER NOT NULL, -- positive atau negative
    reason karma_reason NOT NULL,
    reference_type VARCHAR(50), -- 'job', 'dispute', 'jury'
    reference_id UUID,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 6.2 SpacetimeDB Schema (Real-time State)

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
- High Karma: 300 req/minute
- Blue Check: 600 req/minute

**Specific Endpoints:**
- `POST /jobs`: 5 req/minute (anti-spam)
- `POST /jobs/:id/claim`: 10 req/minute (anti-bot)
- `POST /auth/otp/request`: 3 req/minute per phone

---

## 8. AI Integration (OpenRouter)

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

### 9.1 WebSocket Architecture

**Connection Model:**
- Persistent WebSocket per user session
- Auto-reconnect dengan exponential backoff (max 30s)
- Heartbeat 30 detik

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
- Certificate pinning di PWA

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
| **SpacetimeDB** | In-memory database dengan persistence dan real-time subscriptions. |
| **Dual-Role** | User bisa switch antara customer dan worker dalam satu akun. |
| **Personalized Feed** | Feed job yang berbeda untuk setiap user berdasarkan preferensi. |
| **Karma** | Reputation score user. |
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
- **Version:** 2.0
- **Date:** 2026-03-15
- **Next Review:** 2026-06-15

**Changelog:**
| Version | Date | Changes |
|---------|------|---------|
| 2.0 | 2026-03-15 | Third-party escrow, OpenRouter AI, personalized feed, dual-role, workspace structure |
| 1.0 | 2026-03-01 | Initial specification |

---

*End of Document*
