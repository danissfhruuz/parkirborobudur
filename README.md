# Parkir Borobudur


## 1. Deskripsi Project

Parkir Borobudur adalah aplikasi web untuk membantu pengelolaan parkir wisata. Sistem menyediakan halaman publik, akun pengunjung, portal staff, booking, check-in, check-out, tarif, laporan, dan struk.

## 2. Teknologi yang Digunakan

- **PHP Native**, **MySQL/MariaDB**, dan **MySQLi**.
- **HTML**, **CSS**, dan **JavaScript**.
- **Fetch/AJAX** untuk interaksi dinamis.
- **Tailwind CSS CDN** dan **Chart.js 4.4.4**.
- **PHPMailer** untuk reset password melalui SMTP Gmail.
- API QR eksternal `api.qrserver.com` untuk QR pada struk.
- Apache `.htaccess` untuk aturan akses file.

Tidak ditemukan `composer.json`, `package.json`, atau framework PHP. Versi minimum PHP belum ditentukan di source.

## 3. Role dan Hak Akses

### User atau Pengunjung

Login melalui `index login user/login.php` dan menggunakan session `$_SESSION['user_id']`.

Hak dan tugas yang ditemukan:

- Registrasi akun baru dan login menggunakan username atau email.
- Melihat area, kapasitas, jumlah terisi, tipe kendaraan, dan slot yang tersedia.
- Membuat booking online, melihat antrean/status, dan menampilkan QR booking.
- Membatalkan booking milik sendiri sesuai aturan status.
- Melihat riwayat booking dan menghapus item riwayat melalui AJAX.
- Mengubah profil dan password.
- Menggunakan lupa password/reset password melalui email apabila SMTP tersedia.
- Menyelesaikan booking aktif melalui endpoint user yang tersedia.
- Melihat dan mencetak struk.

### Petugas

Login melalui `login-staff.php` dan diarahkan ke `index petugas/dashboard-petugas.php`.

Hak dan tugas yang ditemukan:

- Melihat dashboard operasional, kendaraan aktif, dan notifikasi booking online.
- Memverifikasi booking online yang berstatus `Menunggu Check-in`.
- Melakukan check-in kendaraan online atau membuat check-in walk-in.
- Memilih area dan slot, bila slot tersedia.
- Melihat booking aktif dan melakukan check-out kendaraan.
- Melihat riwayat checkout dan mencetak struk.

### Admin

Login melalui `login-staff.php` dan diarahkan ke `index admin/dashboard-admin.php`.

Hak dan tugas yang ditemukan:

- Melihat dashboard statistik area, kapasitas, okupansi, user, dan transaksi.
- Mengelola area parkir, slot parkir, tipe kendaraan, dan tarif.
- Melihat/mengelola transaksi dan data user sesuai validasi pada kode.
- Melihat laporan dan mengekspor laporan PDF melalui `export-laporan-pdf.php`.
- Mengakses pengaturan pada halaman yang diizinkan oleh pemeriksaan role.

### Owner

Login melalui `login-staff.php` dan diarahkan ke `index owner/dashboard-owner.php`.

Hak dan tugas yang ditemukan:

- Melihat dashboard ringkasan transaksi, status, tren, dan jenis kendaraan.
- Melihat monitoring kendaraan aktif, laporan, dan ringkasan keuangan.
- Mengelola staff Admin/Petugas melalui `manajemen-staf.php`.
- Melihat/mengubah profil Owner.
- Mengakses pengaturan/tarif pada halaman yang memeriksa role `admin` atau `owner`.

## 4. Alur Sistem

Alur utama yang ditemukan di kode:

```text
Landing page
    -> Registrasi/Login User
    -> Pilih area dan slot
    -> Booking online: Menunggu Check-in
    -> Verifikasi Petugas
    -> Check-in: Aktif
    -> Kendaraan sedang parkir
    -> Check-out: Selesai
    -> Riwayat dan struk
```

1. **Login:** User login dari `index login user/login.php`; staff login dari `login-staff.php`.
2. **Booking:** User memilih area/slot pada `chek_in_user.php`. Query PHP saat ini membuat sumber `online` dan status `Menunggu Check-in`.
3. **Check-in:** Petugas membuka `checkin.php`, memeriksa booking online atau data walk-in, lalu mengubah transaksi menjadi `Aktif`.
4. **Parkir aktif:** Booking `Aktif` dianggap sebagai kendaraan yang sedang parkir. `area_parkir.terisi` disinkronkan oleh `booking-occupancy.php` pada perubahan status tertentu.
5. **Check-out:** Petugas mengirim POST ke `checkout.php`. Sistem mengisi `jam_keluar` dengan `CURTIME()` dari server, mengubah status menjadi `Selesai`, lalu mengurangi okupansi area.
6. **Selesai:** Data dapat muncul di riwayat dan struk. User juga memiliki endpoint untuk menyelesaikan booking aktifnya sendiri.

