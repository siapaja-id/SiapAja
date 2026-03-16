
<div align="center">

# 📜 WHITEPAPER v1.0: SiapAja.id
### Mendekonstruksi Monopoli Gig Economy melalui Efisiensi Engine Real-time dan Komputasi Tinggi.

**Penulis:** Core Developers & Komunitas SiapAja.id  
**Tanggal:** [Bulan, Tahun]  
**Status:** Draf Publik (Open for RFC - Request for Comments)

</div>

---

## 🛑 ABSTRAK
Sistem *gig economy* (ekonomi serabutan) di Indonesia saat ini telah berevolusi menjadi bentuk baru feodalisme digital. Platform raksasa (aplikasi ojol/jasa) yang awalnya bertindak sebagai inovator penghubung, kini bertransformasi menjadi makelar monopolistik yang mengekstraksi hingga 30% dari nilai keringat pekerja, sambil berlindung di balik ilusi "Kemitraan". 

**SiapAja.id** hadir sebagai infrastruktur publik alternatif. Melalui kombinasi **Rust** (untuk efisiensi server ekstrem), **SpacetimeDB** (sebagai state-engine untuk matching instan), dan antarmuka **React PWA** yang modern, kami mendesain ulang ekonomi pekerja lepas dengan **0% komisi untuk mikro, progresif fee untuk makro**. Whitepaper ini menjabarkan arsitektur teknis, model ekonomi anti-bakar duit, mekanisme *Decentralized Justice*, dan strategi monetisasi *Enterprise* (SSPL) yang memastikan keberlanjutan platform tanpa harus menghisap darah pekerja tingkat bawah.

---

## BAB 1: Latar Belakang & Masalah Struktural

### 1.1. Anatomi Penindasan Makelar Digital
Selama satu dekade terakhir, kita diajarkan bahwa untuk menghubungkan "Orang yang butuh jasa" dengan "Orang yang bisa kerja", harus ada perusahaan teknologi di tengahnya yang mengambil jatah 20% hingga 30%. 
*   **Cost of Trust:** Komisi 30% ini pada dasarnya adalah "Biaya Kepercayaan" agar uang *customer* aman dan pekerja dibayar. 
*   **Inefisiensi:** Seiring membesarnya platform, biaya tersebut tidak digunakan untuk menyejahterakan pekerja, melainkan untuk membayar gaji eksekutif, kampanye bakar uang (diskon), dan biaya operasional server (AWS/GCP) yang membengkak karena arsitektur *legacy* (Java/Node.js).

### 1.2. Kematian Transparansi Harga
Harga tidak lagi ditentukan oleh *Supply* dan *Demand* lokal, melainkan oleh algoritma kotak hitam (*black-box algorithm*) milik korporasi. Pekerja dipaksa menerima harga "Gacor" yang seringkali berada di bawah standar hidup layak, memicu fenomena perang harga yang tidak manusiawi.

---

## BAB 2: Solusi Fundamental (The SiapAja Paradigm)

### 2.1. Eliminasi Makelar melalui "Automated Virtual Ledger"
Jika fungsi utama platform adalah memastikan keamanan uang, maka fungsi tersebut bisa digantikan oleh sistem **Virtual Ledger** otomatis.
*   Uang *customer* dikunci di *Virtual Escrow* (Rekening Bersama otomatis).
*   Uang cair 100% ke pekerja begitu tugas selesai secara konsensus.
*   Tidak ada admin keuangan yang perlu menggaji dirinya sendiri dari proses persetujuan ini.

### 2.2. Pembalikan Model UX: "Demand-Driven Feed"
SiapAja.id membuang konsep "Katalog Jasa" (di mana pekerja memajang profil dan menunggu dipanggil). Kami menggunakan model **Hybrid Demand-Social Feed**.
* **Core Transaction:** Postingan kebutuhan bantuan yang di-lock dengan escrow.
* **Social Proof:** Mekanisme "Flex-to-Earn" di mana Jagoan memposting hasil kerja untuk menaikkan Pamor dan Trust.
* **Community Intelligence:** Diskusi lokal non-finansial yang menjaga retensi user (DAU/MAU) tanpa harus membakar duit iklan.
*   **Syarat:** Dana harus di-lock (Pay-to-Post) di awal. Feed terbebas dari *spam* dan order fiktif.
*   Pekerja di radius terdekat tinggal melakukan *claim* (ambil job) secara *first-come-first-serve* atau *bidding* wajar.

---

## BAB 3: Arsitektur Teknis (The "God Mode" Stack)

Untuk menjalankan platform dengan 0% komisi, biaya operasional infrastruktur harus ditekan hingga mendekati angka nol (Zero Marginal Cost).

