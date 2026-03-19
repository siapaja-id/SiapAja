# PAMOR-SYSTEM.md

## Konstitusi Reputasi & Protokol Kontribusi SiapAja.id

**Dokumen ini adalah "Single Source of Truth" untuk seluruh algoritma, metrik, dan logika sistem Pamor. Dokumen ini mengikat secara hukum algoritma bagi Pembuat Job, Jagoan, Juri, dan Kontributor.**

---

## 1. Filosofi Pamor
Pamor bukan sekadar "Rating Bintang". Pamor adalah **Sertifikat Kontribusi Digital**. 
- **Bukan Uang:** Pamor tidak bisa dibeli dengan Rupiah.
- **Bukan Hukuman Seumur Hidup:** Mengenal sistem *Decay* (Penebusan Dosa).
- **Multi-Dimensi:** Menilai profesionalisme (skill), moralitas (etika), dan integritas (kejujuran).

---

## 2. Metrik Pamor: Jagoan (Worker)

Jagoan mengumpulkan Pamor untuk mendapatkan prioritas *feed* dan hak dividen Koperasi.

| Aktivitas | Dampak Pamor | Keterangan |
|-----------|--------------|------------|
| Menyelesaikan Job | **+5 s/d +50** | Tergantung nilai budget & tingkat kesulitan AI. |
| Rating Bintang 5 | **+10** | Bonus kepuasan Pembuat Job. |
| Tepat Waktu (GPS) | **+5** | Terdeteksi otomatis saat tiba di radius 50m. |
| **Creator Conversion** | **+25** | Per 1 Pembuat Job baru yang posting job dari link konten TikTok/IG Jagoan. |
| Pembatalan Sepihak | **-20** | Membatalkan setelah klik "Terima" tanpa alasan valid. |
| Kalah Sengketa | **-50** | Diputuskan oleh Juri Netizen sebagai pihak bersalah. |
| Perilaku Buruk | **-10** | Laporan valid: Kasar, merusak barang, sampah. |

---

## 3. Metrik Pamor: Pembuat Job (Customer)

Pamor bagi Pembuat Job menentukan apakah postingan mereka akan diprioritaskan oleh Jagoan-Jagoan terbaik.

| Aktivitas | Dampak Pamor | Keterangan |
|-----------|--------------|------------|
| Konfirmasi Cepat | **+5** | Klik "Selesai" segera setelah pekerjaan beres. |
| Tips untuk Jagoan | **+2** | Memberikan apresiasi finansial di luar budget. |
| Deskripsi Jelas | **+3** | Memudahkan AI ekstraksi (minim revisi). |
| Harga "Ngehe" | **-5** | Postingan di-downvote massal (-10 net downvote). |
| PHP / Cancel Sering | **-15** | Membuat job tapi sering membatalkan saat Jagoan otw. |
| Eksploitasi | **-30** | Meminta kerjaan tambahan di luar deskripsi tanpa bayar. |

---

## 4. Metrik Pamor: Juri & Kontributor (Governance)

Integritas sistem dijaga oleh mereka yang meluangkan waktu untuk keadilan dan teknologi.

| Aktivitas | Dampak Pamor | Keterangan |
|-----------|--------------|------------|
| Voting Juri (Benar) | **+15** | Voting searah dengan hasil mayoritas final. |
| Voting Juri (Asal) | **-20** | Terdeteksi pola "ikut-ikutan" atau asal klik. |
| Menjadi Utusan RAK | **+50** | Terpilih sebagai delegasi kelompok dan aktif di RAT. |
| Menghadiri RAK | **+5** | Kehadiran aktif dalam diskusi kelompok (bukan cuma silent). |
| Penarikan Mandat | **-30** | Mandat dicabut oleh anggota kelompok karena tidak amanah. |
| Kontribusi Kode | **+100 s/d +1000** | Berdasarkan bobot PR yang di-merge di GitHub. |
| Laporan Bug Valid | **+10** | Laporan via GitHub Issue yang diverifikasi tim infra. |
| Bantuan Darurat | **+50** | Berdonasi/Membantu Jagoan yang kecelakaan saat kas Pool kosong. |

---

## 5. Struktur Tier & Akses (The Utility)

| Tier | Range Pamor | Akses & Keuntungan |
|------|-------------|--------------------|
| **Platinum** | 1000+ | Prioritas Job Premium (>2jt), Voting Power 10x, Dividen Maksimal. |
| **Gold** | 500 - 999 | Prioritas Job Menengah, Voting Power 5x, Dividen Menengah. |
| **Silver** | 100 - 499 | Akses Standar, Voting Power 1x, Dividen Dasar. |
| **Bronze** | 1 - 99 | Akun Baru / Pemulihan. Akses Terbatas, Tidak Ada Dividen. |
| **Frozen** | ≤ 0 | **Deaktivasi Otomatis.** Tidak bisa bidding, tidak bisa post. |

---

## 6. Protokol Deaktivasi & Penebusan (Redemption)

### 6.1 Otomasi Status Frozen (Mati)
Sistem (Backend Rust) akan secara otomatis mengubah status akun menjadi `is_frozen = true` jika:
1. **Pamor Negatif:** Akibat akumulasi sanksi.
2. **Inaktivitas Ekstrem:** Pamor menyusut via *Decay* hingga 0 selama 180 hari tanpa login.

### 6.2 Jalan Penebusan (The Redemption Path)
User yang berstatus **Frozen** tidak perlu bikin akun baru (KTP sudah terikat). Mereka bisa aktif kembali dengan:
1. **Jury Duty Sukarela:** Menjadi juri sengketa (tanpa dibayar Rupiah) hingga Pamor kembali ke angka +1.
2. **Kontribusi Komunitas:** Melaporkan konten ilegal atau membantu user baru di forum diskusi.
3. **Masa Inkubasi:** Menunggu *Pamor Decay* membersihkan poin negatif (hanya jika poin negatif berasal dari sanksi lama).

---

## 7. Mekanisme Anti-Gaming (Security)

Untuk mencegah Jagoan/Pembuat Job saling "tembak" (fake rating) demi menaikkan Pamor:
- **Radius Check:** Transaksi antara dua user yang berdekatan secara terus-menerus akan di-*flag* oleh AI.
- **IP & Device Binding:** Satu perangkat hanya untuk satu identitas KTP.
- **Pamor Weighting:** Rating dari user Tier Platinum memiliki bobot Pamor lebih besar dibanding user baru.
- **Cost of Downvote:** Memberikan downvote mengurangi 1 Pamor Anda sendiri (mencegah *brigading*).

---

## 8. Implementasi Teknis (Immutable Ledger)

Setiap perubahan Pamor **WAJIB** dicatat dalam `ledger_entries` dengan skema:
1. `entry_type`: 'pamor'
2. `payload`: `{ delta: i32, reason: String, current_score: i32 }`
3. `entry_hash`: SHA-256 link ke transaksi sebelumnya.

*Admin tidak memiliki tombol untuk mengubah Pamor secara manual. Pamor hanya berubah melalui aksi user yang valid secara kode.*

---

**Revision 1.0** - *Sistem ini dirancang untuk menciptakan keseimbangan antara ketegasan dan pengampunan.*
