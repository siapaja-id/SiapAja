# SCREEN-LIST.md
## SiapAja.id Platform Architecture
### Version: 3.5
### Date: 2026-03-16
### Philosophy: Markdown-First + AI Extraction + Universal Actions

---

# 0. PRINCIPLES & CONSTRAINTS

## 0.1 Core UX Philosophy
- **Markdown-First**: Everything is a Post, and a Post is just Markdown
- **AI Extraction**: OpenRouter API (Claude 3 Haiku, GPT-3.5) auto-extracts budget, location, category from text
- **No Mode Switching**: Every user can perform any action anytime (Reddit-style)
- **Pay-to-Post**: Job visible only after escrow funded (anti-spam)
- **Fast-Bid Matching**: 15-minute enrollment window, customer selects or auto-match
- **Anti-Senior Bias**: New workers have equal visibility + karma boost
- **Transparent Algorithms**: All scoring factors visible to users

## 0.2 User States (Not Roles)
| State | Description | Unlocks |
|-------|-------------|---------|
| Unverified | Phone only, no KTP | Browse, Chat, Post Job (with limits) |
| Verified | KTP + Bank verified | Claim Jobs, Withdraw, Jury Duty |
| Active | Completed 1+ job | Full platform access |
| Boosted | First 10 jobs | Pamor display boost, "New Jagoan" badge |

---

# 1. AUTHENTICATION & ONBOARDING

## 1.1 Entry Points

### 1.1.1 Splash Screen
- **1.1.1.1** Brand animation (Rust crab + lightning bolt)
- **1.1.1.2** Deep link handler (job invite, referral)
- **1.1.1.3** Version check + forced update if critical

### 1.1.2 Landing Carousel (4 slides)
- **1.1.2.1** "Butuh Bantuan? Post & Bayar Aman"
- **1.1.2.2** "Bisa Bantu? Cari Job & Dapat Duit"
- **1.1.2.3** "0% Komisi untuk Job Kecil"
- **1.1.2.4** "Komunitas yang Adil, Bukan Algoritma"

## 1.2 Authentication

### 1.2.1 Phone Auth (Primary)
- **1.2.1.1** Country code selector (default: Indonesia +62)
- **1.2.1.2** Phone input with format validation
- **1.2.1.3** Operator detection (Telkomsel/XL/Three/etc)
- **1.2.1.4** OTP input (6 digits, auto-focus chain)
- **1.2.1.5** Resend timer (60s) with WhatsApp fallback
- **1.2.1.6** Voice call OTP (final fallback)

### 1.2.2 Social Auth (Secondary)
- **1.2.2.1** Google Sign-In (account selector if multiple)
- **1.2.2.2** Apple Sign-In (iOS only)

### 1.2.3 Security Setup
- **1.2.3.1** Biometric prompt (Face ID/Fingerprint)
- **1.2.3.2** PIN fallback setup (6 digits)

## 1.3 Unified Onboarding (Single Track)

### 1.3.1 Essential Profile (Required)
- **1.3.1.1** Full name input
- **1.3.1.2** Email verification
- **1.3.1.3** Profile photo (camera or gallery)
- **1.3.1.4** Location permission (GPS auto or manual Kecamatan)
- **1.3.1.5** Service radius preference (2km/5km/10km/20km)

### 1.3.2 Verifikasi & Komitmen Konstitusi (Membership Gate)
- **1.3.2.1** Persetujuan AD/ART Digital
  - **1.3.2.1.1** Viewer Konstitusi: Reader AD/ART (termasuk klausul Token $SIAP & Distribusi Fee).
  - **1.3.2.1.2** Digital Signature: Checkbox tunduk pada aturan koperasi.
- **1.3.2.2** KTP Verification (Kyc)
  - Front photo capture with auto-crop
  - Back photo capture
  - Selfie with KTP (liveness detection)
  - OCR review with manual correction
  - Verification pending screen (24-48h)
- **1.3.2.2** Bank Account (for withdrawals)
  - Bank selection (100+ Indonesian banks)
  - Account number input
  - Name validation API
  - Micro-deposit verification (optional instant)

### 1.3.3 Skill Tags (Optional, Enhances Discovery)
- **1.3.3.1** Category selection (multi-select, max 5)
  - Home Repair (Tukang)
  - Electronics (Service AC, TV, etc.)
  - Moving/Logistics
  - Cleaning
  - Automotive
  - Personal Assistant
  - Creative/Design
  - Tech Support
- **1.3.3.2** Experience level (Beginner/Intermediate/Expert)
- **1.3.3.3** Portfolio photo upload (max 10, past work)

### 1.3.4 Onboarding Complete
- **1.3.4.1** Success animation
- **1.3.4.2** "How It Works" quick tutorial (skippable)
- **1.3.4.3** First job recommendation (if location available)

---

# 2. CORE NAVIGATION

## 2.1 Bottom Tab Bar (Fixed 5 Items)

### 2.1.1 Beranda (Home)
- **2.1.1.1** Unified feed (Jobs + Service Offers + Discussions)
- **2.1.1.2** Location indicator with radius editor
- **2.1.1.3** Filter chips (horizontal scroll)

### 2.1.2 Cari (Search)
- **2.1.2.1** Search bar (sticky, with voice input)
- **2.1.2.2** Recent searches
- **2.1.2.3** Trending in your area
- **2.1.2.4** Category explorer (grid)
- **2.1.2.5** Map view toggle (nearby workers/jobs)

### 2.1.3 Buat (Create - Center Action)
- **2.1.3.1** Quick action menu (bottom sheet)
  - "Butuh Bantuan" (Post Job)
  - "Bisa Bantu" (Post Service Offer)
  - "Diskusi" (Community Post)
  - "Pasang Iklan" (Iklan Warga - for local business)

### 2.1.4 Aktivitas (Activity)
- **2.1.4.1** Notification inbox (all types)
- **2.1.4.2** My Jobs tab (posted & applied)
- **2.1.4.3** Messages tab (chat list)

### 2.1.5 Saya (Profile)
- **2.1.5.1** My profile view
- **2.1.5.2** Settings access

## 2.2 Top Navigation (Contextual)