### 3.1. Backend Komputasi Ekstrem (Rust + Axum)
Kami menolak penggunaan bahasa pemrograman *garbage-collected* yang rakus memori. Server inti SiapAja.id ditulis menggunakan **Rust**.
*   **Efisiensi VPS:** Sebuah server *virtual* seharga Rp100.000/bulan dengan RAM 1GB mampu menangani +50.000 koneksi bersamaan berkat asinkronisasi Tokio/Axum.
*   **Crash-Proof:** Jaminan *Memory Safety* Rust memastikan server tidak akan mengalami *Null Pointer Exception* di tengah malam.

### 3.2. Frontend Modern (Flutter + Riverpod + Dart)
Kami menggunakan **Flutter 3+** dengan **Riverpod** untuk state management dan **Dart** sebagai bahasa pemrograman.
*   **Native Mobile:** Aplikasi berjalan sebagai native app di Android/iOS dengan performa tinggi.
*   **Dual Database Architecture:**
  - **REST API (OpenAPI):** Untuk data persisten (PostgreSQL) - user profiles, transactions
  - **SpacetimeDB Binary Protocol:** Untuk data real-time - job feed, GPS tracking, live notifications. Menggunakan community Dart SDK yang production-ready.
*   **Type-Safe API:** Client Dart di-generate otomatis dari OpenAPI spec backend Rust.

### 3.3. State-Sync Engine (Self-hosted SpacetimeDB)
Kami menggunakan SpacetimeDB untuk menghapus latency antara worker dan customer.
*   **In-Memory Speed:** Logika matching (jarak & rating) diproses di memori, bukan query database konvensional yang lambat.
*   **ACID Persistence:** Meskipun kencang, data tetap aman tersimpan di disk.
*   **Real-time Sync:** Perubahan data langsung terpropagasi ke semua client dalam milidetik tanpa polling.

### 3.4. Jembatan Fiat (Xendit Escrow Integration)
*   **In-App Display:** User hanya melihat saldo dalam bentuk Rupiah (IDR).
*   **Escrow Reality:** Di belakang layar, **Xendit Escrow API** (berizin OJK sebagai payment gateway) menampung dana sementara. Saat penarikan (*withdraw*), proses langsung dieksekusi dalam hitungan detik.
*   **Keuntungan:** SiapAja tidak perlu izin OJK karena dana tidak pernah menyentuh rekening perusahaan - Xendit sebagai第三方 escrow provider.

---

## BAB 4: Kecerdasan Buatan sebagai Pelindung Keselamatan (K3)

> **📋 Source of Truth:** Untuk detail lengkap AI pipeline, LLM models, dan JSON schemas, lihat [AI-SPECS.md](./docs/AI-SPECS.md)

Kami menolak keras sistem lelang buta yang membuat pekerja saling banting harga hingga upah tidak manusiawi. Di sinilah **AI Man-Power Estimator** bekerja melalui **OpenRouter API** (Claude 3 Haiku, GPT-3.5) - text-only LLM untuk ekstraksi data terstruktur dari teks tidak terstruktur.

1.  **Text-to-Structured Extraction:** Saat *customer* memposting "Bantu angkut lemari 3 pintu ke lantai 4", LLM mengekstrak: budget, lokasi, estimasi difficulty.
2.  **Risk & Effort Calculation:** AI menghitung estimasi kalori, risiko cedera, dan harga pasar di wilayah tersebut berdasarkan teks saja.
3.  **Price Floor Enforcement:** AI akan mengunci **Harga Bawah**. *Pembuat Job* tidak bisa memposting tawaran di bawah harga minimum yang layak.
4.  **Squad Formation:** Jika beban melampaui kapasitas 1 manusia (>50kg) berdasarkan teks deskripsi, AI secara otomatis memecah job menjadi *Multi-Jagoan Job* (misal: butuh 3 orang).

---

## BAB 5: Model Ekonomi & Ekosistem Finansial

> **📋 Source of Truth:** Untuk detail lengkap fee structure, revenue streams, dan tokenomics, lihat [ECONOMICS.md](./docs/ECONOMICS.md)

Bagaimana *Founder*, Komunitas, dan Platform meraup keuntungan finansial jika komisi operasional adalah 0%? Ini adalah desain makroekonomi kita.

### 5.1. Evolusi Permodalan: Dari SMD ke $SIAP Token
SiapAja.id menggunakan strategi *Hybrid Entity* untuk menjembatani regulasi domestik dan likuiditas global.
*   **Phase 1: Sertifikat Modal Digital (SMD)**
    - Status hukum: Bukti penyertaan modal anggota Koperasi Multi-Pihak yang sah secara digital.
    - Fokus: Kepatuhan regulasi Indonesia (Kemenkop & OJK).
