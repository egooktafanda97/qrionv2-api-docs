# 🚀 Tutorial CI/CD & Deployment - Qrion Backend

## 📚 Daftar Isi
1. [Pengenalan CI/CD](#pengenalan-cicd)
2. [Arsitektur Deployment](#arsitektur-deployment)
3. [Setup Environment Development](#setup-environment-development)
4. [Setup Environment Production](#setup-environment-production)
5. [Cara Kerja CI/CD](#cara-kerja-cicd)
6. [Tutorial Deploy](#tutorial-deploy)

---

## 🎯 Pengenalan CI/CD

### Apa itu CI/CD?

**CI/CD** adalah singkatan dari **Continuous Integration** (Integrasi Berkelanjutan) dan **Continuous Deployment** (Deployment Berkelanjutan).

#### Continuous Integration (CI)
- Setiap kali kode diubah, otomatis di-test dan di-build
- Memastikan kode baru tidak merusak kode yang sudah ada
- Mendeteksi error lebih cepat

#### Continuous Deployment (CD)
- Setelah kode lolos test, otomatis di-deploy ke server
- Tidak perlu manual copy-paste file ke server
- Deployment konsisten dan cepat

### Keuntungan CI/CD untuk Project Ini:

✅ **Deployment Cepat**: 2-3 menit (dulu manual bisa 30 menit)
✅ **Konsisten**: Semua server dapat image yang sama
✅ **Aman**: Ada rollback otomatis jika gagal
✅ **Mudah**: Tinggal push tag, sisanya otomatis

---

## 🏗️ Arsitektur Deployment

### Alur Sederhana:

```
Developer (Kamu)
    ↓
Push Code ke GitHub
    ↓
GitHub Actions (CI/CD)
    ↓ [Build & Test]
    ↓ [Create Docker Image]
    ↓ [Push ke Registry]
    ↓
SSH ke Server
    ↓ [Pull Image]
    ↓ [Restart Container]
    ↓
Server Running ✅
```

### Environment yang Ada:

1. **Local**: Laptop/PC kamu untuk development
2. **Development Server**: Server untuk testing fitur baru
3. **Production Server**: Server utama yang dipakai user

---

## 💻 Setup Environment Development

### 1. Persiapan Server Development

**Spesifikasi Minimum:**
- OS: Ubuntu 20.04+
- RAM: 2GB (recommended 4GB)
- Storage: 20GB free space
- Docker & Docker Compose terinstall

### 2. Install Docker (Jika Belum Ada)

```bash
# Update package
sudo apt update

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verifikasi
docker --version
docker-compose --version
```

### 3. Setup Direktori di Server Development

```bash
# Login ke server development
ssh user@your-dev-server

# Buat direktori
sudo mkdir -p /opt/qrion-dev
cd /opt/qrion-dev

# Buat folder logs
mkdir -p logs
```

### 4. Copy File yang Dibutuhkan

**File yang perlu ada di `/opt/qrion-dev/`:**

a. **docker-compose.dev.yml**
```bash
# Copy dari repository
curl -o docker-compose.dev.yml https://raw.githubusercontent.com/qrionworld/qrionv2-ontuition-be/main/docker-compose.dev.yml
```

b. **File .env** (optional, environment sudah di docker-compose)

### 5. Login ke GitHub Container Registry

```bash
# Buat GitHub Personal Access Token (PAT) dulu:
# 1. Pergi ke GitHub.com → Settings → Developer settings
# 2. Personal access tokens → Generate new token
# 3. Pilih scope: read:packages
# 4. Copy token yang dibuat

# Login ke GHCR
echo "GITHUB_TOKEN_KAMU" | docker login ghcr.io -u USERNAME_GITHUB_KAMU --password-stdin
```

### 6. Test Pull Image & Jalankan

```bash
# Set tag image yang mau dipakai
export IMAGE_TAG=dev-v1.0.0

# Pull image
docker-compose -f docker-compose.dev.yml pull

# Jalankan
docker-compose -f docker-compose.dev.yml up -d

# Cek status
docker-compose -f docker-compose.dev.yml ps

# Cek logs
docker-compose -f docker-compose.dev.yml logs -f qrion-app-dev
```

### 7. Verifikasi Development Berjalan

```bash
# Cek health endpoint
curl http://localhost:8080/actuator/health

# Cek database
docker exec qrion-postgres-dev psql -U postgres -d qrion_dev -c "SELECT count(*) FROM users;"

# Akses PgAdmin
# Buka browser: http://your-dev-server:5050
# Login: admin@qrion.com / admin123
```

---

## 🏭 Setup Environment Production

### 1. Persiapan Server Production

**Spesifikasi Minimum:**
- OS: Ubuntu 20.04+
- RAM: 4GB minimum (8GB recommended)
- Storage: 50GB free space
- CPU: 2 cores minimum
- Docker & Docker Compose terinstall

### 2. Setup Database Production

**PENTING**: Database production TIDAK di-docker, gunakan database eksternal (AWS RDS, managed PostgreSQL, dll)

Jika pakai PostgreSQL lokal:
```bash
# Install PostgreSQL
sudo apt update
sudo apt install postgresql postgresql-contrib

# Login ke PostgreSQL
sudo -u postgres psql

# Buat database
CREATE DATABASE qrion;
CREATE USER postgres WITH PASSWORD 'root';
GRANT ALL PRIVILEGES ON DATABASE qrion TO postgres;
\q
```

### 3. Setup Direktori di Server Production

```bash
# Login ke server production
ssh user@your-prod-server

# Buat direktori
sudo mkdir -p /opt/qrion-prod
cd /opt/qrion-prod

# Buat folder struktur lengkap
mkdir -p logs/prometheus logs/grafana
mkdir -p monitoring/grafana/dashboards monitoring/grafana/datasources
```

### 4. Download File Konfigurasi

```bash
cd /opt/qrion-prod

# Docker compose file
curl -o docker-compose.prod.yml https://raw.githubusercontent.com/qrionworld/qrionv2-ontuition-be/main/docker-compose.prod.yml

# Monitoring configs
curl -o monitoring/prometheus.yml https://raw.githubusercontent.com/qrionworld/qrionv2-ontuition-be/main/monitoring/prometheus.yml

curl -o monitoring/grafana/datasources/prometheus.yml https://raw.githubusercontent.com/qrionworld/qrionv2-ontuition-be/main/monitoring/grafana/datasources/prometheus.yml

curl -o monitoring/grafana/dashboards/dashboard.yml https://raw.githubusercontent.com/qrionworld/qrionv2-ontuition-be/main/monitoring/grafana/dashboards/dashboard.yml
```

### 5. Buat File Environment (.env)

```bash
# Buat file .env
nano /opt/qrion-prod/.env
```

Isi dengan:
```env
SPRING_PROFILES_ACTIVE=prod

# Database Configuration
DB_URL=jdbc:postgresql://localhost:5432/qrion
DB_USERNAME=postgres
DB_PASSWORD=root

# JWT Configuration (GANTI dengan secret yang kuat!)
JWT_SECRET=GantiDenganSecretYangKuatMinimal32KarakterUntukProduksi123
JWT_EXPIRATION=3600000

# Server
SERVER_PORT=8080

# Grafana
GRAFANA_PASSWORD=password_grafana_kamu_yang_kuat

# Image Tag
IMAGE_TAG=latest
```

**PENTING**: 
- Ganti `JWT_SECRET` dengan string random yang kuat
- Ganti password database jika perlu
- Ganti `GRAFANA_PASSWORD` dengan password yang kuat

Generate JWT Secret yang kuat:
```bash
openssl rand -base64 32
```

### 6. Set Permission File .env

```bash
# Set permission agar hanya owner yang bisa baca
chmod 600 /opt/qrion-prod/.env
```

### 7. Login ke GitHub Container Registry

```bash
# Sama seperti dev
echo "GITHUB_TOKEN_KAMU" | docker login ghcr.io -u USERNAME_GITHUB_KAMU --password-stdin
```

### 8. Test Jalankan Production

```bash
cd /opt/qrion-prod

# Pull images
docker-compose pull

# Jalankan
docker-compose up -d

# Cek status
docker-compose ps

# Cek logs
docker-compose logs -f qrion-app-prod
```

### 9. Setup Firewall

```bash
# Allow ports yang dibutuhkan
sudo ufw allow 22/tcp      # SSH
sudo ufw allow 8080/tcp    # Application
sudo ufw allow 9090/tcp    # Prometheus
sudo ufw allow 3000/tcp    # Grafana

# Enable firewall
sudo ufw enable
```

### 10. Verifikasi Production Berjalan

```bash
# Cek application
curl http://localhost:8080/actuator/health

# Cek Prometheus
curl http://localhost:9090/-/healthy

# Cek Grafana
curl http://localhost:3000/api/health
```

**Akses dari Browser:**
- Application: `http://your-prod-server-ip:8080`
- Prometheus: `http://your-prod-server-ip:9090`
- Grafana: `http://your-prod-server-ip:3000` (admin / password dari .env)

---

## ⚙️ Cara Kerja CI/CD

### Arsitektur CI/CD

```
┌─────────────────────────────────────────────────────┐
│  Developer                                          │
│  - Menulis kode                                     │
│  - Commit & Push                                    │
│  - Buat Tag (dev-v1.0.0 atau v1.0.0)              │
└─────────────────┬───────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────┐
│  GitHub Repository (qrionworld/qrionv2-ontuition-be)│
│  - Menerima push tag                                │
│  - Trigger GitHub Actions                           │
└─────────────────┬───────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────┐
│  GitHub Actions Runner (CI/CD Engine)               │
│                                                     │
│  DEVELOPMENT PIPELINE (trigger: dev-v*.*.*)         │
│  ┌─────────────────────────────────────────────┐   │
│  │ 1. Checkout Code                            │   │
│  │ 2. Setup JDK 21                             │   │
│  │ 3. Run Tests (mvn test)                     │   │
│  │ 4. Build JAR (mvn package)                  │   │
│  │ 5. Build Docker Image                       │   │
│  │ 6. Push ke GHCR                             │   │
│  │    ghcr.io/.../qrion-backend-dev:dev-v1.0.0│   │
│  │ 7. SSH ke Dev Server                        │   │
│  │ 8. Pull Image & Restart Container          │   │
│  │ 9. Health Check                             │   │
│  │ 10. Slack Notification                      │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  PRODUCTION PIPELINE (trigger: v*.*.*)              │
│  ┌─────────────────────────────────────────────┐   │
│  │ 1. Security Scan (Snyk)                     │   │
│  │ 2. Run Tests + Coverage                     │   │
│  │ 3. Build JAR                                │   │
│  │ 4. Build Docker Image                       │   │
│  │ 5. Push ke GHCR                             │   │
│  │    ghcr.io/.../qrion-backend-prod:v1.0.0   │   │
│  │ 6. SSH ke Prod Server                       │   │
│  │ 7. Backup Current Container                 │   │
│  │ 8. Pull New Image                           │   │
│  │ 9. Rolling Update (Zero Downtime)           │   │
│  │ 10. Health Check                            │   │
│  │ 11. Rollback jika Gagal                     │   │
│  │ 12. Smoke Tests                             │   │
│  │ 13. Create GitHub Release                   │   │
│  │ 14. Slack Notification                      │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────┬───────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────┐
│  GitHub Container Registry (GHCR)                   │
│  - Menyimpan Docker Images                          │
│  - Private registry untuk organization              │
└─────────────────┬───────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────┐
│  Server (Development / Production)                  │
│  - Pull image dari GHCR                             │
│  - Restart container dengan image baru              │
│  - Monitoring dengan Prometheus + Grafana           │
└─────────────────────────────────────────────────────┘
```

### File CI/CD yang Digunakan

#### 1. `.github/workflows/deploy-dev.yml`

**Trigger**: Tag yang diawali dengan `dev-` (contoh: `dev-v1.0.0`, `dev-v1.2.3`)

**Langkah-langkah:**
1. **Checkout code** - Download kode dari GitHub
2. **Setup JDK 21** - Install Java untuk build
3. **Run tests** - Jalankan semua unit tests
4. **Build JAR** - Compile kode jadi file .jar
5. **Login ke GHCR** - Autentikasi ke GitHub Container Registry
6. **Build Docker Image** - Buat image dari Dockerfile.dev
7. **Push Image** - Upload image ke registry
8. **SSH ke Server** - Koneksi ke development server
9. **Deploy** - Pull image baru dan restart container
10. **Verify** - Cek health endpoint
11. **Notify** - Kirim notifikasi ke Slack

#### 2. `.github/workflows/deploy-prod.yml`

**Trigger**: Tag versi (contoh: `v1.0.0`, `prod-v1.0.0`)

**Langkah-langkah:**
1. **Security Scan** - Scan dependency vulnerabilities dengan Snyk
2. **Run Tests** - Jalankan tests dengan coverage report
3. **Upload Coverage** - Upload ke Codecov
4. **Build JAR** - Compile production build
5. **Login ke GHCR** - Autentikasi ke registry
6. **Build Docker Image** - Buat image dari Dockerfile.prod
7. **Push Image** - Upload dengan multiple tags
8. **SSH ke Server** - Koneksi ke production server
9. **Backup** - Backup container yang sedang berjalan
10. **Pull Image** - Download image baru
11. **Rolling Update** - Deploy dengan zero-downtime
12. **Health Check** - Verifikasi aplikasi berjalan
13. **Rollback** - Otomatis rollback jika gagal
14. **Smoke Tests** - Test endpoint penting
15. **Create Release** - Buat GitHub Release
16. **Notify** - Kirim notifikasi

---

## 🚀 Tutorial Deploy

### A. Deploy ke Development

#### Langkah 1: Pastikan Kode Sudah Siap

```bash
# Di laptop/PC kamu

# 1. Pastikan semua perubahan sudah di-commit
git status

# 2. Commit jika ada perubahan
git add .
git commit -m "feat: tambah fitur baru"

# 3. Push ke GitHub
git push origin main
```

#### Langkah 2: Buat Tag Development

```bash
# Format tag: dev-vMAJOR.MINOR.PATCH
# Contoh: dev-v1.0.0, dev-v1.2.3

# Buat tag
git tag dev-v1.0.0

# Push tag ke GitHub
git push origin dev-v1.0.0
```

#### Langkah 3: Monitor Proses CI/CD

1. Buka GitHub repository
2. Klik tab **Actions**
3. Lihat workflow "Deploy Development" yang sedang berjalan
4. Klik untuk melihat detail setiap step

#### Langkah 4: Verifikasi Deployment

```bash
# Setelah CI/CD selesai (±2-3 menit)

# Test dari laptop
curl http://your-dev-server:8080/actuator/health

# Atau SSH ke server dan cek
ssh user@your-dev-server
cd /opt/qrion-dev
docker-compose logs -f qrion-app-dev
```

#### Contoh Lengkap:

```bash
# Scenario: Kamu baru selesai develop fitur login
cd /path/to/qrion-project

# Commit perubahan
git add .
git commit -m "feat: implement login with JWT"
git push origin main

# Buat tag development
git tag dev-v1.1.0
git push origin dev-v1.1.0

# GitHub Actions akan otomatis:
# ✅ Run tests
# ✅ Build Docker image
# ✅ Push ke registry
# ✅ Deploy ke dev server
# ✅ Restart aplikasi

# Tunggu 2-3 menit, lalu test
curl http://dev-server:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"pass"}'
```

---

### B. Deploy ke Production

#### Langkah 1: Pastikan Development Sudah OK

```bash
# Test di development server dulu
# Pastikan semua fitur berjalan normal
# Minta QA untuk test jika perlu
```

#### Langkah 2: Merge ke Main Branch (Jika Pakai Branch)

```bash
# Jika develop dari branch feature
git checkout main
git merge feature/new-feature
git push origin main
```

#### Langkah 3: Buat Tag Production

```bash
# Format: vMAJOR.MINOR.PATCH
# Semantic Versioning:
# - MAJOR: Breaking changes (v2.0.0)
# - MINOR: New features (v1.1.0)
# - PATCH: Bug fixes (v1.0.1)

# Buat tag
git tag v1.0.0

# Atau dengan annotation (recommended)
git tag -a v1.0.0 -m "Release v1.0.0: Initial production release"

# Push tag
git push origin v1.0.0
```

#### Langkah 4: Monitor Production Deploy

1. Buka GitHub → Actions
2. Workflow "Deploy Production" akan berjalan
3. Monitor setiap step:
   - ✅ Security scan
   - ✅ Tests dengan coverage
   - ✅ Build image
   - ✅ Deploy ke server
   - ✅ Health checks
   - ✅ Smoke tests

#### Langkah 5: Verifikasi Production

```bash
# Test health
curl http://your-prod-server:8080/actuator/health

# Check Prometheus
curl http://your-prod-server:9090/-/healthy

# Check Grafana
curl http://your-prod-server:3000/api/health

# Test API endpoint
curl http://your-prod-server:8080/api/v1/students
```

#### Langkah 6: Monitor di Grafana

1. Buka `http://your-prod-server:3000`
2. Login dengan admin / password dari .env
3. Lihat dashboard:
   - CPU usage
   - Memory usage
   - Request rate
   - Response time
   - Error rate

#### Contoh Lengkap Production Deploy:

```bash
# Scenario: Release versi 1.0.0 ke production

# 1. Pastikan development OK
ssh user@dev-server
curl http://localhost:8080/api/health
# ✅ OK

# 2. Kembali ke laptop
exit

# 3. Pastikan main branch up-to-date
git checkout main
git pull origin main

# 4. Create release tag
git tag -a v1.0.0 -m "Release v1.0.0
- Feature: User authentication with JWT
- Feature: Student CRUD operations
- Feature: Teacher management
- Fix: Database connection pool
- Security: Update dependencies"

# 5. Push tag
git push origin v1.0.0

# 6. Monitor di GitHub Actions (±3-5 menit)
# Browser: https://github.com/qrionworld/qrionv2-ontuition-be/actions

# 7. Setelah deploy sukses, verifikasi
curl http://prod-server:8080/actuator/health

# 8. Check logs jika perlu
ssh user@prod-server
cd /opt/qrion-prod
docker-compose logs -f qrion-app-prod

# 9. Monitor metrics
# Browser: http://prod-server:3000
```

---

## 🔧 Troubleshooting

### Problem: CI/CD Failed di Step "Run Tests"

**Solusi:**
```bash
# Test di local dulu
mvn clean test

# Jika ada test yang fail, fix dulu sebelum push
```

### Problem: Docker Image Push Failed

**Penyebab:** Token GITHUB_TOKEN expired atau tidak ada permission

**Solusi:**
1. Regenerate GitHub token dengan scope `write:packages`
2. Update secret di GitHub repository settings

### Problem: SSH Connection Failed

**Penyebab:** SSH key tidak valid atau server tidak reachable

**Solusi:**
```bash
# Test SSH connection manual
ssh -i ~/.ssh/id_rsa user@server-ip

# Pastikan SSH key sudah di-add ke GitHub Secrets
```

### Problem: Health Check Failed After Deploy

**Penyebab:** Aplikasi gagal start atau database connection error

**Solusi:**
```bash
# SSH ke server
ssh user@server

# Cek logs
docker-compose logs qrion-app-prod

# Cek environment variables
docker exec qrion-app-prod env | grep DB_

# Restart manual jika perlu
docker-compose restart qrion-app-prod
```

### Problem: Rollback Needed

**Manual Rollback:**
```bash
# SSH ke server
ssh user@prod-server
cd /opt/qrion-prod

# Set image tag ke versi sebelumnya
export IMAGE_TAG=v1.0.0

# Pull dan restart
docker-compose pull
docker-compose up -d

# Verify
curl http://localhost:8080/actuator/health
```

---

## 📊 Monitoring Production

### Grafana Dashboard

**Metrics Penting:**
- **JVM Memory**: Heap usage, GC activity
- **HTTP Requests**: Request rate, response time, error rate
- **Database**: Connection pool usage, query time
- **System**: CPU, Memory, Disk usage

### Prometheus Queries

```promql
# Request rate per minute
rate(http_server_requests_seconds_count[1m])

# 95th percentile response time
histogram_quantile(0.95, rate(http_server_requests_seconds_bucket[5m]))

# Error rate
rate(http_server_requests_seconds_count{status=~"5.."}[5m])

# JVM memory used
jvm_memory_used_bytes{area="heap"}
```

---

## 🎓 Best Practices

### 1. Versioning Strategy

```
Development: dev-v1.0.0, dev-v1.1.0, dev-v1.2.0
Production:  v1.0.0, v1.1.0, v2.0.0

Format: vMAJOR.MINOR.PATCH
- MAJOR: Breaking changes
- MINOR: New features (backward compatible)
- PATCH: Bug fixes
```

### 2. Testing Before Production

```
1. Test di local ✅
2. Deploy ke dev ✅
3. QA testing di dev ✅
4. Deploy ke production ✅
```

### 3. Database Migrations

```bash
# Gunakan Flyway untuk production
# File migration di: src/main/resources/db/migration/

# Format: V1__initial_schema.sql
#         V2__add_users_table.sql
#         V3__add_indexes.sql
```

### 4. Backup Strategy

```bash
# Backup database production sebelum deploy
# Otomatis di CI/CD atau manual:

pg_dump -h localhost -U postgres qrion > backup-$(date +%Y%m%d).sql
```

### 5. Monitoring Alerts

Setup alerts di Prometheus untuk:
- CPU > 80%
- Memory > 85%
- Error rate > 5%
- Response time > 1s

---

## 📝 Checklist Deployment

### Before Deploy:
- [ ] Tests passed locally
- [ ] Code reviewed
- [ ] Database migrations ready (if any)
- [ ] Environment variables updated
- [ ] Secrets secured (no hardcoded passwords)

### Development Deploy:
- [ ] Create tag `dev-v*.*.*`
- [ ] Push tag
- [ ] Monitor CI/CD
- [ ] Test endpoints
- [ ] Check logs

### Production Deploy:
- [ ] Development tested OK
- [ ] Backup database
- [ ] Create tag `v*.*.*`
- [ ] Monitor CI/CD
- [ ] Verify health checks
- [ ] Test critical endpoints
- [ ] Monitor metrics
- [ ] Notify team

---

## 🆘 Support

Jika ada masalah:
1. Cek logs di GitHub Actions
2. Cek logs di server: `docker-compose logs`
3. Cek Grafana metrics
4. Contact DevOps team

---

## 🎉 Selesai!

Selamat! Sekarang kamu sudah paham:
- ✅ Cara kerja CI/CD
- ✅ Setup development environment
- ✅ Setup production environment
- ✅ Deploy dengan tag
- ✅ Monitoring dengan Grafana
- ✅ Troubleshooting common issues

**Happy Deploying! 🚀**
