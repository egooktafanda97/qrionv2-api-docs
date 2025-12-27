# Template Renderer - Dynamic Message Building

## Overview

Implementasi sistem template renderer yang memungkinkan penggantian placeholder dinamis dalam pesan notifikasi (SMS, Email, WhatsApp) dengan data real dari database.

## Fitur

- **Dynamic Placeholder Replacement**: Mengganti placeholder seperti `[nama_siswa]`, `[nominal_tagihan]` dengan data aktual
- **Type-Safe DTO**: Menggunakan `TemplateReplaceItem` untuk memastikan tipe data yang benar
- **Indonesian Formatting**: Format tanggal dan mata uang dalam bahasa Indonesia
- **Flexible Template Syntax**: Mendukung format `[key]` dan `{{key}}`
- **Null-Safe**: Menangani nilai null dengan baik (diganti dengan string kosong)

## Struktur File

### 1. TemplateReplaceItem.java
```
Location: src/main/java/com/phoenix/qrion/dto/template/TemplateReplaceItem.java
```

DTO yang menampung data untuk penggantian placeholder.

**Fields yang tersedia:**

| Field Name | Template Key | Description | Example |
|------------|--------------|-------------|---------|
| `namaSiswa` | `[nama_siswa]` | Nama lengkap siswa | "Ahmad Fauzi" |
| `alamatSiswa` | `[alamat_siswa]` | Alamat lengkap siswa | "Jl. Merdeka No. 123, Jakarta" |
| `namaInstitut` | `[nama_institut]` | Nama institut pendidikan | "SMP Negeri 1 Jakarta" |
| `siswaKelas` | `[siswa_kelas]` | Kelas siswa | "7A" |
| `nominalTagihan` | `[nominal_tagihan]` | Nominal tagihan siswa | "500,000" |
| `tanggalTempo` | `[tanggal_tempo]` | Tanggal jatuh tempo | "10 Januari 2025" |
| `namaWali` | `[nama_wali]` | Nama lengkap wali siswa | "Budi Santoso" |
| `bulanPenagihan` | `[bulan_penagihan]` | Bulan penagihan | "Januari" |
| `jenisTagihan` | `[jenis_tagihan]` | Jenis/tahun tagihan | "SPP" |
| `tanggalPenagihan` | `[tanggal_penagihan]` | Tanggal penagihan dibuat | "15 Januari 2025" |

**Helper Methods:**

```java
// Format BigDecimal to Indonesian currency (without "Rp")
String formatCurrency(BigDecimal amount)
// Example: 1500000 -> "1,500,000"

// Format LocalDate to Indonesian date format
String formatDate(LocalDate date)
// Example: 2025-01-10 -> "10 Januari 2025"

// Get Indonesian month name from month index
String getIndonesianMonth(int monthIndex)
// Example: 1 -> "Januari", 12 -> "Desember"
```

### 2. TemplateRenderer.java
```
Location: src/main/java/com/phoenix/qrion/service/template/TemplateRenderer.java
```

Service untuk melakukan rendering template dengan data aktual.

**Main Method:**

```java
public String buildMessage(String message, TemplateReplaceItem item)
```

**Cara Kerja:**
1. Menerima template message dengan placeholder `[key]` atau `{{key}}`
2. Mengekstrak semua field dari `TemplateReplaceItem`
3. Mengganti setiap placeholder dengan value yang sesuai
4. Return message yang sudah dirender

## Cara Penggunaan

### 1. Basic Usage - Manual Data

```java
@Autowired
private TemplateRenderer templateRenderer;

public void sendBillingNotification() {
    // 1. Template message
    String template = "Yth. [nama_wali],\n\n" +
                      "Tagihan [jenis_tagihan] untuk [nama_siswa] " +
                      "sebesar Rp [nominal_tagihan] telah dibuat.\n\n" +
                      "Mohon dibayarkan sebelum [tanggal_tempo].";
    
    // 2. Build data
    TemplateReplaceItem item = TemplateReplaceItem.builder()
        .namaWali("Budi Santoso")
        .namaSiswa("Ahmad Fauzi")
        .jenisTagihan("SPP")
        .nominalTagihan("500,000")
        .tanggalTempo("10 Januari 2025")
        .build();
    
    // 3. Render
    String message = templateRenderer.buildMessage(template, item);
    
    // 4. Send (SMS/Email/WhatsApp)
    sendSMS(parentPhone, message);
}
```

**Output:**
```
Yth. Budi Santoso,

Tagihan SPP untuk Ahmad Fauzi sebesar Rp 500,000 telah dibuat.

Mohon dibayarkan sebelum 10 Januari 2025.
```

### 2. Integration with UserBilling - Real Data

