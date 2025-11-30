# Project: Kursus - PHP + PDO CRUD (Tugas 1)

**Isi repository ini**: proyek backend sederhana untuk entitas `kursus` memenuhi spesifikasi tugas.

> **Catatan**: Jangan jalankan langsung di server publik tanpa konfigurasi. Sesuaikan `config.php` (DB creds) sebelum digunakan.

---

## Struktur folder

```
kursus-app/
├── README.md
├── schema.sql
├── config.php
├── index.php           # list kursus
├── create.php          # form tambah
├── store.php           # proses penyimpanan
├── edit.php            # form edit
├── update.php          # proses update
├── delete.php          # proses hapus
├── /uploads/           # tempat file poster (chmod 755)
├── /classes/
│   └── Kursus.php
└── /templates/
    ├── header.php
    └── footer.php
```

---

## Petunjuk singkat

1. Siapkan database MySQL.
2. Import `schema.sql`.
3. Sesuaikan `config.php` dengan kredensial database.
4. Pastikan folder `uploads/` dapat ditulis oleh webserver (contoh: `chmod 755 uploads` atau `775`).
5. Akses `index.php` pada browser.

---

## FILE: schema.sql

```sql
CREATE DATABASE IF NOT EXISTS tugas1_kursus DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
USE tugas1_kursus;

CREATE TABLE IF NOT EXISTS kursus (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nama_kursus VARCHAR(100) NOT NULL,
  durasi_jam INT NOT NULL,
  level ENUM('beginner','intermediate','advanced') NOT NULL,
  poster_path VARCHAR(255),
  status ENUM('active','inactive') NOT NULL DEFAULT 'active'
);
```

---

## FILE: README.md

```markdown
# Kursus - PHP + PDO CRUD
Aplikasi sederhana untuk entitas `kursus` (Tugas 1)

## Cara pakai
1. Import `schema.sql` ke MySQL.
2. Edit file `config.php` isikan DB host, name, user, pass.
3. Pastikan folder `uploads/` dapat ditulis.
4. Buka `index.php` pada browser.

## Fitur
- Create / Read / Update / Delete kursus
- Upload poster (jpg/png, max 2MB)
- Validasi dasar input
- Menggunakan PDO & prepared statements

## Catatan
- Sesuaikan izin folder `uploads/`.
- Hapus file poster pada proses delete jika ingin; contoh kode sudah menangani penghapusan file poster saat record dihapus.
```

---

## FILE: config.php

```php
<?php
// sesuaikan konfigurasi database
$db_host = '127.0.0.1';
$db_name = 'tugas1_kursus';
$db_user = 'root';
$db_pass = '';
$dsn = "mysql:host={$db_host};dbname={$db_name};charset=utf8mb4";

$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES => false,
];

try {
    $pdo = new PDO($dsn, $db_user, $db_pass, $options);
} catch (PDOException $e) {
    die('Database connection failed: ' . $e->getMessage());
}
```

---

## FILE: classes/Kursus.php

```php
<?php
class Kursus {
    private $pdo;
    private $table = 'kursus';

    public function __construct(PDO $pdo) {
        $this->pdo = $pdo;
    }

    public function all() {
        $stmt = $this->pdo->query("SELECT * FROM {$this->table} ORDER BY id DESC");
        return $stmt->fetchAll();
    }

    public function find($id) {
        $stmt = $this->pdo->prepare("SELECT * FROM {$this->table} WHERE id = ? LIMIT 1");
        $stmt->execute([$id]);
        return $stmt->fetch();
    }

    public function create($data) {
        $sql = "INSERT INTO {$this->table} (nama_kursus,durasi_jam,level,poster_path,status) VALUES (:nama,:durasi,:level,:poster,:status)";
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute([
            ':nama' => $data['nama_kursus'],
            ':durasi' => $data['durasi_jam'],
            ':level' => $data['level'],
            ':poster' => $data['poster_path'],
            ':status' => $data['status']
        ]);
        return $this->pdo->lastInsertId();
    }

    public function update($id, $data) {
        $sql = "UPDATE {$this->table} SET nama_kursus=:nama, durasi_jam=:durasi, level=:level, poster_path=:poster, status=:status WHERE id=:id";
        $stmt = $this->pdo->prepare($sql);
        return $stmt->execute([
            ':nama' => $data['nama_kursus'],
            ':durasi' => $data['durasi_jam'],
            ':level' => $data['level'],
            ':poster' => $data['poster_path'],
            ':status' => $data['status'],
            ':id' => $id
        ]);
    }

    public function delete($id) {
        // ambil data untuk hapus file jika ada
        $k = $this->find($id);
        if ($k && !empty($k['poster_path']) && file_exists($k['poster_path'])) {
            @unlink($k['poster_path']);
        }
        $stmt = $this->pdo->prepare("DELETE FROM {$this->table} WHERE id = ?");
        return $stmt->execute([$id]);
    }
}
```