### 2.2.1 Home Header
- **2.2.1.1** Location pill (tap to change)
- **2.2.1.2** Wallet quick-balance (tap for details)
- **2.2.1.3** Pamor score (tap for dashboard)

### 2.2.2 Universal Search Header
- **2.2.2.1** Back button
- **2.2.2.2** Search input with clear button
- **2.2.2.3** Filter button

---

# 3. FEED SCREENS (Unified Content Stream)

## 3.1 Feed Architecture

### 3.1.1 Feed Tabs (The Social-Utility Mix)
- **3.1.1.1** Untukmu (Algorithmic: 60% Demand, 20% Flex/Portfolio, 20% Community Discussion)
- **3.1.1.2** Cari Cuan (Jobs only - help wanted with active escrow)
- **3.1.1.3** Pamer Skill (Flex/Showcase: Workers posting their completed work results)
- **3.1.1.4** Info Warga (Hyper-local social: Discussion, news, and community warnings)

### 3.1.2 Feed Filters (Sticky Header)
- **3.1.2.1** Category chips (multi-select)
- **3.1.2.2** Price range slider (min-max)
- **3.1.2.3** Distance radius (2km/5km/10km/20km)
- **3.1.2.4** Urgency filter (Sekarang/Hari Ini/Fleksibel)

## 3.2 Card Components

### 3.2.1 Universal Post Card (Threads-style)
- **3.2.1.1** Header: Author avatar + handle + verification badge + time
- **3.2.1.2** Content Body: Full Markdown rendering (H1-H3, lists, bold, links)
- **3.2.1.3** Media Stack: Inline image/video carousel (max 10)
- **3.2.1.4** AI Smart-Metadata Chips (Auto-extracted from text):
  - Budget: "💰 Rp150k" (Extracted)
  - Skill: "🛠️ Reparasi Genteng" (Extracted)
  - Location: "📍 2.5km - Depok" (Extracted)
- **3.2.1.5** Description: Brief auto-summary for collapsed view
- **3.2.1.6** Interaction Thread: Replies count + Upvotes + Pamor Share link

### 3.2.2 Service Offer Card ("Bisa Bantu")
- **3.2.2.1** Header: Jagoan avatar + verification + pamor badge + online status
- **3.2.2.2** Title: "[Nama] bisa [Skill]"
- **3.2.2.3** Skills tags (horizontal scroll)
- **3.2.2.4** Portfolio thumbnail (3 photos)
- **3.2.2.5** Rate range ("Mulai dari Rp50.000")
- **3.2.2.6** Stats: Jobs done + Rating + Response time
- **3.2.2.7** Availability indicator ("Online" / "Sibuk" / "Offline")
- **3.2.2.8** Action bar:
  - "Invite ke Job" (if user has open job)
  - "Chat" (message)
  - "Lihat Profile"

### 3.2.3 Discussion Card (Community)
- **3.2.3.1** Author info
- **3.2.3.2** Title + preview text
- **3.2.3.3** Flair/Tag
- **3.2.3.4** Vote count (Net Score) + comment count
- **3.2.3.5** Actions: 
    - Upvote (Haptic feedback)
    - Downvote (Trigger "Reason Picker" modal: Spam/Scam/Etika/Lainnya)
    - Comment
    - Share

## 3.3 Feed States

### 3.3.1 Loading States
- **3.3.1.1** Skeleton shimmer (6 cards)
- **3.3.1.2** Pull-to-refresh animation
- **3.3.1.3** Infinite scroll spinner

### 3.3.2 Empty States
- **3.3.2.1** No jobs in radius ("Perluas radius?")
- **3.3.2.2** No results for filter ("Reset filter?")
- **3.3.2.3** New user cold start ("Posting pertama?")
- **3.3.2.4** Location permission denied (educational + settings link)

### 3.3.3 Error States
- **3.3.3.1** Network error (retry button)
- **3.3.3.2** GPS weak (manual location fallback)
- **3.3.3.3** Server error (status page link)

## 3.4 Card Component: Iklan Lokal (Contextual Ad Card)

### 3.4.1 Ad Card Structure
- **3.4.1.1** Brand Header: Foto profil toko + Nama Usaha + Label "Promosi" (hijau tipis)
- **3.4.1.2** Creative Canvas: Foto produk/jasa (1:1 atau 4:5 ratio)
- **3.4.1.3** Headline: Judul singkat (misal: "Sedia Pasir Putih & Semen Gresik")
- **3.4.1.4** Distance Tag: Otomatis muncul "📍 800m dari lo"
- **3.4.1.5** CTA Button: Tombol "Chat via SiapAja" atau "Gasin ke Lokasi" (Google Maps)

### 3.4.2 Contextual Targeting Logic
- **3.4.2.1** Job-Based Targeting: Kalau Jagoan lagi liat job "Benerin Atap", iklan yang muncul adalah "Toko Genteng Terdekat"
- **3.4.2.2** Service-Based Targeting: Kalau Tasker lagi cari "Cleaning Service", iklan yang muncul adalah "Layanan Laundry Sat-Set"
- **3.4.2.3** Location-Based: Prioritas radius 1km > 2km > 5km
- **3.4.2.4** Category Match: Ads diselaraskan dengan kategori skill/service yang lagi aktif di feed

---

# 4. CREATE FLOWS (Universal - Anyone Can Create Anything)

## 4.1 Unified Content Input (ChatGPT Style)

### 4.1.1 Universal Post Composer
- **4.1.1.1** Markdown Workspace: Single large-text area for everything (The "Prompt")
- **4.1.1.2** Description: Floating helper for Markdown shortcuts (Table, List, Code)
- **4.1.1.3** Real-time AI Interpreter Sidebar:
  - Extracted Budget tracker (updates as you type "bayar 100rb")
  - Extracted Location tagger (updates as you type "di ruko depan ITC")
  - Extracted Category suggestor
- **4.1.1.4** Media Attachment Bar: Simple paperclip icon for images/docs
- **4.1.1.5** Escrow Preview Overlay: Shows locked amount once AI detects budget in text
- **4.1.1.6** "Publish via Escrow" Button: Trigger payment & post in one tap

