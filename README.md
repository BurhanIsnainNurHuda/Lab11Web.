# Lab11Web.

    Nama: Burhan Isnain Nur Huda  
    NIM: 312410226  
    Kelas: TI.24.A.2  
    Mata Kuliah: Pemrograman Web 1
    Laporan Praktikum 10: PHP OOP

### LAPORAN PRAKTIKUM 11 & 12

### 1. Pendahuluan
Praktikum ini kita bikin framework PHP OOP sederhana + sistem login. Tujuannya biar paham cara bikin aplikasi web yang terstruktur pake OOP, plus ngerti cara kerja session buat login.

### 2. Struktur Folder
Pertama kita setup foldernya dulu di ```htdocs```:
lab11_php_oop/
├── .htaccess              # buat routing
├── config.php             # setting database
├── index.php              # router utama
├── class/                 # class-class core
│   ├── Database.php       # buat konek database
│   └── Form.php           # buat form otomatis
├── module/                # modul-modul aplikasi
│   ├── home/              # halaman depan
│   ├── artikel/           # CRUD artikel
│   └── user/              # login, logout, profile
└── template/              # layout
    ├── header.php
    └── footer.php

### Screenshot struktur folder:
<img width="373" height="564" alt="image" src="https://github.com/user-attachments/assets/3eb97207-9cba-4071-9d31-1180a0973a69" />

### 3. Setup Database

Bikin database dulu di phpMyAdmin:

    sql
    -- database: latihan_oop
    -- password admin: admin123
    CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE,
    password VARCHAR(255),
    nama VARCHAR(100),
    email VARCHAR(100)
    );

    CREATE TABLE artikel (
    id INT AUTO_INCREMENT PRIMARY KEY,
    judul VARCHAR(200),
    isi TEXT,
    tanggal DATE,
    penulis VARCHAR(100)
    );

    -- insert user admin
    INSERT INTO users (username, password, nama) 
    VALUES ('admin', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', 'Administrator');

Password `admin123` udah di-hash pake `password_hash()` biar aman.

### 4. File Penting

### a. config.php

    php
    <?php
    $config = [
    'host' => 'localhost',
    'username' => 'root',
    'password' => '',  // biasanya kosong kalo pake XAMPP
    'db_name' => 'latihan_oop'
    ];
    ?>

### b. .htaccess

Buat biar URL bisa clean (nggak pake `index.php?page=...`):

    RewriteEngine On
    RewriteBase /lab11_php_oop/
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^(.*)$ index.php/$1 [L]

### c. index.php (ROUTER)

Ini jantungnya sistem, semua request diarahkan ke sini:

    <?php
    session_start();
    include "class/Database.php";
    // Ambil URL, misal: /artikel/tambah
    $path = $_SERVER['PATH_INFO'] ?? '/home/index';
    // Pisah jadi modul dan action
    $segments = explode('/', trim($path, '/'));
    $mod = $segments[0] ?? 'home';  // contoh: 'artikel'
    $page = $segments[1] ?? 'index'; // contoh: 'tambah'
    // Cek login kalo mau akses halaman terproteksi
    $public_pages = ['home', 'user']; // halaman yg bisa diakses tanpa login
    if (!in_array($mod, $public_pages) && !isset($_SESSION['is_login'])) {
    header('Location: /lab11_php_oop/user/login');
    exit();
    }

    // Load template
    include "template/header.php";

    // Load halaman sesuai modul
     $file = "module/{$mod}/{$page}.php";
    if (file_exists($file)) {
    include $file;
    } else {
    echo "Halaman gak ketemu bro!";
      }

    include "template/footer.php";
    ?>