---

## FILES: templates/header.php & footer.php

```php
<!-- templates/header.php -->
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Kursus App</title>
  <style>body{font-family:Arial,Helvetica,sans-serif;max-width:900px;margin:20px auto;padding:0 10px}table{width:100%;border-collapse:collapse}th,td{padding:8px;border:1px solid #ddd;text-align:left}a.button{display:inline-block;padding:6px 10px;background:#2d89ef;color:#fff;text-decoration:none;border-radius:4px}form input,form select{padding:6px;width:100%;box-sizing:border-box;margin-bottom:8px}</style>
</head>
<body>
  <h1>Kursus App</h1>
  <p><a href="index.php" class="button">Daftar Kursus</a> <a href="create.php" class="button">Tambah Kursus</a></p>

<!-- templates/footer.php -->
<footer style="margin-top:30px;color:#666;font-size:14px">&copy; Kursus App</footer>
</body>
</html>
```

---

## FILE: index.php (List)

```php
<?php
require 'config.php';
require 'classes/Kursus.php';
$kursus = new Kursus($pdo);
$list = $kursus->all();
include 'templates/header.php';

if (!empty($_GET['msg'])) {
    echo '<p style="color:green">' . htmlspecialchars($_GET['msg']) . '</p>';
}
?>
<table>
    <thead>
        <tr><th>ID</th><th>Nama</th><th>Durasi(jam)</th><th>Level</th><th>Status</th><th>Poster</th><th>Aksi</th></tr>
    </thead>
    <tbody>
        <?php foreach ($list as $row): ?>
        <tr>
            <td><?php echo $row['id'] ?></td>
            <td><?php echo htmlspecialchars($row['nama_kursus']) ?></td>
            <td><?php echo $row['durasi_jam'] ?></td>
            <td><?php echo $row['level'] ?></td>
            <td><?php echo $row['status'] ?></td>
            <td><?php if (!empty($row['poster_path']) && file_exists($row['poster_path'])): ?><img src="<?php echo $row['poster_path'] ?>" alt="poster" style="max-height:50px"><?php else: echo '-' ; endif; ?></td>
            <td>
                <a href="edit.php?id=<?php echo $row['id'] ?>">Edit</a> |
                <a href="delete.php?id=<?php echo $row['id'] ?>" onclick="return confirm('Yakin ingin menghapus?')">Delete</a>
            </td>
        </tr>
        <?php endforeach; ?>
    </tbody>
</table>

<?php include 'templates/footer.php'; ?>
```

---

## FILE: create.php (Form)

```php
<?php
include 'templates/header.php';
?>
<h2>Tambah Kursus</h2>
<form action="store.php" method="post" enctype="multipart/form-data">
    <label>Nama Kursus (required)</label>
    <input type="text" name="nama_kursus" required>

    <label>Durasi (jam) (required)</label>
    <input type="number" name="durasi_jam" min="1" required>

    <label>Level (required)</label>
    <select name="level" required>
        <option value="">--pilih--</option>
        <option value="beginner">beginner</option>
        <option value="intermediate">intermediate</option>
        <option value="advanced">advanced</option>
    </select>

    <label>Poster (jpg/png, max 2MB)</label>
    <input type="file" name="poster">

    <label>Status</label>
    <select name="status">
        <option value="active">active</option>
        <option value="inactive">inactive</option>
    </select>

    <button type="submit">Simpan</button>
</form>

<?php include 'templates/footer.php'; ?>
```

