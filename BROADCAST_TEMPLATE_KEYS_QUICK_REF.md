# Broadcast Template Keys - Quick Reference

**IMPORTANT:** Gunakan HANYA template keys yang terdaftar di `TemplateFieldBootstrap`

---

## ✅ Standard Template Keys

| Key | Description | Example |
|-----|-------------|---------|
| `nama_siswa` | Nama lengkap siswa | "Budi Santoso" |
| `alamat_siswa` | Alamat lengkap siswa | "Jl. Merdeka No. 123" |
| `nama_institut` | Nama institusi | "SMA Negeri 1" |
| `siswa_kelas` | Kelas siswa | "10A" |
| `nominal_tagihan` | Nominal tagihan | "Rp 1.500.000" |
| `tanggal_tempo` | Tanggal jatuh tempo | "2025-01-15" |
| `nama_wali` | Nama wali siswa | "Bapak Setiawan" |
| `bulan_penagihan` | Bulan penagihan | "Januari 2025" |
| `jenis_tagihan` | Jenis/tahun tagihan | "SPP", "2024/2025" |
| `tanggal_penagihan` | Tanggal penagihan dibuat | "2025-01-01" |

---

## 📝 Template Format

```
[nama_siswa]  atau  {{nama_siswa}}
[siswa_kelas] atau  {{siswa_kelas}}
```

---

## ✅ Correct Usage

```json
{
  "broadcastId": 2,
  "students": [
    {
      "studentId": 3,
      "studentName": "Rudi Hartono",
      "whatsappNumber": "6281234567892",
      "messageItem": {
        "nama_siswa": "Rudi Hartono",
        "siswa_kelas": "10B",
        "nama_institut": "SMA Negeri 2",
        "nominal_tagihan": "Rp 2.000.000",
        "tanggal_tempo": "2025-01-15",
        "bulan_penagihan": "Januari 2025"
      }
    }
  ]
}
```

---

## ❌ Incorrect Usage

```json
{
  "messageItem": {
    "name": "Rudi",          // ❌ Use "nama_siswa"
    "class": "10B",          // ❌ Use "siswa_kelas"
    "studentId": "22001",    // ❌ Not standard
    "score": "85"            // ❌ Not standard
  }
}
```

---

## 🧪 Example Template

**Broadcast.finalContent:**
```
Yth. [nama_wali],

Tagihan untuk [nama_siswa] kelas [siswa_kelas]:
- Institusi: [nama_institut]
- Nominal: [nominal_tagihan]
- Jatuh tempo: [tanggal_tempo]
- Bulan: [bulan_penagihan]

Terima kasih.
```

**Rendered Message:**
```
Yth. Bapak Ahmad,

Tagihan untuk Ahmad Rifai kelas 10A:
- Institusi: SMA Negeri 1
- Nominal: Rp 1.500.000
- Jatuh tempo: 2025-01-15
- Bulan: Januari 2025

Terima kasih.
```

---

## 🔗 Full Documentation

See: `documentations/BROADCAST_TEMPLATE_FIELD_STANDARD.md`