### QR pada Alur Booking

- Halaman antrean user menyediakan tampilan/modal QR booking dan tombol konfirmasi.
- `konfirmasi-qr.php` mengubah `booking.qr_shown` menjadi `1` untuk booking milik user yang login.
- Tidak ditemukan implementasi kamera atau scanner QR yang membaca QR booking lalu otomatis melakukan check-in.
- QR pada struk adalah QR lain yang dibuat dari teks transaksi melalui `api.qrserver.com`.

## 5. Struktur Folder dan File Penting

| File/Folder | Fungsi |
|---|---|
| `index.php` | Entry point root yang meneruskan request ke `landing-dynamic-test.php`. |
| `landing-dynamic-test.php` | Landing page publik, status area, peta, bantuan, dan ulasan. |
| `koneksi.php` | Konfigurasi/koneksi MySQLi, timezone Asia/Jakarta, dan charset `utf8mb4`. |
| `csrf.php` | Membuat, menampilkan, dan memvalidasi CSRF token. |
| `session-staff.php` | Session khusus Admin, Petugas, dan Owner. |
| `booking-occupancy.php` | Fungsi `syncTerisiArea()` untuk sinkronisasi `area_parkir.terisi`. |
| `parkir-fee.php` | Pusat konfigurasi tarif, batas waktu, durasi, dan denda. |
| `login-staff.php` | Login staff dari tabel `staff` dan redirect sesuai role. |
| `logout.php` | Logout staff dengan POST dan CSRF. |
| `index login user/` | Modul User. |
| `index petugas/` | Modul operasional Petugas. |
| `index admin/` | Modul pengelolaan Admin. |
| `index owner/` | Modul dashboard dan pengelolaan Owner. |
| `assets/` | CSS branding Admin dan suara notifikasi booking. |
| `backups/` | Backup SQL, panduan SMTP, dan file debug. |
| `.htaccess` | Aturan Apache dan pembatasan file sensitif. |

File proses utama:

| File | Fungsi |
|---|---|
| `index login user/daftar.php` | Registrasi User ke tabel `users`. |
| `index login user/login.php` | Login User dan pembuatan session. |
| `index login user/dashboard-user.php` | Dashboard area, kapasitas, booking aktif, filter, dan pembatalan. |
| `index login user/chek_in_user.php` | Booking online dan pemilihan slot. |
| `index login user/antrian-parkir.php` | Antrean/status booking, QR, pembatalan, dan konfirmasi QR. |
| `index login user/riwayat.php` | Riwayat booking User. |
| `index login user/selesaikan-boking.php` | Endpoint JSON penyelesaian booking User. |
| `index login user/selesai-parkir.php` | Endpoint penyelesaian parkir User. |
| `index login user/struck.php` | Struk User. |
| `index login user/akun-user/` | Profil dan perubahan password User. |
| `index login user/mailer.php` | Helper PHPMailer/SMTP reset password. |
| `index login user/PHPMailer/` | Library PHPMailer yang disertakan. |
| `index petugas/checkin.php` | Verifikasi booking online dan check-in walk-in. |
| `index petugas/checkout.php` | Proses checkout dan status `Selesai`. |
| `index petugas/check-out.php` | Tampilan kendaraan aktif dan form checkout. |
| `index petugas/booking-aktif.php` | Daftar kendaraan aktif. |
| `index petugas/notifikasi-booking.php` | Endpoint JSON notifikasi booking online. |
| `index petugas/riwayat-checkout.php` | Riwayat transaksi checkout. |
| `index petugas/struck.php` | Struk transaksi Petugas. |
| `index admin/manajemen-parkir.php` | CRUD area parkir. |
| `index admin/manajemen-slot.php` | CRUD slot parkir. |
| `index admin/manajemen-user.php` | Daftar/penghapusan User. |
| `index admin/transaksi.php` | Daftar, filter, dan aksi transaksi Admin. |
| `index admin/tarif-parkir.php` | Pengelolaan tarif pada `settings`. |
| `index admin/tipe-kendaraan.php` | CRUD tipe kendaraan. |
| `index admin/laporan.php` | Laporan transaksi. |
| `index admin/export-laporan-pdf.php` | Export laporan PDF/print. |
| `index admin/pengaturan.php` | Pengaturan dan backup database. |
| `index owner/manajemen-staf.php` | CRUD staff Admin/Petugas oleh Owner. |
| `index owner/monitoring-owner.php` | Monitoring kendaraan aktif. |
| `index owner/keuangan-owner.php` | Ringkasan keuangan dan grafik estimasi pendapatan. |
| `index owner/laporan-owner.php` | Laporan Owner. |
| `index owner/laporan-owner.php` | Laporan Owner. |

