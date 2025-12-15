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

Form Tambah Artikel (module/artikel/tambah.php)

     <h2>Tambah Artikel Baru</h2>

    <?php
    $db = new Database();

    if($_SERVER['REQUEST_METHOD'] == 'POST') {
    $data = [
        'judul' => $_POST['judul'],
        'isi' => $_POST['isi'],
        'tanggal' => $_POST['tanggal'],
        'penulis' => $_SESSION['nama'] // ambil dari session login
    ];
    
    if($db->insert('artikel', $data)) {
        echo '<div class="alert alert-success">Artikel berhasil disimpan!</div>';
        echo '<script>setTimeout(() => location.href="../artikel/index", 2000)</script>';
    } else {
        echo '<div class="alert alert-danger">Gagal menyimpan!</div>';
    }
    }
    ?>

    <form method="POST">
    <div class="mb-3">
        <label>Judul Artikel</label>
        <input type="text" name="judul" class="form-control" required>
    </div>
    
    <div class="mb-3">
        <label>Tanggal</label>
        <input type="date" name="tanggal" class="form-control" required>
    </div>
    
    <div class="mb-3">
        <label>Isi Artikel</label>
        <textarea name="isi" class="form-control" rows="5" required></textarea>
    </div>
    
    <button type="submit" class="btn btn-primary">Simpan Artikel</button>
    <a href="../artikel/index" class="btn btn-secondary">Kembali</a>
    </form>
    <?php
    // Handle delete
    if(isset($_GET['hapus'])) {
    $db->delete('artikel', "id=".$_GET['hapus']);
    echo "<script>alert('Data terhapus!'); location.href='../artikel/index';</script>";
    }
    ?>

#### 7. Sistem Login

Login Page (module/user/login.php)

    <?php
    // Kalo udah login, langsung redirect
    if(isset($_SESSION['is_login'])) {
    header('Location: ../home/index');
    exit;
    }

    $error = "";

      // Proses login kalo ada POST
    if($_SERVER['REQUEST_METHOD'] == 'POST') {
    $db = new Database();
    $username = $_POST['username'];
    $password = $_POST['password'];
    
    // Cari user di database
    $sql = "SELECT * FROM users WHERE username='$username' LIMIT 1";
    $result = $db->query($sql);
    $user = $result->fetch_assoc();
    
    // Cocokin password pake password_verify()
    if($user && password_verify($password, $user['password'])) {
        // Set session
        $_SESSION['is_login'] = true;
        $_SESSION['user_id'] = $user['id'];
        $_SESSION['username'] = $user['username'];
        $_SESSION['nama'] = $user['nama'];
        
        // Redirect ke halaman artikel
        header('Location: ../artikel/index');
        exit;
    } else {
        $error = "Username atau password salah!";
     }
    }
    ?>

    <div class="row justify-content-center">
    <div class="col-md-4">
        <div class="card mt-5">
            <div class="card-header bg-primary text-white">
                <h4 class="mb-0">Login System</h4>
            </div>
            <div class="card-body">
                <?php if($error): ?>
                    <div class="alert alert-danger"><?= $error ?></div>
                <?php endif; ?>
                
                <form method="POST">
                    <div class="mb-3">
                        <label>Username</label>
                        <input type="text" name="username" class="form-control" required 
                               placeholder="Masukkan username">
                    </div>
                    
                    <div class="mb-3">
                        <label>Password</label>
                        <input type="password" name="password" class="form-control" required 
                               placeholder="Masukkan password">
                    </div>
                    
                    <button type="submit" class="btn btn-primary w-100">Login</button>
                </form>
                
                <div class="mt-3 text-center">
                    <small class="text-muted">
                        Demo: admin / admin123
                    </small>
                </div>
            </div>
        </div>
    </div>
    </div>

Logout (module/user/logout.php)

    <?php
    // Hapus semua session
    session_destroy();
    // Redirect ke login
    header('Location: ../user/login');
    exit;
    ?>

