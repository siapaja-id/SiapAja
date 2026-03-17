# ECONOMICS.md

## Pusat Informasi Ekonomi & Fee Platform SiapAja.id

**Dokumen ini adalah "Source of Truth" untuk semua hal terkait finansial. Semua dokumen lain HARUS me-reference dokumen ini.**

---

## 1. Terminologi Data vs UI

| Teknis (Backend) | Publik (UI) | Deskripsi |
|------------------|-------------|-----------|
| `tasker` | **Pembuat Job** | User yang posting kebutuhan & bayar escrow. |
| `taskee` | **Jagoan** | User yang ambil job & ngerjain tugas. |
| `platform_fee` | **Biaya Kopi** | Fee operasional platform. |
| `solidarity_pool` | **Dana Solidaritas** | Asuransi komunitas untuk kecelakaan kerja. |

---

## 2. Tiered Fee Structure

Fee platform diterapkan secara progresif berdasarkan budget pekerjaan:

| Budget Pekerjaan | Fee Platform | Beban Biaya |
|------------------|--------------|-------------|
| Dibawah Rp500.000 | **3%** | 100% Pembuat Job (Jagoan 0% Fee) |
| Rp500rb - Rp2jt | **5%** | 50% Pembuat Job : 50% Jagoan |
| Rp2jt - Rp10jt | **7.5%** | 50% Pembuat Job : 50% Jagoan |
| Diatas Rp10jt | **10%** | 50% Pembuat Job : 50% Jagoan |

**Catatan:**
- Transaksi di bawah Rp500.000: Jagoan terima 100% utuh (Zero Commission)
- Fee transaksi besar dibebankan secara adil ke kedua belah pihak (split fee), bukan cuma memeras Jagoan

---

## 3. Fee Distribution (Alokasi Fee Platform)

Dari total fee yang terkumpulkan, distribusi adalah:

| Komponen | Persentase | Keterangan |
|----------|-----------|------------|
| **Community Treasury (Koperasi)** | **60%** | Alokasi utama untuk SHU, Dana Sosial, dan Operasional Koperasi. |
| **Technology Service (Solidarity-ID)** | **40%** | Biaya infrastruktur (Server) + R&D (Gaji Dev) + Lisensi IP. |

---

### 3.1 Transparency of Technology Fee (Solidarity-ID Allocation)
Sebagai bentuk transparansi radikal, alokasi 40% yang dikelola oleh **Solidarity-ID** (PT) diaudit secara terbuka dengan target distribusi sebagai berikut:

| Komponen Riset & Ops | Alokasi | Keterangan |
|-----------------------|---------|------------|
| **Infrastructure Hosting** | 15% | Biaya Cloud (SpacetimeDB, S3, Rust Nodes, API Gateway). |
| **R&D / Core Engineering** | 20% | Biaya pengembangan fitur baru, keamanan, dan optimasi algoritma (Gaji Devs). |
| **Compliance & IP Defense** | 5% | Biaya legalitas, audit keamanan (Pentest), dan perlindungan lisensi SSPL. |

**Prinsip Surplus Riset:** Jika terdapat sisa dari alokasi operasional server (efisiensi Rust), dana tersebut dialokasikan kembali ke dalam **Dana Inovasi** untuk pengembangan fitur perlindungan Jagoan berbasis AI.

---

## 4. Solidarity Pool (Dana Solidaritas)

- **Rate:** 1% auto-deduction dari setiap transaksi
- **Tujuan:** Asuransi komunitas untuk kecelakaan kerja
- **Mekanisme:** Setiap transaksi potong 1%, masuk ke Solidarity Pool di Virtual Ledger
- **Klaim:** Jagoan kecelakaan saat narik barang → klaim diambil dari pool ini lewat persetujuan pengurus Koperasi

---

## 3. Immutable Financial Ledger
Semua mutasi Rupiah (IDR) wajib masuk ke tabel `ledger_entries` yang bersifat *append-only*.

- **Hash Chain**: Tiap transaksi punya `previous_hash`.
- **Integrity**: Perubahan saldo tanpa entry di ledger dianggap invalid oleh logic Rust.
- **Transparency**: Tasker & Jagoan bisa nge-cek "Hash Bukti Transaksi" via dashboard.
- **Audit Trail**: Setiap entry mengandung SHA-256 hash yang bisa diverifikasi publik.

---

## 4. Xendit Escrow Integration

