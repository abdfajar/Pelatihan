## **Arsitektur Microservices: Database per Service & Event Sourcing**  
**Fokus**: Bounded Context, Database Independen, Event Sourcing, dan CQRS  

---

### **1. Database per Service & Bounded Context**  
#### **a. Prinsip Dasar**  
```mermaid  
flowchart TD  
    A[Service Pajak] --> B[(Database Pajak)]  
    C[Service Izin Usaha] --> D[(Database Izin)]  
    E[Service Kependudukan] --> F[(Database Penduduk)]  
```  
- **Aturan Emas**:  
  Setiap microservice memiliki database privatnya sendiri → **tidak ada shared database**!  
- **Bounded Context (DDD)**:  
  Batasan domain di mana model bisnis berlaku konsisten (e.g., "penduduk" di konteks Dukcapil vs Kemendag).  

#### **b. Contoh Penerapan Pemerintah**  
| Service | Database | Bounded Context |  
|---------|----------|----------------|  
| `service-ktp` | PostgreSQL | Identitas Penduduk |  
| `service-pajak` | Oracle | Perpajakan Nasional |  
| `service-bansos` | MongoDB | Bantuan Sosial |  

**Manfaat**:  
- **Isolasi Kegagalan**: Kerusakan database bansos tidak pengaruhi layanan KTP.  
- **Teknologi Tepat Guna**: Gunakan SQL untuk transaksi pajak (ACID), NoSQL untuk data bansos fleksibel.  

---

### **2. Event Sourcing**  
#### **a. Konsep Inti**  
- **State sebagai Deretan Event**:  
  ```mermaid  
  flowchart LR  
      E1[Event 1: PendaftaranKTP] --> E2[Event 2: VerifikasiKTP] --> E3[Event 3: PencetakanKTP]  
  ```  
- **Event Store**: Database khusus menyimpan semua event secara immutable.  

#### **b. Implementasi e-KTP**  
```mermaid  
sequenceDiagram  
    participant A as Aplikasi Mobile  
    participant S as Service KTP  
    participant ES as Event Store  
    
    A->>S: Request buat KTP  
    S->>ES: Simpan event "KTPDibuat"  
    ES-->>S: OK  
    S->>A: 200 "Permintaan diterima"  
    
    loop Proses Background  
        S->>ES: Simpan event "KTPDiverifikasi"  
        S->>ES: Simpan event "KTPDicetak"  
    end  
```  
**Payload Event**:  
```json  
// Event: KTPDiverifikasi  
{  
  "event_id": "evt_kpt_789",  
  "nik": "3273011234567890",  
  "status": "VERIFIED",  
  "timestamp": "2023-11-05T10:15:30Z",  
  "officer_id": "petugas_123"  
}  
```  

#### **c. Keuntungan untuk Sektor Publik**  
- **Audit Trail Otomatis**: Semua perubahan terekam lengkap → transparansi administrasi.  
- **Pemulihan Bencana**: Rekonstruksi state dari event history jika database rusak.  

---

### **3. CQRS (Command Query Responsibility Segregation)**  
#### **a. Pola Desain**  
```mermaid  
flowchart LR  
    C[Command] --> |Write| W[(Write DB)]  
    Q[Query] --> |Read| R[(Read DB)]  
    W --> |Event| E[Event Bus]  
    E --> |Update| R  
```  
- **Pemisahan Jalan**:  
  - **Command Side**: Handle operasi tulis (high consistency).  
  - **Query Side**: Handle operasi baca (high availability).  

#### **b. Studi Kasus: Sistem Perpajakan**  
**Arsitektur**:  
```mermaid  
flowchart TB  
    subgraph Command  
        C1[Service Pajak] -->|"insertPajak()"| DB1[(Write DB)]  
        DB1 -->|Publish| EB[(Event Bus)]  
    end  
    
    subgraph Query  
        EB -->|Update| DB2[(Read DB)]  
        DB2 --> Q1[Dashboard Real-time]  
        DB2 --> Q2[Laporan Bulanan]  
    end  
```  

**Alur Kerja**:  
1. **Command**: Petugas input data pajak → tulis ke database transaksional (ACID).  
2. **Event**: Data baru dipublikasikan ke event bus (Kafka).  
3. **Query**:  
   - Service terpisah konsumsi event → update database baca (optimized for query).  
   - Dashboard real-time baca dari database query (tidak ganggu operasi tulis).  

**Manfaat**:  
- Performa: Beban tulis & baca terdistribusi.  
- Skalabilitas: Database query bisa di-replikasi untuk kebutuhan analitik.  

---

### **Integrasi Event Sourcing + CQRS**  
#### **Arsitektur Lengkap e-Gov**  
```mermaid  
flowchart LR  
    A[Frontend] -->|Command| B(Command Service)  
    B -->|Save Event| C[(Event Store)]  
    C -->|Publish| D[Event Bus]  
    D --> E[Query Service]  
    E -->|Update| F[(Read DB)]  
    A -->|Query| G[[Query Service]]  
    G --> F  
    D --> H[Audit Service]  
    D --> I[Notif Service]  
```  

#### **Use Case: Permohonan Izin Usaha**  
1. **Command**:  
   - `POST /izin` → simpan event `PermohonanIzinDibuat`.  
2. **Event Sourcing**:  
   ```  
   [Event 1] PermohonanIzinDibuat  
   [Event 2] IzinDisetujui  
   [Event 3] IzinDikirim  
   ```
   Berikut diagram untuk alur event **Permohonan Izin Usaha** menggunakan Mermaid:

##### Diagram 1: Alur Event Sederhana (Sequence Flow)
```mermaid
flowchart LR
    A[Event 1\n\nPermohonanIzinDibuat] --> B[Event 2\n\nIzinDisetujui] --> C[Event 3\n\nIzinDikirim]
```

