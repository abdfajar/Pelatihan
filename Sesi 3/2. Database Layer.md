## **Database Layer: PostgreSQL vs MongoDB**  
**Fokus**: Pemilihan database relasional vs NoSQL untuk arsitektur microservices pemerintah  

---

### 🏛️ **1. PostgreSQL (Relational Database)**  
#### **Karakteristik Kunci**  
- **ACID Compliance**: Transaksi atomic, konsisten, terisolasi, tahan lama  
- **SQL Standard**: Dukungan penuh SQL:2016 + ekstensi  
- **Advanced Fitur**:  
  - JSONB (binary JSON)  
  - Full-text search  
  - Geospatial (PostGIS)  
  - Time-series data  

#### **Pola Desain Microservices**  
```mermaid  
flowchart TD  
    Service1[Service Pajak] --> PG1[(PostgreSQL)]  
    Service2[Service Izin Usaha] --> PG2[(PostgreSQL)]  
    style PG1 stroke:#336,stroke-width:2px  
    style PG2 stroke:#336,stroke-width:2px  
```  
**Prinsip**: *Satu service - satu database* (isolasi skema)  

#### **Contoh Implementasi Pemerintah**  
**Kasus**: Sistem Pelaporan Keuangan Daerah  
```sql  
CREATE TABLE reports (  
    id UUID PRIMARY KEY,  
    region_id INT REFERENCES regions(id),  
    year INT NOT NULL,  
    quarter INT CHECK (quarter BETWEEN 1 AND 4),  
    budget NUMERIC(15,2),  
    spent NUMERIC(15,2),  
    document JSONB -- Struktur dinamis dokumen pendukung  
);  
```  
**Optimasi**:  
- Partitioning by `region_id` untuk query cepat  
- GiST index pada kolom geospatial  
- Materialized view untuk dashboard real-time  

#### **Operasi Penting**  
```bash  
# Backup dengan custom format  
pg_dump -Fc dbname > backup.dump  

# Restore  
pg_restore -d newdb backup.dump  

# Monitoring  
SELECT * FROM pg_stat_activity;  
```

---

### 🍃 **2. MongoDB (Document NoSQL)**  
#### **Karakteristik Kunci**  
- **Document-Oriented**: Penyimpanan BSON (binary JSON)  
- **Skema Dinamis**: Struktur berbeda per dokumen  
- **Scalability**: Sharding otomatis  
- **Query Fleksibel**: Aggregation framework powerful  

#### **Pola Desain Microservices**  
```mermaid  
flowchart TD  
    ServiceA[Service Log Aktivitas] --> MDB[(MongoDB)]  
    ServiceB[Service Sensor IoT] --> MDB  
    style MDB stroke:#393,stroke-width:2px  
```  
**Prinsip**: Kumpulkan data unstructured dalam satu cluster  

#### **Contoh Implementasi Pemerintah**  
**Kasus**: Sistem Pelacakan Aset Negara  
```javascript  
db.assets.insertOne({  
  _id: ObjectId("507f191e810c19729de860ea"),  
  type: "kendaraan",  
  location: {  
    type: "Point",  
    coordinates: [106.827, -6.175]  
  },  
  history: [  
    { date: ISODate("2023-01-10"), usage: "dinas kesehatan" },  
    { date: ISODate("2023-03-22"), maintenance: true }  
  ]  
});  
```  
**Optimasi**:  
- Sharding berdasarkan `region_id`  
- TTL index untuk data sensor otomatis expire  
- Aggregation pipeline untuk laporan kompleks  

#### **Operasi Penting**  
```bash  
# Backup  
mongodump --uri="mongodb://user:pwd@host/govdb"  

# Restore  
mongorestore dump/  

# Monitoring  
db.currentOp()  
```

---

### ⚖️ **3. Perbandingan Strategis untuk Pemerintah**  
| Parameter          | PostgreSQL                          | MongoDB                             |  
|--------------------|-------------------------------------|-------------------------------------|  
| **Model Data**     | Relasional (table/row)             | Dokumen (collection/BSON)           |  
| **Transaksi**      | ACID penuh                          | Multi-document ACID (v4.0+)         |  
| **Skema**          | Kaku                                | Dinamis                             |  
| **Performa Baca**  | Sangat cepat untuk query kompleks   | Cepat untuk data hirarkis           |  
| **Performa Tulis** | Optimal dengan indeks               | High-throughput insert              |  
| **Skalabilitas**   | Vertikal + read replica             | Horizontal sharding otomatis        |  
| **Pencarian**      | Full-text + trigram                 | Aggregation framework               |  
| **GIS**            | PostGIS (industri standard)         | GeoJSON (basic support)             |  
| **Compliance**     | Audit logging detil                 | Field-level enkripsi                |  

---