## 6. Database

### Nama Database

Nama database pada `koneksi.php` dan backup SQL adalah `parkir_borobudur`.

Backup yang ditemukan:

- `backups/parkir_borobudur_pre_booking_migration_20260816.sql`
- `backups/staff_pre_reset_20260829.sql`

Backup utama menyebut MariaDB 10.4.32. Database aktif tidak diaudit langsung melalui koneksi, sehingga struktur di bawah berasal dari backup SQL dan query source.

### Tabel `users`

| Kolom penting | Keterangan |
|---|---|
| `id` | Primary key User. |
| `fullname` | Nama lengkap. |
| `username` | Username unik untuk login. |
| `email` | Email unik dan identitas reset password. |
| `phone` | Nomor telepon. |
| `password` | Hash password. |
| `plat_kendaraan` | Plat kendaraan default User. |
| `jenis_kendaraan` | Jenis kendaraan default User. |
| `reset_token`, `reset_expires` | Token dan masa berlaku reset password User. |
| `created_at` | Waktu pembuatan akun. |

Relasi eksplisit: `users.id` menjadi referensi `booking.user_id`.

### Tabel `staff`

| Kolom penting | Keterangan |
|---|---|
| `id` | Primary key staff. |
| `fullname` | Nama staff. |
| `username` | Username staff unik. |
| `password` | Hash password staff. |
| `role` | Enum `admin`, `petugas`, atau `owner`. |
| `created_at` | Waktu pembuatan akun staff. |

Kode reset password staff juga menggunakan `reset_token_hash` dan `reset_expires`, tetapi kolom tersebut tidak ada pada backup staff yang ditemukan. Struktur database aktif perlu dipastikan sebelum fitur reset password staff digunakan.

### Tabel `area_parkir`

| Kolom penting | Keterangan |
|---|---|
| `id` | Primary key area. |
| `kode_area` | Kode unik area, misalnya `PK-A01`. |
| `nama_area` | Nama area parkir. |
| `lokasi` | Deskripsi lokasi area. |
| `jenis` | Jenis kendaraan/area menurut data area. |
| `kapasitas` | Kapasitas maksimum area. |
| `terisi` | Jumlah kendaraan yang dianggap sedang memakai area. |

### Tabel `slot_parkir`

| Kolom penting | Keterangan |
|---|---|
| `id` | Primary key slot. |
| `area_id` | ID area tempat slot berada. |
| `kode_slot` | Kode slot yang ditampilkan pada grid. |
| `status` | Enum `aktif`, `nonaktif`, atau `maintenance`. |
| `keterangan` | Catatan slot, boleh kosong. |
| `created_at` | Waktu slot dibuat. |

Relasi eksplisit: `slot_parkir.area_id` mereferensikan `area_parkir.id`.

### Tabel `booking`

| Kolom pada backup lama | Keterangan |
|---|---|
| `id` | Primary key booking. |
| `user_id` | ID User; kode walk-in terbaru dapat memakai nilai kosong. |
| `area_id` | ID area parkir. |
| `slot_kode` | Kode slot, dapat kosong. |
| `jenis_kendaraan` | Jenis kendaraan. |
| `plat_nomor` | Nomor kendaraan. |
| `tanggal` | Tanggal booking/parkir. |
| `jam_masuk` | Waktu masuk. |
| `jam_keluar` | Waktu keluar, kosong sebelum checkout. |
| `status` | Backup lama: `Aktif`, `Selesai`, `Dibatalkan`. |
| `qr_shown` | Penanda QR booking sudah dikonfirmasi/dilihat. |
| `created_at` | Waktu booking dibuat. |

Query PHP saat ini juga menggunakan `kode_booking`, `sumber`, `tanggal_parkir`, `checkin_staff_id`, dan status `Menunggu Check-in`. Kolom/status tambahan ini tidak terdapat pada definisi `booking` di backup lama. Jadi backup dan source saat ini tidak sepenuhnya berada pada versi schema yang sama.