### 8. Halaman Profil (Tugas)

    <?php
    $db = new Database();
    $user_id = $_SESSION['user_id'];
    $user = $db->get('users', "id=$user_id");

    $pesan = "";

    // Update profil
    if(isset($_POST['update_profil'])) {
    $data = [
        'nama' => $_POST['nama'],
        'email' => $_POST['email']
    ];
    
    if($db->update('users', $data, "id=$user_id")) {
        $_SESSION['nama'] = $_POST['nama']; // update session
        $pesan = "Profil berhasil diupdate!";
        $user = $db->get('users', "id=$user_id"); // refresh data
    }
     }

    // Ubah password
    if(isset($_POST['ubah_password'])) {
    $pass_baru = $_POST['password_baru'];
    $konfirmasi = $_POST['konfirmasi_password'];
    
    if($pass_baru != $konfirmasi) {
        $pesan = "Password tidak cocok!";
    } else {
        // Hash password baru
        $hash_baru = password_hash($pass_baru, PASSWORD_DEFAULT);
        $db->update('users', ['password' => $hash_baru], "id=$user_id");
        $pesan = "Password berhasil diubah!";
    }
    }
    ?>

    <h2>Profil User</h2>

    <?php if($pesan): ?>
    <div class="alert alert-info"><?= $pesan ?></div>
    <?php endif; ?>

    <div class="row">
    <div class="col-md-6">
        <div class="card">
            <div class="card-header">Data Profil</div>
            <div class="card-body">
                <form method="POST">
                    <input type="hidden" name="update_profil" value="1">
                    
                    <div class="mb-3">
                        <label>Username</label>
                        <input type="text" class="form-control" value="<?= $user['username'] ?>" readonly>
                    </div>
                    
                    <div class="mb-3">
                        <label>Nama Lengkap</label>
                        <input type="text" name="nama" class="form-control" value="<?= $user['nama'] ?>" required>
                    </div>
                    
                    <div class="mb-3">
                        <label>Email</label>
                        <input type="email" name="email" class="form-control" value="<?= $user['email'] ?>">
                    </div>
                    
                    <button type="submit" class="btn btn-primary">Update Profil</button>
                </form>
            </div>
        </div>
    </div>
    
    <div class="col-md-6">
        <div class="card">
            <div class="card-header">Ubah Password</div>
            <div class="card-body">
                <form method="POST">
                    <input type="hidden" name="ubah_password" value="1">
                    
                    <div class="mb-3">
                        <label>Password Baru</label>
                        <input type="password" name="password_baru" class="form-control" required>
                    </div>
                    
                    <div class="mb-3">
                        <label>Konfirmasi Password</label>
                        <input type="password" name="konfirmasi_password" class="form-control" required>
                    </div>
                    
                    <button type="submit" class="btn btn-warning">Ubah Password</button>
                </form>
            </div>
        </div>
    </div>
    </div>

### 9. Template