## 4.2 Iklan Warga Creation Flow (Simple & Fast)

### 4.2.1 Drafting Markdown
- **4.2.1.1** Text input: Sama kayak post job, cukup tulis: *"Toko Berkah sedia kopi & gorengan buat Jagoan yang lagi nunggu job. Depan Alfamart Margonda."*
- **4.2.1.2** Photo upload: Satu foto produk/jasa (1:1 atau 4:5 ratio)
- **4.2.1.3** Business info: Nama usaha otomatis dari profil

### 4.2.2 Radius Setting
- **4.2.2.1** Radius selector: 1km, 2km, 5km
- **4.2.2.2** Preview map: Visualisasi jangkauan iklan

### 4.2.3 Budgeting
- **4.2.3.1** Budget input: Masukin budget (misal: Rp10.000 buat 500 tayangan)
- **4.2.3.2** CPM preview: Estimasi jumlah tayangan berdasarkan budget
- **4.2.3.3** Duration indicator: Estimasi lama iklan aktif

### 4.2.4 Payment
- **4.2.4.1** Saldo Rupiah: Bayar pake Saldo Rupiah (Xendit)
- **4.2.4.2** Escrow-like model: Budget dikunci dan diambil per tayangan

### 4.2.5 Success Page
- **4.2.5.1** "Mengudara" animation: Iklan langsung "mengudara" di feed warga sekitar
- **4.2.5.2** Quick link ke Ads Manager

---

# 5. DETAIL SCREENS

## 5.1 Job Detail Screen ("Butuh Bantuan")

### 5.1.1 Header
- **5.1.1.1** Back button
- **5.1.1.2** Share button (native share sheet)
- **5.1.1.3** Bookmark toggle
- **5.1.1.4** More menu (Report, Hide, Copy Link)

### 5.1.2 Poster Profile Card
- **5.1.2.1** Large avatar
- **5.1.2.2** Name + verification + member since
- **5.1.2.3** Stats: Jobs posted + As customer rating
- **5.1.2.4** "Lihat Profil" button
- **5.1.2.5** Chat button (pre-apply inquiry)

### 5.1.3 Main Thread Content
- **5.1.3.1** Content Body: Pure Markdown view with full syntax support
- **5.1.3.2** Description: Meta-data summary card (Budget, Urgency, Status)
- **5.1.3.3** Media Canvas: Full-bleed gallery view for attached evidence
- **5.1.3.4** AI Insights Toggle: Detailed breakdown of how AI extracted the job params

### 5.1.4 Location Section
- **5.1.4.1** Full address (if shared)
- **5.1.4.2** Interactive map with route planning
- **5.1.4.3** Distance from current location
- **5.1.4.4** Landmarks/access notes

### 5.1.5 Budget & Escrow
- **5.1.5.1** Total budget (large)
- **5.1.5.2** Fee breakdown (if applicable)
  - Platform fee: 0% (if <500k) or X%
  - Solidarity pool: 1%
- **5.1.5.3** Escrow status indicator (Locked/Safe)
- **5.1.5.4** "Nego" indicator (if enabled)

### 5.1.6 Enrollment Status (Dynamic)
- **5.1.6.1** Time remaining ("12 menit lagi")
- **5.1.6.2** Applicants count ("3 worker melamar")
- **5.1.6.3** Applicants preview (avatars list)
- **5.1.6.4** "Lihat Semua Pelamar" (if poster)

### 5.1.7 Action Bar (Context-Aware, Sticky Bottom)

**For Unverified Viewers:**
- "Verifikasi untuk Melamar" (primary, educational)
- "Tanya Dulu" (secondary, chat allowed)

**For Verified Viewers (Not Applied):**
- "Saya Bisa Bantu" (primary, apply)
- "Tanya Dulu" (secondary)

**For Verified Viewers (Already Applied):**
- "Sudah Melamar" (disabled, with timestamp)
- "Batalkan" (withdraw application)

**For Poster (Owner):**
- "Lihat Pelamar" (manage applications)
- "Edit" (if no applicants yet)
- "Batalkan Job" (with refund confirmation)

**For Assigned Jagoan:**
- "Menuju Lokasi" (navigation)
- "Update Progress" (photo + status)
- "Selesai" (mark complete)

### 5.1.8 Public Discussion
- **5.1.8.1** Comments/questions about the job
- **5.1.8.2** Poster responses
- **5.1.8.3** "Tanya" input (for anyone)

## 5.2 Service Detail Screen ("Bisa Bantu")

### 5.2.1 Jagoan Profile Header
- **5.2.1.1** Avatar + cover photo
- **5.2.1.2** Name + verification + karma percentile ("Top 15%")
- **5.2.1.3** Online status + last active
- **5.2.1.4** "New Jagoan" badge (if <10 jobs)

### 5.2.2 Service Details
- **5.2.2.1** Skills list with experience level
- **5.2.2.2** Description
- **5.2.2.3** Portfolio gallery
- **5.2.2.4** Rate/range
- **5.2.2.5** Availability calendar

### 5.2.3 Stats & Reviews
- **5.2.3.1** Jobs completed
- **5.2.3.2** Completion rate
- **5.2.3.3** Average rating (stars)
- **5.2.3.4** Review list (as worker)

### 5.2.4 Action Bar
- **5.2.4.1** "Invite ke Job Saya" (if user has open job)
- **5.2.4.2** "Chat" (start conversation)
- **5.2.4.3** "Simpan" (bookmark worker)

## 5.3 Discussion Detail Screen

### 5.3.1 Post Header
- **5.3.1.1** Author info
- **5.3.1.2** Title + flair
- **5.3.1.3** Vote buttons + score
- **5.3.1.4** Time + edit history

### 5.3.2 Content
- **5.3.2.1** Body text
- **5.3.2.2** Attachments

### 5.3.3 Comments Section
- **5.3.3.1** Nested comments (Reddit-style)
- **5.3.3.2** Vote per comment
- **5.3.3.3** Reply input
- **5.3.3.4** Sort options (Best/Top/New)

---

# 6. MATCHING & APPLICATION SCREENS