Relasi eksplisit: `booking.user_id -> users.id` dan `booking.area_id -> area_parkir.id`. Join `checkin_staff_id -> staff.id` digunakan kode terbaru, tetapi foreign key-nya tidak dapat dipastikan dari backup.

### Tabel `settings`

| Kolom penting | Keterangan |
|---|---|
| `id` | Primary key pengaturan. |
| `setting_key` | Nama pengaturan, misalnya `tarif_mobil`. |
| `setting_value` | Nilai pengaturan dalam bentuk text. |
| `created_at` | Waktu dibuat. |
| `updated_at` | Waktu diperbarui. |

Digunakan sebagai sumber tarif dan pengaturan umum seperti nama instansi, singkatan, dan alamat.

### Tabel `tipe_kendaraan`

| Kolom penting | Keterangan |
|---|---|
| `id` | Primary key tipe kendaraan. |
| `kode` | Kode tipe kendaraan. |
| `nama` | Nama tipe kendaraan. |
| `icon` | Nama ikon Material Symbols. |
| `warna` | Kelas/warna tampilan. |
| Kolom waktu | Backup dan query/setup memakai nama berbeda (`dibuat_pada`/`created_at`); perlu dipastikan pada database aktif. |

### Tabel `ulasan`

`landing-dynamic-test.php` membaca dan menyimpan data ulasan. Kolom yang dapat diketahui dari query/HTML adalah `user_id`, `nama`, `rating`, `komentar`, dan `created_at`.

Namun, tabel `ulasan` tidak terdapat pada daftar `CREATE TABLE` backup SQL utama. Definisi lengkap, primary key, dan foreign key tabel ini belum dapat dipastikan dari source yang tersedia.

### Trigger okupansi pada backup

Backup SQL utama berisi trigger `trg_booking_insert_terisi` dan `trg_booking_update_terisi` yang mengubah `area_parkir.terisi`. Source PHP juga memanggil `syncTerisiArea()` dari `booking-occupancy.php`. Keduanya perlu diperiksa agar nilai `terisi` tidak bertambah/berkurang dua kali.

## 7. Fitur Utama

Fitur yang ditemukan:

- Landing page publik dengan area, kapasitas, status ketersediaan, peta, bantuan, dan ulasan.
- Registrasi/login User dan login terpisah Admin, Petugas, Owner.
- Lupa/reset password User melalui PHPMailer/SMTP.
- Profil dan perubahan password User.
- Booking online berdasarkan area dan slot.
- Check-in online yang diverifikasi Petugas.
- Check-in walk-in oleh Petugas.
- Monitoring area dan jumlah terisi.
- Daftar kendaraan parkir aktif.
- Check-out dan pencatatan waktu keluar.
- Riwayat booking User dan riwayat checkout Petugas.
- Struk User dan Petugas.
- QR pada antrean/konfirmasi booking dan QR pada struk.
- Pengelolaan area, slot, tipe kendaraan, tarif, transaksi, laporan, dan user oleh Admin.
- Dashboard Admin, Petugas, dan Owner.
- Export laporan PDF pada modul Admin.
- Ringkasan keuangan dan grafik pada modul Owner.
- Manajemen staff Admin/Petugas oleh Owner.
- Notifikasi booking online untuk Petugas melalui polling Fetch.

Fitur pembayaran online/payment gateway tidak ditemukan. Validasi hasil scan QR otomatis juga belum ditemukan.

## 8. Sistem Tarif dan Denda

`parkir-fee.php` berfungsi sebagai pusat perhitungan tarif dan denda. File ini dipakai oleh halaman struk User, struk Petugas, dan beberapa halaman riwayat/checkout.

| Jenis kendaraan | Key tarif | Tarif fallback di kode | Batas waktu | Denda per jam |
|---|---|---:|---:|---:|
| Mobil | `tarif_mobil` | Rp5.000 | 8 jam | Rp1.000 |
| Motor | `tarif_motor` | Rp3.000 | 8 jam | Rp500 |
| Bus | `tarif_bus` | Rp10.000 | 10 jam | Rp2.000 |

Cara kerja:

1. Sistem membaca ketiga key tarif dari tabel `settings`.
2. Jika query tarif gagal, `parkir-fee.php` memakai nilai fallback di atas.
3. Durasi dihitung dari waktu masuk dan waktu keluar.
4. Kelebihan waktu dibulatkan ke atas per jam dengan `ceil`.
5. Denda = jam kelebihan × denda per jam.
6. Total = tarif dasar + denda.

