# Auth Helper Usage Guide

## Overview
The `Auth` class adalah static helper untuk mengakses informasi authentication di aplikasi Anda. Anda sekarang bisa mendapatkan data user lengkap dari database, bukan hanya dari JWT claims.

## Cara Menggunakan

### 1. Get User Object Lengkap dari Database
```java
// Di controller, service, atau komponen manapun
import com.phoenix.qrion.security.Auth;

Users user = Auth.auth().getUser();
if (user != null) {
    Integer yayasanId = user.getYayasan().getId();
    Integer institutionId = user.getInstitution().getId();
    String name = user.getName();
    String email = user.getEmail();
    // ... akses semua property dari Users entity
}
```

### 2. Get Specific Fields dari Database
```java
// Get dari database user object
Integer yayasanId = Auth.auth().getYayasanIdFromDb();
Integer institutionId = Auth.auth().getInstitutionIdFromDb();
String name = Auth.auth().getNameFromDb();
String email = Auth.auth().getEmailFromDb();
String avatar = Auth.auth().getAvatarFromDb();
String cover = Auth.auth().getCoverFromDb();
String userIstype = Auth.auth().getUserIstypeFromDb();
String uuid = Auth.auth().getUuidFromDb();
```

### 3. Get Dari JWT Claims (Jika ada di token)
```java
// Get dari JWT token (jika tersedia di claims)
Integer userId = Auth.auth().getUserId();
Integer yayasanIdFromJwt = Auth.auth().getYayasanId();
Integer institutionIdFromJwt = Auth.auth().getInstitutionId();
String nameFromJwt = Auth.auth().getName();
String emailFromJwt = Auth.auth().getEmail();
String avatarFromJwt = Auth.auth().getAvatar();
String coverFromJwt = Auth.auth().getCover();
String userIstypeFromJwt = Auth.auth().getUserIstype();
String uuidFromJwt = Auth.auth().getUuid();
```

### 4. Get CustomUserDetails
```java
CustomUserDetails customUserDetails = Auth.auth().getUserDetails();
if (customUserDetails != null) {
    Integer id = customUserDetails.getId();
    String username = customUserDetails.getUsername();
    Collection<? extends GrantedAuthority> authorities = customUserDetails.getAuthorities();
    // ... akses custom user details
}
```

### 5. Basic Auth Methods (Sudah Ada Sebelumnya)
```java
// Check if authenticated
boolean isAuth = Auth.auth().isAuthenticated();

// Get username
String username = Auth.auth().getUsername();

// Get ID
Long id = Auth.auth().getId();

// Check role
boolean isAdmin = Auth.auth().hasRole("ROLE_ADMIN");

// Check privilege
boolean canCreate = Auth.auth().hasPrivilege("CREATE");

// Get authorities
Set<String> authorities = Auth.auth().getAuthorities();
```

## Contoh di Controller

```java
@RestController
@RequestMapping("/api/academic-years")
public class AcademicYearController {
    
    @GetMapping
    public ResponseEntity<?> getAll() {
        try {
            // Get user lengkap dari database
            Users currentUser = Auth.auth().getUser();
            
            if (currentUser == null) {
                return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                    .body(new ApiResponse(false, "User not found", null));
            }
            
            Integer yayasanId = currentUser.getYayasan().getId();
            Integer institutionId = currentUser.getInstitution().getId();
            
            // Gunakan yayasanId dan institutionId untuk filter data
            List<AcademicYear> years = academicYearService.findByYayasanAndInstitution(
                yayasanId, 
                institutionId
            );
            
            return ResponseEntity.ok(new ApiResponse(true, "Success", years));
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(new ApiResponse(false, e.getMessage(), null));
        }
    }
}
```

## Important Notes

1. **UserRepository Injection**: Auth class sudah di-inject dengan `UserRepository`, jadi bisa query ke database.
2. **Null Safety**: Selalu check apakah hasil null sebelum mengakses property.
3. **Performance**: Method `getUser()` akan query database setiap kali dipanggil. Jika perlu digunakan berkali-kali, simpan dalam variable:
   ```java
   Users user = Auth.auth().getUser();
   // Gunakan user variable berkali-kali
   ```

4. **FromDb Methods**: Method dengan suffix `FromDb` akan mengambil dari database user object.
5. **JWT Methods**: Method tanpa suffix akan mencoba membaca dari JWT claims terlebih dahulu.

## Updated Methods

| Method | Source | Return Type | Description |
|--------|--------|-------------|-------------|
| `getUser()` | DB + JWT | Users | Object user lengkap dengan semua relasi |
| `getYayasanIdFromDb()` | DB | Integer | ID yayasan dari database |
| `getInstitutionIdFromDb()` | DB | Integer | ID institusi dari database |
| `getNameFromDb()` | DB | String | Nama user dari database |
| `getEmailFromDb()` | DB | String | Email user dari database |
| `getAvatarFromDb()` | DB | String | Avatar URL dari database |
| `getCoverFromDb()` | DB | String | Cover image URL dari database |
| `getUserIstypeFromDb()` | DB | String | Tipe user dari database |
| `getUuidFromDb()` | DB | String | UUID user dari database |
| `getYayasanId()` | JWT | Integer | ID yayasan dari token |
| `getInstitutionId()` | JWT | Integer | ID institusi dari token |
| `getUserId()` | JWT | Integer | ID user dari token |
| `getName()` | JWT | String | Nama dari token |
| `getEmail()` | JWT | String | Email dari token |
| `getUserIstype()` | JWT | String | Tipe user dari token |
| `getUuid()` | JWT | String | UUID dari token |
| `getAvatar()` | JWT | String | Avatar dari token |
| `getCover()` | JWT | String | Cover dari token |
