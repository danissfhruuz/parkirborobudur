# Dokumentasi Algoritma Sistem Parkir Borobudur

Dokumen ini dibuat berdasarkan pembacaan **READ-ONLY** terhadap source code project pada **24 September 2026**. Isinya hanya menjelaskan proses yang benar-benar ditemukan pada kode PHP. Tidak ada source code, database, konfigurasi, UI, atau struktur project yang diubah.

## Catatan Cara Membaca

- Nama tabel, kolom, status, fungsi, dan file ditulis mengikuti source code.
- Penjelasan query dibuat sederhana. Query tidak disalin seluruhnya karena tujuan dokumen ini adalah menjelaskan algoritma untuk laporan/UKK.
- `Menunggu Check-in` berarti booking online sudah dibuat tetapi belum diverifikasi petugas.
- `Aktif` berarti kendaraan sudah tercatat masuk dan sedang parkir.
- `Selesai` berarti checkout sudah dicatat.
- `Dibatalkan` berarti booking tidak lagi diproses.
- Jika ada perbedaan dengan SQL lama atau backup database, perilaku source code terbaru menjadi acuan dan diberi catatan **PERLU VERIFIKASI**.

## 1. Algoritma Registrasi User

### Algoritma Registrasi User

**Tujuan:** Membuat akun baru pada tabel `users`.

**File sumber:**
- `index login user/daftar.php`
- `koneksi.php`
- `csrf.php`

**Input:**
- `fullname`
- `username`
- `email`
- `phone`
- `password`
- `confirm_password`
- Token CSRF

**Proses:**
1. Sistem memulai session dan koneksi database.
2. Sistem memeriksa token CSRF pada request `POST`.
3. Sistem mengambil dan membersihkan data formulir.
4. Sistem memeriksa kolom wajib, format email, kesamaan password, dan panjang password minimal 6 karakter.
5. Sistem mencari `username` atau `email` yang sama pada tabel `users`.
6. Jika belum ada data yang sama, password diubah menjadi hash dengan `password_hash()`.
7. Sistem menyimpan `fullname`, `username`, `email`, `phone`, dan password hash ke tabel `users`.
8. Jika penyimpanan berhasil, sistem mengarahkan user ke `login.php?daftar=sukses`.

**Decision/Condition:**
- Apakah token dan data formulir valid?
  - **Ya:** lanjut ke pengecekan duplikasi.
  - **Tidak:** proses dihentikan dan pesan error ditampilkan.
- Apakah `username` atau `email` sudah ada?
  - **Ya:** registrasi ditolak karena akun sudah terdaftar.
  - **Tidak:** data user baru dimasukkan ke `users`.

**Output:**
- Akun baru tersimpan di tabel `users`, atau pesan kesalahan jika validasi gagal.

## 2. Algoritma Login User

### Algoritma Login User

**Tujuan:** Memeriksa identitas user dan membuat session user.

**File sumber:**
- `index login user/login.php`
- `koneksi.php`
- `csrf.php`

**Input:**
- Username atau email
- Password
- Token CSRF

**Proses:**
1. Sistem memeriksa apakah user sudah memiliki `user_id` di session.
2. Jika belum login, sistem menerima username/email dan password dari formulir.
3. Sistem memeriksa token CSRF dan memastikan kedua input tidak kosong.
4. Sistem mencari data pada tabel `users` dengan mencocokkan `username` atau `email`.
5. Sistem membandingkan password input dengan password hash memakai `password_verify()`.
6. Jika cocok, session ID dibuat ulang.
7. Sistem menyimpan `user_id`, `nama_user`, `username`, dan `plat_kendaraan` ke session.
8. Sistem mengarahkan user ke `dashboard-user.php`.

**Decision/Condition:**
- Apakah user sudah login?
  - **Ya:** langsung diarahkan ke `dashboard-user.php`.
  - **Tidak:** sistem melanjutkan pemeriksaan formulir.
- Apakah user ditemukan dan password benar?
  - **Ya:** session user dibuat dan login berhasil.
  - **Tidak:** pesan `Username atau password salah` ditampilkan.

**Output:**
- Session user aktif dan dashboard user terbuka, atau login ditolak.

## 3. Algoritma Login Staff

### Algoritma Login Staff

**Tujuan:** Memeriksa akun staff dan membuka session sesuai role `admin`, `petugas`, atau `owner`.

**File sumber:**
- `login-staff.php`
- `session-staff.php`
- `koneksi.php`
- `csrf.php`

**Input:**
- Username staff
- Password staff
- Token CSRF

**Proses:**
1. Sistem memeriksa apakah sudah ada session staff yang aktif.
2. Jika belum ada, sistem memeriksa token CSRF pada request `POST`.
3. Sistem mencari staff berdasarkan `username` pada tabel `staff`.
4. Sistem memeriksa password dengan `password_verify()`.
5. Sistem membaca role staff dari kolom `role` pada tabel `staff`.
6. Sistem memulai session khusus role menggunakan `staffSessionStart()`.
7. Session ID dibuat ulang untuk mengurangi risiko session fixation.
8. Sistem menyimpan `staff_id`, `staff_nama`, username, dan role ke session.
9. Sistem mengarahkan staff ke dashboard sesuai role.

**Decision/Condition:**
- Apakah ada session staff yang masih aktif?
  - **Ya:** staff diarahkan ke dashboard role yang sedang aktif.
  - **Tidak:** sistem melanjutkan proses login.
- Apakah username dan password benar?
  - **Ya:** session role dibuat.
  - **Tidak:** login ditolak dan pesan kesalahan ditampilkan.
- Apakah role termasuk `admin`, `petugas`, atau `owner`?
  - **Ya:** arah dashboard ditentukan dari role tersebut.
  - **Tidak:** role tidak diberi akses ke dashboard staff.

**Output:**
- Session staff sesuai role dan redirect ke dashboard, atau pesan login gagal.

## 4. Algoritma Validasi Role

### Algoritma Validasi Role

**Tujuan:** Membatasi halaman staff hanya untuk role yang berhak mengaksesnya.

**File sumber:**
- `session-staff.php`
- `index admin/dashboard-admin.php`
- `index petugas/dashboard-petugas.php`
- `index owner/dashboard-owner.php`
- Halaman lain pada folder `index admin`, `index petugas`, dan `index owner`

**Input:**
- Nama role halaman: `admin`, `petugas`, atau `owner`
- Cookie session role
- `staff_id` dan `staff_role` pada session

**Proses:**
1. Halaman staff memanggil `staffSessionStart()` dengan role yang sesuai.
2. Fungsi memilih nama cookie session berdasarkan role: `ADMIN_SESSID`, `PETUGAS_SESSID`, atau `OWNER_SESSID`.
3. Sistem memeriksa apakah `staff_id` tersedia.
4. Sistem membandingkan `staff_role` pada session dengan role halaman.
5. Jika valid, halaman melanjutkan query dan menampilkan data.
6. Jika tidak valid, halaman mengarahkan pengguna ke `login-staff.php`.
7. Fungsi `staffCurrentlyLoggedInRole()` digunakan untuk menemukan role yang sedang aktif saat logout staff.

**Decision/Condition:**
- Apakah `staff_id` ada dan role session sama dengan role halaman?
  - **Ya:** akses halaman diberikan.
  - **Tidak:** akses ditolak dan diarahkan ke login staff.
- Apakah cookie session role tersedia?
  - **Ya:** session role tersebut diperiksa.
  - **Tidak:** role tersebut dilewati.

**Output:**
- Halaman staff dapat dibuka oleh role yang benar, sedangkan role lain ditolak.

## 5. Algoritma Booking Parkir

### Algoritma Booking Parkir

**Tujuan:** Menyimpan pemesanan area dan slot parkir oleh user.

**File sumber:**
- `index login user/chek_in_user.php`
- `index login user/dashboard-user.php`
- `koneksi.php`
- `csrf.php`

**Input:**
- `user_id` dari session
- `areaId` atau `kode_area`
- Nomor plat kendaraan
- `slot_kode`
- Jenis kendaraan yang mengikuti jenis area
- Token CSRF