Nilai backup SQL lama tidak seluruhnya sama dengan fallback kode; contoh `tarif_motor` pada backup tercatat Rp2.000. Nilai berjalan bergantung pada isi `settings` database aktif. README ini tidak mengubah nilai tarif.

## 9. Keamanan

Mekanisme keamanan yang ditemukan:

- Session User melalui `$_SESSION['user_id']`.
- Session staff terpisah melalui `ADMIN_SESSID`, `PETUGAS_SESSID`, dan `OWNER_SESSID`.
- Role protection pada setiap dashboard staff.
- CSRF melalui `csrfToken()`, `csrfField()`, `csrfMeta()`, dan `csrfValidate()`.
- Token CSRF pada Fetch melalui header `X-CSRF-Token`.
- Prepared statement dengan `mysqli_prepare()` dan `mysqli_stmt_bind_param()`.
- Password disimpan dengan `password_hash()` dan diperiksa dengan `password_verify()`.
- `session_regenerate_id(true)` setelah login.
- Validasi email, password, role, area, slot, status, dan kepemilikan booking.
- Escaping HTML dengan `htmlspecialchars()`.
- Transaksi database dengan `mysqli_begin_transaction()`, `mysqli_commit()`, dan `mysqli_rollback()`.
- Waktu checkout diisi server melalui `CURTIME()`.
- `.htaccess` membatasi akses file konfigurasi, `backups`, `.git`, dan `.env` jika mod_rewrite aktif.

File mailer memuat konfigurasi SMTP. Credential asli tidak ditulis di README; untuk deployment, credential sebaiknya dipindahkan ke environment/konfigurasi aman dan credential yang pernah tersimpan di source sebaiknya diganti.

## 10. Alur Data

```text
User/Staff mengisi form atau menekan tombol
        |
        v
PHP memeriksa session, role, CSRF, dan input
        |
        v
PHP menjalankan query MySQLi ke database parkir_borobudur
        |
        v
Data booking/status/area diperbarui
        |
        v
area_parkir.terisi disinkronkan bila status berubah
        |
        v
PHP mengirim HTML, redirect, atau JSON ke browser
        |
        v
JavaScript memperbarui tampilan/modal/notifikasi bila diperlukan
```

Contoh alur booking online:

1. User mengirim form dari `chek_in_user.php`.
2. PHP memeriksa session user, area, slot, kendaraan, dan CSRF.
3. PHP menyimpan booking ke tabel `booking`.
4. Status menunggu diverifikasi Petugas.
5. Petugas mengubah booking menjadi `Aktif`.
6. Saat checkout, booking menjadi `Selesai` dan waktu keluar disimpan.
7. Halaman struk membaca gabungan `booking`, `area_parkir`, `users`, dan `settings`.
7. Halaman struk membaca gabungan `booking`, `area_parkir`, `users`, dan `settings`.

## 11. AJAX / Fetch / JavaScript

JavaScript digunakan untuk interaksi tampilan dan request dinamis, antara lain:

- `fetch()` pada dashboard User untuk membatalkan booking.
- `fetch()` pada antrean untuk membatalkan booking dan mengonfirmasi QR.
- `fetch()` pada riwayat untuk menghapus riwayat.
- `fetch()` pada halaman akun untuk update profil dan password.
- `fetch()` pada Petugas untuk polling notifikasi booking setiap 10 detik.
- `fetch()` pada `checkin.php` untuk memuat slot berdasarkan area.
- Response JSON pada endpoint notifikasi, pembatalan, konfirmasi QR, update profil, update password, dan proses lain.
- Modal untuk QR, tipe kendaraan, dan informasi operasional.
- `window.print()` untuk mencetak struk/laporan.
- Chart.js untuk grafik Owner.

Tidak ditemukan library scanner QR kamera seperti `html5-qrcode`. QR dibuat atau ditampilkan, tetapi proses scan kamera otomatis belum dapat dipastikan.

## 12. Struk Parkir

Struk ditampilkan setelah data booking dibaca melalui parameter `booking_id`. Jalur struk yang ditemukan:

- `index login user/struck.php` untuk User.
- `index petugas/struck.php` untuk Petugas.
- `struck.php` di root juga tersedia sebagai file struk.

Data yang ditampilkan meliputi:

