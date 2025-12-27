# Authentication API Documentation

## Daftar Isi
1. [Overview](#overview)
2. [Authentication Flow](#authentication-flow)
3. [Endpoints](#endpoints)
4. [Data Models](#data-models)
5. [Validation Rules](#validation-rules)
6. [Error Handling](#error-handling)
7. [Frontend Integration Examples](#frontend-integration-examples)

---

## Overview

Authentication API mengelola autentikasi dan otorisasi pengguna menggunakan JWT (JSON Web Token) dan verifikasi OTP melalui WhatsApp.

**Base URL**: `/api/auth`

**Features**:
- ✅ Registrasi institusi baru dengan OTP verification
- ✅ Login dengan username/email/telephone + password
- ✅ OTP-based authentication via WhatsApp
- ✅ JWT Token generation dengan refresh token
- ✅ Password management (set, update, reset)
- ✅ Forgot password dengan OTP
- ✅ Multi-tenant support (yayasan & institution)

---

## Authentication Flow

### Flow 1: Registrasi Institusi Baru (dengan OTP)

```
1. POST /api/auth/register
   → System kirim OTP ke WhatsApp
   
2. POST /api/auth/verify-otp
   → Verifikasi OTP + Auto login
   → Return JWT token
   
3. POST /api/auth/set-password (Authenticated)
   → Set password untuk login selanjutnya
```

### Flow 2: Login Normal (dengan Password)

```
1. POST /api/auth/login
   → Login dengan username/email/telephone + password
   → Return JWT token
   
2. Gunakan token untuk akses endpoint lain
   Authorization: Bearer {token}
```

### Flow 3: Forgot Password

```
1. POST /api/auth/forgot-password
   → System kirim OTP ke WhatsApp
   
2. POST /api/auth/reset-password
   → Reset password dengan OTP + password baru
```

---

## Endpoints

### 1. Register Institution

Mendaftarkan institusi baru dan mengirim OTP ke WhatsApp.

**Endpoint**: `POST /api/auth/register`

**Authentication**: ❌ Public

**Request Body**:
```json
{
  "institutionCode": "SDN001",
  "institutionName": "SD Negeri 001 Jakarta",
  "district": "Menteng",
  "city": "Jakarta Pusat",
  "statusNegeri": true,
  "name": "John Doe",
  "telephone": "081234567890",
  "email": "john@example.com",
  "positionId": 1
}
```

**Response Success (200)**:
```json
{
  "success": true,
  "message": "Registrasi berhasil. Silakan cek WhatsApp untuk kode OTP.",
  "data": "Registrasi berhasil. Silakan cek WhatsApp untuk kode OTP.",
  "status": 200,
  "timestamp": "2025-11-27T08:15:30Z",
  "path": "/api/auth/register"
}
```

---

### 2. Verify OTP

Memverifikasi OTP dan langsung login (generate JWT token).

**Endpoint**: `POST /api/auth/verify-otp?telephone={telephone}&otp={otp}`

**Authentication**: ❌ Public

**Query Parameters**:
- `telephone` (String) - Nomor telepon yang didaftarkan
- `otp` (String) - Kode OTP yang diterima via WhatsApp

**Response Success (200)**:
```json
{
  "success": true,
  "message": "OTP verified successfully",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "tokenType": "Bearer",
    "expiresIn": 86400,
    "refreshToken": "refresh_token_here",
    "id": 1,
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "yayasanId": 1,
    "institutionId": 1,
    "username": "john.doe",
    "name": "John Doe",
    "email": "john@example.com",
    "userIstype": "ADMIN",
    "avatar": null,
    "cover": null
  },
  "status": 200,
  "timestamp": "2025-11-27T08:20:00Z",
  "path": "/api/auth/verify-otp"
}
```

---

### 3. Set/Update Password

Mengatur atau update password untuk user yang sudah login.

**Endpoint**: `POST /api/auth/set-password`

**Authentication**: ✅ **Requires JWT Token**

**Request Body**:
```json
{
  "password": "NewSecurePassword123!",
  "confirmPassword": "NewSecurePassword123!"
}
```

**Response Success (200)**:
```json
{
  "success": true,
  "message": "Password berhasil diupdate.",
  "data": "Password berhasil diupdate.",
  "status": 200,
  "timestamp": "2025-11-27T08:25:00Z",
  "path": "/api/auth/set-password"
}
```

---

### 4. Login with Identity & Password

Login menggunakan username/email/telephone dan password.

**Endpoint**: `POST /api/auth/login`

**Authentication**: ❌ Public

**Request Body**:
```json
{
  "userIdentity": "john@example.com",
  "password": "SecurePassword123!"
}
```

**Notes**: `userIdentity` bisa berupa username, email, atau telephone

**Response Success (200)**:
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "tokenType": "Bearer",
    "expiresIn": 86400,
    "refreshToken": "refresh_token_here",
    "id": 1,
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "yayasanId": 1,
    "institutionId": 1,
    "username": "john.doe",
    "name": "John Doe",
    "email": "john@example.com",
    "userIstype": "ADMIN"
  },
  "status": 200
}
```

---

### 5. Resend OTP

Mengirim ulang OTP untuk user yang belum verifikasi.

**Endpoint**: `POST /api/auth/resend-otp`

**Authentication**: ❌ Public

**Request Body**:
```json
{
  "telephone": "081234567890"
}
```

---

### 6. Forgot Password

Memulai proses reset password dengan mengirim OTP ke WhatsApp.

**Endpoint**: `POST /api/auth/forgot-password`

**Authentication**: ❌ Public

**Request Body**:
```json
{
  "telephone": "081234567890"
}
```

---

### 7. Reset Password

Reset password menggunakan OTP yang diterima.

**Endpoint**: `POST /api/auth/reset-password`

**Authentication**: ❌ Public

**Request Body**:
```json
{
  "telephone": "081234567890",
  "otp": "123456",
  "newPassword": "NewSecurePassword123!",
  "confirmPassword": "NewSecurePassword123!"
}
```

---

## Data Models

### RegisterRequest
```typescript
interface RegisterRequest {
  institutionCode: string;
  institutionName: string;
  district: string;
  city: string;
  statusNegeri: boolean;
  name: string;
  telephone: string;
  email: string;
  positionId: number;
}
```

### LoginIdentityRequest
```typescript
interface LoginIdentityRequest {
  userIdentity: string;  // username/email/telephone
  password: string;
}
```

### LoginResponse
```typescript
interface LoginResponse {
  token: string;
  tokenType: string;
  expiresIn: number;
  refreshToken: string;
  id: number;
  uuid: string;
  yayasanId: number;
  institutionId: number;
  username: string;
  name: string;
  email: string;
  userIstype: string;
  avatar?: string;
  cover?: string;
}
```

---

## Validation Rules

### Register Request
- institutionCode: Required, NotBlank
- institutionName: Required, NotBlank
- district: Required, NotBlank
- city: Required, NotBlank
- statusNegeri: Required, Boolean
- name: Required, NotBlank
- telephone: Required, ValidPhone (08xxxxxxxxxx)
- email: Required, ValidEmail
- positionId: Required, Integer

### Login Request
- userIdentity: Required, NotBlank
- password: Required, NotBlank, MinLength(8)

### Password Rules
- Minimal 8 karakter
- confirmPassword harus sama dengan password

### OTP Validation
- OTP: 6 digit angka
- Valid selama 5 menit
- Maksimal 3x percobaan salah

---

## Error Handling

| Error Code | HTTP Status | Description |
|------------|-------------|-------------|
| 1000 | 400 | Validation failed |
| 1006 | 409 | Duplicate entry (telephone/email) |
| 2001 | 401 | Invalid credentials/OTP |
| 3000 | 404 | User not found |
| 5000 | 500 | Internal server error |

---

## Frontend Integration Examples

### React Authentication Context

```typescript
import React, { createContext, useContext, useState } from 'react';
import axios from 'axios';

interface AuthContextType {
  user: LoginResponse | null;
  token: string | null;
  login: (userIdentity: string, password: string) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
};

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<LoginResponse | null>(null);
  const [token, setToken] = useState<string | null>(null);

  const login = async (userIdentity: string, password: string) => {
    const response = await axios.post('http://localhost:8080/api/auth/login', {
      userIdentity,
      password
    });
    
    const loginData = response.data.data;
    setToken(loginData.token);
    setUser(loginData);
    localStorage.setItem('token', loginData.token);
    localStorage.setItem('user', JSON.stringify(loginData));
    axios.defaults.headers.common['Authorization'] = `Bearer ${loginData.token}`;
  };

  const logout = () => {
    setToken(null);
    setUser(null);
    localStorage.removeItem('token');
    localStorage.removeItem('user');
    delete axios.defaults.headers.common['Authorization'];
  };

  return (
    <AuthContext.Provider value={{ user, token, login, logout, isAuthenticated: !!token }}>
      {children}
    </AuthContext.Provider>
  );
};
```

### Login Component

```tsx
import React, { useState } from 'react';
import { Form, Input, Button, message } from 'antd';
import { useAuth } from './authContext';
import { useNavigate } from 'react-router-dom';

const LoginPage: React.FC = () => {
  const { login } = useAuth();
  const navigate = useNavigate();
  const [loading, setLoading] = useState(false);

  const onFinish = async (values: any) => {
    setLoading(true);
    try {
      await login(values.userIdentity, values.password);
      message.success('Login berhasil!');
      navigate('/dashboard');
    } catch (error: any) {
      message.error(error.response?.data?.message || 'Login gagal');
    } finally {
      setLoading(false);
    }
  };

  return (
    <Form onFinish={onFinish} layout="vertical">
      <Form.Item
        label="Username/Email/Telephone"
        name="userIdentity"
        rules={[{ required: true }]}
      >
        <Input />
      </Form.Item>

      <Form.Item
        label="Password"
        name="password"
        rules={[{ required: true }]}
      >
        <Input.Password />
      </Form.Item>

      <Button type="primary" htmlType="submit" loading={loading} block>
        Login
      </Button>
    </Form>
  );
};

export default LoginPage;
```