**Proses:**
1. Sistem memastikan user sudah login.
2. Sistem mencari area pada tabel `area_parkir` berdasarkan `kode_area`.
3. Sistem mengambil `id`, `nama_area`, `jenis`, `kapasitas`, dan `terisi` area.
4. Sistem mengambil daftar slot dari `slot_parkir` dan menentukan slot yang statusnya `aktif` serta belum dipakai.
5. Sistem memeriksa apakah user masih memiliki booking berstatus `Menunggu Check-in` atau `Aktif` pada hari berjalan.
6. Sistem memeriksa nomor plat dan slot yang dipilih.
7. Sistem membuat `kode_booking` dengan format acak `BK-...` dan mengecek ke tabel `booking` agar tidak duplikat.
8. Sistem menambahkan data ke tabel `booking` dengan `sumber = 'online'`, tanggal hari ini, dan status `Menunggu Check-in`.
9. Sistem mengarahkan user kembali ke dashboard dengan ID booking baru.

**Decision/Condition:**
- Apakah area ditemukan?
  - **Ya:** data area digunakan untuk proses booking.
  - **Tidak:** user dikembalikan ke dashboard dengan error area.
- Apakah user sudah memiliki booking berjalan?
  - **Ya:** booking baru ditolak.
  - **Tidak:** proses dilanjutkan.
- Apakah area masih memiliki sisa dan slot yang dipilih valid?
  - **Ya:** data booking disimpan.
  - **Tidak:** sistem menampilkan area penuh atau slot tidak tersedia.

**Output:**
- Satu baris booking baru pada tabel `booking` dengan status `Menunggu Check-in` dan kode booking.

## 6. Algoritma Pengecekan Ketersediaan Area/Slot

### Algoritma Pengecekan Ketersediaan Area/Slot

**Tujuan:** Menentukan apakah area dan slot dapat dipakai untuk booking atau check-in.

**File sumber:**
- `index login user/chek_in_user.php`
- `index petugas/checkin.php`
- `index petugas/manajemen-parkir.php`
- `index admin/manajemen-slot.php`

**Input:**
- `area_id`
- `slot_kode`
- Kapasitas dan `terisi` area
- Status slot
- Tanggal parkir
- Status booking

**Proses:**
1. Sistem mengambil data area dari `area_parkir`.
2. Sisa area dihitung dengan rumus `max(0, kapasitas - terisi)`.
3. Sistem mengambil slot milik area dari `slot_parkir`.
4. Slot dianggap valid jika milik area yang dipilih dan `status = 'aktif'`.
5. Sistem mencari booking dengan slot yang sama dan status `Aktif`.
6. Sistem juga memeriksa booking `Menunggu Check-in` pada `tanggal_parkir = CURDATE()`.
7. Untuk validasi area saat check-in, booking pending hari ini dapat ikut dihitung sebagai reservasi.
8. Sistem menolak area jika jumlah terpakai sudah mencapai `kapasitas`.

**Decision/Condition:**
- Apakah `terisi < kapasitas`?
  - **Ya:** area masih dianggap tersedia.
  - **Tidak:** area dianggap penuh.
- Apakah slot berstatus `aktif` dan belum dipakai?
  - **Ya:** slot boleh dipilih.
  - **Tidak:** slot ditolak karena nonaktif, maintenance, atau sudah dipakai.

**Output:**
- Informasi area/slot tersedia atau alasan penolakan.

## 7. Algoritma Verifikasi / Check-in oleh Petugas

### Algoritma Verifikasi / Check-in oleh Petugas

**Tujuan:** Mengubah booking online menjadi kendaraan yang sudah masuk, atau mencatat kendaraan walk-in.

**File sumber:**
- `index petugas/checkin.php`
- `booking-occupancy.php`
- `session-staff.php`
- `csrf.php`

**Input:**
- Untuk online: kode booking yang diketik atau diperoleh dari scanner QR.
- Untuk walk-in: nomor plat, jenis kendaraan, `area_id`, dan `slot_kode` opsional.
- `staff_id` petugas
- Token CSRF

**Proses:**
1. Petugas harus login dengan role `petugas`.
2. Untuk booking online, sistem mencari booking dari `kode_booking`; source juga dapat mencoba ID dari kode tampilan lama.
3. Sistem memastikan booking berstatus `Menunggu Check-in`.
4. Sistem memeriksa booking terbuka lain dengan nomor plat yang sama.
5. Sistem memeriksa kapasitas area dan ketersediaan slot.
6. Jika valid, tabel `booking` diperbarui menjadi `Aktif`, `tanggal = CURDATE()`, `jam_masuk = CURTIME()`, dan `checkin_staff_id` diisi.
7. `syncTerisiArea()` dipanggil untuk menambah `terisi` area.
8. Untuk walk-in, sistem memeriksa plat terbuka, area, dan slot lalu membuat booking baru dengan `user_id = NULL`, `sumber = 'walk_in'`, `jam_masuk = CURTIME()`, status `Aktif`, dan `checkin_staff_id` petugas.
9. Proses check-in menggunakan transaksi database dan melakukan commit jika semua pemeriksaan berhasil.

**Decision/Condition:**
- Apakah booking online ditemukan dan statusnya `Menunggu Check-in`?
  - **Ya:** verifikasi dilanjutkan.
  - **Tidak:** check-in online ditolak.
- Apakah plat, area, dan slot tidak bermasalah?
  - **Ya:** booking menjadi `Aktif`.
  - **Tidak:** transaksi dibatalkan dan pesan error ditampilkan.
- Apakah walk-in memiliki slot yang dipilih?
  - **Ya:** slot tersebut divalidasi.
  - **Tidak:** check-in tetap dapat dilakukan tanpa slot spesifik karena source mengizinkannya.

**Output:**
- Booking online berubah menjadi `Aktif`, atau booking walk-in baru tersimpan sebagai `Aktif`.
- `jam_masuk` dan petugas check-in tercatat.
- Occupancy area bertambah.

## 8. Algoritma Perubahan Status Booking

### Algoritma Perubahan Status Booking

**Tujuan:** Mengatur perpindahan status booking sesuai tindakan user, petugas, atau admin.

**File sumber:**
- `index login user/chek_in_user.php`
- `index login user/batalkan-boking.php`
- `index petugas/checkin.php`
- `index petugas/checkout.php`
- `index admin/transaksi.php`
- `booking-occupancy.php`

**Input:**
- ID atau kode booking
- Status lama
- Status tujuan
- User/staff yang sedang login
- Token CSRF

**Proses:**
1. Booking online dibuat dengan status `Menunggu Check-in`.
2. Petugas memverifikasi booking dan mengubahnya menjadi `Aktif`.
3. User dapat membatalkan booking yang masih `Menunggu Check-in` atau `Aktif` melalui `batalkan-boking.php`.
4. Petugas atau admin dapat menyelesaikan booking `Aktif` menjadi `Selesai` dan mengisi `jam_keluar`.
5. Admin pada `transaksi.php` juga memiliki aksi pembatalan untuk booking `Aktif`.
6. Pada perubahan yang memengaruhi area, `syncTerisiArea()` dipanggil.

**Decision/Condition:**
- `Menunggu Check-in` ke `Aktif`?
  - **Ya:** hanya setelah verifikasi petugas berhasil.
  - **Tidak:** status tetap atau proses ditolak.
- `Aktif` ke `Selesai`?
  - **Ya:** jam keluar dicatat dan occupancy dikurangi.
  - **Tidak:** checkout tidak diproses.
- `Menunggu Check-in` atau `Aktif` ke `Dibatalkan`?
  - **Ya:** status dibatalkan; occupancy dikurangi hanya jika status lama `Aktif`.
  - **Tidak:** data tidak diubah.

**Output:**
- Status booking menjadi salah satu status yang diizinkan: `Menunggu Check-in`, `Aktif`, `Selesai`, atau `Dibatalkan`.

## 9. Algoritma Update Occupancy

### Algoritma Update Occupancy

**Tujuan:** Menjaga kolom `area_parkir.terisi` sesuai perubahan status booking.

**File sumber:**
- `booking-occupancy.php`
- `index petugas/checkin.php`
- `index petugas/checkout.php`
- `index login user/batalkan-boking.php`
- `index admin/transaksi.php`