## 6.1 Instant Match Discovery (Jagoan Grind Mode)

### 6.1.1 Fast-Match Card Stack (Tinder-style)
- **6.1.1.1** Card Deck: Full-screen gesture-based stack of nearby job posts
- **6.1.1.2** Markdown Card View: The "Post" rendered as a clean, readable markdown sheet
- **6.1.1.3** AI Chips Overlay: Floating badges for 💰 Budget, 📍 Distance, and 🕒 Urgency (auto-extracted)
- **6.1.1.4** Swipe Right Action: Instant "Saya Bisa" (Claim/Enroll) with haptic feedback
- **6.1.1.5** Swipe Left Action: "Skip" and hide this post from the current session stack
- **6.1.1.6** Description: High-velocity UI designed for workers to clear all local demand in seconds without leaving the screen

### 6.1.2 Post-Swipe Feedback & Queue Status
- **6.1.2.1** Match Splash: "Sikat!" full-screen animation if auto-matched instantly
- **6.1.2.2** Enrollment Queue: Real-time progress bar of the 15-minute selection window
- **6.1.2.3** Next Card Trigger: Auto-sliding the next demand card into view after a swipe
- **6.1.2.4** Description: Seamless transition that keeps the worker in the "flow state" of finding work

## 6.2 Applicant Management (Poster Perspective)

### 6.2.1 Applicants List
- **6.2.1.1** Header: "X Jagoan Melamar" + time remaining
- **6.2.1.2** Sort options (Recommended/Jarak/Pamor/Terbaru)
- **6.2.1.3** Applicant cards:
  - Avatar + name + karma percentile
  - Distance + estimated arrival
  - Message preview
  - "New Jagoan" badge (if applicable)
  - Win probability score (transparent algorithm)
  - "Pilih" button

### 6.2.2 Applicant Detail
- **6.2.2.1** Full profile view
- **6.2.2.2** Past work history
- **6.2.2.3** Reviews as Jagoan
- **6.2.2.4** Chat history (if any)
- **6.2.2.5** "Pilih Jagoan Ini" (primary)
- **6.2.2.6** "Tolak" (secondary)

### 6.2.3 Auto-Match Info
- **6.2.3.1** "Jika Anda tidak memilih dalam X menit:"
- **6.2.3.2** Algorithm explanation
- **6.2.3.3** "Disable Auto-Match" toggle (manual only)

## 6.3 Match Confirmation

### 6.3.1 Match Confirmation (The "Winner" State)
- **6.3.1.1** Match Celebration: High-energy visual for "Job is Yours!"
- **6.3.1.2** Thread Unlock: Pembuat Job's markdown post now shows full address & private notes
- **6.3.2.1** "Jagoan Dipilih" confirmation
- **6.3.2.2** Jagoan profile card
- **6.3.2.3** "Chat Jagoan" button
- **6.3.2.4** Cancel window info ("Bisa cancel dalam 5 menit")

---

# 7. JOB EXECUTION SCREENS

## 7.1 Active Job Screen (Jagoan)

### 7.1.1 Job Progress Header
- **7.1.1.1** Job status (Dalam Perjalanan/Sedang Dikerjakan)
- **7.1.1.2** Pembuat Job info
- **7.1.1.3** Chat button

### 7.1.2 Location Tracking
- **7.1.2.1** Map with Pembuat Job location
- **7.1.2.2** "Sampai" button (mark arrival)
- **7.1.2.3** Navigation integration (Google Maps/Waze)

### 7.1.3 Progress Updates
- **7.1.3.1** Photo upload (progress evidence)
- **7.1.3.2** Status notes
- **7.1.3.3** "Minta Bantuan" (escalation if stuck)

### 7.1.4 Completion
- **7.1.4.1** "Selesai" button
- **7.1.4.2** Final photo upload (before/after)
- **7.1.4.3** Completion notes
- **7.1.4.4** "Tunggu Konfirmasi Pembuat Job"

## 7.2 Active Job Screen (Pembuat Job)

### 7.2.1 Jagoan Tracking
- **7.2.1.1** Map with Jagoan location (if en route)
- **7.2.1.2** ETA estimate
- **7.2.1.3** Chat button

### 7.2.2 Progress View
- **7.2.2.1** Photo updates from worker
- **7.2.2.2** Status timeline

### 7.2.3 Completion Confirmation
- **7.2.3.1** "Kerjaan Selesai?" prompt
- **7.2.3.2** Final photos review
- **7.2.3.3** "Konfirmasi & Bayar" (release escrow)
- **7.2.3.4** "Ada Masalah?" (dispute initiation)

---

# 8. WALLET & PAYMENT SCREENS

> **📋 Source of Truth:** Untuk detail lengkap escrow integration, fee structure, dan payment flow, lihat [ECONOMICS.md](./docs/ECONOMICS.md)

## 8.1 Wallet Home

### 8.1.1 Balance Display
- **8.1.1.1** Available balance (large)
- **8.1.1.2** Locked in escrow (tap for details)
- **8.1.1.3** Pending withdrawal

### 8.1.2 Quick Actions
- **8.1.2.1** "Top Up" (deposit)
- **8.1.2.2** "Tarik" (withdraw)
- **8.1.2.3** "Riwayat" (history)

### 8.1.3 Recent Transactions (3 latest)
- **8.1.3.1** Transaction card (icon + desc + amount + status)

## 8.1 WALLET (Unified Fiat & Digital Assets)
### 8.1.1 Asset Display
- **8.1.1.1** Saldo Rupiah: Dana operasional harian.
- **8.1.1.2** Sertifikat Modal Digital (SMD): Jumlah unit penyertaan modal (Phase 1).
- **8.1.1.3** $SIAP Token: (Status: Roadmap - "Ready for Global Swap after Phase 4").
- **8.1.1.4** Transaction Immutability: Tombol cek hash transaksi untuk memastikan data sudah masuk *batch on-chain*.

## 8.2 Top Up Flow

### 8.2.1 Amount Selection
- **8.2.1.1** Preset amounts (50k/100k/200k/500k/1jt)
- **8.2.1.2** Custom amount input
- **8.2.1.3** Quick add buttons

