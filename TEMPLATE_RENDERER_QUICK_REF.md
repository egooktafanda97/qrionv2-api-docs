# Template Renderer - Quick Reference Card

## 🎯 Quick Start (Copy & Paste)

```java
// 1. Inject service
@Autowired
private TemplateRenderer templateRenderer;

// 2. Build data
TemplateReplaceItem item = TemplateReplaceItem.builder()
    .namaSiswa("Ahmad Fauzi")
    .namaWali("Budi Santoso")
    .nominalTagihan("500,000")
    .tanggalTempo("10 Januari 2025")
    .build();

// 3. Render
String message = templateRenderer.buildMessage(
    "Tagihan untuk [nama_siswa] sebesar Rp [nominal_tagihan]", 
    item
);

// Result: "Tagihan untuk Ahmad Fauzi sebesar Rp 500,000"
```

## 📋 Available Fields

| Placeholder | Builder Method | Example |
|-------------|---------------|---------|
| `[nama_siswa]` | `.namaSiswa("...")` | "Ahmad Fauzi" |
| `[alamat_siswa]` | `.alamatSiswa("...")` | "Jl. Merdeka 123" |
| `[nama_institut]` | `.namaInstitut("...")` | "SMP Negeri 1" |
| `[siswa_kelas]` | `.siswaKelas("...")` | "7A" |
| `[nominal_tagihan]` | `.nominalTagihan("...")` | "500,000" |
| `[tanggal_tempo]` | `.tanggalTempo("...")` | "10 Januari 2025" |
| `[nama_wali]` | `.namaWali("...")` | "Budi Santoso" |
| `[bulan_penagihan]` | `.bulanPenagihan("...")` | "Januari" |
| `[jenis_tagihan]` | `.jenisTagihan("...")` | "SPP" |
| `[tanggal_penagihan]` | `.tanggalPenagihan("...")` | "15 Januari 2025" |

## 🛠️ Helper Methods

```java
// Format currency
BigDecimal amount = new BigDecimal("1500000");
String formatted = TemplateReplaceItem.formatCurrency(amount);
// Result: "1,500,000"

// Format date
LocalDate date = LocalDate.of(2025, 1, 10);
String formatted = TemplateReplaceItem.formatDate(date);
// Result: "10 Januari 2025"

// Get month name
String month = TemplateReplaceItem.getIndonesianMonth(1);
// Result: "Januari"
```

## 🔥 Common Use Cases

### Use Case 1: Billing Notification
```java
TemplateReplaceItem item = TemplateReplaceItem.builder()
    .namaSiswa(student.getName())
    .namaWali(student.getParentName())
    .nominalTagihan(TemplateReplaceItem.formatCurrency(billing.getAmount()))
    .tanggalTempo(TemplateReplaceItem.formatDate(billing.getDueDate()))
    .jenisTagihan(billing.getName())
    .build();

String template = "Yth. [nama_wali], tagihan [jenis_tagihan] untuk " +
                  "[nama_siswa] sebesar Rp [nominal_tagihan] " +
                  "jatuh tempo [tanggal_tempo]";

String message = templateRenderer.buildMessage(template, item);
```

### Use Case 2: Payment Reminder
```java
TemplateReplaceItem item = TemplateReplaceItem.builder()
    .namaSiswa(student.getName())
    .siswaKelas(className)
    .nominalTagihan(formatCurrency(amount))
    .tanggalTempo(formatDate(dueDate))
    .build();

String template = "Reminder! Tagihan [nama_siswa] kelas [siswa_kelas] " +
                  "Rp [nominal_tagihan] jatuh tempo [tanggal_tempo]";
```

### Use Case 3: Payment Confirmation
```java
TemplateReplaceItem item = TemplateReplaceItem.builder()
    .namaSiswa(student.getName())
    .nominalTagihan(formatCurrency(amount))
    .tanggalPenagihan(formatDate(LocalDate.now()))
    .build();

String template = "Terima kasih! Pembayaran [nama_siswa] " +
                  "Rp [nominal_tagihan] telah diterima pada [tanggal_penagihan]";
```

## 📱 Integration Examples

### WhatsApp Integration
```java
@Service
public class WhatsAppNotificationService {
    
    @Autowired
    private TemplateRenderer templateRenderer;
    
    @Autowired
    private WhatsAppService whatsappService;
    
    public void sendBillingNotification(UserBilling billing) {
        TemplateReplaceItem item = buildItemFromBilling(billing);
        String template = getTemplate("BILLING_NOTIFICATION");
        String message = templateRenderer.buildMessage(template, item);
        
        whatsappService.send(
            billing.getBilledUser().getStudent().getParentPhone(), 
            message
        );
    }
}
```

### SMS Integration
```java
@Service
public class SmsNotificationService {
    
    @Autowired
    private TemplateRenderer templateRenderer;
    
    @Autowired
    private SmsGateway smsGateway;
    
    public void sendPaymentReminder(UserBilling billing) {
        TemplateReplaceItem item = buildItemFromBilling(billing);
        String template = getTemplate("PAYMENT_REMINDER");
        String message = templateRenderer.buildMessage(template, item);
        
        smsGateway.send(
            billing.getBilledUser().getStudent().getParentPhone(), 
            message
        );
    }
}
```

