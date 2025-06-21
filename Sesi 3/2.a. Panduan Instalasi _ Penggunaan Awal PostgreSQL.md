## 📚 **Panduan Instalasi & Penggunaan Awal PostgreSQL**  
Berikut panduan komprehensif instalasi PostgreSQL di berbagai OS dan penggunaan dasar:

---

### 🛠️ **Instalasi PostgreSQL**  
#### **1. Windows**  
##### **Metode 1: Installer Resmi**  
1. **Download**:  
   - [PostgreSQL Download Page](https://www.postgresql.org/download/windows/)  
   - Pilih versi terbaru (misal: PostgreSQL 16)  
2. **Instalasi**:  
   - Jalankan installer  
   - Centang komponen:  
     ✅ PostgreSQL Server  
     ✅ pgAdmin 4 (GUI tool)  
     ✅ Command Line Tools  
   - Atur password untuk superuser `postgres`  
   - Port default: `5432`  
3. **Verifikasi**:  
   ```powershell
   psql --version
   # psql (PostgreSQL) 16.2
   ```

##### **Metode 2: Chocolatey**  
```powershell
choco install postgresql16
```

---

#### **2. macOS**  
##### **Metode 1: Homebrew**  
```bash
brew install postgresql@16
brew services start postgresql@16
```

##### **Metode 2: Postgres.app**  
1. Download [Postgres.app](https://postgresapp.com/)  
2. Drag ke Applications folder  
3. Klik "Initialize" untuk buat cluster baru  

---

#### **3. Linux (Debian/Ubuntu)**  
```bash
# Tambahkan repository resmi
sudo sh -c 'echo "deb https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Install
sudo apt update
sudo apt install postgresql-16

# Start service
sudo systemctl start postgresql
```

---

#### **4. Linux (Fedora/RHEL/CentOS)**  
```bash
# Fedora
sudo dnf install postgresql-server

# CentOS/RHEL
sudo yum install postgresql-server

# Inisialisasi DB
sudo postgresql-setup --initdb
sudo systemctl start postgresql
```

---

### 🔑 **Konfigurasi Awal**  
#### **1. Login ke PostgreSQL**  
```bash
sudo -u postgres psql  # Linux/macOS
psql -U postgres       # Windows
```

#### **2. Buat User & Database**  
```sql
CREATE USER gov_user WITH PASSWORD 'securePass123!';
CREATE DATABASE gov_db OWNER gov_user;
GRANT ALL PRIVILEGES ON DATABASE gov_db TO gov_user;
```

#### **3. Konfigurasi Akses Remote**  
Edit file `pg_hba.conf` (lokasi: `/etc/postgresql/16/main/` atau `C:\Program Files\PostgreSQL\16\data`):  
```conf
# Tambahkan baris
host    all             all             0.0.0.0/0               scram-sha-256
```

Edit `postgresql.conf`:  
```conf
listen_addresses = '*'
```

Restart service:  
```bash
sudo systemctl restart postgresql  # Linux
# Atau via Services Manager di Windows
```

---

### 🚀 **Penggunaan Dasar**  
#### **1. Operasi Database**  
```bash
# Login sebagai user
psql -h localhost -U gov_user -d gov_db

# Daftar database
\l

# Switch database
\c gov_db
```

#### **2. Buat Tabel & CRUD**  
```sql
-- Buat tabel
CREATE TABLE citizens (
    id SERIAL PRIMARY KEY,
    nik VARCHAR(18) UNIQUE NOT NULL,
    name VARCHAR(100),
    birth_date DATE,
    address JSONB
);

-- Insert data
INSERT INTO citizens (nik, name, birth_date) 
VALUES ('3273011234567890', 'Budi Santoso', '1990-05-15');

-- Query
SELECT * FROM citizens WHERE birth_date > '1985-01-01';

-- Update
UPDATE citizens SET address = '{"street": "Jl. Merdeka 1", "city": "Jakarta"}' 
WHERE nik = '3273011234567890';

-- Delete
DELETE FROM citizens WHERE id = 1;
```

#### **3. Backup & Restore**  
```bash
# Backup
pg_dump -U gov_user -h localhost gov_db > gov_backup.sql

# Restore
psql -U gov_user -h localhost gov_db < gov_backup.sql

# Backup format binary (direkomendasikan)
pg_dump -Fc -U gov_user gov_db > gov_backup.dump
pg_restore -U gov_user -d gov_db gov_backup.dump
```

---

### 📊 **GUI Management Tools**  
1. **pgAdmin 4**:  
   - Termasuk dalam instalasi Windows  
   - Akses via browser: `http://localhost:5050`  
2. **DBeaver**:  
   - Open source: [dbeaver.io](https://dbeaver.io/)  
   - Support semua database  
3. **VS Code Extension**:  
   - [PostgreSQL by Chris Kolkman](https://marketplace.visualstudio.com/items?itemName=ckolkman.vscode-postgres)

---

### ⚠️ **Best Practices Keamanan**  
1. **Ganti Password Default**:  
   ```sql
   ALTER USER postgres PASSWORD 'StrongPassword!123';
   ```
2. **Buat User Khusus Aplikasi**:  
   ```sql
   CREATE USER app_user WITH PASSWORD 'AppPass!456' NOSUPERUSER;
   ```
3. **Aktifkan SSL**:  
   Edit `postgresql.conf`:  
   ```conf
   ssl = on
   ssl_cert_file = 'server.crt'
   ssl_key_file = 'server.key'
   ```
4. **Audit Logging**:  
   ```conf
   log_statement = 'all'
   log_directory = 'pg_log'
   ```

---

### 🔍 **Troubleshooting Umum**  
| Masalah | Solusi |  
|---------|--------|  
| Connection refused | Periksa `listen_addresses` di postgresql.conf |  
| Password authentication failed | Edit `pg_hba.conf` set method `md5` atau `scram-sha-256` |  
| Out of memory | Atur `shared_buffers` di postgresql.conf (25% RAM) |  
| Too many connections | Tingkatkan `max_connections` (default 100) |  

---

### 📚 **Learning Resources**  
1. [Official Docs](https://www.postgresql.org/docs/16/)  
2. [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)  
3. [PGExercises](https://pgexercises.com/)  
4. [Buku: "The Art of PostgreSQL"](https://theartofpostgresql.com/)

```bash
# Cheat sheet cepat
\?          # Bantuan command psql
\h          # Bantuan SQL
\timing     # Aktifkan timer query
\dn         # List schemas
\dt+        # List tables with detail
EXPLAIN ANALYZE SELECT * FROM citizens;  # Analisis performa query
```

> 🚀 **Tip Proyek Pemerintah**:  
> - Gunakan **PostGIS** untuk data geospasial: `CREATE EXTENSION postgis;`  
> - Implementasi partisi tabel untuk data besar (e.g., laporan bulanan)  
> - Setup replication untuk high availability  

## 🏛️ **Contoh Kasus: Layanan Kependudukan Microservice**  
**Database**: PostgreSQL  
**Tabel**: `citizens`  
**Struktur**:  
```sql  
CREATE TABLE citizens (  
    id SERIAL PRIMARY KEY,  
    nik VARCHAR(18) UNIQUE NOT NULL,  -- Nomor Induk Kependudukan  
    name VARCHAR(100) NOT NULL,  
    birth_date DATE NOT NULL,  
    address JSONB NOT NULL,  
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP  
);  
```  

---

### 🔌 **Koneksi Database & Operasi Dasar**  
#### **1. Node.js Implementation (Express + pg)**  
```javascript  
// db.js - Koneksi PostgreSQL  
const { Pool } = require('pg');  
const pool = new Pool({  
  user: 'gov_user',  
  host: 'localhost',  
  database: 'citizen_db',  
  password: 'securePass123!',  
  port: 5432,  
});  

// citizen-service.js - CRUD Operations  
const getCitizenById = async (id) => {  
  const res = await pool.query('SELECT * FROM citizens WHERE id = $1', [id]);  
  return res.rows[0];  
};  

const createCitizen = async (citizen) => {  
  const { nik, name, birth_date, address } = citizen;  
  const res = await pool.query(  
    `INSERT INTO citizens (nik, name, birth_date, address)  
     VALUES ($1, $2, $3, $4) RETURNING *`,  
    [nik, name, birth_date, address]  
  );  
  return res.rows[0];  
};  

// app.js - REST API  
const express = require('express');  
const app = express();  
app.use(express.json());  

app.post('/citizens', async (req, res) => {  
  try {  
    const citizen = await createCitizen(req.body);  
    res.status(201).json(citizen);  
  } catch (err) {  
    res.status(500).json({ error: err.message });  
  }  
});  

app.listen(3000, () => console.log('Server running on port 3000'));  
```  

#### **Cara Eksekusi**:  
```bash  
npm install express pg  
node app.js  
```  

#### **Test Curl**:  
```bash  
curl -X POST http://localhost:3000/citizens \  
  -H "Content-Type: application/json" \  
  -d '{  
    "nik": "3273011234567890",  
    "name": "Budi Santoso",  
    "birth_date": "1990-05-15",  
    "address": {  
      "street": "Jl. Merdeka No.1",  
      "city": "Jakarta Pusat"  
    }  
  }'  
```  

---

### 🐿️ **2. Go Implementation (Gin + pgx)**  
```go  
// db.go - Koneksi Database  
package db  

import (  
	"context"  
	"github.com/jackc/pgx/v5/pgxpool"  
)  

var DB *pgxpool.Pool  

func InitDB(connString string) error {  
	config, err := pgxpool.ParseConfig(connString)  
	if err != nil {  
		return err  
	}  
	DB, err = pgxpool.NewWithConfig(context.Background(), config)  
	return err  
}  

// citizen.go - Model & Repository  
type Citizen struct {  
	ID        int       `json:"id"`  
	NIK       string    `json:"nik"`  
	Name      string    `json:"name"`  
	BirthDate time.Time `json:"birth_date"`  
	Address   string    `json:"address"`  
}  

func GetCitizenByID(id int) (Citizen, error) {  
	var c Citizen  
	err := DB.QueryRow(context.Background(),  
		"SELECT id, nik, name, birth_date, address FROM citizens WHERE id = $1",  
		id,  
	).Scan(&c.ID, &c.NIK, &c.Name, &c.BirthDate, &c.Address)  
	return c, err  
}  

// main.go - REST API  
package main  

import (  
	"net/http"  
	"github.com/gin-gonic/gin"  
	"gov-microservice/db"  
)  

func main() {  
	// Inisialisasi DB  
	if err := db.InitDB("postgres://gov_user:securePass123!@localhost:5432/citizen_db"); err != nil {  
		panic(err)  
	}  

	r := gin.Default()  

	r.GET("/citizens/:id", func(c *gin.Context) {  
		id := c.Param("id")  
		citizen, err := db.GetCitizenByID(id)  
		if err != nil {  
			c.JSON(http.StatusNotFound, gin.H{"error": "Record not found"})  
			return  
		}  
		c.JSON(http.StatusOK, citizen)  
	})  

	r.Run(":8080")  
}  
```  

#### **Cara Eksekusi**:  
```bash  
go mod init gov-microservice  
go get github.com/gin-gonic/gin  
go get github.com/jackc/pgx/v5  
go run main.go  
```  

#### **Test Curl**:  
```bash  
curl http://localhost:8080/citizens/1  
```  

---

### 🔄 **3. Migrasi Database dengan Flyway**  
**File**: `V1__create_citizens_table.sql`  
```sql  
CREATE TABLE citizens (...); -- DDL dari sebelumnya  
```  

**Konfigurasi**:  
```properties  
# flyway.conf  
flyway.url=jdbc:postgresql://localhost:5432/citizen_db  
flyway.user=gov_user  
flyway.password=securePass123!  
```  

**Eksekusi**:  
```bash  
flyway migrate  
```  

---

### 📊 **4. Query Complex: Join dengan Data Wilayah**  
**Tabel Tambahan**:  
```sql  
CREATE TABLE regions (  
    kode_wilayah VARCHAR(10) PRIMARY KEY,  
    nama_provinsi VARCHAR(50),  
    nama_kabupaten VARCHAR(50)  
);  
```  

**Query**:  
```sql  
SELECT c.id, c.nik, c.name, r.nama_provinsi, r.nama_kabupaten  
FROM citizens c  
JOIN regions r ON SUBSTRING(c.nik, 1, 4) = r.kode_wilayah;  
```  

**Implementasi di Node.js**:  
```javascript  
const getCitizensWithRegion = async () => {  
  const query = `  
    SELECT c.id, c.nik, c.name, r.nama_provinsi, r.nama_kabupaten  
    FROM citizens c  
    JOIN regions r ON SUBSTRING(c.nik, 1, 4) = r.kode_wilayah  
  `;  
  return (await pool.query(query)).rows;  
};  
```  

---

### ⚠️ **5. Best Practices untuk Microservices**  
1. **Koneksi Database**:  
   - Gunakan connection pooling (`pg.Pool` di Node.js, `pgxpool` di Go)  
   - Atur max connections sesuai kapasitas DB  

2. **Keamanan**:  
   - Selalu gunakan parameterized query untuk hindari SQL injection  
   ```go  
   // Contoh aman di Go:  
   DB.QueryRow("SELECT ... WHERE id = $1", id)  
   ```  

3. **Error Handling**:  
   - Handle error khusus PostgreSQL:  
   ```javascript  
   // Node.js  
   if (err.code === '23505') { // unique_violation  
     res.status(409).json({ error: 'NIK already exists' });  
   }  
   ```  

4. **Observability**:  
   - Log semua query lambat (> 100ms)  
   - Integrasi dengan Prometheus untuk monitor koneksi DB  

5. **Migrations**:  
   - Gunakan tools migrasi (Flyway, Liquibase, Goose)  
   - Versi kontrol untuk skema database  

---

### 📈 **Studi Kasus Nyata: Sistem Dukcapil Nasional**  
**Arsitektur**:  
```mermaid  
flowchart TB  
    Mobile[App Mobile] --> API[API Gateway]  
    Web[Portal Web] --> API  
    API --> Node[Node.js: Layanan Pencarian]  
    API --> Go[Go: Layanan Validasi]  
    Node --> PG[(PostgreSQL)]  
    Go --> PG  
    PG -->|Replikasi| DR[Disaster Recovery Site]  
```  

**Statistik**:  
- 500+ request/detik saat verifikasi pemilu  
- Latency < 50ms untuk 99% request  
- Data 270 juta penduduk terdistribusi di 12 shard  

**Optimasi**:  
- Indeks GIN pada kolom JSONB `address`  
- Partisi tabel berdasarkan kode provinsi  
- Connection pool dengan PgBouncer  

```sql  
-- Contoh indeks JSONB  
CREATE INDEX idx_address_city ON citizens  
USING gin ((address->>'city'));  
```  

Dengan pola ini, layanan kependudukan dapat diskalakan secara independen dan diintegrasikan dengan sistem pemerintah lainnya melalui API! 🏛️💻
