# GOVERNANCE.md

## Pusat Informasi Sistem Pamor & Pemerintahan Komunitas SiapAja.id

**Dokumen ini adalah "Source of Truth" untuk semua hal terkait reputasi, voting, dan sistem keadilan. Semua dokumen lain HARUS me-reference dokumen ini.**

---

## 1. Terminologi

| Teknis (Backend) | Publik (UI) | Deskripsi |
|------------------|-------------|-----------|
| `karma_score` | **Pamor** | Skor reputasi user di komunitas. |
| `pamor_score` | **Pamor** | Alias untuk backward compatibility. |
| `downvote` | **Downvote** | Sinyal bahwa konten/行为 violate norma komunitas. |

---

## 2. Sistem Pamor

Pamor adalah aset reputasi yang direkam permanen (tapi anonim) di database. Ini BUKAN sistem Kredit Sosial ala China - ini adalah sistem reputasi peer-to-peer.

### 2.1 Metrik Pamor

**Pamor Naik (+):**
- Tepat waktu sampai lokasi
- Rating bintang 5 dari Pembuat Job (dinilai dari kualitas kerja)
- Rajin jadi Juri sengketa yang adil
- Kontribusi kode (GitHub PR merge)

**Pamor Turun (-):**
- Batalin orderan sepihak setelah setuju (Cancel)
- Telat parah tanpa alasan
- Terbukti curang dalam sengketa
- Membuang sampah sembarangan di lokasi Pembuat Job

### 2.2 Pamor Decay (Penyusutan Otomatis)

Kami percaya pada **Penebusan Dosa Digital**:

- **Rate:** Setiap 30 hari, poin Pamor negatif akan otomatis mengalami "Decay" (menyusut) sebesar **15%** jika Jagoan terus berkelakuan baik
- **Mekanisme:** CRON Job di server Rust secara efisien mengkalkulasi jutaan data setiap akhir bulan
- **Filosofi:** Jagoan tidak dihukum seumur hidup. Mereka bisa redemption.

---

## 3. Pamor Tier System

| Tier | Pamor Range | Benefits |
|------|-------------|----------|
| **Bronze** | 0-99 | Basic access |
| **Silver** | 100-499 | Priority feed, higher rate limits |
| **Gold** | 500-999 | Priority matching, voting power |
| **Platinum** | 1000+ | Premium jobs, maximum voting, priority support |

---

## 4. Immutable Pamor Audit
Pamor bukan sekadar angka di profil, tapi deretan event yang di-hash.

- **Event Sourcing**: Pamor dihitung ulang dari nol setiap ada audit, atau pake snapshot yang di-verify hash-nya.
- **Audit Log**: "Pamor Lu naik +10 karena Job #123 (Hash: 0xabc...)"
- **Anti-Manipulation**: Admin gak punya akses `UPDATE pamor_score`. Perubahan skor hanya bisa lewat `Reducer` di SpacetimeDB/Rust yang sudah terprogram.
- **Public Verifiability**: Jagoan bisa export seluruh riwayat Pamor mereka dalam format JSON + hash chain untuk bukti portabilitas reputasi.

---

## 5. Voting Power (Hak Suara)

Pamor bukan cuma buat pamer. Semakin tinggi Pamor, semakin besar **Hak Suara** dalam menentukan arah platform:

- **Konversi:** 100 Pamor = 1 Suara
- **Maksimal:** 10 Suara per orang (supaya Sultan Pamor tidak bisa memonopoli keputusan)
- **Use Cases:**
  - Mau naikin Price Floor (Harga Bawah) di kota Jakarta? Voting!
  - Mau uang denda di Treasury dipakai buat bagi-bagi sembako atau asuransi kecelakaan? Voting!
  - Pemilihan pengurus wilayah

---

## 5. Mekanisme Downvote & Sanksi Sosial

### 5.1 Downvote sebagai Instrumen Akuntabilitas
Downvote bukan sekadar tombol dislike, melainkan sinyal bahwa konten atau perilaku user melanggar norma komunitas.

| Tipe Konten | Dampak Downvote | Syarat Pemberi Suara |
|-------------|-----------------|----------------------|
| **Discussion** | Mengurangi visibilitas (Ranking Drop) | Semua User |
| **Flex/Portfolio** | Mengurangi akumulasi Pamor dari post tsb | Verified User |
| **Demand (Job)** | Alert ke AI/Admin (Potensi Scam/Spam) | Verified User |

### 5.2 Pencegahan Abuse (Anti-Brigading)

