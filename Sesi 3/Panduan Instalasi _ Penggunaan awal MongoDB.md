## 📚 **Panduan Instalasi & Penggunaan Awal MongoDB**  
Berikut panduan komprehensif instalasi MongoDB di berbagai OS dan penggunaan dasar:

---

### 🛠️ **Instalasi MongoDB**  
#### **1. Windows**  
##### **Metode 1: Installer Resmi**  
1. **Download**:  
   - [MongoDB Community Server](https://www.mongodb.com/try/download/community)  
   - Pilih versi terbaru (misal: 7.0)  
2. **Instalasi**:  
   - Jalankan installer `.msi`  
   - Pilih "Complete" setup type  
   - Centang: **Install MongoDB as a Service**  
   - Data directory default: `C:\Program Files\MongoDB`  
3. **Verifikasi**:  
   ```powershell
   mongod --version
   # db version v7.0.5
   ```

##### **Metode 2: Chocolatey**  
```powershell
choco install mongodb
```

---

#### **2. macOS**  
##### **Metode 1: Homebrew**  
```bash
brew tap mongodb/brew
brew install mongodb-community@7.0
brew services start mongodb-community@7.0
```

##### **Metode 2: Manual Install**  
```bash
curl -O https://fastdl.mongodb.org/osx/mongodb-macos-x86_64-7.0.5.tgz
tar -zxvf mongodb-macos-x86_64-7.0.5.tgz
sudo mv mongodb-macos-x86_64-7.0.5 /usr/local/mongodb
echo 'export PATH="/usr/local/mongodb/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

---

#### **3. Linux (Debian/Ubuntu)**  
```bash
# Import public key
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -

# Create list file
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu $(lsb_release -cs)/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Install
sudo apt update
sudo apt install -y mongodb-org

# Start service
sudo systemctl start mongod
sudo systemctl enable mongod
```

---

#### **4. Linux (RHEL/CentOS/Fedora)**  
```bash
# Create repo file
cat <<EOF | sudo tee /etc/yum.repos.d/mongodb-org-7.0.repo
[mongodb-org-7.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/\$releasever/mongodb-org/7.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-7.0.asc
EOF

# Install
sudo yum install -y mongodb-org

# Start service
sudo systemctl start mongod
sudo systemctl enable mongod
```

---

### ⚙️ **Konfigurasi Awal**  
#### **1. Jalankan MongoDB Shell**  
```bash
mongosh
```

#### **2. Buat Database & User**  
```javascript
// Buat database baru
use citizen_db

// Buat user dengan role
db.createUser({
  user: "gov_admin",
  pwd: "SecurePass123!",
  roles: [ { role: "readWrite", db: "citizen_db" } ]
})
```

#### **3. Aktifkan Autentikasi**  
Edit file konfigurasi (`/etc/mongod.conf` atau `C:\Program Files\MongoDB\Server\7.0\bin\mongod.cfg`):  
```yaml
security:
  authorization: enabled
```

Restart service:  
```bash
sudo systemctl restart mongod  # Linux
# Atau via Services Manager di Windows
```

---

### 🚀 **Penggunaan Dasar**  
#### **1. Operasi Database**  
```bash
# Login dengan autentikasi
mongosh -u gov_admin -p SecurePass123! --authenticationDatabase citizen_db

# Lihat database
show dbs

# Gunakan database
use citizen_db
```

#### **2. Buat Collection & CRUD**  
```javascript
// Buat collection citizens
db.createCollection("citizens")

// Insert dokumen
db.citizens.insertOne({
  nik: "3273011234567890",
  name: "Budi Santoso",
  birth_date: new Date("1990-05-15"),
  address: {
    street: "Jl. Merdeka No.1",
    city: "Jakarta Pusat"
  },
  created_at: new Date()
})

// Query
db.citizens.find({ "address.city": "Jakarta Pusat" })

// Update
db.citizens.updateOne(
  { nik: "3273011234567890" },
  { $set: { "address.street": "Jl. Kemerdekaan 45" } }
)

// Delete
db.citizens.deleteOne({ nik: "3273011234567890" })
```

#### **3. Backup & Restore**  
```bash
# Backup
mongodump --uri="mongodb://gov_admin:SecurePass123!@localhost:27017/citizen_db" --out=./backup

# Restore
mongorestore --uri="mongodb://gov_admin:SecurePass123!@localhost:27017/citizen_db" ./backup/citizen_db
```

---

### 📊 **GUI Management Tools**  
1. **MongoDB Compass**:  
   - [Download Resmi](https://www.mongodb.com/try/download/compass)  
   - Fitur lengkap: visual query, performance monitoring  
2. **VS Code Extension**:  
   - [MongoDB for VS Code](https://marketplace.visualstudio.com/items?itemName=mongodb.mongodb-vscode)  
3. **Studio 3T**:  
   - [Free Edition](https://studio3t.com/download/)  

---

### 🔐 **Best Practices Keamanan**  
1. **Aktifkan Enkripsi**:  
   ```yaml
   # mongod.conf
   security:
     enableEncryption: true
     encryptionKeyFile: /etc/mongodb-keyfile
   ```
2. **Batas Akses Jaringan**:  
   ```yaml
   net:
     bindIp: 127.0.0.1,192.168.1.100  # Hanya izinkan IP tertentu
   ```
3. **Audit Logging**:  
   ```yaml
   auditLog:
     destination: file
     format: JSON
     path: /var/log/mongodb/audit.json
   ```
4. **Role-Based Access Control**:  
   ```javascript
   db.createRole({
     role: "data_auditor",
     privileges: [{
       resource: { db: "citizen_db", collection: "citizens" },
       actions: ["find"]
     }],
     roles: []
   })
   ```

---

### 🔍 **Troubleshooting Umum**  
| Masalah | Solusi |  
|---------|--------|  
| Connection refused | Periksa `bindIp` di konfigurasi |  
| Authentication failed | Verifikasi username/db auth |  
| Permission denied | Pastikan hak akses folder data `/var/lib/mongodb` |  
| WiredTiger error | Periksa space disk & journaling |  

---

### 💻 **Integrasi dengan Microservices**  
#### **Node.js (Mongoose)**  
```javascript
// app.js
const mongoose = require('mongoose');

// Koneksi MongoDB
mongoose.connect('mongodb://gov_admin:SecurePass123!@localhost:27017/citizen_db');

// Definisikan schema
const citizenSchema = new mongoose.Schema({
  nik: { type: String, required: true, unique: true },
  name: { type: String, required: true },
  birth_date: Date,
  address: Object
});

// Buat model
const Citizen = mongoose.model('Citizen', citizenSchema);

// Contoh penggunaan
async function createCitizen(data) {
  const citizen = new Citizen(data);
  return await citizen.save();
}

// Test
createCitizen({
  nik: '3273019876543210',
  name: 'Siti Rahayu',
  birth_date: new Date('1985-11-20'),
  address: { city: 'Bandung' }
}).then(console.log);
```

#### **Go (Official Driver)**  
```go
package main

import (
	"context"
	"fmt"
	"log"
	"time"
	
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)

type Citizen struct {
	NIK       string    `bson:"nik"`
	Name      string    `bson:"name"`
	BirthDate time.Time `bson:"birth_date"`
	Address   bson.M    `bson:"address"`
}

func main() {
	// Koneksi MongoDB
	client, err := mongo.Connect(context.TODO(), 
		options.Client().ApplyURI("mongodb://gov_admin:SecurePass123!@localhost:27017"))
	if err != nil {
		log.Fatal(err)
	}
	defer client.Disconnect(context.TODO())

	// Akses collection
	coll := client.Database("citizen_db").Collection("citizens")

	// Insert dokumen
	doc := Citizen{
		NIK:       "3674011122334455",
		Name:      "Ahmad Fauzi",
		BirthDate: time.Date(1995, 3, 12, 0, 0, 0, 0, time.UTC),
		Address:   bson.M{"city": "Depok"},
	}
	
	result, err := coll.InsertOne(context.TODO(), doc)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Inserted ID: %v\n", result.InsertedID)
}
```

---

### 📚 **Learning Resources**  
1. [MongoDB University](https://learn.mongodb.com) (Gratis)  
2. [MongoDB Manual](https://www.mongodb.com/docs/manual/)  
3. [MongoDB Aggregation Framework Guide](https://university.mongodb.com/mercury/M121/2023_October_17/chapter/Chapter_1_Introduction)  
4. [Buku: "MongoDB: The Definitive Guide"](https://www.oreilly.com/library/view/mongodb-the-definitive/9781491954454/)

```javascript
// Cheat sheet cepat MongoDB
db.help()                  // Bantuan database
db.citizens.find().limit(5) // Ambil 5 dokumen
db.citizens.countDocuments() // Hitung total dokumen
db.citizens.createIndex({ "address.city": 1 }) // Buat indeks
db.citizens.explain().find({ "address.city": "Jakarta" }) // Analisis query
```

> 🚀 **Tip Proyek Pemerintah**:  
> - Gunakan **Sharded Cluster** untuk data besar (>1TB)  
> - Implementasi **Change Streams** untuk real-time notifications  
> - Gunakan **Atlas** untuk managed service di lingkungan produksi  
> - Enkripsi field sensitif dengan **Client-Side Field Level Encryption**

Dengan MongoDB terinstal, Anda siap membangun aplikasi berbasis dokumen yang skalabel untuk kebutuhan pemerintah! 🏛️💻