- ID transaksi dan booking.
- Plat nomor dan nama pengunjung bila tersedia.
- Jenis kendaraan, area, kode slot, dan lokasi.
- Waktu masuk dan keluar serta durasi parkir.
- Tarif dasar, kelebihan waktu, denda, dan status.
- Petugas/waktu cetak sesuai konteks halaman.

Cara mencetak:

1. Buka halaman struk setelah checkout atau dari riwayat.
2. Tekan tombol **Cetak Struk**.
3. Browser menjalankan `window.print()`.
4. Toolbar yang diberi aturan print disembunyikan saat mencetak.

QR struk dibuat melalui API QR eksternal dan berisi teks seperti ID transaksi, booking ID, plat nomor, dan status. QR tersebut tidak ditemukan sebagai endpoint verifikasi otomatis di dalam project.

Riwayat checkout tersedia pada `index petugas/riwayat-checkout.php`; riwayat booking User tersedia pada `index login user/riwayat.php`.

## 13. Instalasi di Localhost

### Langkah dasar

1. Install XAMPP.
2. Jalankan **Apache** dan **MySQL** dari XAMPP Control Panel.
3. Letakkan folder project di:

```text
C:\xampp\htdocs\parkir-borobudur1
```

4. Buka phpMyAdmin pada server MySQL yang digunakan.
5. Buat database bernama `parkir_borobudur` atau import backup yang sudah berisi perintah pembuatan database.
6. Import file utama:

```text
backups/parkir_borobudur_pre_booking_migration_20260816.sql
```

7. `backups/staff_pre_reset_20260829.sql` adalah backup terpisah untuk tabel staff. Gunakan hanya bila memang ingin memakai snapshot staff tersebut dan pastikan tidak menimpa data staff yang dibutuhkan.
8. Periksa konfigurasi pada `koneksi.php`.
9. Akses project melalui:

```text
http://localhost/parkir-borobudur1/
```

10. Untuk login User, buka `index login user/login.php`. Untuk Admin/Petugas/Owner, buka `login-staff.php`.

### Catatan schema sebelum pengujian

Backup SQL yang ada tampak lebih lama daripada query booking/check-in terbaru. Jika muncul error `Unknown column` atau status `Menunggu Check-in` tidak diterima, database belum sesuai versi source saat ini. Jangan menambahkan kolom secara asal; cocokkan schema aktif dengan query pada `chek_in_user.php`, `checkin.php`, `booking-aktif.php`, `notifikasi-booking.php`, dan file checkout.

## 14. Konfigurasi Database

File koneksi utama adalah `koneksi.php`.

Konfigurasi localhost yang tertulis di source:

| Pengaturan | Nilai pada source |
|---|---|
| Host | `localhost` |
| User | `root` |
| Password | kosong pada konfigurasi localhost |
| Database | `parkir_borobudur` |
| Port | `3307` |
| Charset | `utf8mb4` |
| Timezone PHP/MySQL | Asia/Jakarta / `+07:00` |

Jika MySQL XAMPP berjalan di port `3306`, ubah `$db_port` di `koneksi.php` menjadi port yang benar. Jangan menampilkan atau menyimpan credential produksi di README, repository, atau source publik.

Untuk hosting InfinityFree, `koneksi.php` memiliki placeholder host/user/password/database yang harus diisi dengan credential hosting sendiri. Nilai asli tidak ditulis di dokumentasi ini.

Fitur reset password email memakai `index login user/mailer.php`, PHPMailer, SMTP Gmail, port 587, dan STARTTLS. Konfigurasi email harus disiapkan sendiri; fitur ini dapat gagal jika SMTP belum dikonfigurasi atau diblokir lingkungan lokal.

## 15. Akun Pengujian

Backup SQL memuat username staff berikut:

| Username | Role | Password |
|---|---|---|
| `admin` | Admin | Tidak tersedia dalam bentuk plaintext; yang tersimpan hanya hash. |
| `owner` | Owner | Tidak tersedia dalam bentuk plaintext; yang tersimpan hanya hash. |
| `petugas1` | Petugas | Tidak tersedia dalam bentuk plaintext; yang tersimpan hanya hash. |

Backup utama juga memuat contoh username User `dnss`, tetapi password plaintext tidak tersedia. Data User pada backup mengandung data pribadi sehingga sebaiknya tidak digunakan untuk demo publik tanpa persetujuan.