```java
@Service
public class BillingNotificationService {
    
    @Autowired
    private TemplateRenderer templateRenderer;
    
    @Autowired
    private UserBillingRepository userBillingRepository;
    
    @Autowired
    private StudentEnrollmentRepository enrollmentRepository;
    
    public void sendBillingNotification(Long userBillingId) {
        // Fetch data from database
        UserBilling ub = userBillingRepository.findById(userBillingId)
            .orElseThrow(() -> new RuntimeException("User billing not found"));
        
        Students student = ub.getBilledUser().getStudent();
        Institution institution = ub.getInstitution();
        Billing billing = ub.getBilling();
        
        // Get current enrollment to find class
        StudentEnrollment enrollment = enrollmentRepository
            .findActiveEnrollmentByStudentId(student.getId())
            .orElse(null);
        
        String className = enrollment != null ? 
            enrollment.getSchoolClass().getName() : "N/A";
        
        // Get template from database
        String template = getTemplateFromDatabase("BILLING_NOTIFICATION");
        
        // Build replacement data
        TemplateReplaceItem item = TemplateReplaceItem.builder()
            .namaSiswa(student.getName())
            .alamatSiswa(student.getFullAddress())
            .namaInstitut(institution.getName())
            .siswaKelas(className)
            .nominalTagihan(TemplateReplaceItem.formatCurrency(ub.getNetAmount()))
            .tanggalTempo(TemplateReplaceItem.formatDate(billing.getBillingDueDate()))
            .namaWali(student.getParentName())
            .bulanPenagihan(TemplateReplaceItem.getIndonesianMonth(ub.getMonthIndex()))
            .jenisTagihan(billing.getName())
            .tanggalPenagihan(TemplateReplaceItem.formatDate(LocalDate.now()))
            .build();
        
        // Render message
        String message = templateRenderer.buildMessage(template, item);
        
        // Send notification
        sendWhatsApp(student.getParentPhone(), message);
    }
}
```

### 3. Broadcast Service Integration

```java
@Service
public class BroadcastService {
    
    @Autowired
    private TemplateRenderer templateRenderer;
    
    public void broadcastBillingReminders(Long academicYearId) {
        // Get all unpaid billings
        List<UserBilling> unpaidBillings = userBillingRepository
            .findUnpaidByAcademicYear(academicYearId);
        
        String template = "Reminder: Tagihan [jenis_tagihan] untuk [nama_siswa] " +
                          "sebesar Rp [nominal_tagihan] akan jatuh tempo pada [tanggal_tempo]. " +
                          "Segera lakukan pembayaran!";
        
        for (UserBilling ub : unpaidBillings) {
            Students student = ub.getBilledUser().getStudent();
            Billing billing = ub.getBilling();
            
            TemplateReplaceItem item = TemplateReplaceItem.builder()
                .namaSiswa(student.getName())
                .jenisTagihan(billing.getName())
                .nominalTagihan(TemplateReplaceItem.formatCurrency(ub.getNetAmount()))
                .tanggalTempo(TemplateReplaceItem.formatDate(billing.getBillingDueDate()))
                .build();
            
            String message = templateRenderer.buildMessage(template, item);
            sendNotification(student.getParentPhone(), message);
        }
    }
}
```

## Template Examples

### 1. Billing Notification (Formal)

```
Kepada Yth. Orang Tua/Wali dari [nama_siswa],

Dengan hormat,

Kami informasikan bahwa tagihan [jenis_tagihan] untuk bulan [bulan_penagihan] telah tersedia dengan detail sebagai berikut:

▪ Nama Siswa: [nama_siswa]
▪ Kelas: [siswa_kelas]
▪ Nominal Tagihan: Rp [nominal_tagihan]
▪ Tanggal Jatuh Tempo: [tanggal_tempo]
▪ Tanggal Penagihan: [tanggal_penagihan]

Alamat: [alamat_siswa]

Mohon untuk segera melakukan pembayaran sebelum tanggal jatuh tempo untuk menghindari keterlambatan.

Hormat kami,
[nama_institut]
```

### 2. Payment Reminder (Friendly)

```
Halo [nama_wali]! 👋

Ini pengingat ramah dari [nama_institut].

Tagihan [jenis_tagihan] untuk [nama_siswa] ([siswa_kelas]) sebesar Rp [nominal_tagihan] akan jatuh tempo pada [tanggal_tempo].

Mohon segera melakukan pembayaran ya!

Terima kasih 😊
```

### 3. Payment Confirmation

```
✅ Pembayaran Berhasil!

Terima kasih [nama_wali]!

Pembayaran [jenis_tagihan] untuk [nama_siswa] ([siswa_kelas]) sebesar Rp [nominal_tagihan] telah kami terima pada [tanggal_penagihan].

Tagihan bulan [bulan_penagihan] telah LUNAS.

[nama_institut]
```

### 4. Overdue Notice

```
⚠️ PEMBERITAHUAN KETERLAMBATAN

Kepada [nama_wali],

Tagihan [jenis_tagihan] untuk [nama_siswa] ([siswa_kelas]) sebesar Rp [nominal_tagihan] telah melewati tanggal jatuh tempo ([tanggal_tempo]).

Mohon segera menghubungi [nama_institut] untuk melakukan pembayaran dan menghindari sanksi administrasi.

Terima kasih atas perhatian Anda.
```

## Testing

Unit test tersedia di: `src/test/java/com/phoenix/qrion/service/template/TemplateRendererTest.java`

