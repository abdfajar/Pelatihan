## **Arsitektur Microservices: Architectural Patterns**  
**Fokus**: BFF, API Gateway, Decomposition by Business Capability/Subdomain  

---

### **1. BFF (Backend For Frontend)**  
**Konsep**:  
- Pola di mana setiap *frontend* (mobile, web, desktop) memiliki *backend khusus* yang melayani kebutuhannya.  
- Contoh:  
  - BFF Mobile: Menyediakan data terkompresi & endpoint untuk fitur offline.  
  - BFF Web: Optimasi untuk rendering cepat dan WebSockets.  

**Mengapa Penting?**  
- **Solusi Over-fetching/Under-fetching**:  
  - Frontend mobile hanya butuh 5 field, sedangkan web butuh 20 field? BFF menyediakan respon khusus.  
- **Isolasi Perubahan**:  
  - Update UI mobile tidak memengaruhi logika web.  

**Studi Kasus Pemerintah**:  
- Aplikasi *Layanan Kesehatan*:  
  - BFF Mobile: Notifikasi vaksin + lokasi faskes (payload minimal).  
  - BFF Web: Dashboard statistik lengkap + grafik.  

---

### **2. API Gateway Pattern**  
**Konsep**:  
- *Single entry point* untuk semua klien yang mengakses kumpulan microservices.  
- Bertugas sebagai:  
  - **Router**: Mengarahkan `/auth` ke service autentikasi, `/payment` ke service pembayaran.  
  - **Aggregator**: Menggabungkan respon dari beberapa service (e.g., data profil user + riwayat transaksi).  
  - **Security Layer**: Otentikasi, rate limiting, dan logging terpusat.  

**Keuntungan untuk Instansi Pemerintah**:  
- **Sederhanakan Integrasi Sistem Warisan**:  
  - Gateway bisa "berbicara" dengan SOAP service lama sambil ekspos RESTful API ke klien modern.  
- **Pusat Kendali Keamanan**:  
  - Implementasi kebijakan OWASP terpusat (e.g., blokir request dari IP mencurigakan).  

**Contoh Tools**:  
- Kong, Apigee, Spring Cloud Gateway.  

---

### **3. Decomposition by Business Capability & Subdomain**  
**Konsep**:  
- Memecah aplikasi berdasarkan **kemampuan bisnis** (*business capability*) atau **batasan domain** (*subdomain*).  

#### **a. Business Capability**  
- Definisi: Fungsi bisnis inti yang mendukung operasi organisasi.  
- Contoh pada Kementerian Keuangan:  
  | Business Capability | Microservice |  
  |---------------------|--------------|  
  | Manajemen Pajak     | `service-pajak-pbb`, `service-pajak-ppn` |  
  | Pembayaran Negara   | `service-billing`, `service-rekonsiliasi` |  

#### **b. Subdomain (Domain-Driven Design)**  
- Definisi: Memisahkan sistem menjadi *boundary context* (bagian domain spesifik).  
- Contoh: Sistem *e-Government* → Subdomain:  
  - `Subdomain Penduduk`: KTP, kelahiran, kematian.  
  - `Subdomain Perizinan`: IMB, izin usaha, izin lingkungan.  

**Langkah Implementasi**:  
1. **Identifikasi Bounded Context**:  
   - Workshop dengan *domain expert* (e.g., tim perpajakan, tim kependudukan).  
2. **Desain Service Granular**:  
   - Hindari god service! Contoh salah: `service-penduduk` yang menangani KTP + izin + pajak.  
   - Contoh benar: `service-ktp`, `service-kelahiran`, `service-pernikahan`.  

**Manfaat untuk Pemerintah**:  
- **Alignment dengan Struktur Organisasi**:  
  - Tim KTP di Dukcapil hanya fokus pada `service-ktp`.  
- **Reuse di Instansi Lain**:  
  - `service-ktp` bisa dipakai oleh aplikasi Dinas Kependudukan *dan* Dinas Kesehatan.  

---

### **Perbandingan Pola Decomposition**  
| Kriteria          | Business Capability                          | Subdomain (DDD)                     |  
|-------------------|----------------------------------------------|-------------------------------------|  
| **Fokus**         | Fungsi bisnis (Apa yang dilakukan instansi?) | Batasan domain (Apa objek utamanya?) |  
| **Contoh**        | Layanan pajak, layanan audit                 | Penduduk, Perizinan, Logistik       |  
| **Kesesuaian**    | Instansi dengan proses baku (e.g., BPJS)     | Instansi multidomain (e.g., Kemendag) |  

---

### **Studi Kasus Integrasi: Transformasi Sistem Monolitik Kementerian X**  
**Latar Belakang**:  
- Sistem monolitik menangani 15+ layanan (pajak, izin, pengaduan) → sulit diperbarui.  

**Strategi Dekomposisi**:  
1. **Business Capability Mapping**:  
   ```mermaid  
   graph LR  
   A[Sistem Monolitik] --> B[Service Pajak]  
   A --> C[Service Izin Usaha]  
   A --> D[Service Pengaduan]  
   ```  
2. **API Gateway sebagai Wrapper**:  
   - Gateway menerjemahkan request REST → format internal SOAP monolitik (selama masa transisi).  
3. **Hasil**:  
   - Tim pengembang pajak bisa deploy update tanpa koordinasi dengan tim izin.  

---

### **Best Practices Arsitektur**  
1. **Jangan Terlalu Granular!**  
   - Targetkan 1 microservice = 1 tim (5-9 orang).  
   - Contoh buruk: Pisahkan `service-ktp-verifikasi` dan `service-ktp-cetak` → terlalu kecil!  
2. **API Gateway + BFF = Kombinasi Powerful**  
   - Gunakan Gateway untuk otentikasi/authorization, lalu BFF untuk customize respon per klien.  
3. **Pastikan Setiap Service Miliki Database Sendiri**:  
   - Hindari shared database! Contoh: `service-pajak` pakai PostgreSQL, `service-log` pakai MongoDB.  

> 💡 **Tip untuk Pemerintah**: Mulai dekomposisi dari layanan yang paling sering berubah (e.g., layanan perizinan) ketimbang yang stabil (e.g., layanan arsip).