**Input:**
- `area_id`
- `statusLama`
- `statusBaru`
- `kapasitas` dan `terisi` area

**Proses:**
1. Fungsi `syncTerisiArea()` membandingkan status lama dan status baru.
2. Jika status lama bukan `Aktif` dan status baru `Aktif`, sistem menambah `terisi` satu.
3. Penambahan dibatasi dengan `LEAST(kapasitas, terisi + 1)`.
4. Jika status lama `Aktif` dan status baru `Selesai` atau `Dibatalkan`, sistem mengurangi `terisi` satu.
5. Pengurangan dibatasi dengan `GREATEST(0, terisi - 1)`.
6. Jika perpindahan status tidak termasuk dua kondisi tersebut, tidak ada perubahan occupancy.

**Decision/Condition:**
- Apakah perubahan masuk ke `Aktif`?
  - **Ya:** `terisi` bertambah maksimal sampai `kapasitas`.
  - **Tidak:** sistem memeriksa kondisi pengurangan.
- Apakah status lama `Aktif` dan status baru `Selesai` atau `Dibatalkan`?
  - **Ya:** `terisi` berkurang minimal sampai 0.
  - **Tidak:** `terisi` tidak berubah.

**Output:**
- Nilai `area_parkir.terisi` diperbarui tanpa melewati batas kapasitas dan tanpa menjadi negatif.

## 10. Algoritma Checkout

### Algoritma Checkout Parkir

**Tujuan:** Menyelesaikan transaksi kendaraan yang keluar dari area parkir.

**File sumber:**
- `index petugas/check-out.php`
- `index petugas/checkout.php`
- `index petugas/struck.php`
- `parkir-fee.php`
- `booking-occupancy.php`
- `index login user/selesaikan-boking.php`
- `index login user/selesai-parkir.php`

**Input:**
- ID booking
- Waktu masuk
- Waktu keluar
- Jenis kendaraan
- Tarif kendaraan
- Token CSRF untuk aksi perubahan

**Proses:**
1. Petugas melihat kendaraan yang masih berstatus `Aktif` pada `check-out.php`.
2. Petugas mengirim `booking_id` melalui `POST` ke `checkout.php`.
3. Sistem memulai transaksi dan mengunci baris booking yang diperiksa.
4. Sistem memastikan booking ada dan statusnya `Aktif`.
5. Sistem mengubah status booking menjadi `Selesai` dan mengisi `jam_keluar = CURTIME()`.
6. Jika perubahan berhasil, sistem memanggil `syncTerisiArea()` untuk status `Aktif` ke `Selesai`.
7. Transaksi database di-commit.
8. Sistem mengarahkan ke `struck.php?booking_id=...` untuk menampilkan tagihan dan struk.
9. User juga memiliki alur penyelesaian sendiri pada `selesaikan-boking.php` dan `selesai-parkir.php` dengan syarat booking milik user dan statusnya `Aktif`.

**Decision/Condition:**
- Apakah request menggunakan `POST` dan token CSRF valid?
  - **Ya:** proses checkout boleh berjalan.
  - **Tidak:** request ditolak.
- Apakah booking ditemukan dan masih `Aktif`?
  - **Ya:** status menjadi `Selesai`.
  - **Tidak:** sistem menampilkan bahwa booking tidak ditemukan atau sudah tidak aktif.
- Apakah update berhasil satu baris?
  - **Ya:** occupancy dikurangi dan struk dibuka.
  - **Tidak:** transaksi di-rollback dan checkout dianggap gagal.

**Output:**
- Status booking `Selesai`.
- `jam_keluar` tersimpan.
- Occupancy area berkurang.
- Halaman struk checkout terbuka.

## 11. Algoritma Perhitungan Durasi Parkir

### Algoritma Perhitungan Durasi Parkir

**Tujuan:** Menghitung lama kendaraan berada di area parkir dari waktu masuk dan waktu keluar.

**File sumber:**
- `parkir-fee.php`
- `index petugas/struck.php`
- `index petugas/riwayat-checkout.php`
- `index login user/struck.php`

**Input:**
- Tanggal parkir
- `jam_masuk`
- `jam_keluar`

**Proses:**
1. `parkirWaktu()` menggabungkan tanggal dan jam menjadi objek `DateTime`.
2. Sistem memeriksa apakah tanggal/jam kosong atau tidak valid.
3. Selisih waktu dihitung dalam detik dengan `waktuKeluar - waktuMasuk`.
4. Selisih negatif dicegah dengan `max(0, ...)` pada fungsi hitung tagihan.
5. Durasi jam dihitung dengan `intdiv(selisih_detik, 3600)`.
6. Sisa menit dihitung dari sisa detik setelah jam diambil.
7. Hasil ditampilkan dalam format `HH Jam MM Menit`.
8. Pada `parkirHitungTagihanDariData()`, jika waktu keluar lebih kecil daripada waktu masuk, waktu keluar ditambah satu hari.

**Decision/Condition:**
- Apakah tanggal dan jam valid?
  - **Ya:** durasi dihitung dari dua waktu tersebut.
  - **Tidak:** sistem memakai nilai aman dengan durasi `0` dan label `-`.
- Apakah waktu keluar lebih kecil dari waktu masuk?
  - **Ya:** dianggap melewati tengah malam dan ditambah satu hari.
  - **Tidak:** waktu digunakan apa adanya.

**Output:**
- `selisih_detik`, `durasi_jam`, `durasi_menit`, dan `durasi_label`.

## 12. Algoritma Perhitungan Tarif

### Algoritma Perhitungan Tarif Dasar

**Tujuan:** Menentukan tarif dasar sesuai jenis kendaraan menggunakan data pada tabel `settings`.

**File sumber:**
- `parkir-fee.php`
- `index admin/tarif-parkir.php`
- `index login user/chek_in_user.php`
- `index petugas/check-out.php`
- `index petugas/struck.php`
- `index login user/struck.php`

**Input:**
- Jenis kendaraan: `Mobil`, `Motor`, atau `Bus`
- Nilai `setting_value` pada `settings`
- Key tarif: `tarif_mobil`, `tarif_motor`, atau `tarif_bus`

**Proses:**
1. `parkirKonfigurasiKendaraan()` mencocokkan jenis kendaraan dengan key tarif dan batas denda.
2. Sistem membaca `setting_key` dan `setting_value` dari tabel `settings`.
3. Nilai yang kosong atau bukan angka tidak dipakai.
4. Nilai tarif negatif dibatasi menjadi 0.
5. Jika query tarif gagal, `parkir-fee.php` memakai nilai default: Mobil Rp5.000, Motor Rp3.000, dan Bus Rp10.000.
6. Tarif dasar dipilih berdasarkan key kendaraan yang aktif.

**Decision/Condition:**
- Apakah jenis kendaraan dikenali oleh `parkirKonfigurasiKendaraan()`?
  - **Ya:** key tarif dan aturan denda digunakan.
  - **Tidak:** konfigurasi tarif mengembalikan `null`; halaman pemanggil harus menangani kondisi tersebut.
- Apakah nilai tarif dari `settings` valid?
  - **Ya:** nilai database digunakan.
  - **Tidak:** nilai default dari `parkir-fee.php` digunakan.

**Output:**
- Tarif dasar kendaraan dalam rupiah dan key tarif yang dipakai.

## 13. Algoritma Perhitungan Denda

### Algoritma Perhitungan Denda Parkir

**Tujuan:** Menghitung denda ketika durasi melewati batas waktu jenis kendaraan.

**File sumber:**
- `parkir-fee.php`
- `index petugas/struck.php`
- `index petugas/riwayat-checkout.php`
- `index login user/struck.php`

**Input:**
- Durasi parkir dalam detik
- Batas waktu kendaraan
- Tarif denda per jam

