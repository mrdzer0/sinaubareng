# Batch 2 - Vulnerable & Outdated Components Labs

Repository ini berisi kumpulan lab exercise untuk pelatihan **Vulnerable & Outdated Components** (Batch 2).  
Di dalam repo ini, peserta dapat mencoba hands-on berbagai teknik identifikasi dan eksploitasi komponen yang rentan pada aplikasi.

## 📚 Daftar Lab Exercise

- **CVE-2019-11358**  
  Studi kasus kerentanan jQuery pada aplikasi berbasis JavaScript.
- **log4j-poc**  
  Proof-of-Concept (PoC) eksploitasi Log4Shell (CVE-2021-44228) beserta environment Java.
- **snyk-lab**  
  Latihan penggunaan Snyk untuk deteksi dan manajemen dependency vulnerability.

---

## 🚦 Tutorial Scanning Menggunakan Dependency-Check

### 1. Jalankan Dependency-Check (Linux/Mac)
```bash
cd dependency-check
./bin/dependency-check.sh --scan <path-folder-proyek-anda> --out .
```

### 2. Tips
- Ganti `<path-folder-proyek-anda>` dengan folder yang ingin Anda scan (misal: `../log4j-poc`).
- Hasil report biasanya berbentuk file .html di folder yang sama, dengan nama `dependency-check-report.html`.
```bash
./dependency-check.sh --scan <path-folder-proyek-anda> --disableCentral --out .
```

## 🚦 Tutorial Scanning Menggunakan Snyk

### 1. Install Snyk CLI
```bash
npm install -g snyk
```

### 2. Login ke Snyk
```bash
snyk auth
```

### 3. Jalankan Scan
Pindah ke folder project yang ingin di scan terlebih dahulu:
```bash
cd <path-folder-project>
snyk test
```

### 4. Melihat Hasil
- Hasil scan akan langsung muncul di terminal.
- Jika ada kerentanan, Snyk akan menampilkan detil dan rekomendasi perbaikannya.

## 🚦 Tutorial Supplychain attack
### 1. Create a fake (malicious) module
```sh
mkdir fake-package
cd fake-package
npm init -y
```
### 2. Edit `package.json` dengan menambahkan `postinstall` pada script
```sh
"scripts": {
  "postinstall": "echo 'You have been pwned!' > pwned.txt"
}
```
### 3. Repack modulenya / bisa refer ke folder **fake-package**
```sh
npm pack
```
### 4. Coba install modulnya
```sh
npm install ../fake-package/fake-package-1.0.0.tgz
```

## 📌 Note
- `report_cve2019.html`, `report_log4j.html`, `report_snyklab.html` merupakan contoh hasil report dari tools OWASP Dependency-Check

---
Selamat mencoba & belajar!