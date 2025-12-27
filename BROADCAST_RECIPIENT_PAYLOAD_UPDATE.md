# Broadcast Recipient Payload Update Documentation

## Overview
File `.http` untuk Broadcast Recipient telah diperbarui untuk mendukung **message personalization per-recipient** melalui field `messageItem` yang baru.

## Files Updated

### 1. **BroadcastRecipient.http** (NEW)
File HTTP testing yang komprehensif untuk semua operasi broadcast recipient dengan contoh payload lengkap.

**Lokasi**: `/http/BroadcastRecipient.http`

**Isi**:
- Send broadcast immediately (single & bulk)
- Schedule broadcast for specific date/time
- Monthly recurring broadcast
- Bulk operations (by IDs dan direct)
- Complete payload examples dengan message personalization
- Documentation lengkap

### 2. **Broadcasts.http** (UPDATED)
File HTTP original diperbarui dengan:
- Informasi tentang message personalization baru
- Contoh payload dengan `messageItem`
- Penjelasan field dan options
- Link ke dokumentasi lengkap di BroadcastRecipient.http

**Lokasi**: `/http/Broadcasts.http`

## Payload Structure

### Basic Recipient Payload (Simple)
```json
{
  "broadcastId": 1,
  "studentId": 12345,
  "studentName": "Budi Santoso",
  "whatsappNumber": "6281234567890"
}
```

### Enhanced Payload with Message Personalization
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
    "tuition": "Rp 1.500.000",
    "score": "85",
    "occasion": "Christmas"
  }
}
```

## Key Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `broadcastId` | Long | Yes | ID of the broadcast to send |
| `studentId` | Long | Yes | ID of the recipient student |
| `studentName` | String | Yes | Name of recipient student |
| `whatsappNumber` | String | Yes | WhatsApp number (format: 62XXXXXXXXXX) |
| `recipientGroup` | String | No | Group identifier (e.g., "class-10a", "billing") |
| `messageItem` | Map<String, Object> | No | **NEW**: Key-value pairs for message personalization |

## Message Personalization

### How It Works
1. Broadcast contains template content with placeholders: `{{key}}`
2. `messageItem` provides values for each placeholder
3. System replaces placeholders with values from `messageItem`
4. Final message stored in `BroadcastRecipient.message` (TEXT column)

### Example

**Template Content**:
```
Halo {{name}}, tuition Anda adalah {{tuition}} untuk kelas {{class}}.
Nilai Anda: {{score}}.
```

**messageItem**:
```json
{
  "name": "Budi",
  "tuition": "Rp 1.500.000",
  "class": "10A",
  "score": "85"
}
```

**Final Message**:
```
Halo Budi, tuition Anda adalah Rp 1.500.000 untuk kelas 10A.
Nilai Anda: 85.
```

## API Endpoints

### Send Now (Immediate Delivery)
```
POST /api/broadcasts/recipients/send-now
POST /api/broadcasts/recipients/bulk/send-now
POST /api/broadcasts/recipients/bulk-id/send-now
```

### Schedule (Delayed Delivery)
```
POST /api/broadcasts/recipients/schedule?scheduleDate=YYYY-MM-DDTHH:mm:ss
POST /api/broadcasts/recipients/bulk/schedule?scheduleDate=YYYY-MM-DDTHH:mm:ss
POST /api/broadcasts/recipients/bulk-id/schedule?scheduleDate=YYYY-MM-DDTHH:mm:ss
```

### Monthly (Recurring)
```
POST /api/broadcasts/recipients/monthly?dayOfMonth=1-31
POST /api/broadcasts/recipients/bulk/monthly?dayOfMonth=1-31
POST /api/broadcasts/recipients/bulk-id/monthly?dayOfMonth=1-31
```

## Bulk Operations

### Bulk by IDs (recipients array)
```json
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

## Example Use Cases

