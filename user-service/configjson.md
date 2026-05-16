# Dokumentasi Konfigurasi Aplikasi (`config.json`)

File konfigurasi ini berfungsi sebagai **pusat pengaturan** untuk aplikasi berbasis Golang yang sedang Anda bangun. 

Daripada menuliskan pengaturan secara langsung (*hardcode*) di dalam kode program, memisahkannya ke dalam file JSON membuat aplikasi menjadi lebih fleksibel, aman, dan mudah dikonfigurasi saat berpindah *environment* (misal dari komputer lokal ke server produksi) tanpa perlu melakukan kompilasi ulang kode Golang.

---

## 📋 Rincian Kegunaan Konfigurasi

### 1. Pengaturan Dasar Aplikasi (`App Settings`)
* **`port` (8001):** Menentukan di "pintu" mana server Golang Anda akan berjalan dan menerima permintaan (contoh akses: `http://localhost:8001`).
* **`appName` ("user-service"):** Nama layanan aplikasi Anda. Berguna untuk identitas pencatatan (*logging*) agar Anda tahu log tersebut berasal dari microservice *User*.
* **`appEnv` ("local"):** Menandakan lingkungan tempat aplikasi berjalan saat ini. Nilai `"local"` berarti dijalankan di komputer Anda sendiri. Nanti saat dirilis, ini bisa diubah menjadi `"production"` atau `"staging"`.

### 2. Kunci Keamanan (`Security Key`)
* **`signatureKey`:** Kunci rahasia yang digunakan untuk menandatangani atau memverifikasi data tertentu, seperti memvalidasi *webhook* dari pihak ketiga (misal: payment gateway) atau enkripsi internal.

### 3. Pengaturan Database PostgreSQL (`Database Settings`)
Berdasarkan standar `port: 5432`, aplikasi ini dikonfigurasi untuk terhubung ke database **PostgreSQL**.
* **Kredensial Utama:** `host`, `port`, `name`, `username`, dan `password` merupakan informasi wajib agar Golang bisa masuk dan mengelola data di database `user_service`.
* **Pengaturan Connection Pooling (Manajemen Koneksi):**
    * **`maxOpenConnection` (10):** Batas maksimal koneksi ke database yang boleh dibuka secara bersamaan oleh aplikasi.
    * **`maxLifetimeConnection` (10):** Batas waktu berapa lama sebuah koneksi boleh digunakan sebelum dihancurkan dan dibuat ulang (mencegah kebocoran memori).
    * **`maxIdleConnection` (10):** Jumlah koneksi yang tetap dibiarkan menyala (*standby*) meskipun sedang tidak ada aktivitas, supaya saat ada *request* baru aplikasi tidak perlu membuat koneksi dari awal lagi.
    * **`maxIdleTime` (10):** Batas waktu koneksi *idle* boleh menganggur sebelum akhirnya diputus otomatis oleh sistem.

### 4. Pengaturan Komponen Pertahanan (`Rate Limiter`)
* **`rateLimiterMaxRequest` (1000)** & **`rateLimiterTimeSecond` (60):** Berfungsi membatasi agar satu pengguna atau satu IP hanya boleh mengirim maksimal 1000 *request* dalam kurun waktu 60 detik. Berguna untuk mencegah serangan *spamming*, *brute-force*, atau DDoS.

### 5. Pengaturan Autentikasi (`JWT Settings`)
* **`jwtSecretKey`:** Kunci rahasia untuk membuat (*sign*) dan memvalidasi token login user. Kunci ini harus dijaga ketat agar tidak bocor ke publik.
* **`jwtExpirationTime` (1440):** Waktu kedaluwarsa token JWT dalam satuan menit (1440 menit = 24 jam). Jika lewat dari batas waktu ini, sesi user habis dan mereka harus *login* kembali.

---

## 🚀 Implementasi pada Golang (Struct Matcher)

Di dalam project Golang, data JSON di atas biasanya akan di-*parse* ke dalam sebuah struktur data (`struct`) seperti berikut:

```go
type Config struct {
	Port                  int      `json:"port"`
	AppName               string   `json:"appName"`
	AppEnv                string   `json:"appEnv"`
	SignatureKey          string   `json:"signatureKey"`
	Database              DBConfig `json:"database"`
	RateLimiterMaxRequest int      `json:"rateLimiterMaxRequest"`
	RateLimiterTimeSecond int      `json:"rateLimiterTimeSecond"`
	JwtSecretKey          string   `json:"jwtSecretKey"`
	JwtExpirationTime     int      `json:"jwtExpirationTime"`
}

type DBConfig struct {
	Host                  string `json:"host"`
	Port                  int    `json:"port"`
	Name                  string `json:"name"`
	Username              string `json:"username"`
	Password              string `json:"password"`
	MaxOpenConnection     int    `json:"maxOpenConnection"`
	MaxLifetimeConnection int    `json:"maxLifetimeConnection"`
	MaxIdleConnection     int    `json:"maxIdleConnection"`
	MaxIdleTime           int    `json:"maxIdleTime"`
}