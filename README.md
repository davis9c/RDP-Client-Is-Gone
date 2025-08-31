Mantap! Saya akan bikin:

1. **Script PowerShell** otomatis:

   * Deteksi otomatis `install.wim` atau `install.esd`.
   * Prompt input untuk `$isoDrive`.
   * Copy otomatis `mstsc.exe`.
2. **README.md**:

   * Cerita masalah awal, analisis, dan solusi.
   * Dibuat menarik untuk GitHub.

---

### 🔹 PowerShell Script: `Restore-MSTSC.ps1`

```powershell
<#
.SYNOPSIS
    Script untuk mengembalikan Remote Desktop Client (mstsc.exe) di Windows 11
.DESCRIPTION
    Script ini akan:
    - Meminta input lokasi drive ISO Windows 11
    - Mencari file install.wim atau install.esd
    - Mount image Windows
    - Menyalin mstsc.exe dan file bahasa
    - Melepas mount image
.NOTES
    Dibuat oleh Davis Aji
#>

# ===== INPUT DRIVE ISO =====
$isoDrive = Read-Host "Masukkan drive letter tempat ISO Windows 11 (contoh: E:)"

# ===== CEK FILE WIM/ESD =====
$wimFile = Join-Path $isoDrive "sources\install.wim"
$esdFile = Join-Path $isoDrive "sources\install.esd"

if (Test-Path $wimFile) {
    $imageFile = $wimFile
    Write-Host "Ditemukan install.wim" -ForegroundColor Green
} elseif (Test-Path $esdFile) {
    $imageFile = $esdFile
    Write-Host "Ditemukan install.esd" -ForegroundColor Green
} else {
    Write-Host "File install.wim atau install.esd tidak ditemukan di $isoDrive\sources" -ForegroundColor Red
    exit
}

# ===== CARI INDEX PERTAMA =====
Write-Host "Mencari index pertama di $imageFile ..." -ForegroundColor Cyan
$indexInfo = dism /Get-WimInfo /WimFile:$imageFile | Out-String
$firstIndexLine = ($indexInfo -split "`n" | Select-String -Pattern "Index :")[0]

if ($firstIndexLine) {
    $proIndex = ($firstIndexLine -split ":")[1].Trim()
    Write-Host "Index pertama ditemukan: $proIndex" -ForegroundColor Green
} else {
    Write-Host "Tidak ada index ditemukan! Script dihentikan." -ForegroundColor Red
    exit
}

# ===== MOUNT IMAGE =====
$mountDir = "C:\Mount"
Write-Host "Membuat folder mount di $mountDir ..." -ForegroundColor Cyan
New-Item -ItemType Directory -Force -Path $mountDir | Out-Null

Write-Host "Mounting image Windows..." -ForegroundColor Cyan
dism /Mount-Wim /WimFile:$imageFile /Index:$proIndex /MountDir:$mountDir

# ===== SALIN MSTSC =====
Write-Host "Menyalin Remote Desktop Client..." -ForegroundColor Cyan
Copy-Item "$mountDir\Windows\System32\mstsc.exe" "C:\Windows\System32\" -Force
Copy-Item "$mountDir\Windows\System32\*.mui" "C:\Windows\System32\id-ID\" -Force -ErrorAction SilentlyContinue

# ===== UNMOUNT IMAGE =====
Write-Host "Melepas mount image..." -ForegroundColor Cyan
dism /Unmount-Wim /MountDir:$mountDir /Discard

Write-Host "✅ Selesai! Coba jalankan mstsc dari Run (Win+R)." -ForegroundColor Green
```

---

### 🔹 README.md (Menarik untuk GitHub)

````markdown
# 🔧 Windows 11 Remote Desktop Client (mstsc.exe) Recovery

Pernah tidak sengaja **menghapus fitur Remote Desktop Client (MSTSC)** di Windows 11?  
Tiba-tiba `mstsc.exe` hilang, dan Remote Desktop tidak bisa dipakai?  
Repo ini menyediakan **solusi otomatis** untuk mengembalikan MSTSC hanya dengan **ISO Windows 11**.

---

## 🧩 Gejala Awal
- `mstsc.exe` hilang dari `C:\Windows\System32\`
- Tidak bisa membuka Remote Desktop
- Windows Features tidak menampilkan opsi untuk mengaktifkan kembali Remote Desktop Client
- `DISM /Online /Add-Capability` gagal dengan **Error 87**

---

## 🔍 Alur Analisa
1. Mengecek integritas sistem dengan `sfc /scannow` → tidak mengembalikan MSTSC.
2. Mencoba `DISM /Online /Add-Capability /CapabilityName:Microsoft.Windows.RemoteDesktop.Client` → gagal.
3. Ternyata Remote Desktop Client adalah bagian dari **image Windows**, bukan package yang bisa diinstal ulang via DISM Online.
4. Solusi: **Mount ISO Windows 11** dan ambil file `mstsc.exe` langsung dari `install.wim` atau `install.esd`.

---

## 🚀 Solusi Otomatis
Script `Restore-MSTSC.ps1` akan:
1. Meminta Anda memasukkan drive letter ISO Windows 11.
2. Mendeteksi `install.wim` atau `install.esd`.
3. Mount image Windows 11.
4. Menyalin `mstsc.exe` dan file bahasa ke `C:\Windows\System32\`.
5. Melepas image.

---

## 💻 Cara Pakai
1. **Download ISO Windows 11** dari [Microsoft](https://www.microsoft.com/software-download/windows11).  
2. Mount ISO ke Windows (klik kanan → Mount).  
3. Clone repo ini:
   ```powershell
   git clone https://github.com/username/restore-mstsc.git
   cd restore-mstsc
````

4. Jalankan script sebagai Administrator:

   ```powershell
   powershell -ExecutionPolicy Bypass -File .\Restore-MSTSC.ps1
   ```
5. Masukkan drive letter ISO (contoh: `E:`).
6. Tunggu sampai selesai, lalu jalankan:

   ```
   mstsc
   ```

---

## 📸 Screenshot

![Demo Script](screenshot.png)

---

## 🧑‍💻 Kontribusi

Pull request terbuka untuk:

* Mendukung Windows 10
* GUI berbasis PowerShell

---

## ⚠️ Disclaimer

Script ini memodifikasi file sistem Windows.
Gunakan dengan hati-hati dan pastikan ISO Windows resmi dari Microsoft.

---

✨ **Enjoy Remote Desktop kembali tanpa reinstall Windows!** ✨

```

---

📦 **Output**:  
- `Restore-MSTSC.ps1` → Script otomatis full.  
- `README.md` → Siap untuk GitHub, ada cerita & dokumentasi menarik.  

---

Mau sekalian saya bikin **screenshot step-by-step** supaya repo kamu lebih profesional?
```
