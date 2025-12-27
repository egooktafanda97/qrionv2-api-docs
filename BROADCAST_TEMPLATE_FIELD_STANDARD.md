# Broadcast Template Field Standardization

**Author:** System Documentation  
**Date:** December 21, 2025  
**Status:** ✅ Implemented  

---

## 📋 Overview

Dokumen ini menjelaskan **standardisasi template field keys** yang HARUS digunakan dalam broadcast message personalization. Semua template key mengacu pada `TemplateField` entity yang didefinisikan di database dan di-seed melalui `TemplateFieldBootstrap`.

**CRITICAL:** Hanya template keys yang terdaftar di `TemplateFieldBootstrap` yang dapat digunakan untuk message replacement. Custom keys tidak akan di-render.

---

## 🔑 Standard Template Field Keys

### Keys yang Tersedia (dari `TemplateFieldBootstrap`)

| **Key Name**          | **Description**                          | **Example Value**           |
|-----------------------|------------------------------------------|-----------------------------|
| `nama_siswa`          | Nama lengkap siswa                       | "Budi Santoso"              |
| `alamat_siswa`        | Alamat lengkap siswa                     | "Jl. Merdeka No. 123"       |
| `nama_institut`       | Nama institusi pendidikan                | "SMA Negeri 1 Jakarta"      |
| `siswa_kelas`         | Kelas siswa                              | "10A", "11 IPA 2"           |
| `nominal_tagihan`     | Nominal tagihan siswa                    | "Rp 1.500.000"              |
| `tanggal_tempo`       | Tanggal jatuh tempo pembayaran           | "2025-01-15"                |
| `nama_wali`           | Nama lengkap wali/orang tua siswa        | "Bapak Setiawan"            |
| `bulan_penagihan`     | Bulan penagihan                          | "Januari 2025"              |
| `jenis_tagihan`       | Jenis tagihan atau tahun penagihan       | "SPP", "2024/2025"          |
| `tanggal_penagihan`   | Tanggal penagihan dibuat                 | "2025-01-01"                |

### ⚠️ Keys yang TIDAK BOLEH Digunakan

Berikut adalah contoh keys yang **TIDAK STANDARD** dan tidak akan di-render:

- ❌ `name` → gunakan `nama_siswa`
- ❌ `class` → gunakan `siswa_kelas`
- ❌ `studentId` → tidak ada di standard keys
- ❌ `score` → tidak ada di standard keys
- ❌ `totalTuition` → gunakan `nominal_tagihan`
- ❌ `dueDate` → gunakan `tanggal_tempo`
- ❌ `parentName` → gunakan `nama_wali`
- ❌ `schoolYear` → gunakan `jenis_tagihan`

---

## 📝 Template Syntax

Template keys dapat menggunakan dua format:

### 1. Square Bracket Format (Recommended)
```
[nama_siswa]
[siswa_kelas]
[nama_institut]
```

### 2. Double Curly Brace Format
```
{{nama_siswa}}
{{siswa_kelas}}
{{nama_institut}}
```

**Note:** Kedua format didukung oleh `TemplateRenderer`, tetapi **square bracket format** lebih direkomendasikan untuk konsistensi dengan contoh di dokumentasi.

---

## 🔄 Template Rendering Flow

### Architecture Overview

```
JSON Request (messageItem)
        ↓
BulkStudentIdRequest.StudentInfo.messageItem (Map<String, Object>)
        ↓
BroadcastMessageProcessor.buildTemplateReplaceItem()
        ↓
TemplateReplaceItem (Java object dengan standard fields)
        ↓
TemplateRenderer.buildMessage(template, item)
        ↓
Rendered Message (stored in broadcast_recipient.message)
```

### Processing Steps

1. **Client sends JSON** dengan `messageItem`:
   ```json
   {
     "studentId": 3,
     "studentName": "Rudi Hartono",
     "whatsappNumber": "6281234567892",
     "messageItem": {
       "nama_siswa": "Rudi Hartono",
       "siswa_kelas": "10B",
       "nama_institut": "SMA Negeri 2",
       "nominal_tagihan": "Rp 2.000.000",
       "tanggal_tempo": "2025-01-15"
     }
   }
   ```

2. **BroadcastMessageProcessor** converts `Map<String, Object>` → `TemplateReplaceItem`:
   ```java
   TemplateReplaceItem item = TemplateReplaceItem.builder()
       .namaSiswa("Rudi Hartono")
       .siswaKelas("10B")
       .namaInstitut("SMA Negeri 2")
       .nominalTagihan("Rp 2.000.000")
       .tanggalTempo("2025-01-15")
       .build();
   ```