**Proses:**
1. Sistem mengambil aturan kendaraan dari `parkirKonfigurasiKendaraan()`.
2. Batas Mobil adalah 8 jam dengan denda Rp1.000 per jam.
3. Batas Motor adalah 8 jam dengan denda Rp500 per jam.
4. Batas Bus adalah 10 jam dengan denda Rp2.000 per jam.
5. Sistem menghitung `kelebihan_detik = max(0, durasi_detik - batas_jam * 3600)`.
6. Sistem menghitung bagian jam untuk tampilan dengan pembagian bulat.
7. Untuk penagihan, `kelebihan_jam = ceil(kelebihan_detik / 3600)` jika ada kelebihan waktu.
8. Denda dihitung dengan `kelebihan_jam * denda_per_jam`.
9. Tidak ada pembulatan rupiah tambahan; pembulatan waktu dilakukan ke atas per jam kelebihan.

**Decision/Condition:**
- Apakah durasi melewati batas?
  - **Ya:** setiap bagian jam kelebihan dihitung sebagai satu jam denda melalui `ceil()`.
  - **Tidak:** `kelebihan_jam` dan denda bernilai 0.
- Apakah tarif denda negatif?
  - **Ya:** nilai dibatasi menjadi 0 dengan `max(0, ...)`.
  - **Tidak:** tarif denda digunakan.

**Output:**
- `kelebihan_detik`, `kelebihan_jam`, label kelebihan waktu, dan nilai `denda`.

## 14. Algoritma Perhitungan Total Pembayaran

### Algoritma Perhitungan Total Pembayaran

**Tujuan:** Menggabungkan tarif dasar dan denda menjadi total tagihan parkir.

**File sumber:**
- `parkir-fee.php`
- `index petugas/check-out.php`
- `index petugas/struck.php`
- `index petugas/riwayat-checkout.php`
- `index login user/struck.php`

**Input:**
- `tarif_dasar`
- Durasi parkir
- `batas_jam`
- `denda_per_jam`

**Proses:**
1. Sistem menghitung durasi dari `jam_masuk` dan `jam_keluar`.
2. Sistem menentukan tarif dasar dari jenis kendaraan.
3. Sistem menentukan denda berdasarkan kelebihan waktu.
4. Sistem menghitung total dengan rumus `total = tarif_dasar + denda`.
5. Nilai tarif dasar dan denda negatif dibatasi menjadi 0.
6. Total diformat dengan `number_format()` saat ditampilkan sebagai rupiah.

**Decision/Condition:**
- Apakah data waktu valid?
  - **Ya:** durasi dan denda dihitung.
  - **Tidak:** denda menjadi 0 dan tarif dasar tetap menjadi dasar total.
- Apakah ada kelebihan waktu?
  - **Ya:** total tarif dasar ditambah denda.
  - **Tidak:** total sama dengan tarif dasar.

**Output:**
- Nilai `tarif_dasar`, `denda`, dan `total` tagihan.

## 15. Algoritma Pembuatan/Tampilan Struk

### Algoritma Pembuatan dan Tampilan Struk

**Tujuan:** Menampilkan bukti detail booking dan hasil perhitungan tagihan.

**File sumber:**
- `index petugas/struck.php`
- `index login user/struck.php`
- `parkir-fee.php`

**Input:**
- `booking_id`
- `user_id` untuk struk user
- Data booking, area, slot, kendaraan, jam masuk, jam keluar, dan status
- Tarif pada `settings`

**Proses:**
1. Halaman menerima `booking_id` dari parameter request.
2. Sistem mengambil data booking dari tabel `booking` dan data area dari `area_parkir`.
3. Struk user dibatasi dengan `booking_id` dan `user_id` agar user hanya melihat booking miliknya.
4. Struk petugas mengambil booking yang selesai sesuai ID.
5. Sistem membaca tarif menggunakan fungsi pada `parkir-fee.php`.
6. Sistem menghitung durasi, denda, dan total tagihan.
7. Sistem menampilkan kode booking, kendaraan, area, slot, waktu, status, rincian tarif, denda, dan total.
8. Tombol/cetak struk hanya menampilkan atau mencetak hasil; source tidak menunjukkan transaksi pembayaran online.

**Decision/Condition:**
- Apakah `booking_id` valid dan data ditemukan?
  - **Ya:** rincian struk ditampilkan.
  - **Tidak:** halaman menghentikan proses dengan pesan struk tidak ditemukan.
- Apakah status dan `jam_keluar` sudah tersedia?
  - **Ya:** durasi checkout dapat dihitung.
  - **Tidak:** halaman mengikuti nilai aman yang disediakan helper atau menampilkan status yang ada.

**Output:**
- Tampilan struk parkir berisi detail booking, durasi, tarif dasar, denda, total, dan status.

## 16. Algoritma Riwayat Checkout

### Algoritma Riwayat Checkout

**Tujuan:** Menampilkan daftar kendaraan yang sudah selesai checkout dan menyediakan akses ke struk.

**File sumber:**
- `index petugas/riwayat-checkout.php`
- `index petugas/struck.php`
- `parkir-fee.php`

**Input:**
- Kata pencarian opsional
- Data booking dengan status `Selesai`
- `jam_keluar`
- Tarif dan waktu booking

**Proses:**
1. Sistem memastikan staff yang membuka halaman memiliki role `petugas`.
2. Sistem mengambil data dari `booking` dengan status `Selesai` dan `jam_keluar` tidak kosong.
3. Jika ada kata pencarian, sistem menambahkan penyaringan sesuai data yang dicari.
4. Sistem menghitung jumlah riwayat untuk statistik halaman.
5. Untuk setiap baris, sistem mengambil tarif berdasarkan `jenis_kendaraan`.
6. Sistem memanggil `parkirHitungTagihanDariData()` untuk menghitung durasi dan total.
7. Sistem menampilkan data riwayat dan tautan `struck.php?booking_id=...`.

**Decision/Condition:**
- Apakah booking berstatus `Selesai` dan `jam_keluar` tersedia?
  - **Ya:** booking masuk ke riwayat checkout.
  - **Tidak:** booking tidak ditampilkan pada riwayat ini.
- Apakah kata pencarian cocok?
  - **Ya:** baris ditampilkan.
  - **Tidak:** baris disaring dari hasil.

**Output:**
- Daftar riwayat checkout, durasi, total tagihan, status, dan tautan struk.

## 17. Algoritma Pengelolaan Area Parkir oleh Admin

### Algoritma Pengelolaan Area dan Slot Parkir oleh Admin

**Tujuan:** Menambah, mengubah, menghapus, dan memantau area serta slot parkir.

**File sumber:**
- `index admin/manajemen-parkir.php`
- `index admin/manajemen-slot.php`
- `koneksi.php`
- `csrf.php`

**Input:**
- Area: `kode_area`, `nama_area`, `lokasi`, `jenis`, `kapasitas`, `terisi`
- Slot: `area_id`, `kode_slot`, `status`, `keterangan`
- Aksi tambah, ubah status, ubah, atau hapus
- Token CSRF

**Proses:**
1. Sistem memvalidasi role `admin` dan token CSRF.
2. Admin menambah area; sistem mengecek `kode_area` agar tidak duplikat.
3. Area baru disimpan ke `area_parkir` dengan `terisi` awal 0.
4. Saat area diubah, sistem memvalidasi kapasitas dan terisi agar tidak negatif serta `terisi` tidak melebihi kapasitas.
5. Admin dapat menambah satu slot atau beberapa slot sekaligus ke `slot_parkir`.
6. Sistem mengecek kombinasi `area_id` dan `kode_slot` agar tidak duplikat.
7. Admin dapat mengubah status slot menjadi `aktif`, `nonaktif`, atau `maintenance` sesuai aksi pada source.
8. Sebelum area atau slot dihapus, sistem memeriksa booking yang masih menggunakan data tersebut.
9. Sistem mengambil ulang data area, kapasitas, terisi, sisa, dan persentase okupansi untuk tampilan.

**Decision/Condition:**
- Apakah kode area/slot sudah ada?
  - **Ya:** penyimpanan baru ditolak.
  - **Tidak:** data dapat ditambahkan.
- Apakah `terisi` negatif atau lebih besar dari `kapasitas`?
  - **Ya:** perubahan area ditolak.
  - **Tidak:** perubahan disimpan.
- Apakah area/slot masih dipakai booking aktif?
  - **Ya:** penghapusan dibatasi atau ditolak oleh validasi source.
  - **Tidak:** data dapat dihapus sesuai aksi admin.