---

## FILE: store.php (Proses Create + Upload)

```php
<?php
require 'config.php';
require 'classes/Kursus.php';
$kursus = new Kursus($pdo);

// simple validation
$nama = trim($_POST['nama_kursus'] ?? '');
$durasi = $_POST['durasi_jam'] ?? '';
$level = $_POST['level'] ?? '';
$status = $_POST['status'] ?? 'active';

$errors = [];
if ($nama === '') $errors[] = 'Nama wajib diisi.';
if (!is_numeric($durasi) || intval($durasi) <= 0) $errors[] = 'Durasi harus angka lebih dari 0.';
if (!in_array($level, ['beginner','intermediate','advanced'])) $errors[] = 'Level tidak valid.';

$poster_path = null;
if (!empty($_FILES['poster']) && $_FILES['poster']['error'] !== UPLOAD_ERR_NO_FILE) {
    $f = $_FILES['poster'];
    if ($f['error'] !== UPLOAD_ERR_OK) $errors[] = 'Upload poster gagal.';
    $allowed = ['image/jpeg','image/png'];
    if (!in_array($f['type'], $allowed)) $errors[] = 'Tipe poster harus JPG atau PNG.';
    if ($f['size'] > 2 * 1024 * 1024) $errors[] = 'Ukuran poster maksimal 2MB.';

    if (empty($errors)) {
        $ext = pathinfo($f['name'], PATHINFO_EXTENSION);
        $fname = 'uploads/poster_' . time() . '_' . bin2hex(random_bytes(4)) . '.' . $ext;
        if (!move_uploaded_file($f['tmp_name'], $fname)) {
            $errors[] = 'Gagal menyimpan file poster.';
        } else {
            $poster_path = $fname;
        }
    }
}

if (!empty($errors)) {
    // tampilkan error sederhana
    foreach ($errors as $e) echo '<p style="color:red">' . htmlspecialchars($e) . '</p>';
    echo '<p><a href="create.php">Kembali</a></p>';
    exit;
}

$data = [
    'nama_kursus' => $nama,
    'durasi_jam' => intval($durasi),
    'level' => $level,
    'poster_path' => $poster_path,
    'status' => $status
];

$id = $kursus->create($data);
header('Location: index.php?msg=' . urlencode('Data berhasil ditambahkan.'));
exit;
```

---

## FILE: edit.php (Form Edit)

```php
<?php
require 'config.php';
require 'classes/Kursus.php';
$kursus = new Kursus($pdo);
$id = $_GET['id'] ?? null;
if (!$id) { header('Location: index.php'); exit; }
$item = $kursus->find($id);
if (!$item) { echo 'Data tidak ditemukan'; exit; }
include 'templates/header.php';
?>
<h2>Edit Kursus</h2>
<form action="update.php" method="post" enctype="multipart/form-data">
    <input type="hidden" name="id" value="<?php echo $item['id'] ?>">

    <label>Nama Kursus (required)</label>
    <input type="text" name="nama_kursus" value="<?php echo htmlspecialchars($item['nama_kursus']) ?>" required>

    <label>Durasi (jam) (required)</label>
    <input type="number" name="durasi_jam" min="1" value="<?php echo $item['durasi_jam'] ?>" required>

    <label>Level (required)</label>
    <select name="level" required>
        <option value="beginner" <?php if($item['level']==='beginner') echo 'selected' ?>>beginner</option>
        <option value="intermediate" <?php if($item['level']==='intermediate') echo 'selected' ?>>intermediate</option>
        <option value="advanced" <?php if($item['level']==='advanced') echo 'selected' ?>>advanced</option>
    </select>

    <p>Poster saat ini: <?php if (!empty($item['poster_path']) && file_exists($item['poster_path'])): ?>
        <br><img src="<?php echo $item['poster_path'] ?>" style="max-height:80px">
    <?php else: echo ' - (tidak ada)'; endif; ?></p>

    <label>Ganti poster? (biarkan kosong untuk mempertahankan)</label>
    <input type="file" name="poster">

    <label>Status</label>
    <select name="status">
        <option value="active" <?php if($item['status']==='active') echo 'selected' ?>>active</option>
        <option value="inactive" <?php if($item['status']==='inactive') echo 'selected' ?>>inactive</option>
    </select>

    <button type="submit">Update</button>
</form>

<?php include 'templates/footer.php'; ?>
```

