MenuUtama.php
php
<!DOCTYPE html>
<html>
<head>
    <title>Form Penerimaan Siswa Baru</title>
</head>
<body>
    <h2>Form Penerimaan Siswa Baru</h2>
    <form method="post" action="proses1.php">
        <input type="hidden" name="action" value="insert">
        <label>Nama Siswa:</label>
        <input type="text" name="nama" required>
        <label>Usia:</label>
        <input type="number" name="usia" required>
        <input type="submit" value="Simpan">
    </form>
    <hr>
    <h2>Daftar Siswa</h2>
    <table border="1">
        <tr>
            <th>Nama Siswa</th>
            <th>Usia</th>
            <th>Aksi</th>
        </tr>
        <?php
        // Memuat daftar siswa dari file JSON
        $data_file = 'data_siswa.json';
        if (file_exists($data_file)) {
            $data_siswa = json_decode(file_get_contents($data_file), true);
            foreach ($data_siswa as $siswa) {
                echo '<tr>';
                echo '<td>'.$siswa['nama'].'</td>';
                echo '<td>'.$siswa['usia'].'</td>';
                echo '<td><a href="edit1.php?id='.$siswa['id'].'">Edit</a> | <a href="proses1.php?action=delete&id='.$siswa['id'].'">Hapus</a></td>';
                echo '</tr>';
            }
        }
        ?>
    </table>
</body>
</html>

proses1.php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['action'])) {
    $data_file = 'data_siswa.json';
    $data_siswa = [];

    if (file_exists($data_file)) {
        $data_siswa = json_decode(file_get_contents($data_file), true);
    }

    if ($_POST['action'] === 'insert') {
        // Periksa apakah semua field formulir diisi
        if (isset($_POST['nama']) && isset($_POST['usia'])) {
            // Ambil data yang dikirimkan dari formulir
            $nama = $_POST['nama'];
            $usia = $_POST['usia'];

            // Buat ID unik untuk entri baru
            $id = uniqid();

            // Buat entri baru untuk data yang dikirimkan
            $new_siswa = [
                'id' => $id,
                'nama' => $nama,
                'usia' => $usia
            ];

            // Tambahkan entri baru ke array data siswa
            $data_siswa[] = $new_siswa;

            // Simpan kembali data array yang telah diperbarui ke file JSON
            file_put_contents($data_file, json_encode($data_siswa));
        }
    } elseif ($_POST['action'] === 'edit') {
        // Periksa apakah semua field formulir diisi
        if (isset($_POST['id']) && isset($_POST['nama']) && isset($_POST['usia'])) {
            // Ambil data yang dikirimkan dari formulir
            $id_siswa = $_POST['id'];
            $nama = $_POST['nama'];
            $usia = $_POST['usia'];

            // Cari siswa dengan ID yang sesuai
            $siswa_to_edit = null;
            foreach ($data_siswa as $index => $siswa) {
                if ($siswa['id'] === $id_siswa) {
                    $siswa_to_edit = $index;
                    break;
                }
            }

            // Jika siswa ditemukan, update data siswa
            if ($siswa_to_edit !== null) {
                $data_siswa[$siswa_to_edit]['nama'] = $nama;
                $data_siswa[$siswa_to_edit]['usia'] = $usia;

                // Simpan kembali data array yang telah diperbarui ke file JSON
                file_put_contents($data_file, json_encode($data_siswa));
            }
        }
    } elseif ($_POST['action'] === 'delete') {
        // Periksa apakah parameter ID siswa untuk dihapus sudah diberikan
        if (isset($_POST['id'])) {
            // Ambil ID siswa yang akan dihapus dari parameter
            $id_siswa = $_POST['id'];

            // Cari indeks siswa yang sesuai dengan ID
            $index_to_remove = null;
            foreach ($data_siswa as $index => $siswa) {
                if ($siswa['id'] === $id_siswa) {
                    $index_to_remove = $index;
                    break;
                }
            }

            // Jika ID siswa ditemukan, hapus siswa dari array data
            if ($index_to_remove !== null) {
                unset($data_siswa[$index_to_remove]);

                // Simpan kembali data array yang telah diperbarui ke file JSON
                file_put_contents($data_file, json_encode(array_values($data_siswa)));
            }
        }
    }
}

// Alihkan kembali ke MenuUtama.php setelah aksi berhasil
header('Location: MenuUtama.php');
exit;
?>

edit1.php
php
<!DOCTYPE html>
<html>
<head>
    <title>Edit Data Siswa</title>
</head>
<body>
    <h2>Edit Data Siswa</h2>
    <?php
    if (isset($_GET['id'])) {
        $id_siswa = $_GET['id'];

        // Muat data yang ada dari file JSON
        $data_file = 'data_siswa.json';
        $data_siswa = [];
        if (file_exists($data_file)) {
            $data_siswa = json_decode(file_get_contents($data_file), true);
            $siswa_to_edit = null;
            foreach ($data_siswa as $siswa) {
                if ($siswa['id'] === $id_siswa) {
                    $siswa_to_edit = $siswa;
                    break;
                }
            }

            if ($siswa_to_edit !== null) {
                ?>
                <form method="post" action="proses1.php">
                    <input type="hidden" name="action" value="edit">
                    <input type="hidden" name="id" value="<?php echo $siswa_to_edit['id']; ?>">
                    <label>Nama Siswa:</label>
                    <input type="text" name="nama" value="<?php echo $siswa_to_edit['nama']; ?>" required>
                    <label>Usia:</label>
                    <input type="number" name="usia" value="<?php echo $siswa_to_edit['usia']; ?>" required>
                    <input type="submit" value="Simpan">
                </form>
                <?php
            } else {
                echo "Data siswa tidak ditemukan.";
            }
        }
    } else {
        echo "ID siswa tidak diberikan.";
    }
    ?>
</body>
</html>

hapus_siswa.php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'GET' && isset($_GET['id'])) {
    // Ambil ID siswa yang akan dihapus dari parameter URL
    $id_siswa = $_GET['id'];

    // Muat data yang ada dari file JSON
    $data_file = 'data_siswa.json';
    $data_siswa = [];
    if (file_exists($data_file)) {
        $data_siswa = json_decode(file_get_contents($data_file), true);
    }

    // Cari indeks siswa yang sesuai dengan ID
    $index_to_remove = null;
    foreach ($data_siswa as $index => $siswa) {
        if ($siswa['id'] === $id_siswa) {
            $index_to_remove = $index;
            break;
        }
    }

    // Jika ID siswa ditemukan, hapus siswa dari array data
    if ($index_to_remove !== null) {
        unset($data_siswa[$index_to_remove]);

        // Simpan kembali data array yang telah diperbarui ke file JSON
        file_put_contents($data_file, json_encode(array_values($data_siswa)));
    }
}

// Alihkan kembali ke MenuUtama.php setelah penghapusan berhasil
header('Location: MenuUtama.php');
exit;
?>
