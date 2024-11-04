# Dimas-Luthfi

### Nama : DIMAS LUTHFI AZIS
### NIM  : 21.01.55.0010
### SISTEM INFORMASI

### MEMBUAT TABBEL PADA DATABSE MYSQL
  ```SQL
        CREATE TABLE `buku` (
      `id` int(11) NOT NULL,
      `title` varchar(255) NOT NULL,
      `author` varchar(255) NOT NULL,
      `stock` int(11) NOT NULL,
      `year` int(11) NOT NULL
        ) 
        INSERT INTO `buku` (`id`, `title`, `author`, `stock`, `year`) VALUES
    (1, 'Kailono the piggy', 'Kailana', 10, 2022),
    (2, 'Iwan the piggy', 'Cik irwan', 11, 2021),
    (3, 'Tegur the piggy', 'Tegar', 12, 2020),
    (4, 'Brian guru agomo', 'Nugroho', 13, 2019),
    (5, 'Barca menang ', 'Xavi hernandes', 14, 2040);
  ```

### MEMBUAT FILE `BUKU_API.PHP` PADA FOLDER `BUKU`
```PHP
  <?php
header("Content-Type: application/json; charset=UTF-8");
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");
header("Access-Control-Allow-Headers: Content-Type, Access-Control-Allow-Headers, Authorization, X-Requested-With");

$method = $_SERVER['REQUEST_METHOD'];
$request = [];

if (isset($_SERVER['PATH_INFO'])) {
    $request = explode('/', trim($_SERVER['PATH_INFO'],'/'));
}

function getConnection() {
    $host = 'localhost';
    $db   = 'buku';
    $user = 'root';
    $pass = ''; // Ganti dengan password MySQL Anda jika ada
    $charset = 'utf8mb4';

    $dsn = "mysql:host=$host;dbname=$db;charset=$charset";
    $options = [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false,
    ];
    try {
        return new PDO($dsn, $user, $pass, $options);
    } catch (\PDOException $e) {
        throw new \PDOException($e->getMessage(), (int)$e->getCode());
    }
}

function response($status, $data = NULL) {
    header("HTTP/1.1 " . $status);
    if ($data) {
        echo json_encode($data);
    }
    exit();
}

$db = getConnection();

switch ($method) {
    case 'GET':
        if (!empty($request) && isset($request[0])) {
            $id = $request[0];
            $stmt = $db->prepare("SELECT * FROM buku WHERE id = ?");
            $stmt->execute([$id]);
            $book = $stmt->fetch();
            if ($book) {
                response(200, $book);
            } else {
                response(404, ["message" => "Book not found"]);
            }
        } else {
            $stmt = $db->query("SELECT * FROM buku");
            $buku = $stmt->fetchAll();
            response(200, $buku);
        }
        break;
    
    case 'POST':
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->title) || !isset($data->author) || !isset($data->stock) || !isset($data->year)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "INSERT INTO buku (title, author, stock, year) VALUES (?, ?, ?, ?)";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->title, $data->author, $data->stock, $data->year])) {
            response(201, ["message" => "Book created", "id" => $db->lastInsertId()]);
        } else {
            response(500, ["message" => "Failed to create book"]);
        }
        break;
    
    
    case 'PUT':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "Book ID is required"]);
        }
        $id = $request[0];
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->title) || !isset($data->author) || !isset($data->stock) || !isset($data->year)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "UPDATE buku SET title = ?, author = ?, stock = ?, year = ? WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->title, $data->author, $data->stock, $data->year, $id])) {
            response(200, ["message" => "Book updated"]);
        } else {
            response(500, ["message" => "Failed to update book"]);
        }
        break;
    
    case 'DELETE':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "Book ID is required"]);
        }
        $id = $request[0];
        $sql = "DELETE FROM buku WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$id])) {
            response(200, ["message" => "Book deleted"]);
        } else {
            response(500, ["message" => "Failed to delete book"]);
        }
        break;
    
    default:
        response(405, ["message" => "Method not allowed"]);
        break;
}
?>
```
### MENAMPILKAN SEMUA DAFTAR BUKU
1. MENGGUNAKAN METOD GET `http://localhost/buku/bUKU_api.php`
2. MWNAMPILKAN HASIL :
   ```JSON
   [
    {
        "id": 1,
        "title": "Kailono the piggy",
        "author": "Kailana",
        "stock": 10,
        "year": 2022
    },
    {
        "id": 2,
        "title": "Iwan the piggy",
        "author": "Cik irwan",
        "stock": 11,
        "year": 2021
    },
    {
        "id": 3,
        "title": "Tegur the piggy",
        "author": "Tegar",
        "stock": 12,
        "year": 2020
    },
    {
        "id": 4,
        "title": "Brian guru agomo",
        "author": "Nugroho",
        "stock": 13,
        "year": 2019
    },
    {
        "id": 5,
        "title": "Barca menang ",
        "author": "Xavi hernandes",
        "stock": 14,
        "year": 2040
    }
    ]
   ```
   ### MENAMPILKAN SALAH SATU/SPESIFIKASI BUKU MELALUI ID
   1.MENGGUNAKAN METOD GET`http://localhost/buku/buku_api.php/3`
   2.MENAMPILKAN HASIL :
   ```
   {
    "id": 3,
    "title": "Tegur the piggy",
    "author": "Tegar",
    "stock": 12,
    "year": 2020
   }
   ```
   ### MENAMBAHKAN BUKU BARU PADA TABEL
   1.MENGGUNAKAN METOD POST`http://localhost/buku/buku_api.php/6`
   ```
   {
    "title": "RANTI MINION",
    "author": "Ranti Nurmala",
    "stock":"22",
    "year": 2015
   }
   ```
   2.MENAMPILKAN HASIL :
   ```
   {
    "message": "Book created",
    "id": "7"
   }
   ```
   ### MEMPERBAHARUI BUKU PADA TABEL
   1.MENGGUNAKAN METOD PUT `http://localhost/buku/buku_api.php/7`
   2.MENAMPILKAN HASIL:
   ```
   {
    "message": "Book updated"
   }
   ```
   ### MENGHAPUS BUKU PADA TABEL
   1.MENGGUNAKAN METOD DELETE `http://localhost/buku/buku_api.php/7`
   2.MENAMPILKAN HASIL :
   ```
   {
    "message": "Book deleted"
   }
   ```