3. **TemplateRenderer** replaces placeholders:
   - Template: `"Halo [nama_siswa], tagihan kelas [siswa_kelas] sebesar [nominal_tagihan]"`
   - Rendered: `"Halo Rudi Hartono, tagihan kelas 10B sebesar Rp 2.000.000"`

4. **BroadcastRecipient** entity stores rendered message:
   ```java
   BroadcastRecipient.builder()
       .studentName("Rudi Hartono")
       .message("Halo Rudi Hartono, tagihan kelas 10B sebesar Rp 2.000.000")
       .build();
   ```

---

## 🎯 Code Implementation

### Entity: TemplateField.java
```java
@Entity
@Table(name = "template_field")
public class TemplateField {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 100, nullable = false, unique = true)
    private String keyName; // e.g. nama_siswa, siswa_kelas

    @Column(length = 255)
    private String description;

    private LocalDateTime createdAt = LocalDateTime.now();
}
```

### Bootstrap: TemplateFieldBootstrap.java
```java
@Component
public class TemplateFieldBootstrap implements CommandLineRunner {

    @Override
    public void run(String... args) {
        if (repo.count() == 0) {
            List<TemplateField> defaults = Arrays.asList(
                TemplateField.builder().keyName("nama_siswa").description("Nama lengkap siswa").build(),
                TemplateField.builder().keyName("alamat_siswa").description("Alamat lengkap siswa").build(),
                TemplateField.builder().keyName("nama_institut").description("Nama institut pendidikan").build(),
                TemplateField.builder().keyName("siswa_kelas").description("Kelas siswa").build(),
                TemplateField.builder().keyName("nominal_tagihan").description("Nominal tagihan siswa").build(),
                TemplateField.builder().keyName("tanggal_tempo").description("Tanggal jatuh tempo pembayaran").build(),
                TemplateField.builder().keyName("nama_wali").description("Nama lengkap wali siswa").build(),
                TemplateField.builder().keyName("bulan_penagihan").description("Bulan penagihan").build(),
                TemplateField.builder().keyName("jenis_tagihan").description("Tahun penagihan").build(),
                TemplateField.builder().keyName("tanggal_penagihan").description("Tanggal penagihan dibuat").build()
            );
            repo.saveAll(defaults);
        }
    }
}
```

### DTO: BulkStudentIdRequest.java
```java
@Data
public static class StudentInfo {
    @JsonProperty("studentId")
    private Long studentId;
    
    @JsonProperty("studentName")
    private String studentName;
    
    @JsonProperty("whatsappNumber")
    private String whatsappNumber;
    
    @JsonProperty("messageItem")
    private Map<String, Object> messageItem; // ← Accepts template data
}
```

### Processor: BroadcastMessageProcessor.java
```java
public TemplateReplaceItem buildTemplateReplaceItem(Map<String, Object> messageItem) {
    if (messageItem == null) {
        return TemplateReplaceItem.builder().build();
    }

    TemplateReplaceItem.TemplateReplaceItemBuilder builder = TemplateReplaceItem.builder();

    // Map standard keys to TemplateReplaceItem fields
    if (messageItem.containsKey("nama_siswa")) {
        builder.namaSiswa(toString(messageItem.get("nama_siswa")));
    }
    
    if (messageItem.containsKey("siswa_kelas")) {
        builder.siswaKelas(toString(messageItem.get("siswa_kelas")));
    }
    
    // ... (mapping untuk semua standard keys)
    
    return builder.build();
}
```

### Renderer: TemplateRenderer.java
```java
public String buildMessage(String message, TemplateReplaceItem item) {
    if (message == null || item == null) {
        return message;
    }

    Map<String, String> replacements = new HashMap<>();

    if (item.getNamaSiswa() != null) {
        replacements.put("nama_siswa", item.getNamaSiswa());
    }
    
    if (item.getSiswaKelas() != null) {
        replacements.put("siswa_kelas", item.getSiswaKelas());
    }
    
    // ... (mapping untuk semua fields)

    return render(message, replacements);
}
```

---

## 📤 HTTP Request Examples

### ✅ CORRECT - Using Standard Keys

#### Example 1: Bulk Send Now with Personalization
```http
POST {{baseUrl}}/api/broadcast-recipients/bulk-id/send-now
Content-Type: application/json
Authorization: Bearer {{token}}

{
  "broadcastId": 2,
  "recipientGroup": "class-10b",
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
    },
    {
      "studentId": 4,
      "studentName": "Siti Nurhaliza",
      "whatsappNumber": "6281234567893",
      "messageItem": {
        "nama_siswa": "Siti Nurhaliza",
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

#### Example 2: Monthly Billing Reminder
```http
POST {{baseUrl}}/api/broadcast-recipients/bulk-id/monthly?dayOfMonth=5
Content-Type: application/json
Authorization: Bearer {{token}}