### 🛡️ **4. Best Practices Keamanan**  
#### **PostgreSQL**  
```sql  
-- Enkripsi data sensitif  
CREATE EXTENSION pgcrypto;  
INSERT INTO users (ssn) VALUES (pgp_sym_encrypt('123456789', 'secret_key'));  

-- Row-level security  
ALTER TABLE laporan ENABLE ROW LEVEL SECURITY;  
CREATE POLICY officer_policy ON laporan  
  USING (departemen = current_user);  
```

#### **MongoDB**  
```javascript  
// Field-level enkripsi  
const encryptedFields = {  
  fields: [  
    {  
      path: "nik",  
      keyId: UUID("12345678-1234-1234-1234-123456789012"),  
      bsonType: "string"  
    }  
  ]  
};  

// Role-based access  
db.createRole({  
  role: "auditor",  
  privileges: [{ resource: { db: "gov", collection: "reports" }, actions: ["find"] }],  
  roles: []  
});  
```

---

### 🔄 **5. Integrasi dengan Microservices**  
#### **Pola Umum**  
```mermaid  
flowchart LR  
    Service -->|Connection Pool| DB[(Database)]  
    DB -->|Replication| Replica  
    Replica --> BI[Business Intelligence]  
```  

#### **Kode Contoh**  
**Spring Boot + PostgreSQL**:  
```java  
@Repository  
public interface ReportRepo extends JpaRepository<FinancialReport, UUID> {  
    @Query("SELECT region, SUM(budget) FROM reports WHERE year = :year GROUP BY region")  
    List<RegionBudget> findBudgetByYear(@Param("year") int year);  
}  
```  

**Node.js + MongoDB**:  
```javascript  
app.get('/assets/nearby', async (req, res) => {  
  const point = { type: 'Point', coordinates: [parseFloat(req.query.lng), parseFloat(req.query.lat)] };  
  const assets = await db.collection('assets').find({  
    location: {  
      $near: {  
        $geometry: point,  
        $maxDistance: 1000  
      }  
    }  
  }).toArray();  
  res.json(assets);  
});  
```

---

### 🚨 **6. Disaster Recovery & Compliance**  
#### **PostgreSQL**  
- WAL Archiving + Point-in-Time Recovery  
- Replikasi: Streaming + Logical replication  
- Tools: pgBackRest, Barman  

#### **MongoDB**  
- Ops Manager untuk backup otomatis  
- Replica set (min 3 node)  
- Sharded cluster untuk HA regional  

**Panduan BSSN**:  
- Enkripsi data-at-rest menggunakan LUKS (PostgreSQL) atau KMIP (MongoDB)  
- Audit log wajib untuk semua akses data sensitif  
- Isolasi jaringan: Database tidak boleh terpublik internet  

---

### 📊 **7. Studi Kasus: Sistem Terintegrasi**  
**Arsitektur Hybrid**:  
```mermaid  
flowchart LR  
    A[Service Pajak] --> P[(PostgreSQL)]  
    B[Service Log] --> M[(MongoDB)]  
    C[Service Geospasial] --> G[(PostGIS)]  
    D[Service Asset] --> M  
```  

**Alasan Pemilihan**:  
- PostgreSQL: Transaksi keuangan butuh ACID  
- MongoDB: Fleksibilitas log & data sensor  
- PostGIS: Standard industri untuk pemetaan  

**Hasil**:  
- 99.95% availability selama 12 bulan  
- Pengurangan biaya penyimpanan 40% dengan tiered storage  

---

### 📝 **8. Checklist Implementasi**  
1. [ ] **PostgreSQL**:  
   - Setup connection pooling (PgBouncer)  
   - Konfigurasi WAL archiving  
   - Buat read replica untuk reporting  
2. [ ] **MongoDB**:  
   - Atur shard key yang optimal  
   - Aktifkan enkripsi TLS 1.3  
   - Set oplog size minimal 20GB  
3. [ ] **Umum**:  
   - Backup harian + test restore bulanan  
   - Monitoring: Query latency, connection usage  
   - Patch security rutin  

> 💡 **Tip Ahli**:  
> - Untuk sistem baru: Mulai dengan PostgreSQL kecuali butuh skema sangat dinamis  
> - Gunakan **MongoDB** untuk:  
>   - Logging terpusat  
>   - Data sensor IoT  
>   - Konten manajemen (CMS)  
> - Gunakan **PostgreSQL** untuk:  
>   - Sistem keuangan  
>   - Registrasi penduduk  
>   - Layanan dengan regulasi ketat  

**Referensi Resmi**:  
- [PostgreSQL Docs](https://www.postgresql.org/docs/)  
- [MongoDB Docs](https://www.mongodb.com/docs/)  
- [BSSN Panduan Keamanan Database](https://bssn.go.id/publikasi/)