---

## FILE: update.php (Proses Update)

```php
<?php
require 'config.php';
require 'classes/Kursus.php';
$kursus = new Kursus($pdo);

$id = $_POST['id'] ?? null;
if (!$id) { header('Location: index.php'); exit; }
$item = $kursus->find($id);
if (!$item) { echo 'Data tidak ditemukan'; exit; }

$nama = trim($_POST['nama_kursus'] ?? '');
$durasi = $_POST['durasi_jam'] ?? '';
$level = $_POST['level'] ?? '';
$status = $_POST['status'] ?? 'active';

$errors = [];
if ($nama === '') $errors[] = 'Nama wajib diisi.';
if (!is_numeric($durasi) || intval($durasi) <= 0) $errors[] = 'Durasi harus angka lebih dari 0.';
if (!in_array($level, ['beginner','intermediate','advanced'])) $errors[] = 'Level tidak valid.';

$poster_path = $item['poster_path']; // default pakai yang lama
if (!empty($_FILES['poster']) && $_FILES['poster']['error'] !== UPLOAD_ERR_NO_FILE) {
    $f = $_FILES['poster'];
    if ($f['error'] !== UPLOAD_ERR_OK) $errors[] = 'Upload poster gagal.';
    $allowed = ['image/jpeg','image/png'];
    if (!in_array($f['type'], $allowed)) $errors[] = 'Tipe poster harus JPG atau PNG.';
    if ($f['size'] > 2 * 1024 * 1024) $errors[] = 'Ukuran poster maksimal 2MB.';

    if (empty($errors)) {
        $ext = pathinfo($f['name'], PATHINFO_EXTENSION);
        $fname = 'uploads/poster_' . time() . '_' . bin2hex(random_bytes(4)) . '.' . $ext;
        if (!move_uploaded_file($f['tmp_name'], $fname)) {
            $errors[] = 'Gagal menyimpan file poster.';
        } else {
            // hapus poster lama jika ada
            if (!empty($poster_path) && file_exists($poster_path)) @unlink($poster_path);
            $poster_path = $fname;
        }
    }
}

if (!empty($errors)) {
    foreach ($errors as $e) echo '<p style="color:red">' . htmlspecialchars($e) . '</p>';
    echo '<p><a href="edit.php?id=' . $item['id'] . '">Kembali</a></p>';
    exit;
}

$data = [
    'nama_kursus' => $nama,
    'durasi_jam' => intval($durasi),
    'level' => $level,
    'poster_path' => $poster_path,
    'status' => $status
];

$kursus->update($id, $data);
header('Location: index.php?msg=' . urlencode('Data berhasil diupdate.'));
exit;
```

---

## FILE: delete.php

```php
<?php
require 'config.php';
require 'classes/Kursus.php';
$kursus = new Kursus($pdo);
$id = $_GET['id'] ?? null;
if (!$id) { header('Location: index.php'); exit; }

$kursus->delete($id);
header('Location: index.php?msg=' . urlencode('Data berhasil dihapus.'));
exit;
```

---

## Ketentuan tambahan

* Validasi file upload: hanya `image/jpeg` dan `image/png`, maksimal 2MB.
* Kode memakai PDO prepared statements untuk keamanan terhadap SQL Injection.
* Penghapusan file poster dilakukan saat update jika diganti, dan saat delete.

---

Jika kamu mau, saya bisa:

* Mengemas semua file ini menjadi satu *zip* (berikan izin) atau
* Menambahkan file `.gitignore` dan instruksi commit & push ke GitHub.

Sebutkan jika mau: **(1)** zip file, **(2)** .gitignore + instruksi Git, atau **(3)** langsung saya buatkan contoh isi README lebih lengkap.
