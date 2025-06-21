### 📚 **Panduan Instalasi Go (Golang) untuk Semua OS**  
Berikut panduan komprehensif instalasi Go di berbagai sistem operasi, termasuk optimasi dan verifikasi:

---

#### **1. Windows**  
##### **Metode 1: Installer Resmi (Rekomendasi)**  
1. **Download**:  
   - Kunjungi [golang.org/dl](https://golang.org/dl)  
   - Pilih installer `.msi` untuk Windows  
2. **Instalasi**:  
   - Jalankan installer (default akan install di `C:\Go`)  
   - Centang opsi: **Add to PATH**  
3. **Verifikasi**:  
   ```powershell
   go version
   # go version go1.22.1 windows/amd64
   ```

##### **Metode 2: Scoop (Package Manager)**  
1. **Instal Scoop**:  
   ```powershell
   Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
   irm get.scoop.sh | iex
   ```
2. **Instal Go**:  
   ```powershell
   scoop install go
   ```

---

#### **2. macOS**  
##### **Metode 1: Homebrew (Rekomendasi)**  
1. **Instal Homebrew**:  
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
2. **Instal Go**:  
   ```bash
   brew install go
   ```

##### **Metode 2: PKG Installer**  
1. **Download**:  
   - [Go macOS Installer](https://go.dev/dl/go1.22.1.darwin-amd64.pkg)  
2. **Instalasi**:  
   - Ikuti wizard instalasi  
3. **Verifikasi**:  
   ```bash
   go version
   ```

---

#### **3. Linux (Debian/Ubuntu)**  
##### **Metode 1: Official Binary**  
1. **Download**:  
   ```bash
   wget https://go.dev/dl/go1.22.1.linux-amd64.tar.gz
   ```
2. **Ekstrak**:  
   ```bash
   sudo tar -C /usr/local -xzf go1.22.1.linux-amd64.tar.gz
   ```
3. **Atur PATH**:  
   ```bash
   echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
   source ~/.bashrc
   ```

##### **Metode 2: Snap**  
```bash
sudo snap install go --classic
```

---

#### **4. Linux (Fedora/CentOS/RHEL)**  
```bash
sudo dnf install golang  # Fedora
sudo yum install golang  # CentOS/RHEL
```

---

### ⚙️ **Post-Installation Setup**  
#### **Konfigurasi Environment**  
1. **Buat Workspace**:  
   ```bash
   mkdir ~/go-projects
   cd ~/go-projects
   ```
2. **Atur GOPATH**:  
   ```bash
   echo 'export GOPATH=$HOME/go-projects' >> ~/.bashrc
   echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.bashrc
   source ~/.bashrc
   ```
   Untuk Windows:  
   ```powershell
   [System.Environment]::SetEnvironmentVariable('GOPATH', "$env:USERPROFILE\go-projects", 'User')
   ```

#### **Optimasi**  
1. **Private Repository**:  
   ```bash
   go env -w GOPRIVATE=github.com/gov-project
   ```
2. **Proxy Module**:  
   ```bash
   go env -w GOPROXY=https://proxy.golang.org,direct
   ```

---

### 🔍 **Troubleshooting Umum**  
| Masalah | Solusi |  
|---------|--------|  
| `go: command not found` | Periksa PATH dan restart terminal |  
| Permission denied | Gunakan `sudo` atau fix permission folder instalasi |  
| Module proxy error | Set `GOPROXY=https://goproxy.io,direct` |  
| CGO errors | Instal gcc: `sudo apt install build-essential` |

---

### 🧪 **Test Instalasi**  
Buat file `hello.go`:  
```go
package main

import "fmt"

func main() {
    fmt.Println("Go berjalan sukses!")
}
```
Jalankan:  
```bash
go run hello.go
# Output: Go berjalan sukses!
```

---

### 🚀 **Workflow Development**  
1. **Inisialisasi Modul**:  
   ```bash
   go mod init github.com/username/project
   ```
2. **Tambah Dependency**:  
   ```bash
   go get github.com/gin-gonic/gin
   ```
3. **Build Binary**:  
   ```bash
   go build -o myapp
   ```
4. **Cross-Compile**:  
   ```bash
   GOOS=windows GOARCH=amd64 go build -o app.exe
   ```

---

### 📊 **Perbandingan Metode Instalasi**  
| Metode | Kelebihan | Kekurangan |  
|--------|-----------|------------|  
| **Official Installer** | Paling stabil | Tidak otomatis update |  
| **Package Manager** | Auto-update, mudah | Versi mungkin tidak terbaru |  
| **Source Build** | Custom optimasi | Kompleks untuk pemula |  

> 💡 **Best Practices**:  
> - Gunakan **Go Modules** untuk manajemen dependency  
> - Set `GO111MODULE=on` untuk proyek baru  
> - Gunakan `.go-version` file untuk konsistensi versi  
> - Untuk proyek pemerintah, gunakan versi stabil (bukan beta)  

Dokumen resmi:  
- [Go Documentation](https://go.dev/doc/)  
- [Effective Go](https://go.dev/doc/effective_go)  
- [Go by Example](https://gobyexample.com/)  

Dengan Go terinstal, Anda siap membangun aplikasi performa tinggi untuk layanan pemerintah! 🚀🇮🇩
