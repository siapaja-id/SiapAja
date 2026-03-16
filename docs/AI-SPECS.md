# AI-SPECS.md

## Pusat Spesifikasi AI & LLM Pipeline SiapAja.id

**Dokumen ini adalah "Source of Truth" untuk semua hal terkait AI, LLM, dan automasi. Semua dokumen lain HARUS me-reference dokumen ini.**

---

## 1. Terminologi

| Teknis (Backend) | Publik (UI) | Deskripsi |
|------------------|-------------|-----------|
| `price_floor` | **Harga Bawah** | Harga minimum yang direkomendasikan AI. |
| `taskee_count_required` | **Jagoan Dibutuhkan** | Jumlah Jagoan untuk satu job. |
| `risk_level` | **Tingkat Risiko** | low/medium/high dari AI assessment. |

---

## 2. OpenRouter Architecture

### Why OpenRouter?
- Single API key untuk access 200+ LLM models
- Fallback otomatis jika satu model down
- Cost optimization (pilih model berdasarkan price/performance)
- Tidak perlu manage multiple API keys (OpenAI, Anthropic, etc)

### Implementation
- HTTP client menggunakan `reqwest` dengan connection pooling
- Async calls dengan timeout 10 detik
- Retry logic: 3 attempts dengan exponential backoff
- Caching: identical prompts di-cache 1 jam (SpacetimeDB)

---

## 3. AI Use Cases

### 3.1 Price Floor Estimation

**Purpose:** Mencegah perang harga yang tidak manusiawi

**Input:**
- Job title
- Job description
- Category
- Location (GPS coordinates)

**Prompt Template:**
```
Estimate fair price for [job description] in [location]. 
Consider: labor cost, difficulty, market rate, distance.
Return JSON: {
  "min_price": number,
  "recommended_price": number,
  "confidence": number (0-1),
  "reasoning": string
}
```

**Model:** Claude 3 Haiku (fast, cheap, good enough)
**Fallback:** GPT-3.5 Turbo

**Logic:**
- AI menghitung biaya hidup per wilayah, tingkat kesulitan kerja, dan jarak tempuh
- Kalau AI bilang harga wajar benerin pompa air adalah Rp150.000, maka tombol "Bidding" di bawah angka itu akan **dimatikan**
- Kita pastikan kompetisi terjadi di *kualitas*, bukan di *kemiskinan*

---

### 3.2 Manpower Estimation (Squad Formation)

**Purpose:** Menentukan apakah pekerjaan butuh multiple workers

**Input:**
- Job description
- Image URL (optional - worker upload foto, AI analyze)

**Prompt Template:**
```
Analyze if this job needs multiple workers.
Consider: weight, stairs, tools, complexity, hazard level.
Return JSON: {
  "workers_needed": number (1-5),
  "reasoning": string,
  "risk_level": "low" | "medium" | "high",
  "required_skills": string[]
}
```

**Threshold:** >50kg beban → wajib minimal 2 worker

**Squad Formation Logic:**
- Jika AI mendeteksi butuh 3 orang untuk angkat lemari raksasa
- Sistem otomatis bikin "Lobby" pencarian 3 worker terdekat
- Begitu 3 orang kumpulkan, mereka jalan bareng
- Setelah kerjaan selesai, Virtual Ledger otomatis pecah pembayaran ke 3 akun worker secara adil

---

### 3.3 Content Moderation

**Purpose:** Mencegah konten ilegal/berbahaya

**Input:**
- Job title
- Job description

**Prompt Template:**
```
Check if this content violates safety guidelines.
Return JSON: {
  "approved": boolean,
  "reason": string? (if not approved),
  "flags": string[] (e.g., "illegal", "dangerous", "scam")
}
```

**Model:** Claude 3 Haiku

**Examples of Blocked Content:**
- "butuh orang buat nagih utang sambil bawa sajam" → BLOCKED
- Illegal activities → BLOCKED
- Dangerous requests → BLOCKED

---

### 3.4 Feed Personalization (Batch)

**Purpose:** Menyajikan feed yang relevan untuk setiap user

**Input:**
- User history (categories, budgets, locations)
- Recent job claims
- Rating patterns

**Prompt Template:**
```
Given user history, predict preferred job categories and price range.
Return JSON: {
  "preferred_categories": string[],
  "avg_budget": number,
  "price_sensitivity": number (0-1, lower = suka job mahal)
}
```

**Frequency:** Daily batch untuk semua active users
**Cache:** Store di SpacetimeDB `feed_personalization` table

---

## 4. AI Output JSON Schema

### 4.1 PriceEstimation Response (OpenRouter)
```json
{
  "price_floor": 150000,
  "recommended_price": 200000,
  "is_heavy_lift": false,
  "taskee_count_required": 1,
  "risk_level": "low"
}
```

### 4.2 ManpowerEstimation Response
```json
{
  "workers_needed": 3,
  "reasoning": "Heavy furniture (refrigerator + oven) estimated >50kg, 4th floor no elevator",
  "risk_level": "high",
  "required_skills": ["heavy_lifting", "moving"]
}
```

### 4.3 ContentModeration Response
```json
{
  "approved": true,
  "reason": null,
  "flags": []
}
```

---

## 5. Error Handling AI

- **Timeout:** fallback ke rule-based heuristic
- **API error:** log dan retry, jika gagal gunakan default values
- **Invalid JSON response:** retry dengan stricter prompt

---

## 6. Model Selection Matrix

| Use Case | Primary Model | Fallback Model | Timeout |
|----------|---------------|----------------|---------|
| Price Floor | Claude 3 Haiku | GPT-3.5 Turbo | 10s |
| Manpower | Claude 3 Haiku | GPT-3.5 Turbo | 10s |
| Content Moderation | Claude 3 Haiku | None (strict) | 5s |
| Feed Personalization | GPT-3.5 Turbo | Claude 3 Haiku | 15s |

---

## Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-16 | Initial creation - consolidated from readme.md, WHITEPAPER.md & TECHNICAL.md |
| 1.1 | 2026-03-16 | Added terminology table, updated PriceEstimation schema with price_floor, is_heavy_lift, taskee_count_required. Fixed section numbering. |

---

*Dokumen ini adalah Source of Truth. Untuk detail implementasi teknis, rujuk ke TECHNICAL.md.*