*   **Phase 4: Global Tokenization ($SIAP)**
    - Status hukum: Token tata kelola yang dirilis melalui *Offshore Foundation* untuk menarik modal global.
    - Mekanisme: Pemegang SMD memiliki hak prioritas untuk konversi (swap) menjadi $SIAP Token saat infrastruktur *Dual-Entity* diaktifkan.

### 5.2. Revenue Streams (The Management Carry)
Platform bukan hanya infrastruktur, tapi mesin ekonomi bagi para *Founder* dan Anggota.
*   **Pembuat Job-Side Admin Fee (3%):** Untuk transaksi <Rp500.000, biaya dibebankan 100% ke Pembuat Job. Jagoan menerima 100% utuh (Zero Commission Branding).
*   **The Founder's Carry (30%):** Dari total *Fee Platform* yang terkumpul, 30% dialokasikan langsung sebagai royalti IP dan biaya manajemen bagi tim *Founder* (Pengurus Inti).

### 5.3. Multi-Channel Revenue & Hyper-Local Ads
Platform membiayai dirinya sendiri tanpa memeras pekerja harian melalui:

1.  **Tiered Transaction Fee:** Transaksi di atas Rp500.000 dikenakan fee progresif (2.5% - 10%). Fee dibebankan secara adil (split 50:50) ke kedua belah pihak. Pekerja harian (mikro) tetap bebas potongan.

2.  **Contextual Ads Engine:** Sistem menyisipkan iklan hyper-local (radius <5km) yang relevan berdasarkan konteks pencarian. Contoh: User cari "Tukang AC" → Sistem tampilin iklan "Toko Sparepart AC Terdekat". Warung, laundry, toko bangunan bisa pasang iklan dengan budget lokal.

3.  **Identity-as-a-Service:** Biaya verifikasi Centang Biru sebagai filter trust. **PENTING:** Ini adalah *trust signal*, BUKAN garansi. Platform tidak bertanggung jawab atas kerugian finansial atau fisik.

### 5.4. Manajemen Likuiditas & Open Ledger (Transparansi Radikal)
Dana IDR yang tersimpan di *Escrow* menghasilkan *Float* (dana mengendap) yang dikelola secara kolektif.
*   **Auto-Yield Logic:** Dana diputar di instrumen berisiko rendah (SBN/Reksadana Pasar Uang). Hasilnya (Yield) 100% masuk ke kas Koperasi.
*   **Open Ledger Policy:** Setiap Rupiah yang masuk ke kas (dari yield, fee makro, atau ads) bisa di-audit secara real-time oleh semua anggota via aplikasi. Tidak ada "biaya siluman".
*   **Mekanisme SHU:** Sisa Hasil Usaha dibagikan secara otomatis via Virtual Ledger setiap akhir periode kepada anggota aktif (berdasarkan proporsi Pamor dan partisipasi), bukan cuma buat pemilik modal.

---

## BAB 6: Desentralisasi Keadilan (Pengadilan Netizen)

> **📋 Source of Truth:** Untuk detail lengkap dispute resolution dan jury selection, lihat [GOVERNANCE.md](./docs/GOVERNANCE.md)

Pusat panggilan (Call Center) yang berisi CS robot adalah sumber frustrasi terbesar di era modern. SiapAja.id menggantinya dengan **Decentralized Justice Protocol**.

1.  **Trigger Sengketa:** *Pembuat Job* merasa pekerjaan tidak sesuai, menekan tombol "Dispute". Dana otomatis terkunci (*Frozen*).
2.  **Jury Selection (Expertise-Based):** Sistem memilih 7 juri dari pool Jagoan yang punya **track record di kategori job yang sama**. Contoh: Sengketa "Tukang AC" → Juri adalah Jagoan rating tinggi di kategori "AC & Elektronik".
3.  **Blind Voting:** Juri diberikan waktu 24 jam untuk meninjau bukti foto *Sebelum/Sesudah* tanpa mengetahui identitas pelapor. 
4.  **Resolusi & Insentif:** Suara mayoritas (misal 4 vs 3) menjadi keputusan final yang dieksekusi oleh Virtual Ledger. Platform tidak intervensi. Keputusan juri adalah **FINAL**.

---

## BAB 7: Sistem Pamor (The Anti-Dystopian Social Credit)

> **📋 Source of Truth:** Untuk detail lengkap Pamor calculation, voting power, dan tier system, lihat [GOVERNANCE.md](./docs/GOVERNANCE.md)

Di SiapAja.id, kami memperkenalkan konsep **PAMOR** sebagai alternatif dari sistem Kredit Sosial yanghoror. Pamor bukan alat pengawasan negara, melainkan **sistem reputasi peer-to-peer** yang memberdayakan komunitas.