### 8.2.2 Payment Method
- **8.2.2.1** E-Wallet (GoPay/OVO/Dana/LinkAja) - deep link
- **8.2.2.2** Virtual Account (bank transfer) - number display with copy
- **8.2.2.3** QRIS (display QR)
- **8.2.2.4** Retail (Alfamart/Indomaret) - barcode

### 8.2.3 Processing
- **8.2.3.1** Pending screen with countdown
- **8.2.3.2** Success animation + balance update
- **8.2.3.3** Failure/retry options

## 8.3 Withdrawal Flow

### 8.3.1 Amount & Destination
- **8.3.1.1** Amount (max: available balance)
- **8.3.1.2** Saved bank accounts list
- **8.3.1.3** Add new account
- **8.3.1.4** Fee preview (if any)
- **8.3.1.5** Estimated arrival

### 8.3.2 Security Verification
- **8.3.2.1** PIN/biometric
- **8.3.2.2** OTP (for large amounts)

### 8.3.3 Processing
- **8.3.3.1** "Sedang Diproses" (under review if >10jt)
- **8.3.3.2** "Sedang Ditransfer"
- **8.3.3.3** "Selesai" with receipt

## 8.4 Transaction History

### 8.4.1 List View
- **8.4.1.1** Filter chips (All/In/Out/Escrow)
- **8.4.1.2** Date group headers
- **8.4.1.3** Transaction cards:
  - Icon (category)
  - Description
  - Amount (+green/-red)
  - Status (success/pending/failed)
  - Timestamp

### 8.4.2 Transaction Detail (Audit Mode)
- **8.4.2.1** Full details
- **8.4.2.2** Reference numbers
- **8.4.2.3** Related job link (if applicable)
- **8.4.2.4** Download receipt
- **8.4.2.5** Report issue
- **8.4.2.6** **Immutable Receipt**: Menampilkan Hash SHA-256 transaksi (contoh: `0x7f83b1657ff1fc53b92dc18148a1d65dfa1350f`).
- **8.4.2.7** **Verify Link**: Tombol buat verifikasi data ke Public Ledger Browser (transparansi radikal) - buka modal yang nampilin payload JSON + hash chain.

### 8.4.3 Ledger Browser (Public Audit View)
- **8.4.3.1** Search by hash: Input field buat cari transaksi berdasarkan hash.
- **8.4.3.2** Filter by type: Toggle antara 'financial' dan 'pamor'.
- **8.4.3.3** Chain visualization: Timeline entry dengan indikator hash validity (centang hijau kalau chain intact).
- **8.4.3.4** Export data: Download JSON + Merkle proof untuk audit eksternal.

---

### 9.2.3 Pamor Audit Trail
- **9.2.3.1** List kronologis mutasi Pamor (contoh: "+10 dari Job #123", "-5 dari downvote").
- **9.2.3.2** Tiap baris punya indikator "Verified by Ledger" (Centang hijau kecil).
- **9.2.3.3** Hash preview: Klik buat liat full hash entry di modal.
- **9.2.3.4** Filter by source: Job completion, downvote, jury duty, decay.
- **9.2.3.5** Export Pamor Ledger: Tombol download entire Pamor history sebagai JSON + hash chain (buat portabilitas reputasi).

---

# 9. KARMA & REPUTATION SCREENS

> **📋 Source of Truth:** Untuk detail lengkap sistem Pamor, voting power, decay rate, dan tier system, lihat [GOVERNANCE.md](./docs/GOVERNANCE.md)

## 9.1 Pamor Dashboard

### 9.1.1 Score Display
- **9.1.1.1** Current karma (large number)
- **9.1.1.2** Percentile rank ("Anda di Top 20% bulan ini")
- **9.1.1.3** Trend indicator (↗️ improving)
- **9.1.1.4** Progress to next tier

### 9.1.2 Tier Information
- **9.1.2.1** Current tier (Bronze/Silver/Gold/Platinum)
- **9.1.2.2** Tier benefits list
- **9.1.2.3** Next tier requirements

### 9.1.3 Pamor Breakdown
- **9.1.3.1** Category scores (per skill)
- **9.1.3.2** As Pembuat Job score
- **9.1.3.3** As Jagoan score

## 9.2 Pamor History

- **9.2.2.2** Pamor change
- **9.2.2.3** Related job/user (if applicable)
- **9.2.2.4** Appeal option (if disputed)

## 9.3 Fairness Audit (Transparency Feature)

### 9.3.1 Application History
- **9.3.1.1** List of applied jobs
- **9.3.1.2** Win/loss status
- **9.3.1.3** Reason for loss (transparent)
  - "Pamor 15% lebih rendah dari pemenang"
  - "Jarak 2km lebih jauh"
  - "Pesan kurang detail"
  - "Pembuat Job memilih manual"
- **9.3.1.4** Tips for improvement

---

# 10. DISPUTE & JUSTICE SCREENS

> **📋 Source of Truth:** Untuk detail lengkap dispute resolution, jury selection, dan voting mechanism, lihat [GOVERNANCE.md](./docs/GOVERNANCE.md)

## 10.1 Dispute Initiation

### 10.1.1 Dispute Form
- **10.1.1.1** Job reference (auto-filled)
- **10.1.1.2** Your role (Poster/Jagoan - auto)
- **10.1.1.3** Reason selection:
  - Pekerjaan tidak selesai
  - Kualitas tidak sesuai
  - Kerusakan barang
  - Keterlambatan parah
  - Lainnya (text input)
- **10.1.1.4** Desired outcome:
  - Full refund
  - Partial refund
  - Revisi pekerjaan
- **10.1.1.5** Evidence upload (photos/videos, max 10)
- **10.1.1.6** Description
- **10.1.1.7** Escrow freeze warning
- **10.1.1.8** "Ajukan Sengketa" confirmation

## 10.2 Jury Duty (For Verified Users)

### 10.2.1 Jury Summons
- **10.2.1.1** Notification: "Anda Dipilih sebagai Juri"
- **10.2.1.2** Case preview (anonymized)
- **10.2.1.3** Time commitment (24h to vote)
- **10.2.1.4** Reward preview (karma + fee)
- **10.2.1.5** Accept/Decline

