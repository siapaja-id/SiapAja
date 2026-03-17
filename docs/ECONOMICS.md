# ECONOMICS.md

## Pusat Informasi Ekonomi & Fee Platform SiapAja.id

**Dokumen ini adalah "Source of Truth" untuk semua hal terkait finansial. Semua dokumen lain HARUS me-reference dokumen ini.**

---

## 1. Terminologi Data vs UI

| Teknis (Backend) | Publik (UI) | Deskripsi |
|------------------|-------------|-----------|
| `tasker` | **Pembuat Job** | User yang posting kebutuhan & bayar escrow. |
| `taskee` | **Jagoan** | User yang ambil job & ngerjain tugas. |
| `platform_fee` | **Kontribusi Koperasi** | Kontribusi transaksi untuk dana bersama & pengembangan. |
| `solidarity_pool` | **Dana Solidaritas** | Asuransi komunitas untuk kecelakaan kerja. |

---

## 2. Tiered Fee Structure

Koperasi tidak menarik iuran anggota. Sebagai gantinya, dana bersama (Treasury) dibangun melalui kontribusi progresif per transaksi:

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

## 3. Revenue & Technology Service Model

Seluruh porsi **Platform Fee** (100%) masuk ke dalam **Community Treasury (Kas Kopi)**. Kopi kemudian membayar biaya operasional teknologi kepada **Solidarity-ID** sebagai vendor infrastruktur eksklusif.

### 3.1 Komponen Biaya Teknologi (Triple-Tax Logic)
Solidarity-ID menagih Kopi berdasarkan penggunaan nyata di server lokal (Self-Hosted):

1. **SA-TEV (SiapAja Total Execution Value):** Biaya per unit komputasi (CPU/RAM/IO) pada setiap interaksi aplikasi. Solidarity-ID berhak mengatur harga unit SA-TEV secara mandiri.
2. **Success Transaction Fee:** Biaya tetap (Fixed Fee) per transaksi sukses untuk pemeliharaan ledger.
3. **Annual License Fee:** Biaya tahunan hak pakai kekayaan intelektual (SSPL) dan dukungan R&D inti.

### 3.2 Prinsip Surplus Optimasi (Non-Audit)
Solidarity-ID memiliki hak penuh atas efisiensi kode Rust yang dikembangkan. 
- Jika pengembang Solidarity-ID berhasil mengoptimasi kode sehingga penggunaan **SA-TEV** menjadi rendah (murah), margin keuntungan tetap milik Solidarity-ID sebagai insentif inovasi.
- Kopi tidak berhak meminta pengembalian atau audit atas margin efisiensi teknis Solidarity-ID.

### 3.3 Aliran Dana (The Flow)
1. **Platform Fee** ditarik dari transaksi besar → Masuk **100%** ke Kas Kopi.
2. **SA-TEV Tracker** mencatat beban kerja server per request/action.
3. **Invoice Bulanan:** Solidarity-ID menagih Kopi berdasarkan akumulasi SA-TEV + Transaction Fee.
4. **Settlement:** Kopi membayar tagihan dari Kas Treasury. Sisa saldo Treasury sepenuhnya milik Kopi untuk SHU dan asuransi.

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

### 5.4 Revenue Surplus: Partisipasi sebagai Modal
Karena tidak ada Simpanan Pokok, modal koperasi berasal dari akumulasi surplus transaksi. Anggota yang aktif bertransaksi secara otomatis meningkatkan "Penyertaan Modal" mereka yang tercatat di sistem sebagai SMD (Sertifikat Modal Digital) berbasis partisipasi.

### 5.5 B2B API Integration
Mall atau Apartemen yang mau pakai worker kita secara borongan wajib langganan API (Enterprise Tier).

---

## 7. Auto-Yield Treasury

Dana Treasury (dari denda dan sisa fee) yang belum terpakai tidak dibiarkan nganggur:

- **Mekanisme:** Sistem otomatis memutar dana di instrumen Reksadana Pasar Uang berisiko rendah
- **Hasil (Yield):** Dibagikan sebagai dividen bulanan kepada pemegang Pamor tertinggi
- **Filosofi:** Pekerja menjadi investor melalui keringat (jasa), bukan melalui iuran tunai.

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
- Status hukum: Bukti penyertaan modal berbasis akumulasi kontribusi transaksi (bukan setoran tunai).
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