##### Diagram 2: Arsitektur Lengkap dengan Event Sourcing & CQRS
```mermaid
flowchart TB
    subgraph Command Side
        direction TB
        A[Frontend] -->|1. Buat Permohonan| B[Command Service]
        B -->|2. Simpan Event| C[(Event Store)]
        C -->|3. Publish| D[Event Bus]
    end

    subgraph Event Processing
        direction LR
        D --> E[Projection Service]
        E -->|Update| F[(Read Database)]
    end

    subgraph Query Side
        G[Frontend] -->|5. Query Status| H[Query Service]
        H --> F
    end

    D -->|4. Trigger| I[Notification Service]
    I -->|SMS/Email| J[Pemohon]

    style Command Side fill:#f9f,stroke:#333
    style Query Side fill:#9f9,stroke:#333
```

##### Penjelasan Diagram:
1. ***Alur Command*** (Warna Ungu):
   - User membuat permohonan izin → Command Service menyimpan event `PermohonanIzinDibuat` di Event Store
   - Event dipublikasikan ke Event Bus (Kafka/RabbitMQ)

2. ***Event Processing***:
   - Projection Service mengonsumsi event dan meng-update Read Database (optimized for query)
   - Notification Service mengirim notifikasi ke pemohon

3. ***Alur Query*** (Warna Hijau):
   - User mengecek status izin → Query Service membaca dari Read Database

##### Diagram 3: State Rekonstruksi (Event Sourcing)
```mermaid
flowchart LR
    S0[State Awal] -->|Event 1\nPermohonanIzinDibuat| S1[State: DRAFT]
    S1 -->|Event 2\nIzinDisetujui| S2[State: APPROVED]
    S2 -->|Event 3\nIzinDikirim| S3[State: DELIVERED]
```

##### Contoh Payload Event:
```json
// Event 1: PermohonanIzinDibuat
{
  "event_id": "evt_iz001",
  "application_id": "IZ2023-789",
  "applicant": "PT. Maju Jaya",
  "created_at": "2023-11-05T09:15:00Z",
  "service_type": "Izin Usaha"
}

// Event 2: IzinDisetujui
{
  "event_id": "evt_iz002",
  "application_id": "IZ2023-789",
  "approver_id": "officer_456",
  "approved_at": "2023-11-07T14:20:00Z",
  "notes": "Lengkapi dokumen lingkungan"
}

// Event 3: IzinDikirim
{
  "event_id": "evt_iz003",
  "application_id": "IZ2023-789",
  "delivery_method": "EMAIL",
  "sent_at": "2023-11-08T10:05:00Z",
  "recipient": "contact@majujaya.id"
}
```

##### Diagram 4: Interaksi Sistem (Sequence Diagram)
```mermaid
sequenceDiagram
    participant User
    participant CommandService
    participant EventStore
    participant EventBus
    participant ProjectionService
    participant ReadDB
    participant QueryService
    
    User->>CommandService: Ajukan Izin Usaha
    CommandService->>EventStore: Save Event: PermohonanIzinDibuat
    EventStore->>EventBus: Publish Event
    CommandService-->>User: Permohonan diterima
    
    EventBus->>ProjectionService: Consume Event
    ProjectionService->>ReadDB: Update view_permohonan_izin
    
    User->>QueryService: Cek Status Izin
    QueryService->>ReadDB: SELECT status FROM view_permohonan_izin
    ReadDB-->>QueryService: Status: DRAFT
    QueryService-->>User: Tampilkan status
```

##### Keuntungan Penerapan di Sektor Pemerintah:
1. ***Transparansi Proses***:
   - Setiap tahap (pengajuan > persetujuan > pengiriman) terekam immutable
2. ***Pemulihan Data***:
   - Jika database rusak, state bisa direkonstruksi dari event history
3. ***Integrasi Sistem***:
   - Dinas terkait (perdagangan, lingkungan) bisa subscribe event tertentu
   ```mermaid
   flowchart LR
       EventBus -->|Event IzinDisetujui| DinasPerdagangan
       EventBus -->|Event IzinDisetujui| DinasLingkungan
   ```
3. **CQRS**:  
   - **Query Side**: Bangun view `izin_status` di database baca untuk tracking real-time.  

---

### **Best Practices untuk Pemerintah**  
1. **Pilih Teknologi Database Berdasarkan Kebutuhan**:  
   | Service | Jenis Database | Alasan |  
   |---------|----------------|--------|  
   | Transaksi Keuangan | SQL (PostgreSQL) | Butuh ACID compliance |  
   | Log Aktivitas | NoSQL (MongoDB) | Skema fleksibel untuk audit log |  
   | Cache Data | Redis | Akses cepat untuk data statis |  

2. **Implementasi Event-Driven untuk Integrasi Kementerian**:  
   - Contoh:  
     ```mermaid  
     flowchart LR  
         Kemdikbud --"Event: NISNDiverifikasi"--> Bus  
         Kemendagri --"Event: KKDiupdate"--> Bus  
         Dukcapil -->|Subscribe| Bus[(Event Bus Nasional)]  
     ```  

3. **Hindari Anti-Pattern**:  
   - ❌ Shared database antar kementerian → gunakan API/event.  
   - ❌ Query langsung ke database command → gunakan CQRS.  

> 💡 **Tip Transformasi Digital**:  
> Mulai penerapan Event Sourcing dari sistem yang:  
> - Memiliki regulasi ketat (e.g., perpajakan, perizinan)  
> - Butuh audit trail lengkap (e.g., pengadaan barang)  
> - Sering berkolaborasi lintas instansi (e.g., data kependudukan)