### 10.2.2 Evidence Review
- **10.2.2.1** Case summary
- **10.2.2.2** Poster evidence
- **10.2.2.3** Jagoan evidence
- **10.2.2.4** Job details (anonymized location)
- **10.2.2.5** Chat history (filtered)

### 10.2.3 Voting
- **10.2.3.1** Blind voting interface
  - "Pihak yang Posting Menang"
  - "Pihak yang Mengerjakan Menang"
  - "Pembagian" (split)
- **10.2.3.2** Confidence level (1-5)
- **10.2.3.3** Reasoning (optional, private)
- **10.2.3.4** Submit vote

### 10.2.4 Results
- **10.2.4.1** "Menunggu juror lain" progress
- **10.2.4.2** Final decision reveal
- **10.2.4.3** Your reward (karma + rupiah)
- **10.2.4.4** Case archive

## 10.3 Dispute History

### 10.3.1 My Disputes
- **10.3.1.1** As initiator
- **10.3.1.2** As respondent
- **10.3.1.3** As juror

### 10.3.2 Dispute Detail
- **10.3.2.1** Full timeline
- **10.3.2.2** Evidence from both sides
- **10.3.2.3** Jury votes (anonymized)
- **10.3.2.4** Final outcome
- **10.3.2.5** Financial impact

---

# 11. ACTIVITY & NOTIFICATION CENTER

## 11.1 Notification Inbox

### 11.1.1 Tabs
- **11.1.1.1** Semua (All)
- **11.1.1.2** Pekerjaan (Job-related)
- **11.1.1.3** Keuangan (Payment)
- **11.1.1.4** Komunitas (Jury, Pamor, Reviews)
- **11.1.1.5** Sistem (Updates, Maintenance)

### 11.1.2 Notification Types
- **11.1.2.1** Job alerts (new job in radius, applied, selected, completed)
- **11.1.2.2** Payment alerts (received, withdrawal, escrow released)
- **11.1.2.3** Pamor alerts (gained, decay, tier up)
- **11.1.2.4** Jury summons
- **11.1.2.5** Review received
- **11.1.2.6** Chat messages
- **11.1.2.7** System announcements

## 11.2 My Jobs Screen

### 11.2.1 Diminta (Posted by Me)
- **11.2.1.1** Drafts
- **11.2.1.2** Open (enrolling, no applicants yet)
- **11.2.1.3** Has Applicants (select needed)
- **11.2.1.4** Assigned (in progress)
- **11.2.1.5** Completed (pending review or done)

### 11.2.2 Dikerjakan (Done by Me)
- **11.2.2.1** Applied (pending selection)
- **11.2.2.2** Selected (assigned, not started)
- **11.2.2.3** In Progress (en route, working)
- **11.2.2.4** Completed (pending confirmation or done)

### 11.2.3 Disengketakan (In Dispute)
- **11.2.3.1** Active disputes
- **11.2.3.2** Pending jury
- **11.2.3.3** Resolved

## 11.3 Messages (Chat)

### 11.3.1 Chat List
- **11.3.1.1** Conversation list with context labels:
  - "Tentang: Service AC"
  - "Tentang: Pindahan Kos"
- **11.3.1.2** Unread badges
- **11.3.1.3** Online indicators
- **11.3.1.4** Archive/Delete

### 11.3.2 Chat Room
- **11.3.2.1** Header with job context (tappable to job detail)
- **11.3.2.2** Message bubbles (text, image, location)
- **11.3.2.3** Quick actions (send location, send photo)
- **11.3.2.4** Voice message (future)
- **11.3.2.5** Block/Report

---

# 12. PROFILE SCREENS

## 12.1 Public Profile View

### 12.1.1 Header
- **12.1.1.1** Cover photo
- **12.1.1.2** Avatar (large)
- **12.1.1.3** Name + verification badge
- **12.1.1.4** Pamor tier badge
- **12.1.1.5** "New Jagoan" badge (if <10 jobs)

### 12.1.2 Stats Row
- **12.1.2.1** Jobs posted (as customer)
- **12.1.2.2** Jobs completed (as worker)
- **12.1.2.3** Pamor score
- **12.1.2.4** Member since

### 12.1.3 Content Tabs (Reddit-style)
- **12.1.3.1** Overview (mixed recent activity)
- **12.1.3.2** Butuh Bantuan (their job posts)
- **12.1.3.3** Bisa Bantu (their service offers)
- **12.1.3.4** Diskusi (community posts)
- **12.1.3.5** Reviews (received, as customer + worker)

### 12.1.4 Action Buttons (Context-Aware)
- **If self:** Edit Profile
- **If other:** Message, Invite to Job, Block, Report

## 12.2 Edit Profile

### 12.2.1 Basic Info
- **12.2.1.1** Profile photo (change/remove)
- **12.2.1.2** Cover photo (change/remove)
- **12.2.1.3** Full name
- **12.2.1.4** Username/handle (@)
- **12.2.1.5** Bio/description

### 12.2.2 Contact & Verification
- **12.2.2.1** Phone (change with re-verification)
- **12.2.2.2** Email (verify/change)
- **12.2.2.3** KTP status (re-verify if needed)

### 12.2.3 Skills & Services
- **12.2.3.1** Edit categories
- **12.2.3.2** Edit experience levels
- **12.2.3.3** Edit portfolio
- **12.2.3.4** Edit availability

### 12.2.4 Finance
- **12.2.4.1** Bank accounts (add/edit/remove)
- **12.2.4.2** E-wallet bindings

---

# 13. SETTINGS SCREENS

## 13.1 Account Settings

### 13.1.1 Preferensi
- **13.1.1.1** Language (Bahasa Indonesia/English)
- **13.1.1.2** Currency (IDR fixed for now)
- **13.1.1.3** Time format (24h/12h)
- **13.1.1.4** Notifications
  - Push (master toggle + per-type)
  - Email
  - SMS

### 13.1.2 Privasi
- **13.1.2.1** Profile visibility (Public/Active Only/Friends)
- **13.1.2.2** Location sharing (Exact/Approximate/Off)
- **13.1.2.3** Online status (Show/Hide)
- **13.1.2.4** Blocked users list
- **13.1.2.5** Data download (GDPR-style)