**Output:**
- Data `area_parkir` dan `slot_parkir` tersimpan, diperbarui, atau dihapus sesuai validasi.
- Dashboard admin menampilkan kapasitas, terisi, sisa, dan persentase okupansi.

## 18. Algoritma Pengelolaan User oleh Admin

### Algoritma Pengelolaan User oleh Admin

**Tujuan:** Menampilkan statistik dan daftar user serta menghapus user yang memenuhi aturan source.

**File sumber:**
- `index admin/manajemen-user.php`
- `koneksi.php`
- `csrf.php`

**Input:**
- `user_id`
- Aksi `hapus_user`
- Token CSRF
- Data user pada tabel `users`

**Proses:**
1. Sistem memvalidasi role `admin` dan token CSRF.
2. Sistem menghitung jumlah seluruh user pada tabel `users`.
3. Sistem menghitung user baru pada bulan berjalan berdasarkan `created_at`.
4. Sistem menghitung user unik yang memiliki booking `Aktif`.
5. Sistem mengambil daftar user dan jumlah booking masing-masing dengan relasi `users` dan `booking`.
6. Saat admin meminta penghapusan, sistem menghitung jumlah booking milik user tersebut.
7. Jika user tidak memiliki riwayat booking, baris user dihapus dari `users`.
8. Jika masih memiliki riwayat booking, penghapusan ditolak agar riwayat tetap terjaga.

**Decision/Condition:**
- Apakah user memiliki riwayat booking?
  - **Ya:** user tidak boleh dihapus.
  - **Tidak:** user dapat dihapus.
- Apakah `user_id` valid?
  - **Ya:** proses statistik atau aksi diteruskan.
  - **Tidak:** aksi tidak dijalankan.

**Output:**
- Daftar dan statistik user tampil.
- Penghapusan hanya berhasil untuk user tanpa riwayat booking.

## 19. Algoritma Pengelolaan Tarif oleh Admin

### Algoritma Pengelolaan Tarif oleh Admin

**Tujuan:** Menampilkan dan memperbarui tarif dasar kendaraan yang digunakan perhitungan parkir.

**File sumber:**
- `index admin/tarif-parkir.php`
- `parkir-fee.php`
- `koneksi.php`
- `csrf.php`

**Input:**
- `tarif_mobil`
- `tarif_motor`
- `tarif_bus`
- Aksi `simpan_tarif`
- Token CSRF

**Proses:**
1. Sistem memvalidasi role `admin` dan token CSRF.
2. Sistem mengambil nilai tarif dari tabel `settings` berdasarkan tiga key tarif.
3. Admin mengisi nilai tarif baru pada formulir.
4. Nilai input diubah menjadi bilangan bulat oleh source.
5. Sistem menyimpan setiap key dengan proses insert atau update jika key sudah ada.
6. Fungsi `parkirAmbilTarif()` pada file `parkir-fee.php` membaca nilai yang sama saat menghitung tagihan.

**Decision/Condition:**
- Apakah aksi yang dikirim adalah `simpan_tarif`?
  - **Ya:** nilai tarif diproses dan disimpan.
  - **Tidak:** halaman hanya menampilkan data tarif.
- Apakah key tarif sudah ada di `settings`?
  - **Ya:** nilai `setting_value` diperbarui.
  - **Tidak:** baris setting baru dibuat.

**Output:**
- Tarif Mobil, Motor, dan Bus tersimpan pada tabel `settings` dan dapat dibaca oleh algoritma tagihan.

## 20. Algoritma Dashboard Owner / Rekap Pendapatan

### Algoritma Dashboard Owner dan Rekap Pendapatan

**Tujuan:** Menampilkan monitoring area, jumlah booking, status transaksi, dan estimasi pendapatan.

**File sumber:**
- `index owner/dashboard-owner.php`
- `index owner/laporan-owner.php`
- `index owner/keuangan-owner.php`
- `index owner/monitoring-owner.php`

**Input:**
- Data `area_parkir`: `kapasitas` dan `terisi`
- Data `booking`: `tanggal`, `status`, `jenis_kendaraan`, dan `area_id`
- Tarif aktif dari `settings`
- Rentang tanggal laporan jika dipilih

**Proses:**
1. Sistem memvalidasi role `owner` melalui session staff.
2. Sistem mengambil tarif aktif `tarif_mobil`, `tarif_motor`, dan `tarif_bus`.
3. Sistem mengambil area dan menghitung total kapasitas, total terisi, serta total tersedia.
4. Sistem menghitung jumlah booking berdasarkan `status` dan `jenis_kendaraan`.
5. Sistem menghitung kendaraan sedang parkir dari booking berstatus `Aktif`.
6. Untuk booking berstatus `Selesai`, sistem mengalikan jumlah booking dengan tarif aktif sesuai jenis kendaraan.
7. Laporan dan keuangan mengelompokkan data berdasarkan tanggal, jenis kendaraan, dan area sesuai halaman yang dibuka.
8. Grafik dan ringkasan periode dibentuk dari hasil pengelompokan tersebut.
9. Source secara eksplisit menyatakan nominal ini adalah estimasi karena database belum menyimpan pembayaran aktual.

**Decision/Condition:**
- Apakah status booking `Selesai`?
  - **Ya:** booking dihitung sebagai pendapatan estimasi.
  - **Tidak:** booking hanya masuk statistik status, bukan pendapatan selesai.
- Apakah kapasitas area lebih dari 0?
  - **Ya:** persentase okupansi dihitung dari `terisi / kapasitas * 100`.
  - **Tidak:** persentase ditampilkan 0.
- Apakah tanggal masuk dalam rentang laporan?
  - **Ya:** data masuk rekap periode.
  - **Tidak:** data tidak ikut dihitung.

**Output:**
- Dashboard monitoring area.
- Jumlah booking per status dan jenis kendaraan.
- Rekap transaksi.
- Estimasi pendapatan, bukan pembayaran aktual.

## 21. Algoritma Logout

### Algoritma Logout User dan Staff

**Tujuan:** Mengakhiri session user atau staff dengan aman.

**File sumber:**
- `index login user/logout.php`
- `logout.php`
- `session-staff.php`
- `csrf.php`

**Input:**
- Request `POST`
- Token CSRF
- Session user atau cookie session staff

**Proses:**
1. Sistem hanya menerima logout melalui `POST`.
2. Sistem memeriksa token CSRF.
3. Logout user mengosongkan `$_SESSION` dan menjalankan `session_destroy()`.
4. Logout staff mencari role aktif melalui `staffCurrentlyLoggedInRole()`.
5. Sistem membuka session role tersebut dengan `staffSessionStart()`.
6. Sistem mengosongkan session staff dan menjalankan `session_destroy()`.
7. Sistem mengarahkan kembali ke `login.php` untuk user atau `login-staff.php` untuk staff.

**Decision/Condition:**
- Apakah method request `POST`?
  - **Ya:** token CSRF diperiksa.
  - **Tidak:** request ditolak dengan HTTP 405.
- Apakah token CSRF valid?
  - **Ya:** session dihancurkan.
  - **Tidak:** logout dihentikan dengan pesan token tidak valid.
- Apakah staff role ditemukan?
  - **Ya:** hanya session role tersebut yang dihancurkan.
  - **Tidak:** sistem langsung kembali ke halaman login staff.

**Output:**
- Session login berakhir dan halaman login ditampilkan.

## 22. Algoritma Penting Lain yang Ditemukan

### Algoritma Antrean dan Konfirmasi QR Booking

**Tujuan:** Menampilkan posisi booking user dan menyediakan data QR yang dapat dicari petugas, tanpa menganggap QR sebagai pembayaran atau verifikasi otomatis.

**File sumber:**
- `index login user/antrian-parkir.php`
- `index login user/konfirmasi-qr.php`
- `index petugas/checkin.php`
- `csrf.php`

**Input:**
- `booking_id`
- `user_id` dari session user
- Data booking, area, slot, plat, dan status
- Hasil pembacaan QR atau kode booking pada sisi petugas
- Token CSRF untuk konfirmasi QR

