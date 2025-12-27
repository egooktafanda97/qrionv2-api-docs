# Qrion Backend - Docker & CI/CD Setup

Complete Docker and CI/CD setup for Qrion Backend with Development and Production environments.

## 📋 Table of Contents
- [Profiles Configuration](#profiles-configuration)
- [Docker Setup](#docker-setup)
- [CI/CD Pipeline](#cicd-pipeline)
- [Monitoring](#monitoring)
- [Quick Start](#quick-start)

## 🔧 Profiles Configuration

### Available Profiles
- **local**: For local development (localhost database)
- **dev**: For Docker development environment (with PostgreSQL container)
- **prod**: For production environment (external database)

### Profile Files
- `application.yaml` - Base configuration
- `application-local.yaml` - Local development
- `application-dev.yaml` - Docker development
- `application-prod.yaml` - Production

## 🐳 Docker Setup

### Development Environment (with PostgreSQL)

**File**: `docker-compose.dev.yml`

Includes:
- PostgreSQL 15 database
- Spring Boot application (pre-built image from GHCR)
- PgAdmin (database management UI)

**Important**: This uses pre-built Docker images from GitHub Container Registry, not local builds.

```bash
# Set image tag (optional, defaults to 'latest')
export IMAGE_TAG=dev-v1.0.0

# Start development environment
docker-compose -f docker-compose.dev.yml up -d

# Stop environment
docker-compose -f docker-compose.dev.yml down

# View logs
docker-compose -f docker-compose.dev.yml logs -f qrion-app-dev

# View application logs from mounted volume
tail -f logs/qrion.log
```

**Access URLs**:
- Application: http://localhost:8080
- PgAdmin: http://localhost:5050
  - Email: admin@qrion.com
  - Password: admin123

### Production Environment (external database)

**File**: `docker-compose.prod.yml`

Includes:
- Spring Boot application (pre-built image from GHCR)
- Prometheus (metrics collection with logs)
- Grafana (monitoring dashboards with logs)

**Important**: This uses pre-built Docker images from GitHub Container Registry.

```bash
# Create .env file from template
cp .env.example .env
# Edit .env and fill in production values

# Set image tag
export IMAGE_TAG=v1.0.0

# Start production environment
docker-compose -f docker-compose.prod.yml up -d

# Stop environment
docker-compose -f docker-compose.prod.yml down

# View monitoring logs
tail -f logs/prometheus/prometheus.log
tail -f logs/grafana/grafana.log
```

**Access URLs**:
- Application: http://localhost:8080
- Prometheus: http://localhost:9090
- Grafana: http://localhost:3000
  - Username: admin
  - Password: (from .env GRAFANA_PASSWORD)

**Logs Location**:
- Application logs: `./logs/`
- Prometheus logs: `./logs/prometheus/`
- Grafana logs: `./logs/grafana/`

## 🚀 CI/CD Pipeline

### GitHub Actions Workflows

Two workflows are configured for **automated image-based deployment via SSH**.

#### 1. Development Deployment
**Trigger**: Tags matching `dev-*` or `dev-v*.*.*`

```bash
# Create and push development tag
git tag dev-v1.0.0
git push origin dev-v1.0.0
```

**Workflow**: `.github/workflows/deploy-dev.yml`
1. Runs tests
2. Builds Docker image
3. Pushes image to GitHub Container Registry (GHCR)
4. **SSH to dev server**
5. **Pulls pre-built image** (no building on server)
6. Restarts containers with new image
7. Sends Slack notification

**Key Feature**: Server only pulls and runs pre-built images, no source code or build process on server.

#### 2. Production Deployment
**Trigger**: Tags matching `prod-v*.*.*` or `v*.*.*`

```bash
# Create and push production tag
git tag v1.0.0
git push origin v1.0.0
```

**Workflow**: `.github/workflows/deploy-prod.yml`
1. Security scanning (Snyk)
2. Runs tests with coverage
3. Builds Docker image
4. Pushes image to GitHub Container Registry (GHCR)
5. **SSH to production server**
6. **Pulls pre-built image** (no building on server)
7. Creates backup of current container
8. Rolling update with zero-downtime
9. Health check verification
10. Automatic rollback on failure
11. Runs smoke tests
12. Creates GitHub release
13. Sends Slack notification

**Key Features**: 
- Server only pulls and runs pre-built images
- Zero-downtime deployment
- Automatic backup and rollback on failure
- No source code or build tools required on production server

### Required GitHub Secrets

Set these in GitHub repository settings (Settings → Secrets → Actions):

```
# Development Server
DEV_SERVER_HOST=your-dev-server-ip
DEV_SERVER_USER=your-ssh-user
DEV_SERVER_SSH_KEY=your-private-ssh-key
DEV_SERVER_PORT=22

# Production Server
PROD_SERVER_HOST=your-prod-server-ip
PROD_SERVER_USER=your-ssh-user
PROD_SERVER_SSH_KEY=your-private-ssh-key
PROD_SERVER_PORT=22

# Optional
SLACK_WEBHOOK_URL=your-slack-webhook
SNYK_TOKEN=your-snyk-token
CODECOV_TOKEN=your-codecov-token
```

### Server Requirements

**What's needed on deployment servers**:
- Docker installed
- Docker Compose installed
- SSH access configured
- Directory structure:
  - Development: `/opt/qrion-dev/`
  - Production: `/opt/qrion-prod/`
- Files on server:
  - `docker-compose.dev.yml` (for dev server)
  - `docker-compose.prod.yml` (for prod server)
  - `.env` file with environment variables (prod only)
  - `monitoring/` directory with configs (prod only)

**What's NOT needed on servers**:
- ❌ Source code
- ❌ Maven/Java build tools
- ❌ Git repository
- ❌ Application dependencies

**Deployment Process**:
1. CI/CD builds image in GitHub Actions
2. Image pushed to GitHub Container Registry
3. SSH to server
4. Pull pre-built image
5. Restart containers with new image

## 📊 Monitoring

### Logs Structure

All logs are stored in the `./logs/` directory:

```
logs/
├── qrion.log              # Application logs (Spring Boot)
├── qrion-error.log        # Application error logs
├── prometheus/            # Prometheus logs
│   └── prometheus.log
└── grafana/              # Grafana logs
    └── grafana.log
```

**Viewing Logs**:
```bash
# Application logs
tail -f logs/qrion.log

# Error logs only
tail -f logs/qrion-error.log

# Prometheus logs
tail -f logs/prometheus/prometheus.log

# Grafana logs
tail -f logs/grafana/grafana.log

# All logs combined
tail -f logs/**/*.log
```

### Health Check Endpoints

Spring Boot Actuator provides several monitoring endpoints:

- **Health**: http://localhost:8080/actuator/health
- **Info**: http://localhost:8080/actuator/info
- **Metrics**: http://localhost:8080/actuator/metrics
- **Prometheus**: http://localhost:8080/actuator/prometheus

### Prometheus Metrics

Prometheus collects metrics from the application:
- JVM metrics (memory, threads, GC)
- HTTP request metrics
- Database connection pool metrics
- Custom application metrics

### Grafana Dashboards

Access Grafana at http://localhost:3000

Pre-configured datasource:
- Prometheus (http://prometheus:9090)

**Creating Dashboards**:
1. Login to Grafana
2. Create new dashboard
3. Add panels with Prometheus queries

**Useful Queries**:
```promql
# Request rate
rate(http_server_requests_seconds_count[5m])

# 95th percentile response time
histogram_quantile(0.95, rate(http_server_requests_seconds_bucket[5m]))

# JVM memory usage
jvm_memory_used_bytes{area="heap"}

# Active database connections
hikaricp_connections_active
```

## 🚀 Quick Start

### Local Development

```bash
# 1. Start local PostgreSQL (or use Docker)
docker run -d \
  --name postgres-local \
  -e POSTGRES_DB=qrion \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=root \
  -p 5432:5432 \
  postgres:15-alpine

# 2. Run application with local profile
mvn spring-boot:run -Dspring-boot.run.profiles=local

# Or with environment variable
export SPRING_PROFILES_ACTIVE=local
mvn spring-boot:run
```

### Docker Development

```bash
# 1. Build and start all services
docker-compose -f docker-compose.dev.yml up -d --build

# 2. Check application logs
docker-compose -f docker-compose.dev.yml logs -f qrion-app-dev

# 3. Access application
curl http://localhost:8080/actuator/health
```

### Docker Production

```bash
# 1. Create .env file with production values
cp .env.example .env
nano .env

# 2. Build and start services
docker-compose -f docker-compose.prod.yml up -d --build

# 3. Monitor with Grafana
open http://localhost:3000
```

## 📦 Building Docker Images

### Development Image
```bash
docker build -f Dockerfile.dev -t qrion-backend-dev .
```

### Production Image
```bash
docker build -f Dockerfile.prod -t qrion-backend-prod .
```

## 🔍 Troubleshooting

### Check application logs
```bash
# Development
docker-compose -f docker-compose.dev.yml logs qrion-app-dev

# Production
docker-compose -f docker-compose.prod.yml logs qrion-app-prod
```

### Check health endpoint
```bash
curl http://localhost:8080/actuator/health
```

### Connect to database (development)
```bash
docker exec -it qrion-postgres-dev psql -U postgres -d qrion_dev
```

### Restart services
```bash
docker-compose -f docker-compose.dev.yml restart
```

## 📝 Environment Variables

### Development (.env for dev)
```env
SPRING_PROFILES_ACTIVE=dev
DB_URL=jdbc:postgresql://postgres-dev:5432/qrion_dev
DB_USERNAME=postgres
DB_PASSWORD=postgres_dev_pass
JWT_SECRET=DevSecretKeyForDevelopmentEnvironment1234567890
```

### Production (.env for prod)
```env
SPRING_PROFILES_ACTIVE=prod
DB_URL=jdbc:postgresql://your-prod-db:5432/qrion_prod
DB_USERNAME=your_db_user
DB_PASSWORD=your_secure_password
JWT_SECRET=YourVerySecureProductionSecret123456789012345
JWT_EXPIRATION=3600000
SERVER_PORT=8080
GRAFANA_PASSWORD=your_grafana_password
```

## 🔐 Security Considerations

1. **Never commit .env files** - Use .env.example as template
2. **Use strong JWT secrets** - Generate with: `openssl rand -base64 32`
3. **Rotate secrets regularly** - Especially in production
4. **Use HTTPS in production** - Configure reverse proxy (nginx/traefik)
5. **Secure database access** - Use private networks, strong passwords
6. **Enable firewall rules** - Limit access to necessary ports only

## 📚 Additional Resources

- [Spring Boot Actuator Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

## 🤝 Contributing

1. Create feature branch
2. Make changes
3. Test locally with `docker-compose.dev.yml`
4. Push and create tag for deployment
5. CI/CD will automatically deploy

## 📄 License

[Your License Here]