### 13.1.3 Keamanan
- **13.1.3.1** Change PIN
- **13.1.3.2** Biometric settings
- **13.1.3.3** Active devices (view/logout)
- **13.1.3.4** Login history
- **13.1.3.5** Two-factor auth (future)

## 13.2 App Settings

### 13.2.1 Tampilan
- **13.2.1.1** Theme (Light/Dark/System)
- **13.2.1.2** Font size
- **13.2.1.3** Compact mode (dense UI)

### 13.2.2 Data & Storage
- **13.2.2.1** Cache clear
- **13.2.2.2** Downloaded media
- **13.2.2.3** Offline data settings

### 13.2.3 Jaringan
- **13.2.3.1** Data saver mode
- **13.2.3.2** Image quality (Auto/Low/High)
- **13.2.3.3** Wi-Fi only for media

## 13.3 Support & About

### 13.3.1 Bantuan
- **13.3.1.1** FAQ / Knowledge base
- **13.3.1.2** Contact support (chat/email)
- **13.3.1.3** Report bug
- **13.3.1.4** Suggest feature

### 13.3.2 Safety
- **13.3.2.1** Emergency contacts
- **13.3.2.2** Safety checklist
- **13.3.2.3** Report incident

### 13.3.3 About
- **13.3.3.1** Version info
- **13.3.3.2** Terms of service
- **13.3.3.3** Privacy policy
- **13.3.3.4** Open source licenses
- **13.3.3.5** Build in Public link (GitHub)

---

# 14. SPECIAL SCREENS

## 14.0 MANAGEMENT & FOUNDER HUB (The Sultan View)
### 14.0.1 Global Revenue Stream
- **14.0.1.1** Real-time Fee Tracker: Pantau jatah 3% dari transaksi harian.
- **14.0.1.2** Founder Wallet: Tombol withdraw untuk jatah pengurus.

## 14.1 Contributor Zone (Developer)

### 14.1.1 Build in Public
- **14.1.1.1** GitHub issues feed
- **14.1.1.2** Pull request activity
- **14.1.1.3** Contributor leaderboard
- **14.1.1.4** Bounty board

### 14.1.2 Pamor Equity
- **14.1.2.1** Contribution stats (commits, PRs, issues)
- **14.1.2.2** Equity points balance
- **14.1.2.3** Vesting schedule
- **14.1.2.4** Dividend history

## 14.2 KOPERASI HUB (The Ownership Experience)
### 14.2.1 Dashboard Anggota
- **14.2.1.1** Status Anggota & Porsi Pamor Shares.
- **14.2.1.2** Open Ledger: Transparansi kas kopi (Yield & Fee).
- **14.2.1.3** Governance: Voting amandemen AD/ART.

### 14.2.2 Open Ledger (Transparansi Radikal)
- **14.2.2.1** Real-time Treasury Clock: Counter duit yang masuk ke kas koperasi per detik
- **14.2.2.2** Allocation Chart: Visualisasi kemana duit fee & yield pergi (Ops vs SHU vs Solidarity)
- **14.2.2.3** Public Log: Daftar transaksi treasury (anonymized) yang bisa di-scroll siapa aja
- **14.2.2.4** Solidarity Pool Balance: Total dana asuransi yang siap pake

### 14.2.3 Governance & Voting (Satu Orang Satu Suara)
- **14.2.3.1** Proposal Policy: List usulan perubahan sistem (misal: "Naikin tarif min di Depok")
- **14.2.3.2** Voting Room: Interface buat submit suara (pake Pamor-weighted voting)
- **14.2.3.3** Forum Diskusi Kebijakan: Chat thread khusus per-proposal
- **14.2.3.4** Election Center: Pemilihan pengurus wilayah secara periodik

### 14.2.4 Solidarity & Social (Bela Beli Teman)
- **14.2.4.1** List Klaim Bantuan: Daftar anggota yang butuh dana bantuan (sakit/kecelakaan)
- **14.2.4.2** Verifikasi Komunitas: Tombol bagi anggota terdekat buat verifikasi kondisi teman
- **14.2.4.3** "Beli Jasa Teman": Filter khusus buat prioritasin panggil worker yang lagi butuh bantuan dana

## 14.3 Verification Gates (Soft Prompts)

### 14.3.1 Unlock Feature Modal
- **14.3.1.1** "Lengkapi Verifikasi"
- **14.3.1.2** Benefits list with icons
- **14.3.1.3** Quick actions (Upload KTP, Add Bank)
- **14.3.1.4** "Nanti" (dismissible)

## 14.5 Ads Manager (Dashboard Toko)

### 14.5.1 Active Ads
- **14.5.1.1** Daftar iklan yang lagi jalan
- **14.5.1.2** Status (Aktif/Expired/Paused)
- **14.5.1.3** Quick actions (Pause/Edit/Delete)

### 14.5.2 Real-time Stats
- **14.5.2.1** Pamor Reach: Berapa banyak warga yang liat
- **14.5.2.2** Engagement: Berapa banyak yang klik atau nanya via chat
- **14.5.2.3** Cost per engagement: Efisiensi biaya iklan
- **14.5.2.4** Time remaining: Sisa waktu iklan aktif

### 14.5.3 Top-up Budget
- **14.5.3.1** Tambah saldo buat perpanjang durasi iklan
- **14.5.3.2** Quick amounts (50k/100k/200k)
- **14.5.3.3** Payment method selection

---

# 15. ERROR & EDGE CASE SCREENS

## 15.1 Network Errors

### 15.1.1 No Connection
- **15.1.1.1** Full-screen illustration
- **15.1.1.2** "Coba Lagi" button
- **15.1.1.3** Offline mode indicator (if cached data available)

### 15.1.2 Server Error
- **15.1.2.1** "Terjadi Kesalahan" message
- **15.1.2.2** Error code (for support)
- **15.1.2.3** Retry button
- **15.1.2.4** Contact support link

## 15.2 Permission Errors