**Proses:**
1. `antrian-parkir.php` mengambil booking berdasarkan `booking_id` dan `user_id` agar user hanya melihat booking miliknya.
2. Sistem membuat kode tampilan dari `kode_booking`; untuk data lama yang kosong, source memakai fallback berbasis ID.
3. Posisi antrean dihitung dari jumlah booking `Aktif` pada area yang sama dan dibuat lebih dulu, lalu ditambah satu.
4. Sistem membuat payload QR yang berisi data tampilan seperti kode booking, kendaraan, dan area.
5. Payload dikirim ke layanan pembuat gambar QR melalui URL pada source; QR hanya menjadi tampilan data tiket/check-in.
6. Saat user menekan konfirmasi, `konfirmasi-qr.php` memastikan booking tersebut milik user yang login.
7. Jika valid, kolom `qr_shown` pada tabel `booking` diubah menjadi 1.
8. Petugas dapat memasukkan kode secara manual atau menggunakan scanner kamera pada `checkin.php`.
9. Setelah kode dibaca, server tetap mencari dan memvalidasi booking, status, plat, area, dan slot sebelum check-in.

**Decision/Condition:**
- Apakah booking milik user yang login?
  - **Ya:** data antrean/QR dapat ditampilkan atau `qr_shown` diperbarui.
  - **Tidak:** data tidak ditemukan dan tidak boleh diubah.
- Apakah hasil scan memiliki format QR yang dikenali?
  - **Ya:** kode booking diambil dan dikirim untuk pencarian server.
  - **Tidak:** scanner menampilkan QR tidak valid dan petugas dapat memakai input manual.
- Apakah booking hasil pencarian lolos validasi server?
  - **Ya:** proses dilanjutkan ke verifikasi check-in.
  - **Tidak:** check-in ditolak.

**Output:**
- Posisi antrean, tampilan QR, atau nilai `qr_shown = 1`.
- Kode booking yang dapat dipakai petugas untuk proses verifikasi.

## Catatan Keamanan dan Database

- Form dan request yang mengubah data menggunakan CSRF token dari `csrf.php`.
- Password user dan staff disimpan sebagai hash melalui `password_hash()` dan diverifikasi dengan `password_verify()`.
- Query penting menggunakan prepared statement MySQLi pada banyak bagian source.
- Source checkout petugas memakai transaksi dan `FOR UPDATE`; booking online user tidak memakai mekanisme lock/transaksi penuh yang sama saat reservasi. **PERLU VERIFIKASI** jika sistem dipakai pada banyak request bersamaan karena ada potensi dua request memilih slot yang sama.
- Backup SQL lama memiliki trigger occupancy, sedangkan source PHP juga memanggil `syncTerisiArea()`. **PERLU VERIFIKASI** agar occupancy tidak bertambah atau berkurang dua kali.
- Backup SQL lama tidak sepenuhnya mencerminkan source terbaru, termasuk beberapa kolom seperti `kode_booking`, `sumber`, `tanggal_parkir`, `checkin_staff_id`, `qr_shown`, dan status `Menunggu Check-in`. **PERLU VERIFIKASI** sebelum backup lama dipakai sebagai schema instalasi.
- Source tidak menunjukkan payment gateway, pembayaran online, atau tabel pembayaran aktual. Untuk algoritma pembayaran online: **Tidak ditemukan implementasi yang dapat diverifikasi di source code.**
- Source tidak menunjukkan audit log formal. Untuk algoritma audit log: **Tidak ditemukan implementasi yang dapat diverifikasi di source code.**
- File `struck.php` di root merupakan implementasi legacy dengan jalur dan formula berbeda dari `index petugas/struck.php` serta `index login user/struck.php`. **PERLU VERIFIKASI** jika file legacy tersebut masih digunakan.

## Ringkasan Alur Sistem

1. User membuat akun pada `users` lalu login.
2. User memilih area dan slot yang tersedia.
3. Sistem membuat booking online dengan status `Menunggu Check-in`.
4. Petugas mencari kode booking atau membaca QR, tetapi validasi akhir tetap dilakukan server.
5. Jika valid, status berubah menjadi `Aktif`, `jam_masuk` diisi, dan `area_parkir.terisi` bertambah.
6. Saat kendaraan keluar, checkout mengubah status menjadi `Selesai`, mengisi `jam_keluar`, dan mengurangi occupancy.
7. Durasi, tarif dasar, denda, dan total dihitung oleh fungsi pada `parkir-fee.php`.
8. Struk dan riwayat checkout menampilkan hasil perhitungan tersebut.
9. Owner melihat statistik dan pendapatan estimasi dari booking `Selesai` dikalikan tarif aktif.

### Ringkasan Registrasi/Login

- User melakukan registrasi dengan mengisi `fullname`, `username`, `email`, `phone`, `password`, dan `confirm_password`.
- Sistem memeriksa data wajib, format email, kesamaan password, panjang password minimal 6 karakter, serta duplikasi `username` atau `email` pada tabel `users`.
- Password user disimpan dalam bentuk hash menggunakan `password_hash()`.
- Saat login, user dapat menggunakan `username` atau `email` bersama password.
- Sistem mencari data pada tabel `users`, lalu memeriksa password menggunakan `password_verify()`.
- Jika berhasil, sistem membuat session berisi `user_id`, `nama_user`, `username`, dan `plat_kendaraan`, kemudian mengarahkan user ke `dashboard-user.php`.
- Registrasi dan login memakai token CSRF pada request `POST`.
- Urutan alur end-to-end sesuai implementasi source code:
  1. **Registrasi User:** Calon pengguna membuat akun di `index login user/daftar.php`.
  2. **Login User:** Pengguna login di `index login user/login.php` (tabel `users`), sistem membuat session pengguna (`user_id`).
  3. **Penentuan Role Staff (Khusus Staff):** Staff login terpisah di `login-staff.php` (tabel `staff`), sistem membaca kolom `role` (`admin`, `petugas`, atau `owner`) lalu mengarahkan ke dashboard masing-masing role.
  4. **Pemesanan / Booking:** User memilih area dan slot di `index login user/chek_in_user.php`, sistem membuat transaksi pada tabel `booking`.
  5. **Status Menunggu Check-in:** Booking online tersimpan dengan status awal `Menunggu Check-in` dan jam masuk belum diisi.
  6. **Verifikasi Petugas:** Petugas memeriksa booking online (atau memproses walk-in) di `index petugas/checkin.php`.
  7. **Status Aktif:** Status booking diubah menjadi `Aktif`, `jam_masuk = CURTIME()`, dan occupancy bertambah (`syncTerisiArea()`).
  8. **Kendaraan Parkir:** Kendaraan berada di area parkir dan muncul pada monitoring kendaraan aktif.
  9. **Checkout:** Petugas memproses keluar di `index petugas/checkout.php`.
  10. **Status Selesai:** Sistem mengubah status menjadi `Selesai`, mengisi `jam_keluar = CURTIME()`, mengurangi occupancy area, lalu commit transaksi.
  11. **Hitung Durasi:** Halaman struk/riwayat (`index petugas/struck.php` & `parkir-fee.php`) menghitung durasi dari `jam_masuk` dan `jam_keluar`.
  12. **Hitung Tarif + Denda:** Sistem membaca tarif dari tabel `settings`, menghitung batas toleransi jam, denda jika ada kelebihan waktu, dan total pembayaran.
  13. **Struk:** Rincian data transaksi, waktu, denda, dan total tagihan ditampilkan pada struk checkout.
  14. **Riwayat:** Data tersimpan pada database dan dapat dilihat kembali di `index petugas/riwayat-checkout.php` maupun riwayat user.
- Catatan audit urutan: Pada source code, status berubah menjadi **Selesai** terlebih dahulu di `checkout.php` saat jam keluar dicatat ke database. Perhitungan **Hitung Durasi** dan **Hitung Tarif + Denda** dieksekusi setelahnya pada saat halaman **Struk** atau **Riwayat** memuat data booking yang sudah selesai tersebut.
## Daftar File Sumber

Berikut adalah daftar file PHP utama dalam source code yang menjadi dasar dokumentasi algoritma ini:

