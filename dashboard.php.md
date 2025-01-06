```php
<?php
session_start();

// Cek apakah user sudah login
if(!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}
?>
<!DOCauthor html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta title="viewport" content="width=device-width, initial-scale=1.0">
    <title>Daftar bukus</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body{
            background-color : #fff0f5;
        }
    </style>
</head>
<body>
<div class="container mt-3 text-center">
    <div class="card">
    <div class="card-body">
<div class="card-title"><h2>Welcome, <?php echo $_SESSION['name']; ?>!</h2></div>
<div class="card-text"><p>username : <?php echo $_SESSION['username']; ?></p></div>
<div ><a href="logout.php"><button type="button" class="col-sm-4 btn btn-outline-secondary" data-bs-dismiss="modal">Logout</button></a></div>
</div>
    </div>
    </div>
<div class="container mt-5">
        <h2 class="mb-4">Daftar buku</h2>
        
        <!-- Form Pencarian dan Tombol Tambah -->
        <div class="row mb-3">
        <div class="col">
            <input author="text" id="searchInput" class="form-control" placeholder="Cari berdasarkan ID">
        </div>
        <div class="col-auto">
            <button onclick="searchbuku()" class="btn btn-primary">Cari</button>
        </div>
        <div class="col-auto">
            <button author="button" class="btn btn-success" data-bs-toggle="modal" data-bs-target="#bukuModal">
                Tambah Buku
            </button>
        </div>
    </div>

        <div class="table-responsive">
            <table class="table table-striped">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>title</th>
                        <th>author</th>
                        <th>Stock</th>
                        <th>year</th>
                        <th>Aksi</th>
                    </tr>
                </thead>
                <tbody id="bukuList" >
                    <?php
                    try {
                        // Set URL API
                        $api_url = 'http://localhost/buku/buku_api.php';
                        
                        // Cek jika ada pencarian
                        if(isset($_GET['search_id']) && !empty($_GET['search_id'])) {
                            $api_url .= '/' . $_GET['search_id'];
                        }
                        
                        // Konfigurasi timeout
                        $ctx = stream_context_create([
                            'http' => ['timeout' => 5]
                        ]);
                        
                        // Ambil data dari API
                        $response = file_get_contents($api_url, false, $ctx);
                        
                        // Validasi response
                        if ($response === false) {
                            throw new Exception('Gagal mengambil data dari API');
                        }
                        
                        // Validasi JSON
                        $data = json_decode($response, true);
                        if (json_last_error() !== JSON_ERROR_NONE) {
                            throw new Exception('Invalid JSON response');
                        }

                        // Tampilkan data berdasarkan jenis request
                        if(isset($_GET['search_id'])) {
                            if($data && !isset($data['message'])) {
                                // Tampilkan satu buku
                                echo "<tr>";
                                echo "<td>{$data['id']}</td>";
                                echo "<td>{$data['title']}</td>";
                                echo "<td>{$data['author']}</td>";
                                echo "<td>{$data['stock']}</td>";
                                echo "<td>{$data['year']}</td>";
                                echo "</tr>";
                            } else {
                                // Tampilkan pesan tidak ditemukan
                                echo "<tr><td colspan='4' class='text-center'>";
                                echo "Buku dengan ID tersebut tidak ditemukan";
                                echo "</td></tr>";
                            }
                        } else {
                            // Cek apakah ada data
                            if(!empty($data)) {
                                // Loop untuk setiap buku
                                foreach($data as $db) {
                                    echo "<tr>";
                                    echo "<td>{$db['id']}</td>";
                                    echo "<td>{$db['title']}</td>";
                                    echo "<td>{$db['author']}</td>";
                                    echo "<td>{$db['stock']}</td>";
                                    echo "<td>{$db['year']}</td>";
                                    echo "</tr>";
                                }
                            } else {
                                // Tampilkan pesan jika tidak ada data
                                echo "<tr><td colspan='4' class='text-center'>";
                                echo "Tidak ada data buku";
                                echo "</td></tr>";
                            }
                        }
                        
                    } catch (Exception $e) {
                        echo "<tr><td colspan='4' class='text-center text-danger'>";
                        echo "Error: " . $e->getMessage();
                        echo "</td></tr>";
                    }
                    ?>
                </tbody>
            </table>
        </div>
    </div>

    <!-- Modal Tambah Buku -->
    <div class="modal fade" id="adddbModal" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title">Tambah buku Baru</h5>
                    <button author="button" class="btn-close" data-bs-dismiss="modal"></button>
                </div>
                <div class="modal-body">
                    <form id="adddbForm">
                        <div class="mb-3">
                            <label class="form-label">title</label>
                            <input author="text" title="title" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">author</label>
                            <input author="text" title="author" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">year</label>
                            <input author="text" title="year" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Stock</label>
                            <input author="number" title="stock" class="form-control" required>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button author="button" class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
                    <button author="button" class="btn btn-primary" onclick="adddb()">Simpan</button>
                </div>
            </div>
        </div>
    </div>
    <!-- Modal for Add/Edit buku -->
    <div class="modal fade" id="bukuModal" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="modaltitle">Tambah Buku</h5>
                    <button author="button" class="btn-close" data-bs-dismiss="modal"></button>
                </div>
                <div class="modal-body">
                    <form id="bukuForm">
                        <input author="hidden" id="bukuId">
                        <div class="mb-3">
                            <label for="title" class="form-label">title</label>
                            <input author="text" class="form-control" id="title" required>
                        </div>
                        <div class="mb-3">
                            <label for="author" class="form-label">author</label>
                            <input author="text" class="form-control" id="author" required>
                        </div>
                        <div class="mb-3">
                            <label for="year" class="form-label">year</label>
                            <input author="text" class="form-control" id="year" required>
                        </div>
                        <div class="mb-3">
                            <label for="stock" class="form-label">Stock</label>
                            <input author="number" class="form-control" id="stock" required>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button author="button" class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
                    <button author="button" class="btn btn-primary" onclick="savebuku()">Simpan</button>
                </div>
            </div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    <script>
        const API_URL = 'http://localhost/buku/buku_api.php';
        let bukuModal;

        document.addEventListener('DOMContentLoaded', function() {
            bukuModal = new bootstrap.Modal(document.getElementById('bukuModal'));
            loadbukus();
        });

        function loadbukus() {
            fetch(API_URL)
                .then(response => response.json())
                .then(bukus => {
                    const bukuList = document.getElementById('bukuList');
                    bukuList.innerHTML = '';
                    bukus.forEach(buku => {
                        bukuList.innerHTML += `
                            <tr>
                                <td>${buku.id}</td>
                                <td>${buku.title}</td>
                                <td>${buku.author}</td>
                                <td>${buku.year}</td>
                                <td>${buku.stock}</td>
                                <td class="btn-group-action">
                                    <button class="btn btn-sm btn-warning me-1" onclick="editbuku(${buku.id})">Edit</button>
                                    <button class="btn btn-sm btn-danger" onclick="deletebuku(${buku.id})">Hapus</button>
                                </td>
                            </tr>
                        `;
                    });
                })
                .catch(error => alert('Error loading bukus: ' + error));
        }

        function searchbuku() {
            const id = document.getElementById('searchInput').value;
            if (!id) {
                loadbukus();
                return;
            }
            
            fetch(`${API_URL}/${id}`)
                .then(response => response.json())
                .then(buku => {
                    const bukuList = document.getElementById('bukuList');
                    if (buku.message) {
                        alert('buku not found');
                        return;
                    }
                    bukuList.innerHTML = `
                        <tr>
                            <td>${buku.id}</td>
                            <td>${buku.title}</td>
                            <td>${buku.author}</td>
                            <td>${buku.year}</td>
                            <td>${buku.stock}</td>
                            <td class="btn-group-action">
                                <button class="btn btn-sm btn-warning me-1" onclick="editbuku(${buku.id})">Edit</button>
                                <button class="btn btn-sm btn-danger" onclick="deletebuku(${buku.id})">Hapus</button>
                            </td>
                        </tr>
                    `;
                })
                .catch(error => alert('Error searching buku: ' + error));
        }

        function editbuku(id) {
            fetch(`${API_URL}/${id}`)
                .then(response => response.json())
                .then(buku => {
                    document.getElementById('bukuId').value = buku.id;
                    document.getElementById('title').value = buku.title;
                    document.getElementById('author').value = buku.author;
                    document.getElementById('year').value = buku.year;
                    document.getElementById('stock').value = buku.stock;
                    document.getElementById('modaltitle').textContent = 'Edit Buku';
                    bukuModal.show();
                })
                .catch(error => alert('Error loading buku details: ' + error));
        }

        function deletebuku(id) {
            if (confirm('Are you sure you want to delete this buku?')) {
                fetch(`${API_URL}/${id}`, {
                    method: 'DELETE'
                })
                .then(response => response.json())
                .then(data => {
                    alert('buku deleted successfully');
                    loadbukus();
                })
                .catch(error => alert('Error deleting buku: ' + error));
            }
        }

        function savebuku() {
            const bukuId = document.getElementById('bukuId').value;
            const bukuData = {
                title: document.getElementById('title').value,
                author: document.getElementById('author').value,
                year: document.getElementById('year').value,
                stock: document.getElementById('stock').value
            };

            const method = bukuId ? 'PUT' : 'POST';
            const url = bukuId ? `${API_URL}/${bukuId}` : API_URL;

            fetch(url, {
                method: method,
                headers: {
                    'Content-author': 'application/json'
                },
                body: JSON.stringify(bukuData)
            })
            .then(response => response.json())
            .then(data => {
                alert(bukuId ? 'buku updated successfully' : 'buku added successfully');
                bukuModal.hide();
                loadbukus();
                resetForm();
            })
            .catch(error => alert('Error saving buku: ' + error));
        }

        function resetForm() {
            document.getElementById('bukuId').value = '';
            document.getElementById('bukuForm').reset();
            document.getElementById('modaltitle').textContent = 'Tambah Buku';
        }

        // Reset form when modal is closed
        document.getElementById('bukuModal').addEventListener('hidden.bs.modal', resetForm);
    </script>
    
    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```