### 15.2.1 Location Denied
- **15.2.1.1** Educational illustration (why we need location)
- **15.2.1.2** "Buka Pengaturan" button (deep link)
- **15.2.1.3** Manual location fallback (search Kecamatan)

### 15.2.2 Camera/Mic Denied
- **15.2.2.1** Educational message
- **15.2.2.2** Settings link
- **15.2.2.3** Alternative (gallery upload for camera)

## 15.3 Business Logic Errors

### 15.3.1 Insufficient Funds
- **15.3.1.1** "Saldo Tidak Cukup" message
- **15.3.1.2** Current vs required amount
- **15.3.1.3** "Top Up Sekarang" button
- **15.3.1.4** "Kurangi Budget" option (if applicable)

### 15.3.2 Verification Required
- **15.3.2.1** Feature explanation
- **15.3.2.2** Verification benefits
- **15.3.2.3** "Verifikasi Sekarang" CTA

### 15.3.3 Job Unavailable
- **15.3.3.1** "Job Sudah Diambil" (claimed by other)
- **15.3.3.2** "Job Dibatalkan" (cancelled by poster)
- **15.3.3.3** "Job Kedaluwarsa" (expired)
- **15.3.3.4** Similar jobs suggestion

---

# 16. MODALS & OVERLAYS

## 16.1 Action Modals

### 16.1.1 Share Modal
- **16.1.1.1** Copy link
- **16.1.1.2** QR code
- **16.1.1.3** Native share sheet
- **16.1.1.4** App-specific (WhatsApp, Telegram, etc.)

### 16.1.2 Filter Modal
- **16.1.2.1** Category multi-select
- **16.1.2.2** Price range slider
- **16.1.2.3** Distance radius
- **16.1.2.4** Sort options
- **16.1.2.5** Save preset

### 16.1.3 Sort Modal
- **16.1.3.1** Relevance (default)
- **16.1.3.2** Terbaru (newest)
- **16.1.3.3** Terdekat (nearest)
- **16.1.3.4** Termahal (highest budget)
- **16.1.3.5** Termurah (lowest budget)

## 16.2 Confirmation Dialogs

### 16.2.1 Destructive Actions
- **16.2.1.1** Cancel job (with refund warning)
- **16.2.1.2** Withdraw application
- **16.2.1.3** Block user
- **16.2.1.4** Delete account (with data export option)

### 16.2.2 Financial Confirmations
- **16.2.2.1** Confirm payment
- **16.2.2.2** Confirm withdrawal
- **16.2.2.3** Confirm escrow release

## 16.3 Bottom Sheets

### 16.3.1 Quick Actions
- **16.3.1.1** Job actions (Apply/Share/Report)
- **16.3.1.2** Payment method selector
- **16.3.1.3** Location picker
- **16.3.1.4** Category selector

---

# 17. SCREEN FLOW SUMMARIES

## 17.1 Critical Path: First Job Post
```
Splash → Onboarding → Home → Buat → "Butuh Bantuan" → 
Category → Details → Photos → Location → Budget → 
Review → Fund Escrow → Success → Job Live
```

## 17.2 Critical Path: First Job Claim (New Jagoan)
```
Splash → Onboarding → Home → Feed → Job Card → 
"Saya Bisa" → Verification Prompt → Upload KTP → 
Wait Verification → Verified → Apply → Wait Selection → 
Selected → Chat → Navigate → Complete → Payment Received
```

## 17.3 Critical Path: Dispute Resolution
```
Job Complete → "Ada Masalah" → Dispute Form → 
Evidence Upload → Submit → Jury Selection → 
Jury Review → Voting → Result → Fund Release
```

---

# 18. DESIGN TOKENS & COMPONENTS

## 18.1 Core Components
- **Button**: Primary (green), Secondary (grey), Destructive (red), Ghost
- **Input**: Text, Number, TextArea, Search, OTP, Select
- **Card**: Job Card, Service Card, User Card, Transaction Card
- **List**: Feed List, Chat List, Notification List
- **Modal**: Center, Bottom Sheet, Full Screen
- **Feedback**: Toast, Banner, Badge, Skeleton, Empty State, Loading Spinner

## 18.2 Color Palette
- **Primary**: #00A86B (Islamic green - trust, growth)
- **Secondary**: #1A1A1A (Near black - strength)
- **Accent**: #FFD700 (Gold - karma, quality)
- **Success**: #00C853
- **Warning**: #FFAB00
- **Error**: #FF1744
- **Info**: #2979FF

## 18.3 Typography
- **Display**: Inter Bold (headlines)
- **Body**: Inter Regular (content)
- **Mono**: JetBrains Mono (numbers, codes)

---

# 19. RESPONSIVE BREAKPOINTS

| Breakpoint | Width | Adaptations |
|------------|-------|-------------|
| Mobile | 320-428px | Default, single column |
| Large Mobile | 429-767px | Larger touch targets |
| Tablet | 768-1024px | Side panels, master-detail |
| Desktop PWA | >1024px | Full layout, multi-column feed |

---

# 20. VERSION HISTORY

| Version | Date | Key Changes |
|---------|------|-------------|
| 1.0 | 2026-03-01 | Initial draft with mode switching |
| 2.0 | 2026-03-10 | Reddit-style universal actions |
| 3.0 | 2026-03-15 | Final: Fast-bid matching, anti-senior bias, transparency features |
| 3.1 | 2026-03-16 | Markdown-first paradigm: AI extraction, Universal Post Card, ChatGPT-style composer |
| 3.2 | 2026-03-16 | Swipe-engine matching: Tinder-style card stack for worker discovery |
| 3.3 | 2026-03-16 | Downvote accountability: Reason Picker modal, Net Score display |
| 3.4 | 2026-03-16 | Immutable Ledger UI: Transaction hash display, Ledger Browser, Pamor Audit Trail with verification |
| 3.5 | 2026-03-16 | Hyper-Local Ads: Iklan Warga (Contextual Ad Card, Creation Flow, Ads Manager Dashboard) |

---

**Document Status**: 3.5 - Hyper-Local Ads Integration
**Next Review**: Pre-MVP Launch
**Owner**: Product & Design Team
```