### 1. Modul Autentikasi, Session, & Keamanan
- `koneksi.php`: Konfigurasi basis data MySQLi, zona waktu server (`Asia/Jakarta`), dan penyesuaian host lingkungan.
- `csrf.php`: Pengelolaan token CSRF (pembuatan token 32 byte dan validasi dengan `hash_equals()`).
- `session-staff.php`: Isolasi multi-session staff (`ADMIN_SESSID`, `PETUGAS_SESSID`, `OWNER_SESSID`) dan pendeteksi role aktif.
- `login-staff.php`: Autentikasi dan pengalihan dashboard berdasarkan kolom `role` pada tabel `staff`.
- `logout.php`: Penghentian session login staff aktif melalui metode `POST` dengan verifikasi token CSRF.
- `index login user/daftar.php`: Registrasi pengguna baru, enkripsi password menggunakan `password_hash()`, dan validasi duplikasi.
- `index login user/login.php`: Autentikasi pengguna pada tabel `users`, verifikasi dengan `password_verify()`, dan pembentukan session user.
- `index login user/logout.php`: Penghentian session pengguna melalui metode `POST` dengan proteksi token CSRF.

### 2. Modul Operasional Booking & Kendaraan
- `booking-occupancy.php`: Fungsi `syncTerisiArea()` untuk sinkronisasi nilai `terisi` pada tabel `area_parkir` saat status booking berubah.
- `parkir-fee.php`: Logika perhitungan durasi parkir, penentuan tarif dasar, perhitungan denda kelebihan jam (`ceil()`), dan total tagihan.
- `index login user/chek_in_user.php`: Pembuatan reservasi booking online oleh pengguna (status `Menunggu Check-in`).
- `index login user/antrian-parkir.php`: Pemantauan antrean booking oleh pengguna dan pembuatan payload gambar QR tiket.
- `index login user/konfirmasi-qr.php`: Penandaan kolom `qr_shown = 1` saat QR tiket telah ditampilkan/dikonfirmasi pengguna.
- `index login user/batalkan-boking.php`: Pembatalan booking oleh pengguna (status `Dibatalkan`) beserta penyesuaian okupansi area.
- `index login user/selesaikan-boking.php`: Penyelesaian booking secara mandiri oleh pengguna yang bersangkutan.
- `index petugas/checkin.php`: Pemeriksaan booking online, pemindaian QR kamera, dan pendaftaran kendaraan walk-in menjadi status `Aktif`.
- `index petugas/check-out.php`: Antarmuka monitoring kendaraan yang sedang parkir (status `Aktif`) untuk persiapan checkout.
- `index petugas/checkout.php`: Eksekusi transaksi perubahan status kendaraan dari `Aktif` menjadi `Selesai` dan pencatatan `jam_keluar`.
- `index petugas/struck.php`: Pembuatan dan penampilan bukti struk checkout kendaraan di sisi petugas.
- `index login user/struck.php`: Tampilan bukti struk parkir khusus untuk pengguna pemilik booking.
- `index petugas/riwayat-checkout.php`: Rekapitulasi transaksi kendaraan yang berstatus `Selesai` dengan jam keluar valid.

### 3. Modul Admin (Pengelolaan Master Data)
- `index admin/dashboard-admin.php`: Ringkasan statistik operasional, status harian, dan notifikasi untuk admin.
- `index admin/manajemen-parkir.php`: Tambah, ubah, dan hapus master data area parkir pada tabel `area_parkir`.
- `index admin/manajemen-slot.php`: Tambah slot tunggal/massal dan pembaruan status slot pada tabel `slot_parkir`.
- `index admin/tarif-parkir.php`: Pengaturan nilai tarif dasar kendaraan pada tabel `settings`.
- `index admin/manajemen-user.php`: Pemantauan daftar user terdaftar dan validasi penghapusan akun tanpa riwayat transaksi.
- `index admin/transaksi.php`: Monitoring dan penyesuaian status seluruh transaksi booking di sistem oleh admin.

### 4. Modul Owner (Monitoring & Laporan)
- `index owner/dashboard-owner.php`: Dashboard eksekutif monitoring area, okupansi zona, dan ringkasan transaksi.
- `index owner/laporan-owner.php`: Rekapitulasi laporan transaksi berkala berdasarkan status dan tipe kendaraan.
- `index owner/keuangan-owner.php`: Estimasi pendapatan keuangan dari akumulasi transaksi berstatus `Selesai` dikalikan tarif aktif.
- `index owner/monitoring-owner.php`: Pemantauan langsung daftar kendaraan yang sedang berstatus `Aktif` di seluruh area parkir.
## Daftar File Penting yang Benar-Benar Dianalisis

Berdasarkan audit teknis secara **READ-ONLY**, berikut adalah 25 file penting inti (core files) yang dianalisis secara mendalam per baris kode untuk menyusun dokumentasi algoritma ini:

| No | File Sumber | Kategori / Modul | Peran Utama & Logika Kunci yang Dianalisis |
|---|---|---|---|
| 1 | `koneksi.php` | Sistem & Basis Data | Koneksi MySQLi, set timezone `Asia/Jakarta`, penanganan port 3307 & host lingkungan. |
| 2 | `csrf.php` | Keamanan | Pembuatan token acak 32-byte dan validasi timing-attack safe via `hash_equals()`. |
| 3 | `session-staff.php` | Manajemen Sesi | Isolasi cookie multi-role (`ADMIN_SESSID`, `PETUGAS_SESSID`, `OWNER_SESSID`). |
| 4 | `login-staff.php` | Autentikasi Staff | Validasi login staff, pencocokan hash password, dan redirect berbasis kolom `role`. |
| 5 | `logout.php` | Manajemen Sesi | Logout staff aktif secara aman melalui proteksi CSRF dan pembersihan sesi. |
| 6 | `booking-occupancy.php` | Operasional / Okupansi | Logika fungsi `syncTerisiArea()` dengan batasan `LEAST()` dan `GREATEST()`. |
| 7 | `parkir-fee.php` | Finansial & Tarif | Logika durasi, tarif jenis kendaraan, pembulatan denda `ceil()`, dan total bayar. |
| 8 | `index login user/daftar.php` | Registrasi Pengguna | Validasi input, cek duplikasi email/username, dan enkripsi `password_hash()`. |
| 9 | `index login user/login.php` | Autentikasi Pengguna | Login multi-identifier (email/username), verifikasi password, dan sesi user. |
| 10 | `index login user/logout.php` | Manajemen Sesi | Pembersihan sesi pengguna terproteksi token CSRF. |
| 11 | `index login user/chek_in_user.php` | Booking Pengguna | Pengecekan sisa kuota, pemilihan slot aktif, generate kode booking `BK-...`. |
| 12 | `index login user/antrian-parkir.php` | Antrean & QR | Perhitungan posisi antrean riil dan generate string format payload QR. |
| 13 | `index login user/konfirmasi-qr.php` | Konfirmasi Tiket | Mutasi kolom `qr_shown = 1` dengan verifikasi kepemilikan user. |
| 14 | `index login user/batalkan-boking.php` | Pembatalan Transaksi | Pembatalan booking oleh user dan penyesuaian otomatis counter `terisi`. |
| 15 | `index login user/struck.php` | Bukti Transaksi | Struk parkir sisi pengguna dengan rincian biaya dari `parkir-fee.php`. |
| 16 | `index petugas/checkin.php` | Operasional Petugas | Verifikasi booking online, scanner kamera, dan check-in walk-in (`Aktif`). |
| 17 | `index petugas/check-out.php` | Operasional Petugas | Antarmuka monitoring kendaraan `Aktif` dengan live preview durasi dan tarif. |
| 18 | `index petugas/checkout.php` | Operasional Petugas | Transaksi DB (`FOR UPDATE`), set `jam_keluar = CURTIME()`, dan status `Selesai`. |
| 19 | `index petugas/struck.php` | Bukti Transaksi | Cetak struk checkout resmi petugas berserta rincian denda overstay. |
| 20 | `index petugas/riwayat-checkout.php` | Rekapitulasi Petugas | Query log kendaraan selesai (`jam_keluar IS NOT NULL`) dan filter pencarian. |
| 21 | `index admin/manajemen-parkir.php` | Master Data Admin | CRUD area parkir, validasi kapasitas vs terisi, dan visualisasi marker peta. |
| 22 | `index admin/manajemen-slot.php` | Master Data Admin | Pembuatan slot tunggal/massal serta pengubahan status `aktif`/`maintenance`. |
| 23 | `index admin/tarif-parkir.php` | Finansial Admin | Pembaruan tarif dasar kendaraan pada tabel `settings` via upsert. |
| 24 | `index admin/manajemen-user.php` | Manajemen Akun Admin | Monitoring daftar user dan perlindungan integritas data (blokir hapus user aktif). |
| 25 | `index owner/dashboard-owner.php` | Eksekutif Owner | Agregasi statistik harian, okupansi zona, dan estimasi omzet dari booking `Selesai`. |

