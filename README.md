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

