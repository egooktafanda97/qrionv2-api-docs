# 🚀 Quick Deployment Reference

## Image-Based Deployment Flow

```
┌─────────────────┐
│  Git Tag Push   │
│  dev-v1.0.0 or  │
│    v1.0.0       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ GitHub Actions  │
│  - Run Tests    │
│  - Build Image  │
│  - Push to GHCR │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  SSH to Server  │
│  - Pull Image   │
│  - Restart App  │
│  - Verify       │
└─────────────────┘
```

## 📋 Quick Commands

### Development Deployment

```bash
# Tag and deploy
git tag dev-v1.0.0
git push origin dev-v1.0.0

# Manual on server
export IMAGE_TAG=dev-v1.0.0
cd /opt/qrion-dev
docker-compose pull
docker-compose up -d
```

### Production Deployment

```bash
# Tag and deploy
git tag v1.0.0
git push origin v1.0.0

# Manual on server
export IMAGE_TAG=v1.0.0
cd /opt/qrion-prod
docker-compose pull
docker-compose up -d
```

### View Logs

```bash
# Application logs
tail -f /opt/qrion-prod/logs/qrion.log

# Prometheus logs
tail -f /opt/qrion-prod/logs/prometheus/prometheus.log

# Grafana logs
tail -f /opt/qrion-prod/logs/grafana/grafana.log

# Container logs
docker-compose logs -f qrion-app-prod
```

### Health Checks

```bash
# Application health
curl http://localhost:8080/actuator/health

# Prometheus targets
curl http://localhost:9090/api/v1/targets

# Container status
docker-compose ps
```

### Rollback

```bash
# Production rollback
cd /opt/qrion-prod
export IMAGE_TAG=v1.0.0-previous
docker-compose pull
docker-compose up -d
```

## 🔧 Server Setup (One-Time)

### Development Server

```bash
curl -o setup.sh https://raw.githubusercontent.com/qrionworld/qrionv2-ontuition-be/main/scripts/setup-dev-server.sh
chmod +x setup.sh
./setup.sh
```

### Production Server

```bash
curl -o setup.sh https://raw.githubusercontent.com/qrionworld/qrionv2-ontuition-be/main/scripts/setup-prod-server.sh
chmod +x setup.sh
./setup.sh
# IMPORTANT: Edit /opt/qrion-prod/.env with production values!
```

## 🔐 GitHub Secrets Needed

```
DEV_SERVER_HOST=10.0.1.100
DEV_SERVER_USER=ubuntu
DEV_SERVER_SSH_KEY=<private-key-content>

PROD_SERVER_HOST=10.0.2.100
PROD_SERVER_USER=ubuntu
PROD_SERVER_SSH_KEY=<private-key-content>
```

## 📊 Access URLs

### Development
- App: http://dev-server:8080
- PgAdmin: http://dev-server:5050

### Production
- App: http://prod-server:8080
- Prometheus: http://prod-server:9090
- Grafana: http://prod-server:3000 (admin / from .env)

## ⚡ Troubleshooting

### Image Pull Failed
```bash
docker login ghcr.io -u YOUR_USERNAME
docker pull ghcr.io/qrionworld/qrionv2-ontuition-be/qrion-backend-prod:latest
```

### Container Won't Start
```bash
docker-compose logs qrion-app-prod
docker-compose restart qrion-app-prod
```

### Out of Space
```bash
docker system prune -af
docker volume prune -f
```

## 📁 Server Directory Structure

```
/opt/qrion-prod/
├── docker-compose.prod.yml
├── .env
├── logs/
│   ├── qrion.log
│   ├── prometheus/
│   └── grafana/
└── monitoring/
    ├── prometheus.yml
    └── grafana/
        ├── datasources/
        └── dashboards/
```

## ✅ Deployment Checklist

- [ ] Server has Docker and Docker Compose installed
- [ ] Directory `/opt/qrion-dev` or `/opt/qrion-prod` created
- [ ] `docker-compose.yml` downloaded
- [ ] `.env` file configured with correct values
- [ ] Monitoring configs downloaded (prod only)
- [ ] Logged in to GHCR: `docker login ghcr.io`
- [ ] Firewall rules configured
- [ ] SSH keys added to GitHub Secrets
- [ ] First deployment successful
- [ ] Health check passing

## 📚 Documentation

- **Full Setup Guide**: `SERVER_SETUP.md`
- **Docker Guide**: `DOCKER_SETUP.md`
- **API Documentation**: `documentations/`
