# Summary: Broadcast Recipient Feature Updates

## What's New

Broadcast recipient system telah diupdate dengan dukungan **message personalization per-recipient**.

## Quick Reference

### Files Created/Updated

| File | Status | Purpose |
|------|--------|---------|
| `/http/BroadcastRecipient.http` | **NEW** | Complete HTTP testing file untuk broadcast recipient APIs |
| `/http/Broadcasts.http` | **UPDATED** | Original broadcast file dengan info tentang personalization baru |
| `/BROADCAST_RECIPIENT_PAYLOAD_UPDATE.md` | **NEW** | Dokumentasi lengkap (file ini yang dibaca sedang) |

### Key New Feature: messageItem Field

```json
{
  "broadcastId": 1,
  "studentId": 12345,
  "studentName": "Budi Santoso",
  "whatsappNumber": "6281234567890",
  "recipientGroup": "class-10a",
  "messageItem": {
    "name": "Budi",
    "class": "10A",
    "tuition": "Rp 1.500.000"
  }
}
```

## What Changed

### Before
- Simple payload dengan basic info
- Pesan yang sama untuk semua recipient
- Tidak ada personalisasi

### After
- **Tambahan `messageItem`**: Object dengan variable untuk personalisasi
- **Per-recipient personalization**: Setiap penerima bisa dapat pesan berbeda
- **Template replacement**: `{{key}}` dalam broadcast content diganti dengan value dari `messageItem`

## How to Use

### 1. Simple Send (No Personalization)
```bash
POST http://localhost:8081/api/broadcasts/recipients/send-now
{
  "broadcastId": 1,
  "studentId": 12345,
  "studentName": "Budi Santoso",
  "whatsappNumber": "6281234567890"
}
```

### 2. Send with Personalization
```bash
POST http://localhost:8081/api/broadcasts/recipients/send-now
{
  "broadcastId": 1,
  "studentId": 12345,
  "studentName": "Budi Santoso",
  "whatsappNumber": "6281234567890",
  "messageItem": {
    "name": "Budi",
    "class": "10A"
  }
}
```

**Result**: Broadcast content "Halo {{name}}, kelas {{class}}" menjadi "Halo Budi, kelas 10A"

### 3. Bulk Send with Per-Recipient Personalization
```bash
POST http://localhost:8081/api/broadcasts/recipients/bulk/send-now
{
  "broadcastId": 1,
  "recipients": [
    {
      "studentId": 12345,
      "studentName": "Budi",
      "whatsappNumber": "6281234567890",
      "messageItem": {"name": "Budi", "class": "10A"}
    },
    {
      "studentId": 12346,
      "studentName": "Ani",
      "whatsappNumber": "6281234567891",
      "messageItem": {"name": "Ani", "class": "10A"}
    }
  ]
}
```

**Result**: Masing-masing recipient dapat pesan yang dipersonalisasi sesuai messageItem mereka.

## API Endpoints Overview

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/broadcasts/recipients/send-now` | POST | Kirim sekarang (single) |
| `/api/broadcasts/recipients/bulk/send-now` | POST | Kirim sekarang (bulk direct) |
| `/api/broadcasts/recipients/bulk-id/send-now` | POST | Kirim sekarang (bulk by IDs) |
| `/api/broadcasts/recipients/schedule` | POST | Kirim terjadwal |
| `/api/broadcasts/recipients/bulk/schedule` | POST | Kirim terjadwal (bulk) |
| `/api/broadcasts/recipients/bulk-id/schedule` | POST | Kirim terjadwal (bulk by IDs) |
| `/api/broadcasts/recipients/monthly` | POST | Kirim bulanan |
| `/api/broadcasts/recipients/bulk/monthly` | POST | Kirim bulanan (bulk) |
| `/api/broadcasts/recipients/bulk-id/monthly` | POST | Kirim bulanan (bulk by IDs) |

## Payload Fields

### Required Fields
- `broadcastId` (Long): ID broadcast
- `studentId` (Long): ID recipient
- `studentName` (String): Nama recipient
- `whatsappNumber` (String): No. WhatsApp (format: 62XXXXXXXXXX)

### Optional Fields
- `recipientGroup` (String): Untuk pengelompokan recipient
- `messageItem` (Map): **NEW** - Key-value untuk template replacement

## Query Parameters

### For Scheduled Delivery
```
?scheduleDate=2025-12-25T09:00:00
```
Format: ISO 8601 (YYYY-MM-DDTHH:mm:ss)

### For Monthly Recurring
```
?dayOfMonth=1
```
Range: 1-31

## Testing

### Option 1: VS Code REST Client
1. Open `/http/BroadcastRecipient.http`
2. Set `{{baseUrl}}` dan `{{token}}`
3. Click "Send Request"

### Option 2: curl
```bash
curl -X POST http://localhost:8081/api/broadcasts/recipients/send-now \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "broadcastId": 1,
    "studentId": 12345,
    "studentName": "Budi",
    "whatsappNumber": "6281234567890",
    "messageItem": {"name": "Budi", "class": "10A"}
  }'