Kesimpulan: **akun pengujian dengan password siap pakai tidak tersedia di source/documentasi**. Buat akun baru melalui registrasi User atau buat staff melalui database yang sudah disiapkan secara aman.
Kesimpulan: **akun pengujian dengan password siap pakai tidak tersedia di source/documentasi**. Buat akun baru melalui registrasi User atau buat staff melalui database yang sudah disiapkan secara aman.

## 16. Skenario Penggunaan

### User

1. Buka landing page.
2. Registrasi melalui `index login user/daftar.php`.
3. Login.
4. Pilih area dan slot yang tersedia pada dashboard.
5. Isi jenis kendaraan dan plat nomor.
6. Kirim booking online.
7. Lihat status antrean dan QR booking.
8. Tunggu Petugas memverifikasi check-in.
9. Setelah selesai parkir, gunakan fitur penyelesaian yang tersedia atau minta Petugas melakukan checkout.
10. Buka riwayat/struk dan cetak bila diperlukan.

### Petugas

1. Login melalui portal staff sebagai Petugas.
2. Buka notifikasi atau halaman check-in.
3. Pilih booking online yang menunggu, periksa kode/plat/area/slot, lalu verifikasi.
4. Untuk pengunjung langsung, pilih alur walk-in dan isi data kendaraan.
5. Pantau kendaraan pada halaman booking aktif.
6. Saat kendaraan keluar, lakukan checkout.
7. Buka struk atau riwayat checkout untuk mencetak bukti.

### Admin

1. Login melalui portal staff sebagai Admin.
2. Periksa dashboard area, kapasitas, user, dan transaksi.
3. Atur area parkir dan slot.
4. Kelola tipe kendaraan.
5. Atur tarif pada halaman tarif parkir.
6. Periksa transaksi dan laporan.
7. Kelola pengaturan sesuai hak akses yang diberikan kode.

### Owner

1. Login melalui portal staff sebagai Owner.
2. Lihat dashboard dan grafik operasional.
3. Pantau kendaraan aktif.
4. Periksa laporan dan ringkasan keuangan.
5. Kelola staff Admin/Petugas.
6. Perbarui profil Owner bila diperlukan.

## 17. Troubleshooting

### Database tidak terkoneksi

- Pastikan MySQL berjalan di XAMPP.
- Periksa database `parkir_borobudur` sudah dibuat.
- Cocokkan port di `koneksi.php`; source localhost memakai `3307`, sedangkan XAMPP sering memakai `3306`.
- Pastikan extension PHP `mysqli` aktif.

### `Unknown column` atau status booking ditolak

- Backup SQL yang tersedia lebih lama daripada sebagian query PHP saat ini.
- Periksa kolom `kode_booking`, `sumber`, `tanggal_parkir`, `checkin_staff_id`, dan status `Menunggu Check-in`.
- Pastikan database aktif sesuai versi source, bukan hanya backup lama.

### Session atau redirect kembali ke login

- Pastikan browser mengizinkan cookie.
- Akses project melalui Apache, bukan membuka file PHP langsung dari File Explorer.
- Jangan menghapus atau mengganti `session-staff.php`.
- Pastikan role pada tabel `staff` sama dengan role yang diperiksa halaman.

### Akses role ditolak

- Admin memakai halaman pada folder `index admin/`.
- Petugas memakai folder `index petugas/`.
- Owner memakai folder `index owner/`.
- Login staff memakai session cookie berbeda per role.

### File tidak ditemukan

- Nama folder project mengandung spasi, misalnya `index login user` dan `index admin`.
- Gunakan URL melalui Apache dan jangan mengganti nama file seperti `chek_in_user.php` atau `struck.php` tanpa memperbarui link.
- Periksa `require`/`require_once` jika memindahkan file.

### CSRF token tidak valid

- Muat ulang halaman sebelum mengirim form.
- Pastikan session aktif.
- Jangan menghapus `csrfField()`/`csrfMeta()` dari halaman yang mengirim POST atau Fetch.
- Pastikan Fetch mengirim header `X-CSRF-Token` bila endpoint membutuhkannya.

### Reset password email gagal

- Periksa konfigurasi SMTP pada `mailer.php`.
- Pastikan akun email dan App Password valid.
- Pastikan koneksi TLS/port 587 dapat digunakan.
- Jangan menaruh credential SMTP produksi langsung di repository.

### Nilai okupansi tidak sesuai

- Periksa `booking-occupancy.php`.
- Periksa trigger `trg_booking_insert_terisi` dan `trg_booking_update_terisi` pada database.
- Source PHP dan trigger backup sama-sama dapat mengubah `area_parkir.terisi`, sehingga perlu dipastikan hanya mekanisme yang sesuai yang digunakan.