**Run tests:**
```bash
./mvnw test -Dtest=TemplateRendererTest
```

**Test coverage:**
- ✅ Basic placeholder replacement
- ✅ Null handling
- ✅ Currency formatting
- ✅ Date formatting (Indonesian)
- ✅ Month name translation
- ✅ Real-world notification examples
- ✅ Missing fields (replaced with empty string)

## API Integration Example

### Controller Example

```java
@RestController
@RequestMapping("/api/notifications")
public class NotificationController {
    
    @Autowired
    private TemplateRenderer templateRenderer;
    
    @PostMapping("/preview")
    public ResponseEntity<String> previewMessage(
            @RequestBody NotificationPreviewRequest request) {
        
        TemplateReplaceItem item = TemplateReplaceItem.builder()
            .namaSiswa(request.getStudentName())
            .namaWali(request.getParentName())
            .siswaKelas(request.getClassName())
            .nominalTagihan(request.getAmount())
            .tanggalTempo(request.getDueDate())
            .build();
        
        String message = templateRenderer.buildMessage(
            request.getTemplate(), item);
        
        return ResponseEntity.ok(message);
    }
}
```

## Database Schema Reference

Template fields dari tabel `template_field`:

| ID | Description | Key Name |
|----|-------------|----------|
| 8 | Nama lengkap siswa | nama_siswa |
| 9 | Alamat lengkap siswa | alamat_siswa |
| 10 | Nama institut pendidikan | nama_institut |
| 11 | Kelas siswa | siswa_kelas |
| 12 | Nominal tagihan siswa | nominal_tagihan |
| 13 | Tanggal jatuh tempo pembayaran | tanggal_tempo |
| 14 | Nama lengkap wali siswa | nama_wali |
| 15 | Bulan penagihan | bulan_penagihan |
| 16 | Tahun penagihan | jenis_tagihan |
| 17 | Tanggal penagihan dibuat | tanggal_penagihan |

## Best Practices

### 1. Always Use Helper Methods for Formatting

❌ **Bad:**
```java
item.setNominalTagihan(amount.toString()); // "1500000.00"
item.setTanggalTempo(date.toString()); // "2025-01-10"
```

✅ **Good:**
```java
item.setNominalTagihan(TemplateReplaceItem.formatCurrency(amount)); // "1,500,000"
item.setTanggalTempo(TemplateReplaceItem.formatDate(date)); // "10 Januari 2025"
```

### 2. Handle Null Values

```java
TemplateReplaceItem item = TemplateReplaceItem.builder()
    .namaSiswa(student != null ? student.getName() : "N/A")
    .siswaKelas(enrollment != null ? enrollment.getSchoolClass().getName() : "-")
    .nominalTagihan(TemplateReplaceItem.formatCurrency(amount))
    .build();
```

### 3. Reuse Templates

Store templates in database (`template_broadcast` table) and retrieve them:

```java
String template = templateBroadcastRepository
    .findByCategory("BILLING_NOTIFICATION")
    .map(TemplateBroadcast::getContent)
    .orElse(DEFAULT_TEMPLATE);
```

### 4. Validate Before Sending

```java
public void sendNotification(String phone, String message) {
    // Validate that all placeholders are replaced
    if (message.contains("[") || message.contains("]")) {
        log.warn("Message still contains unreplaced placeholders: {}", message);
        // Handle error
    }
    
    // Send message
    smsService.send(phone, message);
}
```

## Troubleshooting

### Problem: Placeholder tidak terganti

**Cause:** Key name tidak match dengan field di `TemplateReplaceItem`

**Solution:** Pastikan key di template sama dengan yang ada di mapping:
- Template: `[nama_siswa]`
- Java field: `namaSiswa`
- Key yang digunakan: `"nama_siswa"` (underscore, lowercase)

### Problem: Format tanggal/currency salah

**Cause:** Tidak menggunakan helper method

**Solution:** Selalu gunakan:
- `TemplateReplaceItem.formatCurrency()` untuk BigDecimal
- `TemplateReplaceItem.formatDate()` untuk LocalDate
- `TemplateReplaceItem.getIndonesianMonth()` untuk month index

### Problem: NullPointerException

**Cause:** Field null tapi tidak di-handle

**Solution:** Gunakan conditional atau default value:
```java
.namaSiswa(student != null ? student.getName() : "N/A")
```

## Future Enhancements

- [ ] Template preview in UI
- [ ] Template validation before saving
- [ ] More template fields (e.g., nis, parent_email, billing_type)
- [ ] Multi-language support (English templates)
- [ ] Template versioning
- [ ] Conditional rendering (if/else in templates)

## Related Files

- `TemplateReplaceItem.java`: DTO for template data
- `TemplateRenderer.java`: Service for rendering
- `TemplateRendererTest.java`: Unit tests
- `TemplateRendererExample.http`: Usage examples
- `TemplateBroadcast.java`: Entity for storing templates
- `TemplateField.java`: Entity for template field definitions

## Support

For questions or issues, contact the development team or refer to:
- Project documentation in `/documentations`
- API documentation in Postman collection
- Unit tests for usage examples
