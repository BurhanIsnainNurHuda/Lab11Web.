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

#### 5. Class Database

Ini class buat handle semua operasi database:

    class Database {
    private $conn;
    
    public function __construct() {
        // Koneksi ke database
        include "config.php";
        $this->conn = new mysqli(
            $config['host'], 
            $config['username'], 
            $config['password'], 
            $config['db_name']
        );
    }
    
    // SELECT data
    public function getAll($table) {
        $result = $this->conn->query("SELECT * FROM $table");
        $data = [];
        while($row = $result->fetch_assoc()) {
            $data[] = $row;
        }
        return $data;
    }
    
    // INSERT data
    public function insert($table, $data) {
        $columns = implode(",", array_keys($data));
        $values = "'" . implode("','", array_values($data)) . "'";
        $sql = "INSERT INTO $table ($columns) VALUES ($values)";
        return $this->conn->query($sql);
    }
    
    // UPDATE data
    public function update($table, $data, $where) {
        $set = [];
        foreach($data as $key => $value) {
            $set[] = "$key='$value'";
        }
        $set = implode(",", $set);
        $sql = "UPDATE $table SET $set WHERE $where";
        return $this->conn->query($sql);
    }
    
    // DELETE data
    public function delete($table, $where) {
        return $this->conn->query("DELETE FROM $table WHERE $where");
    }
    }

### 6. Modul Artikel (CRUD)

List Artikel (module/artikel/index.php)

    <?php
    $db = new Database();
    $artikel = $db->getAll('artikel');
     ?>

    <h2>Daftar Artikel</h2>
    <a href="../artikel/tambah" class="btn btn-primary mb-3">Tambah Baru</a>

    <table class="table table-bordered">
    <tr class="table-dark">
        <th>No</th>
        <th>Judul</th>
        <th>Tanggal</th>
        <th>Aksi</th>
    </tr>
    
    <?php if(empty($artikel)): ?>
        <tr><td colspan="4" class="text-center">Belum ada artikel</td></tr>
    <?php else: ?>
        <?php $no = 1; foreach($artikel as $a): ?>
        <tr>
            <td><?= $no++ ?></td>
            <td><?= $a['judul'] ?></td>
            <td><?= $a['tanggal'] ?></td>
            <td>
                <a href="../artikel/ubah/<?= $a['id'] ?>" class="btn btn-sm btn-warning">Edit</a>
                <a href="?hapus=<?= $a['id'] ?>" class="btn btn-sm btn-danger" 
                   onclick="return confirm('Yakin hapus?')">Hapus</a>
            </td>
        </tr>
        <?php endforeach; ?>
    <?php endif; ?>
    </table>

    <?php
    // Handle delete
    if(isset($_GET['hapus'])) {
    $db->delete('artikel', "id=".$_GET['hapus']);
    echo "<script>alert('Data terhapus!'); location.href='../artikel/index';</script>";
    }
    ?>