{
  "broadcastId": 5,
  "recipientGroup": "monthly-billing",
  "students": [
    {
      "studentId": 10,
      "studentName": "Budi Santoso",
      "whatsappNumber": "6281234567890",
      "messageItem": {
        "nama_siswa": "Budi Santoso",
        "siswa_kelas": "10A",
        "nama_institut": "SMA Negeri 1",
        "nominal_tagihan": "Rp 1.500.000",
        "tanggal_tempo": "2025-01-05",
        "bulan_penagihan": "Januari 2025",
        "nama_wali": "Bapak Santoso"
      }
    }
  ]
}
```

### ❌ INCORRECT - Using Non-Standard Keys

```http
# ❌ DON'T DO THIS
{
  "broadcastId": 2,
  "students": [
    {
      "studentId": 3,
      "studentName": "Rudi",
      "whatsappNumber": "6281234567892",
      "messageItem": {
        "name": "Rudi",           // ❌ Wrong - use "nama_siswa"
        "class": "10B",           // ❌ Wrong - use "siswa_kelas"
        "studentId": "22001",     // ❌ Not a standard key
        "score": "85"             // ❌ Not a standard key
      }
    }
  ]
}
```

**Result:** Template keys like `[name]`, `[class]`, `[score]` will NOT be replaced and will remain as-is in the message.

---

## 📊 Database Schema

### Table: template_field
```sql
CREATE TABLE template_field (
    id BIGSERIAL PRIMARY KEY,
    key_name VARCHAR(100) NOT NULL UNIQUE,
    description VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Sample Data (Seeded by Bootstrap)
```sql
INSERT INTO template_field (key_name, description) VALUES
('nama_siswa', 'Nama lengkap siswa'),
('alamat_siswa', 'Alamat lengkap siswa'),
('nama_institut', 'Nama institut pendidikan'),
('siswa_kelas', 'Kelas siswa'),
('nominal_tagihan', 'Nominal tagihan siswa'),
('tanggal_tempo', 'Tanggal jatuh tempo pembayaran'),
('nama_wali', 'Nama lengkap wali siswa'),
('bulan_penagihan', 'Bulan penagihan'),
('jenis_tagihan', 'Tahun penagihan'),
('tanggal_penagihan', 'Tanggal penagihan dibuat');
```

---

## 🧪 Testing Template Rendering

### Step 1: Create Broadcast with Template

```http
POST {{baseUrl}}/api/broadcasts
Content-Type: application/json
Authorization: Bearer {{token}}

{
  "title": "Tagihan Bulanan",
  "finalContent": "Yth. [nama_wali], 

Berikut tagihan untuk [nama_siswa] kelas [siswa_kelas] di [nama_institut]:
- Nominal: [nominal_tagihan]
- Jatuh tempo: [tanggal_tempo]
- Bulan: [bulan_penagihan]

Terima kasih.",
  "status": "draft"
}
```

**Response:** Assume `broadcastId: 10`

### Step 2: Send with Message Personalization

```http
POST {{baseUrl}}/api/broadcast-recipients/bulk-id/send-now
Content-Type: application/json
Authorization: Bearer {{token}}

{
  "broadcastId": 10,
  "recipientGroup": "test-template",
  "students": [
    {
      "studentId": 100,
      "studentName": "Test Student",
      "whatsappNumber": "628123456789",
      "messageItem": {
        "nama_siswa": "Ahmad Rifai",
        "nama_wali": "Bapak Ahmad",
        "siswa_kelas": "10A",
        "nama_institut": "SMA Negeri 1 Jakarta",
        "nominal_tagihan": "Rp 1.500.000",
        "tanggal_tempo": "2025-01-15",
        "bulan_penagihan": "Januari 2025"
      }
    }
  ]
}
```

### Step 3: Verify Database

```sql
SELECT id, student_name, message, send_status 
FROM broadcast_recipient 
WHERE broadcast_id = 10 
ORDER BY created_at DESC 
LIMIT 1;
```

**Expected Result:**
```
id: 1234
student_name: Test Student
message: "Yth. Bapak Ahmad, 

Berikut tagihan untuk Ahmad Rifai kelas 10A di SMA Negeri 1 Jakarta:
- Nominal: Rp 1.500.000
- Jatuh tempo: 2025-01-15
- Bulan: Januari 2025

Terima kasih."
send_status: pending
```

✅ **All placeholders replaced correctly!**

---

## 🔍 Troubleshooting

### Problem 1: Template Keys Not Replaced

**Symptom:** Message contains `[nama_siswa]` instead of actual name.

**Possible Causes:**
1. ❌ Using non-standard key in `messageItem` (e.g., `"name"` instead of `"nama_siswa"`)
2. ❌ Key name typo (e.g., `"nama_siwa"` instead of `"nama_siswa"`)
3. ❌ `messageItem` is `null` or empty

**Solution:**
- ✅ Verify `messageItem` uses exact standard keys from `TemplateFieldBootstrap`
- ✅ Check JSON key spelling matches exactly (case-sensitive)
- ✅ Ensure `messageItem` is not null in request payload

### Problem 2: Message Field is NULL in Database

**Symptom:** `broadcast_recipient.message` column is NULL.

**Possible Causes:**
1. ❌ `BulkStudentIdRequest.StudentInfo` missing `messageItem` field
2. ❌ Service method not calling `templateRenderer.buildMessage()`
3. ❌ Broadcast template (`finalContent`) is NULL

**Solution:**
- ✅ Verify `BulkStudentIdRequest.StudentInfo` has `@JsonProperty("messageItem")` field
- ✅ Check service methods call template rendering logic
- ✅ Ensure broadcast entity has valid `finalContent`

### Problem 3: Partial Replacement

**Symptom:** Some keys replaced, others not.

**Example:**
- Template: `"[nama_siswa] kelas [siswa_kelas] tagihan [nominal_tagihan]"`
- Result: `"Ahmad Rifai kelas [siswa_kelas] tagihan Rp 1.500.000"`

**Cause:** `messageItem` missing some keys (e.g., `siswa_kelas` not provided).

**Solution:**
- ✅ Include ALL template keys in `messageItem` that exist in template
- ✅ Use `BroadcastMessageProcessor.extractTemplateKeys()` to identify required keys

---

## 📚 Related Files

### Core Implementation
- `src/main/java/com/phoenix/qrion/entities/TemplateField.java`
- `src/main/java/com/phoenix/qrion/bootstrap/TemplateFieldBootstrap.java`
- `src/main/java/com/phoenix/qrion/service/template/TemplateRenderer.java`
- `src/main/java/com/phoenix/qrion/service/broadcast/BroadcastMessageProcessor.java`

### DTOs & Requests
- `src/main/java/com/phoenix/qrion/dto/broadcast/BulkStudentIdRequest.java`
- `src/main/java/com/phoenix/qrion/dto/broadcast/BulkBroadcastRecipientRequest.java`
- `src/main/java/com/phoenix/qrion/dto/template/TemplateReplaceItem.java`

### Service Layer
- `src/main/java/com/phoenix/qrion/service/broadcast/impl/BroadcastRecipientServiceImpl.java`

### HTTP Examples
- `http/BroadcastRecipient.http` ← **UPDATED with standard keys**

### Documentation
- `documentations/BROADCAST_HTTP_PAYLOAD_FIX.md`
- `documentations/BROADCAST_MESSAGE_PERSONALIZATION.md`
- `documentations/BROADCAST_TEMPLATE_FIELD_STANDARD.md` ← **This document**

---

## ✅ Validation Checklist

Before sending broadcast with personalization, verify:

- [ ] Broadcast entity has valid `finalContent` with template placeholders
- [ ] All template keys use **standard format** (e.g., `[nama_siswa]` not `[name]`)
- [ ] JSON request includes `messageItem` object
- [ ] All keys in `messageItem` match **exactly** with `TemplateFieldBootstrap` keys
- [ ] No typos in key names (case-sensitive)
- [ ] Service method calls `templateRenderer.buildMessage()` before saving
- [ ] Database column `broadcast_recipient.message` is NOT NULL after save

---

## 🚀 Summary

**Key Points:**
1. ✅ Use ONLY standard template field keys from `TemplateFieldBootstrap`
2. ✅ Template format: `[nama_siswa]` or `{{nama_siswa}}`
3. ✅ JSON payload: `"messageItem": { "nama_siswa": "..." }`
4. ✅ Rendering happens in service layer via `TemplateRenderer`
5. ✅ Rendered message stored in `broadcast_recipient.message`

**Standard Keys Reference:**
```
nama_siswa | alamat_siswa | nama_institut | siswa_kelas | nominal_tagihan
tanggal_tempo | nama_wali | bulan_penagihan | jenis_tagihan | tanggal_penagihan
```

**Do NOT use custom keys** like `name`, `class`, `score`, `studentId` - they will not be replaced!

---

**Document Version:** 1.0  
**Last Updated:** December 21, 2025  
**Status:** ✅ Production Ready