### 7.1. Mekanisme Pamor
*   **Akuisisi:** Pamor didapat dari penyelesaian tugas sukses, partisipasi juri, dan kontribusi kode (bagi *developer*).
*   **Pengurangan:** Pembatalan mendadak, keterlambatan parah, atau membuang sampah sembarangan di lokasi *Pembuat Job*.
*   **Decay Mechanism (Pengampunan):** Pamor buruk akan menyusut (hilang) dengan sendirinya sebesar 15% setiap bulan jika user kembali berkelakuan baik. Kami menghargai penebusan kesalahan, bukan menghukum seumur hidup.
*   **Utilitas:** Pamor tinggi memberikan prioritas tayangan di *Feed*, hak suara (*Voting Power*) untuk menentukan regulasi harga di wilayahnya, dan syarat mutlak pembagian dividen *Treasury*.

---

## BAB 8: Kerangka Hukum & Platform Non-Liability

### 8.1. Non-Liability Disclaimer (FFFI Principle)
SiapAja.id bertindak **MURNI sebagai penyedia infrastruktur kode** (Software-as-a-Service).

*   **Zero Responsibility:** Platform TIDAK bertanggung jawab atas kualitas kerja, penipuan antar user, kecelakaan kerja, atau kerugian finansial/fisik apapun.
*   **Jury-Led Resolution:** Semua sengketa diselesaikan secara *peer-to-peer* melalui sistem Juri Netizen. Keputusan juri adalah final dan mengikat secara algoritma.
*   **User Autonomy:** User setuju bahwa penggunaan aplikasi ini adalah **atas resiko sendiri** (*Use at your own risk*).

### 8.2. Struktur Koperasi Multi-Pihak
SiapAja.id menolak bentuk Perseroan Terbatas (PT) yang murni kapitalis. Platform akan didaftarkan sebagai **Koperasi Multi-Pihak**. Pekerja di lapangan, pengembang perangkat lunak, dan dewan pengawas memiliki porsi kepemilikan yang sah secara hukum negara, menghindari label "Mitra Palsu".

### 8.3. Strategi Lisensi Ganda (AGPL + SSPL)
Kode sumber kami adalah senjata komersial yang dilindungi secara hukum:
1.  **Untuk Rakyat (GNU AGPL v3):** Siapa pun (Universitas, LSM, RT/RW) bebas *fork* kode ini dan menjalankannya secara mandiri (gratis), selama mereka juga mempublikasikan modifikasinya ke ranah *open source*.
2.  **Untuk Korporasi Rakus (SSPL):** Jika sebuah BUMN, perusahaan raksasa multinasional, atau *startup unicorn* mencoba mengambil kode kami, mengganti logonya, dan menutup *source code*-nya untuk bisnis komersial pribadi... **Mereka secara hukum wajib membayar Lisensi Enterprise kepada pemegang Pamor Shares dengan nilai kontrak yang kami tentukan.** Ini adalah jalan raya menuju valuasi triliunan tanpa harus menjual jiwa komunitas.

---

## BAB 9: Peta Jalan (Roadmap)

*   **Fase 1: Asimilasi Kode (Q1 - Q2):** Penyelesaian inti Rust Axum, Flutter App, dan SpacetimeDB module. Uji coba integrasi Virtual Ledger.
*   **Fase 2: Hyper-Local Beta (Q3):** Peluncuran *real-money* secara eksklusif di 1 Kecamatan di Jabodetabek. Menguji keandalan *Price Floor AI* dan stabilitas *Virtual Escrow*.
*   **Fase 3: Kedaulatan Fiat & Ekspansi (Q4):** Integrasi BI-FAST untuk *bypass* biaya *Payment Gateway*. Ekspansi ke kota-kota lapis kedua.
*   **Fase 4: Era Enterprise (Tahun ke-2):** Pembukaan API komersial untuk korporasi, pendirian Koperasi resmi berskala nasional, dan pembagian dividen *Treasury* pertama.

---

## BAB 10: Kesimpulan (Panggilan Beraksi)

Monopoli *gig economy* saat ini bukan didasarkan pada teknologi yang mustahil dikalahkan, melainkan pada keengganan kita untuk bersatu membangun infrastruktur tandingan. 

**SiapAja.id** mengundang para *Rustaceans*, *Flutter Devs*, penggiat hukum, aktivis pekerja, dan siapa saja yang muak melihat tetangga mereka kelelahan di jalan raya hanya demi potongan 30% yang masuk ke gedung kaca di SCBD.

Kita tidak butuh miliaran Dolar dari investor asing untuk membuat sistem ini berjalan. Kita hanya butuh kompilasi kode yang bersih, konsensus algoritma yang adil, dan semangat gotong royong digital yang nyata.

**Pilih senjata kalian. Buka Pull Request. Mari kita ambil alih ekonomi ini.** 

---
*(Dokumen ini merupakan properti intelektual komunitas terbuka SiapAja.id. Silakan ajukan Issue di GitHub untuk mengusulkan amendemen).*