```

### Option 3: Postman
1. Import requests dari `/http/BroadcastRecipient.http`
2. Setup environment variables
3. Send requests

## Real-World Examples

### Example 1: Billing Reminder
```json
{
  "broadcastId": 4,
  "studentId": 12345,
  "studentName": "Budi Santoso",
  "whatsappNumber": "6281234567890",
  "recipientGroup": "billing-jan",
  "messageItem": {
    "name": "Budi",
    "month": "January",
    "amount": "Rp 1.500.000",
    "dueDate": "10 January 2026"
  }
}
```
Template: "Halo {{name}}, reminder pembayaran SPP {{month}} sebesar {{amount}}, jatuh tempo {{dueDate}}"
Result: "Halo Budi, reminder pembayaran SPP January sebesar Rp 1.500.000, jatuh tempo 10 January 2026"

### Example 2: Exam Results
```json
{
  "broadcastId": 2,
  "studentId": 12347,
  "studentName": "Rudi Hartono",
  "whatsappNumber": "6281234567892",
  "recipientGroup": "exam-results",
  "messageItem": {
    "name": "Rudi",
    "subject": "Mathematics",
    "score": "92",
    "grade": "A+"
  }
}
```

### Example 3: Scholarship Status
```json
{
  "broadcastId": 5,
  "studentId": 12349,
  "studentName": "Dewi Lestari",
  "whatsappNumber": "6281234567894",
  "recipientGroup": "scholarship-active",
  "messageItem": {
    "name": "Dewi",
    "type": "Prestasi Akademik",
    "amount": "Rp 2.000.000",
    "month": "December 2025"
  }
}
```

## Key Improvements

✅ **Per-Recipient Personalization**: Setiap penerima dapat pesan yang customized
✅ **Template Key Replacement**: Automatic replacement `{{key}}` dengan nilai dari messageItem
✅ **Message Storage**: Final message disimpan di BroadcastRecipient.message (TEXT column)
✅ **Backward Compatible**: Payload lama tanpa messageItem tetap bekerja
✅ **Bulk Operations**: Mendukung bulk dengan personalisasi individual
✅ **Flexible Scheduling**: Send now, scheduled, atau recurring monthly
✅ **Group Management**: recipientGroup untuk kategorisasi dan management

## Database

### Column Added
```sql
ALTER TABLE broadcast_recipient ADD COLUMN message TEXT;
```

### Storage
Final rendered message (setelah template replacement) disimpan di field `message`.

## Documentation Files

1. **BroadcastRecipient.http** - Complete API testing file dengan semua contoh
2. **Broadcasts.http** - Updated dengan info tentang personalization
3. **BROADCAST_RECIPIENT_PAYLOAD_UPDATE.md** - Dokumentasi detail (file ini)

## Next Steps

1. ✅ File `.http` sudah dibuat
2. ✅ Payload documentation sudah lengkap
3. 🔄 **Test endpoints** menggunakan file `.http`
4. 🔄 **Verify message personalization** di actual delivery
5. 🔄 **Monitor** WhatsApp integration untuk delivery status

## Support & References

- Full examples: See `/http/BroadcastRecipient.http`
- API integration: See `/http/Broadcasts.http`
- Detailed docs: See `/BROADCAST_RECIPIENT_PAYLOAD_UPDATE.md`
- Source code: `src/main/java/com/phoenix/qrion/service/impl/BroadcastRecipientServiceImpl.java`

---

**Last Updated**: December 21, 2025
**Status**: ✅ Ready for Testing