### 1. Billing Reminder with Personalization
```json
{
  "broadcastId": 4,
  "studentId": 12345,
  "studentName": "Budi Santoso",
  "whatsappNumber": "6281234567890",
  "recipientGroup": "billing-reminder",
  "messageItem": {
    "name": "Budi",
    "month": "January",
    "amount": "Rp 1.500.000",
    "dueDate": "10th"
  }
}
```

### 2. Exam Announcement with Student Scores
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
    "grade": "A",
    "date": "2025-12-21"
  }
}
```

### 3. Scholarship Notification
```json
{
  "broadcastId": 5,
  "studentId": 12349,
  "studentName": "Dewi Lestari",
  "whatsappNumber": "6281234567894",
  "recipientGroup": "scholarship",
  "messageItem": {
    "name": "Dewi",
    "scholarshipType": "Prestasi",
    "amount": "Rp 2.000.000",
    "month": "December"
  }
}
```

## Testing with .http Files

### Using VS Code REST Client Extension

1. **Open file**: `/http/BroadcastRecipient.http`
2. **Set variables** in VS Code settings (if needed):
   - `{{baseUrl}}`: http://localhost:8081
   - `{{token}}`: Your JWT token
3. **Click "Send Request"** above any request block
4. **View response** in sidebar

### Single Request
- Click "Send Request" button above the request
- Response appears in VS Code's REST Client panel

### Multiple Recipients
- Use bulk endpoint: `/bulk/send-now`
- Provide array of recipients with individual `messageItem` values

## Database Updates

### BroadcastRecipient Entity
```java
@Column(columnDefinition = "TEXT")
private String message; // Actual message text (after template replacement)
```

**Storage**: Final rendered message stored here after personalization.

## Implementation Details

### Message Processing Flow
1. User submits broadcast with `messageItem`
2. System extracts template keys: `{{key}}`
3. Template key extraction identifies required variables
4. For each key, system replaces with value from `messageItem`
5. Final message stored in `BroadcastRecipient.message`
6. Message sent to WhatsApp via provider

### Template Key Extraction
- Pattern: `{{` + key name + `}}`
- Supports nested/hierarchical keys
- Case-sensitive matching

### Fallback Behavior
- If `messageItem` missing for a key → placeholder remains (e.g., "{{name}}")
- If `messageItem` null → uses broadcast base content
- Single recipients use message personalization
- Bulk recipients each get individual personalized message

## Notes

- **messageItem is optional**: Simple payloads without personalization still work
- **Per-recipient personalization**: Each recipient can have different message
- **Group categorization**: recipientGroup helps organize and manage recipients
- **WhatsApp integration required**: For actual delivery to WhatsApp
- **Valid JWT token required**: For all API calls
- **scheduleDate format**: ISO 8601 (YYYY-MM-DDTHH:mm:ss)
- **dayOfMonth range**: 1-31 (system validates for month length)

## Testing Checklist

- [ ] Create broadcast with template or manual content
- [ ] Add single recipient with messageItem
- [ ] Verify final message stored correctly
- [ ] Test bulk delivery with different messageItems
- [ ] Test scheduled delivery
- [ ] Test monthly recurring delivery
- [ ] Verify WhatsApp delivery (if provider configured)
- [ ] Test without messageItem (backward compatibility)
- [ ] Test with missing messageItem values (fallback)

## Related Files

- Source: `src/main/java/com/phoenix/qrion/controllers/BroadcastRecipientController.java`
- Entity: `src/main/java/com/phoenix/qrion/entities/BroadcastRecipient.java`
- DTO: `src/main/java/com/phoenix/qrion/dto/broadcast/BroadcastRecipientRequest.java`
- Service: `src/main/java/com/phoenix/qrion/service/impl/BroadcastRecipientServiceImpl.java`
- Tests: `/http/BroadcastRecipient.http` (this file)

## Support

For complete request examples and all delivery methods, see:
- **BroadcastRecipient.http**: Comprehensive examples for all endpoints
- **Broadcasts.http**: Integration examples with broadcast creation