### Broadcast Service
```java
@Service
public class BroadcastService {
    
    @Autowired
    private TemplateRenderer templateRenderer;
    
    public void broadcastToStudents(List<UserBilling> billings, String templateContent) {
        for (UserBilling billing : billings) {
            TemplateReplaceItem item = buildItemFromBilling(billing);
            String message = templateRenderer.buildMessage(templateContent, item);
            sendNotification(billing, message);
        }
    }
}
```

## 🎨 Template Examples

### Short Template (SMS)
```
Tagihan [jenis_tagihan] [nama_siswa] Rp [nominal_tagihan] 
jatuh tempo [tanggal_tempo]. Info: [nama_institut]
```

### Medium Template (WhatsApp)
```
Yth. [nama_wali],

Tagihan [jenis_tagihan] untuk [nama_siswa] ([siswa_kelas])
Nominal: Rp [nominal_tagihan]
Jatuh Tempo: [tanggal_tempo]

Mohon segera melakukan pembayaran.

Terima kasih,
[nama_institut]
```

### Long Template (Email)
```
Kepada Yth. Orang Tua/Wali dari [nama_siswa],

Dengan hormat,

Kami informasikan bahwa tagihan [jenis_tagihan] untuk bulan 
[bulan_penagihan] telah tersedia dengan detail sebagai berikut:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━
▪ Nama Siswa: [nama_siswa]
▪ Kelas: [siswa_kelas]
▪ Nominal Tagihan: Rp [nominal_tagihan]
▪ Tanggal Jatuh Tempo: [tanggal_tempo]
▪ Tanggal Penagihan: [tanggal_penagihan]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Alamat: [alamat_siswa]

Mohon untuk segera melakukan pembayaran sebelum tanggal jatuh 
tempo untuk menghindari keterlambatan.

Hormat kami,
[nama_institut]
```

## ⚠️ Common Mistakes

### ❌ Don't do this:
```java
// Wrong: Not using helper methods
item.setNominalTagihan(amount.toString()); // "1500000.00"
item.setTanggalTempo(date.toString());     // "2025-01-10"

// Wrong: Forgetting null check
item.setNamaSiswa(student.getName()); // NPE if student is null
```

### ✅ Do this instead:
```java
// Correct: Use helper methods
item.setNominalTagihan(TemplateReplaceItem.formatCurrency(amount)); // "1,500,000"
item.setTanggalTempo(TemplateReplaceItem.formatDate(date));         // "10 Januari 2025"

// Correct: Null-safe
item.setNamaSiswa(student != null ? student.getName() : "N/A");
```

## 🔍 Debugging Tips

```java
// Check if placeholders are replaced
String message = templateRenderer.buildMessage(template, item);
if (message.contains("[") || message.contains("]")) {
    log.warn("Unreplaced placeholders found: {}", message);
}

// Log template and data
log.info("Template: {}", template);
log.info("Data: {}", item);
log.info("Result: {}", message);
```

## 📊 Testing

```java
@Test
void testTemplateRenderer() {
    TemplateRenderer renderer = new TemplateRenderer();
    
    String template = "Hello [nama_siswa]";
    TemplateReplaceItem item = TemplateReplaceItem.builder()
        .namaSiswa("Ahmad")
        .build();
    
    String result = renderer.buildMessage(template, item);
    assertEquals("Hello Ahmad", result);
}
```

## 🚀 Performance Tips

```java
// ✅ Good: Cache template
private String cachedTemplate;

public void sendNotifications(List<UserBilling> billings) {
    if (cachedTemplate == null) {
        cachedTemplate = getTemplate("BILLING_NOTIFICATION");
    }
    
    for (UserBilling billing : billings) {
        TemplateReplaceItem item = buildItem(billing);
        String message = templateRenderer.buildMessage(cachedTemplate, item);
        send(message);
    }
}

// ❌ Bad: Load template every time
public void sendNotifications(List<UserBilling> billings) {
    for (UserBilling billing : billings) {
        String template = getTemplate("BILLING_NOTIFICATION"); // Slow!
        // ...
    }
}
```

## 📚 More Information

- **Full Guide**: `/documentations/TEMPLATE_RENDERER_GUIDE.md`
- **Implementation Details**: `/documentations/TEMPLATE_RENDERER_IMPLEMENTATION_SUMMARY.md`
- **Flow Diagrams**: `/documentations/TEMPLATE_RENDERER_FLOW_DIAGRAM.md`
- **Test Examples**: `/src/test/java/.../TemplateRendererTest.java`
- **Code Examples**: `/http/TemplateRendererExample.http`

## 🆘 Support

Need help? Check:
1. Unit tests for working examples
2. Documentation files listed above
3. Ask the development team

---

**Version**: 1.0  
**Last Updated**: December 2024  
**Status**: Production Ready ✅