1. **Pamor Weighting**: Satu downvote dari user Pamor 1000 lebih berasa dibanding dari user Pamor 100.
2. **Rate Limiting**: User tidak bisa memberikan >5 downvote dalam 1 jam (mencegah botting).
3. **Transparency**: User yang di-downvote bisa melihat "Alasan" (dipilih dari preset: Spam, Harga Gak Masuk Akal, Kasar, dll).
4. **Anti-Spam**: Downvote berulang dari orang yang sama ke target yang sama dalam waktu singkat bakal di-*ignore* sama Rust backend.

### 5.3 Cost of Downvoting
- Memberikan downvote akan mengurangi **1 poin Pamor** dari pemberi suara
- Ini memastikan downvote bukanlah tindakan ringan, melainkan pernyataan serius

### 5.4 Pamor Decay on Negative Feedback
Setiap **10 downvote bersih (net)** pada konten user akan otomatis mengurangi **1 poin Pamor permanen**, yang hanya bisa dipulihkan melalui mekanisme "Penebusan Dosa Digital" (Bagian 2.2).

### 5.5 Appeal Mechanism
Kalau user merasa jadi korban downvote massal tanpa alasan jelas, user bisa ajukan **Dispute** ke Juri (lihat Bagian 6).

---

## 6. Decentralized Justice (Pengadilan Netizen)

Kami nggak pakai CS yang jawabannya "Mohon maaf atas ketidaknyamanannya". Kami pake hukum komunitas.

### 6.1 Alur Sengketa (Dispute Lifecycle)

1. **Trigger:** Pembuat Job klaim "Kerjaan nggak beres!" → Dana di Virtual Escrow otomatis **BEKU**
2. **Jury Selection:** Sistem mencari 7 Juri dengan **expertise di bidang yang sama**
3. **Evidence Upload:** Jagoan & Pembuat Job upload bukti foto/video
4. **Blind Voting:** Juri voting secara anonymous. Juri nggak bisa lihat hasil voting juri lain sebelum submit
5. **Resolution:** Pemenang voting dapet dananya, Juri dapet komisi kecil sebagai imbalan kejujuran

### 6.2 Algoritma Pemilihan Juri (Expertise-Based)

- **Kriteria:** Juri dipilih dari pool Jagoan yang punya **track record di kategori job yang sama**
- **Contoh:** Sengketa job "Tukang AC" → Juri adalah Jagoan dengan rating tinggi di kategori "AC & Elektronik"
- **Netralitas:** Juri **nggak saling kenal** dan **nggak satu radius** dengan pelaku sengketa
- **Anti-Herd:** Juri nggak bisa lihat hasil voting juri lain sebelum dia sendiri submit (mencegah "Ikut-ikutan")

### 6.3 Voting Rules

- **Quorum:** Minimal 5 dari 7 juri harus voting
- **Outcome:** Suara mayoritas (misal 4 vs 3) menjadi keputusan final
- **Finality:** Keputusan juri adalah **FINAL** dan dieksekusi oleh Virtual Ledger secara otomatis

---

## 7. Contribution Logic: Technology vs Operations

Sistem ini memisahkan kontribusi menjadi dua jalur:
1. **Jalur Operasional (Koperasi):** Jagoan berkontribusi melalui kerja lapangan dan mendapatkan SHU dari porsi 60% pendapatan Koperasi.
2. **Jalur Riset (Solidarity-ID):** Developer berkontribusi melalui kode (GitHub PR) dan mendapatkan **Pamor Equity** yang bersumber dari alokasi 40% biaya teknologi.

Koperasi tidak perlu mempekerjakan developer secara internal, sehingga terhindari dari beban manajemen teknologi yang kompleks. Koperasi cukup memastikan layanan dari Solidarity-ID berjalan sesuai standar yang disepakati (SLA).

---

## 8. User States (Not Roles)

| State | Description | Unlocks |
|-------|-------------|---------|
| Unverified | Phone only, no KTP | Browse, Chat, Post Job (with limits) |
| Verified | KTP + Bank verified | Claim Jobs, Withdraw, Jury Duty |
| Active | Completed 1+ job | Full platform access |
| Boosted | First 10 jobs | Pamor display boost, "New Jagoan" badge |

---

## Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-16 | Initial creation - consolidated from readme.md, WHITEPAPER.md & SCREEN-LIST.md |
| 1.1 | 2026-03-16 | Added Bab 3: Mekanisme Downvote & Sanksi Sosial (Anti-Brigading, Cost of Downvoting, Pamor Decay on Negative Feedback) |
| 1.2 | 2026-03-16 | Terminology update: Karma → Pamor, Worker → Jagoan, Customer → Pembuat Job. Added UI vs Technical terminology tables. |
| 1.3 | 2026-03-16 | Added Immutable Pamor Audit section with event sourcing and hash chain verification |

---

*Dokumen ini adalah Source of Truth. Untuk detail implementasi teknis, rujuk ke TECHNICAL.md. Untuk detail economics, rujuk ke ECONOMICS.md.*