## 18. Catatan Pengembangan

Hal-hal penting jika project dikembangkan lebih lanjut:

- Buat migration/schema resmi yang menyamakan backup database dengan query PHP terbaru.
- Tetapkan satu sumber perubahan okupansi: trigger database atau helper PHP, lalu uji agar tidak terjadi double count.
- Dokumentasikan status booking resmi, termasuk `Menunggu Check-in`, `Aktif`, `Selesai`, dan `Dibatalkan`.
- Pastikan struktur `staff` memiliki kolom reset password yang benar-benar dipakai kode.
- Pindahkan konfigurasi SMTP dan credential database ke environment yang aman.
- Tambahkan pengujian untuk booking bersamaan, slot bentrok, pembatalan, checkout ganda, dan perubahan `terisi`.
- Pertahankan prepared statement, CSRF, validasi ownership booking, dan transaksi database.
- Jika QR ingin dipakai sebagai verifikasi, tambahkan endpoint/verifier resmi; saat ini source baru membuat/menampilkan data QR.
- Tentukan apakah fitur pembayaran diperlukan; payment gateway belum ditemukan.
- Hindari membuka folder backup pada server publik dan pastikan aturan keamanan juga diterapkan bila hosting bukan Apache.
- Tambahkan dokumentasi versi PHP dan struktur database yang menjadi acuan rilis.

## 19. Ringkasan Project

Parkir Borobudur adalah aplikasi parkir berbasis PHP Native dan MySQL/MariaDB. Sistem memiliki empat role utama: User, Petugas, Admin, dan Owner. Alur utamanya adalah User membuat booking, Petugas memverifikasi check-in, kendaraan berstatus aktif, lalu dilakukan checkout dan pembuatan struk.

Fitur utamanya meliputi booking online, slot dan area parkir, check-in online/walk-in, checkout, riwayat, struk, tarif/denda, dashboard, laporan, monitoring, dan manajemen staff. Keamanan yang ditemukan meliputi session, role protection, CSRF token, prepared statement, password hashing/verification, validasi input, escaping output, dan transaksi database.

## 20. Catatan untuk UKK

Poin-poin singkat untuk presentasi:

- **Mengapa PHP Native?** Project dapat berjalan langsung di Apache/XAMPP tanpa framework besar. File PHP menangani proses, query, validasi, dan tampilan.
- **Fungsi MySQL/MariaDB:** Menyimpan User, staff, area, slot, booking, tarif, tipe kendaraan, dan data yang dipakai sistem.
- **Fungsi session:** Menandai siapa yang sedang login. User memakai `user_id`, staff memakai `staff_id` dan `staff_role`.
- **Fungsi CSRF:** Mencegah request POST palsu. Token dibuat di session dan divalidasi melalui `csrfValidate()`.
- **Prepared statement:** Memisahkan query dan input sehingga membantu mencegah SQL Injection.
- **`password_hash()` dan `password_verify()`:** Password tidak disimpan sebagai teks biasa; login menguji password terhadap hash.
- **`booking.status`:** Menunjukkan tahap transaksi: `Menunggu Check-in`, `Aktif`, `Selesai`, dan `Dibatalkan` pada source terbaru. Backup lama belum mencantumkan semuanya.
- **`area_parkir.terisi`:** Menunjukkan jumlah kendaraan yang memakai area untuk menghitung kapasitas dan status penuh/tersedia.
- **Fetch/AJAX:** Dipakai untuk pembatalan, konfirmasi QR, update profil, pemuatan slot, dan notifikasi tanpa selalu memuat ulang halaman.
- **Transaksi database:** `begin_transaction`, `commit`, dan `rollback` menjaga perubahan booking dan okupansi tetap konsisten.
- **Role dan hak akses:** Setiap role memiliki folder dan pemeriksaan session/role sendiri.
- **Alur booking sampai checkout:** User membuat booking online, Petugas memverifikasi check-in, booking aktif, kendaraan dipantau, checkout dilakukan, waktu keluar dicatat, status selesai, dan struk dicetak.
- **QR:** QR antrean ditampilkan/dikonfirmasi melalui `qr_shown`, sedangkan QR struk dibuat dari data transaksi melalui layanan QR eksternal. Scanner kamera otomatis belum ditemukan.
- **Tarif dan denda:** `parkir-fee.php` membaca tarif dari `settings`, menghitung durasi, menambahkan denda jika melewati batas, lalu menghasilkan total.