Platform menggunakan Xendit Escrow (berizin OJK) untuk menampung dana:

### Alur Pembayaran:
1. **Deposit:** Pembuat Job bayar via GoPay/OVO/VA/QRIS ke Xendit
2. **Escrow Lock:** Dana dikunci di escrow, tidak bisa diambil sampai pekerjaan selesai
3. **Release:** Jagoan terima uang (dikurangi 1% solidarity pool) langsung ke rekening bank/e-wallet
4. **Ledger Record:** Backend otomatis catat di `ledger_entries` dengan cryptographic hash

### Keuntungan:
- SiapAja tidak perlu izin OJK karena dana tidak pernah menyentuh rekening perusahaan
- Fraud risk ditanggung Xendit (licensed payment gateway)
- Compliance: Xendit sudah licensed sebagai Payment Gateway oleh BI
- **Auditability**: Jagoan bisa verifikasi sendiri transaksi mereka masuk ledger yang immutable

---

## 6. Revenue Streams (Triple-Engine)

### 5.1 Tiered Transaction Fee
Dari proyek besar (renovasi, borongan kantor). Pekerja harian tetap bebas potongan.

### 5.2 Premium Blue Check (Trust Signal)
User bisa beli verifikasi centang biru. **PENTING:** Ini murni signal, BUKAN garansi. Platform TIDAK bertanggung jawab atas kerugian finansial/fisik. Sengketa diselesaikan Juri Netizen.

### 5.3 Hyper-Local Ads Marketplace
Warung, laundry, toko bangunan di radius 2km bisa pasang iklan kontekstual di Feed. Iklan muncul pas user scroll nyari jasa relevan.

### 5.4 B2B API Integration
Mall atau Apartemen yang mau pakai worker kita secara borongan wajib langganan API (Enterprise Tier).

---

## 7. Auto-Yield Treasury

Dana Treasury (dari denda dan sisa fee) yang belum terpakai tidak dibiarkan nganggur:

- **Mekanisme:** Sistem otomatis memutar dana di instrumen Reksadana Pasar Uang berisiko rendah
- **Hasil (Yield):** Dibagikan sebagai dividen bulanan kepada pemegang Pamor tertinggi
- **Filosofi:** Pekerja bukan cuma buruh, mereka adalah Investor Ekosistem

---

## 8. Downvote sebagai Social Price Controller

Di feed **Demand**, Jagoan bisa downvote postingan Pembuat Job yang naruh harga "Ngehe" (misal: "Pindahan rumah 3 lantai, bayar 50rb").

### 7.1 Mekanisme Price Downvote
- **Threshold:** Kalau downvote tembus **-10 bersih**, postingan otomatis di-*hide* dari feed utama
- **Notifikasi:** Pembuat Job تلقى notif: *"Warga ngerasa harga lo kemurahan, coba cek AI Price Floor lagi deh."*
- **AI Price Floor:** Sistem AI bakal rekomendasikan harga fair berdasarkan histori pekerjaan sejenis di radius yang sama

### 7.2 Filosofi
Ini bukan monopoli harga. Ini **Social Correction** - komunitas punya suara buat ngasih tau kalau harga terlalu rendah (yang bisa menyebabkan worker melayani dengan kualitas rendah) atau terlalu tinggi (yang bisa tanda scam).

---

## 9. Tokenomics Roadmap

### Phase 1: Sertifikat Modal Digital (SMD)
- Status hukum: Bukti penyertaan modal anggota Koperasi Multi-Pihak
- Fokus: Kepatuhan regulasi Indonesia (Kemenkop & OJK)

### Phase 4: Global Tokenization ($SIAP)
- Status hukum: Token tata kelola yang dirilis melalui Offshore Foundation
- Mekanisme: Pemegang SMD memiliki hak prioritas untuk konversi (swap) menjadi $SIAP Token

---

## Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-16 | Initial creation - consolidated from readme.md & WHITEPAPER.md |
| 1.1 | 2026-03-16 | Added Bab 7: Downvote sebagai Social Price Controller (Anti-Price Gouging via Community Voting) |
| 1.2 | 2026-03-16 | Terminology update: Worker → Jagoan, Customer → Pembuat Job. Added UI vs Technical terminology tables. |
| 1.3 | 2026-03-16 | Added Immutable Financial Ledger section with hash chain architecture for audit trail |

---

*Dokumen ini adalah Source of Truth. Untuk detail implementasi teknis, rujuk ke TECHNICAL.md.*
