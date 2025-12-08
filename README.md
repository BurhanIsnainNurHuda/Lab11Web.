# Lab11Web.

    Nama: Burhan Isnain Nur Huda  
     NIM: 312410226  
    Kelas: TI.24.A.2  
    Mata Kuliah: Pemrograman Web 1
    Laporan Praktikum 10: PHP OOP

### LAPORAN PRAKTIKUM 11

### 1. Tujuan
   
Paham konsep Framework Modular

Paham konsep Routing

Bikin Framework sederhana pakai PHP OOP

### 2. Langkah-langkah
Langkah 1: Buat Folder
Bikin folder ``lab11_php_oop`` di ``htdocs``

### Langkah 2: Buat File .htaccess

``RewriteEngine On
RewriteBase /lab11_php_oop/
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^(.*)$ index.php/$1 [L]``

### Langkah 3: Buat File Utama
config.php:

``<?php
$config = [
    'host' => 'localhost',
    'username' => 'root',
    'password' => '',
    'db_name' => 'latihan_oop'
];
?>``

index.php:

``<?php
include "config.php";
include "class/Database.php";
include "class/Form.php";``

``$path = $_SERVER['PATH_INFO'] ?? '/artikel/index';
$segments = explode('/', trim($path, '/'));
$mod = $segments[0] ?? 'artikel';
$page = $segments[1] ?? 'index';``

``include "template/header.php";
if(file_exists("module/$mod/$page.php")){
    include "module/$mod/$page.php";
} else {
    echo "Halaman tidak ditemukan";
}
include "template/footer.php";
?>``

### Langkah 4: Bikin Class
Database.php: Class untuk koneksi dan CRUD database
Form.php: Class untuk bikin form otomatis

### Langkah 5: Bikin Template
header.php: Navigasi + style CSS
footer.php: Copyright + penutup HTML

### Langkah 6: Bikin Modul Artikel
module/artikel/index.php: Tampilkan data artikel
module/artikel/tambah.php: Form tambah artikel
module/artikel/ubah.php: Form edit artikel

### Langkah 7: Buat Database
``CREATE DATABASE latihan_oop;
USE latihan_oop;
CREATE TABLE artikel (
    id INT AUTO_INCREMENT PRIMARY KEY,
    judul VARCHAR(255),
    isi TEXT,
    tanggal TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);``

### 3. Hasil
Struktur Akhir:
lab11_php_oop/
├── .htaccess
├── config.php
├── index.php
├── class/
│   ├── Database.php
│   └── Form.php
├── module/
│   └── artikel/
│       ├── index.php
│       ├── tambah.php
│       └── ubah.php
└── template/
    ├── header.php
    └── footer.php

### Screenshoot
<img width="1914" height="957" alt="image" src="https://github.com/user-attachments/assets/2eca4f6f-8e19-4bd5-9eec-22a11b088971" />
<img width="1912" height="950" alt="image" src="https://github.com/user-attachments/assets/bf8e96d4-de49-4ff4-950a-6832a2e88c89" />

