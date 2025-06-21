### 📚 **Panduan Instalasi Node.js untuk Semua Lingkungan OS**  
Berikut panduan komprehensif instalasi Node.js di berbagai sistem operasi, termasuk optimasi dan verifikasi:

---

#### **1. Windows**  
##### **Metode 1: Installer Resmi (Rekomendasi)**  
1. **Download**:  
   - Kunjungi [Node.js Official Site](https://nodejs.org)  
   - Pilih versi **LTS** (Long Term Support)  
2. **Instalasi**:  
   - Jalankan installer `.msi`  
   - Centang opsi:  
     ✅ **Automatically install tools for native modules**  
     ✅ **Add to PATH**  
3. **Verifikasi**:  
   ```powershell
   node -v
   # v20.12.1
   npm -v
   # 10.5.0
   ```

##### **Metode 2: Package Manager (Chocolatey)**  
1. **Instal Chocolatey** (Admin PowerShell):  
   ```powershell
   Set-ExecutionPolicy Bypass -Scope Process -Force
   iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
   ```
2. **Instal Node.js**:  
   ```powershell
   choco install nodejs-lts
   ```

---

#### **2. macOS**  
##### **Metode 1: Homebrew (Rekomendasi)**  
1. **Instal Homebrew**:  
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
2. **Instal Node.js**:  
   ```bash
   brew install node@20
   echo 'export PATH="/opt/homebrew/opt/node@20/bin:$PATH"' >> ~/.zshrc
   source ~/.zshrc
   ```

##### **Metode 2: PKG Installer**  
1. **Download**:  
   - [Node.js macOS Installer](https://nodejs.org/dist/v20.12.1/node-v20.12.1.pkg)  
2. **Instalasi**:  
   - Ikuti wizard instalasi  
3. **Verifikasi**:  
   ```bash
   node -v && npm -v
   ```

---

#### **3. Linux (Debian/Ubuntu)**  
##### **Metode 1: NodeSource Repository**  
1. **Tambahkan Repo**:  
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
   ```
2. **Instal Node.js**:  
   ```bash
   sudo apt install -y nodejs
   ```
3. **Verifikasi**:  
   ```bash
   node --version
   ```

##### **Metode 2: nvm (Node Version Manager)**  
1. **Instal nvm**:  
   ```bash
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
   source ~/.bashrc
   ```
2. **Instal Node.js**:  
   ```bash
   nvm install 20
   nvm use 20
   ```

---

#### **4. Linux (CentOS/RHEL)**  
##### **Metode Resmi**  
1. **Tambahkan Repo**:  
   ```bash
   curl -fsSL https://rpm.nodesource.com/setup_20.x | sudo bash -
   ```
2. **Instal Node.js**:  
   ```bash
   sudo yum install -y nodejs
   ```

---

#### **5. Linux (Fedora)**  
```bash
sudo dnf install nodejs
```

---

### ⚙️ **Post-Installation Setup**  
#### **Optimasi Global**  
1. **Atur Default Registry**:  
   ```bash
   npm config set registry https://registry.npmjs.org/
   ```
2. **Instal Tools Penting**:  
   ```bash
   npm install -g npm@latest typescript ts-node yarn pnpm
   ```
3. **Permission Fix (Linux/macOS)**:  
   ```bash
   mkdir ~/.npm-global
   npm config set prefix '~/.npm-global'
   echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
   source ~/.bashrc
   ```

---

### 🔍 **Troubleshooting Umum**  
| Masalah | Solusi |  
|---------|--------|  
| `EACCES` permission error | Jalankan: `sudo chown -R $(whoami) ~/.npm` |  
| Command not found (Linux) | Pastikan PATH sudah benar: `echo $PATH` |  
| Version mismatch | Gunakan `nvm` untuk multi-versi |  
| Native module build error | Instal build tools: `sudo apt install build-essential` |

---

### 🧪 **Test Instalasi**  
Buat file `test.js`:  
```javascript
const http = require('http');
http.createServer((req, res) => {
  res.end('Node.js berjalan!');
}).listen(3000, () => {
  console.log('Server aktif di http://localhost:3000');
});
```
Jalankan:  
```bash
node test.js
```

---

### 📊 **Perbandingan Metode Instalasi**  
| Metode | Kelebihan | Kekurangan |  
|--------|-----------|------------|  
| **Official Installer** | Paling sederhana | Tidak mendukung multi-versi |  
| **Package Manager** | Auto-update, integrasi OS | Versi mungkin tidak latest |  
| **nvm** | Multi-versi, fleksibel | Konfigurasi manual |  

> 💡 **Tip Pro**:  
> - Untuk development: Gunakan `nvm` untuk switch versi  
> - Untuk production: Pakai package manager OS  
> - Selalu gunakan versi **LTS** untuk proyek pemerintah!  

Dokumen resmi: [Node.js Docs](https://nodejs.org/en/docs/) | [nvm GitHub](https://github.com/nvm-sh/nvm)