## Catatan Verifikasi

Bagian ini merangkum poin-poin hasil audit teknis mendalam terhadap source code yang memerlukan perhatian atau verifikasi lanjutan sebelum sistem diuji atau di-deploy:

1. **Perbedaan Skema Database Terbaru vs File Backup SQL:**
   - **Status:** `PERLU VERIFIKASI`
   - **Keterangan:** File cadangan database lama tidak memuat beberapa kolom penting yang aktif dipakai pada source code PHP terbaru, antara lain: `booking.kode_booking`, `booking.sumber`, `booking.tanggal_parkir`, `booking.checkin_staff_id`, `booking.qr_shown`, status `'Menunggu Check-in'`, serta tabel `ulasan`. Pastikan database yang berjalan di server lokal/hosting sudah menggunakan skema termutakhir.

2. **Sinkronisasi Okupansi Area (Trigger vs Logika PHP):**
   - **Status:** `PERLU VERIFIKASI`
   - **Keterangan:** Pada backup SQL lama terdapat trigger database yang otomatis mengubah counter `area_parkir.terisi`, sementara pada source code PHP terbaru pembaruan dilakukan secara eksplisit memanggil fungsi `syncTerisiArea()` di `booking-occupancy.php`. Harus diverifikasi agar trigger lama dinonaktifkan supaya nilai `terisi` tidak terhitung berkurang atau bertambah dua kali lipat (double count).

3. **Status Booking & Jam Keluar pada Checkout:**
   - **Status:** Terverifikasi Sesuai Source Code
   - **Keterangan:** Pada `index petugas/checkout.php`, status transaksi diubah menjadi `Selesai` dan `jam_keluar = CURTIME()` dicatat lebih dulu ke dalam database dalam satu transaksi aman (`FOR UPDATE`). Perhitungan selisih durasi serta denda dihitung setelahnya saat halaman `struck.php` dibuka menggunakan helper `parkir-fee.php`.

4. **Metode Pembayaran Online / Gateway:**
   - **Status:** Tidak ditemukan implementasi yang dapat diverifikasi di source code
   - **Keterangan:** Sistem tidak memiliki modul payment gateway (seperti Midtrans, Xendit, atau transfer bank otomatis). Tagihan yang dihitung sistem adalah tarif resmi parkir Borobudur untuk keperluan pelaporan kasir/petugas dan cetak struk fisik.

5. **Fungsi QR Scanner Tiket:**
   - **Status:** Terverifikasi Sesuai Source Code
   - **Keterangan:** QR Code yang dihasilkan pada aplikasi user (`antrian-parkir.php`) berfungsi sebagai tiket digital yang memuat teks gabungan data booking. Pada sisi petugas (`checkin.php`), scanner kamera HTML5/JavaScript mengekstrak `kode_booking`, tetapi validasi keaslian, ketersediaan area, kuota slot, dan status kendaraan tetap divalidasi penuh oleh server di backend PHP.

6. **File Struk Legacy:**
   - **Status:** `PERLU VERIFIKASI`
   - **Keterangan:** Terdapat file `struck.php` di direktori root aplikasi yang menggunakan logika dan formula lama. Halaman operasional aktif yang benar-benar digunakan sistem saat ini berada di `index petugas/struck.php` dan `index login user/struck.php`.

## Bagian yang Masih Perlu Diverifikasi

Berdasarkan hasil penelusuran menyeluruh secara **READ-ONLY**, berikut adalah poin-poin teknis dan operasional yang **masih perlu diverifikasi** oleh penguji, pembimbing, atau pengembang sebelum sistem digunakan pada pengujian live atau deployment produksi:

1. **Sinkronisasi Trigger Database vs Fungsi PHP `syncTerisiArea()`:**
   - **Kondisi di Kode:** Source code PHP di `booking-occupancy.php` secara eksplisit menambah/mengurangi `area_parkir.terisi` saat booking berubah status (`Aktif`, `Selesai`, `Dibatalkan`). Di sisi lain, pada beberapa dump/backup SQL lama tersimpan trigger database pada event `AFTER UPDATE booking`.
   - **Risiko:** Jika trigger database aktif bersamaan dengan fungsi PHP, counter okupansi akan terhitung ganda (berkurang/bertambah 2 untuk satu kendaraan).
   - **Rekomendasi Verifikasi:** Cek tabel `information_schema.TRIGGERS` di database aktif; pastikan trigger okupansi lama sudah didrop/dinonaktifkan jika sistem mengandalkan fungsi PHP.

2. **Konsistensi Kolom Skema Tabel `booking` dan `staff`:**
   - **Kondisi di Kode:** File PHP aktif mengandalkan kolom-kolom baru seperti `booking.kode_booking`, `booking.sumber`, `booking.tanggal_parkir`, `booking.checkin_staff_id`, `booking.qr_shown`, status `'Menunggu Check-in'`, serta `staff.role`. Kolom-kolom ini tidak ditemukan pada dump database awal (`parkir_borobudur.sql` lama).
   - **Rekomendasi Verifikasi:** Jalankan `DESCRIBE booking;` dan `DESCRIBE staff;` pada database MySQL server untuk memastikan seluruh kolom tersebut sudah ada.

3. **Status Penggunaan File Legacy `struck.php` di Direktori Root:**
   - **Kondisi di Kode:** Terdapat file `struck.php` lama di root project dengan penghitungan tarif hardcoded yang berbeda dengan aturan tarif dinamis di `parkir-fee.php` yang dipakai oleh `index petugas/struck.php` dan `index login user/struck.php`.
   - **Rekomendasi Verifikasi:** Pastikan tidak ada tautan atau tombol lama yang masih mengarah ke `/struck.php` root agar struk yang dicetak selalu konsisten dengan tarif resmi di tabel `settings`.

4. **Ketiadaan Database Lock Penuh pada Form Booking Pengguna:**
   - **Kondisi di Kode:** Pada `index login user/chek_in_user.php`, validasi ketersediaan area dan slot dilakukan secara bertahap tanpa transaksi database eksklusif (`FOR UPDATE`) seperti halnya pada check-in petugas (`index petugas/checkin.php`).
   - **Risiko:** Berpotensi terjadi *race condition* jika dua pengunjung memilih slot parkir yang persis sama pada detik yang sama di jaringan ramai.
   - **Rekomendasi Verifikasi:** Perlu diuji skenario reservasi simultan untuk slot yang sama atau ditambahkan transaksi penguncian baris (`FOR UPDATE`) pada rilis mendatang.

5. **Kredensial dan Alur Reset Password Staff:**
   - **Kondisi di Kode:** File `reset_password_staff.php` dan `lupa_password_staff.php` merujuk pada token reset, namun alur otomatis pengiriman token via email untuk role staff tidak ditemukan modul lengkapnya di source code (berbeda dengan reset password user yang memiliki `mailer.php`).
   - **Rekomendasi Verifikasi:** Verifikasi apakah reset password staff dilakukan secara manual melalui database/admin atau memang fitur tersebut belum diimplementasikan penuh.

6. **Konfigurasi Port MySQL pada Lingkungan Pengujian:**
   - **Kondisi di Kode:** File `koneksi.php` secara default menentukan `$port = 3307;` untuk lingkungan `localhost`.
   - **Rekomendasi Verifikasi:** Jika komputer penguji atau laptop presentasi UKK menggunakan konfigurasi default XAMPP MySQL standar (port `3306`), koneksi akan gagal kecuali port di `koneksi.php` disesuaikan atau MySQL server dipindahkan ke port 3307.