Header (template/header.php)

    <!DOCTYPE html>
    <html lang="id">
    <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PHP OOP Framework</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body { padding-top: 20px; background-color: #f8f9fa; }
        .navbar { margin-bottom: 30px; }
    </style>
    </head>
    <body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
        <div class="container">
            <a class="navbar-brand" href="../home/index">
                <b>PHP OOP Framework</b>
            </a>
            
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
                <span class="navbar-toggler-icon"></span>
            </button>
            
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav me-auto">
                    <li class="nav-item">
                        <a class="nav-link" href="../home/index">Home</a>
                    </li>
                    
                    <?php if(isset($_SESSION['is_login'])): ?>
                    <li class="nav-item">
                        <a class="nav-link" href="../artikel/index">Artikel</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="../user/profile">Profil</a>
                    </li>
                    <?php endif; ?>
                </ul>
                
                <ul class="navbar-nav">
                    <?php if(isset($_SESSION['is_login'])): ?>
                    <li class="nav-item">
                        <span class="nav-link text-white">
                            Hi, <?= $_SESSION['nama'] ?>
                        </span>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link text-white" href="../user/logout">Logout</a>
                    </li>
                    <?php else: ?>
                    <li class="nav-item">
                        <a class="nav-link text-white" href="../user/login">Login</a>
                    </li>
                    <?php endif; ?>
                </ul>
            </div>
        </div>
    </nav>
    
    <div class="container">

Footer (template/footer.php)


    </div> <!-- end container -->
    
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    <script>
        // Auto hide alert setelah 5 detik
        setTimeout(() => {
            const alerts = document.querySelectorAll('.alert');
            alerts.forEach(alert => {
                alert.style.transition = 'opacity 0.5s';
                alert.style.opacity = '0';
                setTimeout(() => alert.remove(), 500);
            });
        }, 5000);
    </script>
    </body>
    </html>

### 10. Testing & Screenshot

Test 1: Akses tanpa login

Buka `http://localhost/lab11_php_oop/artikel/index`

Hasil: Langsung redirect ke halaman login ✅

Screenshot: Redirect ke login
<img width="1918" height="946" alt="image" src="https://github.com/user-attachments/assets/0804920c-0c24-4067-a5d6-6784579d6d6c" />

Test 2: Login dengan admin/admin123
    
Masuk ke halaman login

Isi username: `admin`, password: `admin123`

Hasil: Berhasil login, redirect ke artikel ✅

Screenshot: Login Success

<img width="1919" height="891" alt="image" src="https://github.com/user-attachments/assets/4ce5179d-ab95-442f-bd91-18a044affcfc" />

<img width="1919" height="934" alt="image" src="https://github.com/user-attachments/assets/9e5682eb-b202-44a5-af4e-0dc7a9e09d73" />
    
 <img width="1919" height="947" alt="image" src="https://github.com/user-attachments/assets/a83e864a-677c-45b7-a197-07ebfc5d3b12" />

Test 3: CRUD Artikel

Tambah artikel: ✅ Bisa

Edit artikel: ✅ Bisa

Hapus artikel: ✅ Bisa dengan konfirmasi

Screenshot: List Artikel
<img width="1915" height="942" alt="image" src="https://github.com/user-attachments/assets/6c255c0b-1071-4ab2-9de8-246d4bd17636" />

Test 4: Halaman Profil (Tugas)

Update nama & email: ✅ Bisa

Ubah password: ✅ Bisa dengan hashing

Screenshot: Profile Page
<img width="1919" height="949" alt="image" src="https://github.com/user-attachments/assets/20c19aef-a01b-4b9c-9d4e-dd9430de862d" />

Test 5: Logout
Klik logout

Hasil: Session terhapus, kembali ke login ✅

<img width="1919" height="951" alt="image" src="https://github.com/user-attachments/assets/9cf0bcee-6808-4d8c-8f89-f05ee1c57886" />

11. Kendala & Solusi
    
Kendala 1: .htaccess gak jalan
Problem: URL masih muncul `index.php?`
Solusi: Cek Apache `mod_rewrite` aktif, dan di `httpd.conf` setting `AllowOverride All`

Kendala 2: Session hilang setelah redirect
Problem: Session gak nempel
Solusi: Pastikan `session_start()` dipanggil SEBELUM ada output HTML

Kendala 3: Password verify gagal
Problem: Password admin123 gak bisa login
Solusi: Pastikan di database passwordnya udah di-hash pake `password_hash()`, bukan plain text

12. Kesimpulan
Yang berhasil dibuat:
✅ Framework modular dengan routing clean URL

✅ Class Database untuk handle semua query

✅ Sistem login dengan session & password hashing

✅ CRUD artikel lengkap

✅ Halaman profil + ubah password (tugas)

✅ Template Bootstrap yang responsive

Yang dipelajari:

Cara bikin aplikasi PHP yang terstruktur

Pake OOP buat bikin class reusable

Cara kerja session buat login

Routing biar URL rapi

Password security dengan hashing

Yang bisa ditambahin:

Validasi form lebih ketat

Upload gambar buat artikel

Pagination kalo data banyak

Role-based access (admin vs user biasa)
