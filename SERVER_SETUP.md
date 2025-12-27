# Server Setup Guide

This guide explains how to prepare your servers for **image-based deployment** using Docker.

## 📋 Overview

The deployment process works as follows:
1. **CI/CD Pipeline** builds Docker images in GitHub Actions
2. Images are pushed to **GitHub Container Registry (GHCR)**
3. **SSH connection** is used to deploy to servers
4. Servers **pull pre-built images** and restart containers
5. **No source code or build tools** are needed on servers

## 🖥️ Server Requirements

### Minimum Requirements
- **OS**: Ubuntu 20.04+ or any Linux with Docker support
- **RAM**: 2GB minimum (4GB recommended for production)
- **CPU**: 2 cores minimum
- **Disk**: 20GB free space
- **Network**: Stable internet connection for pulling images

### Required Software
- Docker 20.10+
- Docker Compose 2.0+
- SSH server configured
- curl (for setup scripts)

## 🚀 Development Server Setup

### Quick Setup (Automated)

```bash
# Download and run setup script
curl -o setup-dev-server.sh https://raw.githubusercontent.com/qrionworld/qrionv2-ontuition-be/main/scripts/setup-dev-server.sh
chmod +x setup-dev-server.sh
./setup-dev-server.sh
```

### Manual Setup

```bash
# 1. Create application directory
sudo mkdir -p /opt/qrion-dev
cd /opt/qrion-dev

# 2. Create logs directories
mkdir -p logs/prometheus logs/grafana

# 3. Download docker-compose file
curl -o docker-compose.dev.yml https://raw.githubusercontent.com/qrionworld/qrionv2-ontuition-be/main/docker-compose.dev.yml

# 4. Create .env file
cat > .env << 'EOF'
SPRING_PROFILES_ACTIVE=dev
DB_URL=jdbc:postgresql://postgres-dev:5432/qrion_dev
DB_USERNAME=postgres
DB_PASSWORD=postgres_dev_pass
JWT_SECRET=DevSecretKeyForDevelopmentEnvironment1234567890
JWT_EXPIRATION=86400000
IMAGE_TAG=latest
EOF

# 5. Set permissions
sudo chown -R $USER:$USER /opt/qrion-dev
chmod 755 /opt/qrion-dev

# 6. Login to GitHub Container Registry
echo "YOUR_GITHUB_TOKEN" | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```

### First Deployment

```bash
# Pull and run development environment
export IMAGE_TAG=dev-v1.0.0
docker-compose -f /opt/qrion-dev/docker-compose.dev.yml pull
docker-compose -f /opt/qrion-dev/docker-compose.dev.yml up -d

# Check status
docker-compose -f /opt/qrion-dev/docker-compose.dev.yml ps

# View logs
docker-compose -f /opt/qrion-dev/docker-compose.dev.yml logs -f
```

## 🏭 Production Server Setup

### Quick Setup (Automated)

```bash
# Download and run setup script
curl -o setup-prod-server.sh https://raw.githubusercontent.com/qrionworld/qrionv2-ontuition-be/main/scripts/setup-prod-server.sh
chmod +x setup-prod-server.sh
./setup-prod-server.sh
```

### Manual Setup

```bash
# 1. Create application directory
sudo mkdir -p /opt/qrion-prod
cd /opt/qrion-prod

# 2. Create logs and monitoring directories
mkdir -p logs/prometheus logs/grafana
mkdir -p monitoring/grafana/dashboards monitoring/grafana/datasources

# 3. Download docker-compose file
curl -o docker-compose.prod.yml https://raw.githubusercontent.com/qrionworld/qrionv2-ontuition-be/main/docker-compose.prod.yml

# 4. Download monitoring configurations
curl -o monitoring/prometheus.yml https://raw.githubusercontent.com/qrionworld/qrionv2-ontuition-be/main/monitoring/prometheus.yml
curl -o monitoring/grafana/datasources/prometheus.yml https://raw.githubusercontent.com/qrionworld/qrionv2-ontuition-be/main/monitoring/grafana/datasources/prometheus.yml
curl -o monitoring/grafana/dashboards/dashboard.yml https://raw.githubusercontent.com/qrionworld/qrionv2-ontuition-be/main/monitoring/grafana/dashboards/dashboard.yml

# 5. Create .env file (EDIT WITH YOUR VALUES!)
cat > .env << 'EOF'
SPRING_PROFILES_ACTIVE=prod
DB_URL=jdbc:postgresql://your-prod-db-host:5432/qrion_prod
DB_USERNAME=your_db_user
DB_PASSWORD=your_secure_password
JWT_SECRET=YourVerySecureProductionSecret123456789012345
JWT_EXPIRATION=3600000
SERVER_PORT=8080
GRAFANA_PASSWORD=your_grafana_password
IMAGE_TAG=latest
EOF

# 6. Set permissions
sudo chown -R $USER:$USER /opt/qrion-prod
chmod 755 /opt/qrion-prod
chmod 600 /opt/qrion-prod/.env  # Secure the environment file

# 7. Login to GitHub Container Registry
echo "YOUR_GITHUB_TOKEN" | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```

### ⚠️ Production Configuration

**IMPORTANT**: Edit `/opt/qrion-prod/.env` with your production values:

```bash
nano /opt/qrion-prod/.env
```

**Required Changes**:
- `DB_URL`: Your production database connection string
- `DB_USERNAME`: Production database username
- `DB_PASSWORD`: Strong production database password
- `JWT_SECRET`: Generate with `openssl rand -base64 32`
- `GRAFANA_PASSWORD`: Strong Grafana admin password

### First Deployment

```bash
# Pull and run production environment
export IMAGE_TAG=v1.0.0
docker-compose -f /opt/qrion-prod/docker-compose.prod.yml pull
docker-compose -f /opt/qrion-prod/docker-compose.prod.yml up -d

# Check status
docker-compose -f /opt/qrion-prod/docker-compose.prod.yml ps

# View logs
docker-compose -f /opt/qrion-prod/docker-compose.prod.yml logs -f
```

## 🔐 SSH Configuration for CI/CD

### Generate SSH Key

On your local machine:

```bash
# Generate SSH key pair
ssh-keygen -t ed25519 -C "github-actions-qrion" -f ~/.ssh/qrion-deploy

# This creates:
# - Private key: ~/.ssh/qrion-deploy (add to GitHub Secrets)
# - Public key: ~/.ssh/qrion-deploy.pub (add to server)
```

### Add Public Key to Server

```bash
# On the server
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Add the public key
cat >> ~/.ssh/authorized_keys << 'EOF'
# Paste your public key here (content of qrion-deploy.pub)
EOF

chmod 600 ~/.ssh/authorized_keys
```

### Test SSH Connection

```bash
# From your local machine
ssh -i ~/.ssh/qrion-deploy user@your-server-ip

# Should connect without password
```

### Add to GitHub Secrets

1. Go to your GitHub repository
2. Settings → Secrets and variables → Actions
3. Add secrets:

**Development Server**:
- `DEV_SERVER_HOST`: Your dev server IP/hostname
- `DEV_SERVER_USER`: SSH username
- `DEV_SERVER_SSH_KEY`: Content of `~/.ssh/qrion-deploy` (private key)
- `DEV_SERVER_PORT`: SSH port (usually 22)

**Production Server**:
- `PROD_SERVER_HOST`: Your prod server IP/hostname
- `PROD_SERVER_USER`: SSH username
- `PROD_SERVER_SSH_KEY`: Content of `~/.ssh/qrion-deploy` (private key)
- `PROD_SERVER_PORT`: SSH port (usually 22)

## 🔥 Firewall Configuration

### Development Server

```bash
# Allow SSH
sudo ufw allow 22/tcp

# Allow application
sudo ufw allow 8080/tcp

# Allow PostgreSQL (if accessing externally)
sudo ufw allow 5433/tcp

# Allow PgAdmin
sudo ufw allow 5050/tcp

# Enable firewall
sudo ufw enable
```

### Production Server

```bash
# Allow SSH
sudo ufw allow 22/tcp

# Allow application
sudo ufw allow 8080/tcp

# Allow Prometheus
sudo ufw allow 9090/tcp

# Allow Grafana
sudo ufw allow 3000/tcp

# Enable firewall
sudo ufw enable
```

## 📊 Monitoring Access

### Production Monitoring URLs

After deployment, access monitoring tools:

- **Application**: `http://your-server-ip:8080`
- **Prometheus**: `http://your-server-ip:9090`
- **Grafana**: `http://your-server-ip:3000`

### Grafana Login

- **Username**: `admin`
- **Password**: Value from `GRAFANA_PASSWORD` in `.env`

## 🔄 Deployment Workflow

### Automated Deployment (Recommended)

```bash
# Development
git tag dev-v1.0.0
git push origin dev-v1.0.0

# Production
git tag v1.0.0
git push origin v1.0.0
```

GitHub Actions will:
1. Build Docker image
2. Push to GHCR
3. SSH to server
4. Pull new image
5. Restart containers
6. Verify health

### Manual Deployment

```bash
# On the server
cd /opt/qrion-prod  # or /opt/qrion-dev

# Login to registry
echo "YOUR_TOKEN" | docker login ghcr.io -u YOUR_USERNAME --password-stdin

# Pull new image
export IMAGE_TAG=v1.0.0
docker-compose pull

# Restart with new image
docker-compose up -d

# Check status
docker-compose ps
docker-compose logs -f
```

## 🛠️ Maintenance Commands

### View Logs

```bash
# Application logs
tail -f /opt/qrion-prod/logs/qrion.log

# Container logs
docker-compose -f /opt/qrion-prod/docker-compose.prod.yml logs -f

# Specific service
docker-compose -f /opt/qrion-prod/docker-compose.prod.yml logs -f qrion-app-prod
```

### Restart Services

```bash
cd /opt/qrion-prod

# Restart all services
docker-compose restart

# Restart specific service
docker-compose restart qrion-app-prod
```

### Update to Latest

```bash
cd /opt/qrion-prod

# Pull latest images
docker-compose pull

# Recreate containers with new images
docker-compose up -d

# Clean up old images
docker image prune -af
```

### Backup

```bash
# Backup logs
tar -czf qrion-logs-$(date +%Y%m%d).tar.gz /opt/qrion-prod/logs/

# Backup configuration
tar -czf qrion-config-$(date +%Y%m%d).tar.gz /opt/qrion-prod/.env /opt/qrion-prod/monitoring/

# Backup database (if PostgreSQL on same server)
docker exec qrion-postgres-dev pg_dump -U postgres qrion_dev > backup-$(date +%Y%m%d).sql
```

## 🐛 Troubleshooting

### Container Won't Start

```bash
# Check logs
docker-compose logs qrion-app-prod

# Check container status
docker ps -a

# Restart service
docker-compose restart qrion-app-prod
```

### Cannot Pull Image

```bash
# Check registry login
docker login ghcr.io

# Try manual pull
docker pull ghcr.io/qrionworld/qrionv2-ontuition-be/qrion-backend-prod:latest

# Check network
ping ghcr.io
```

### Database Connection Failed

```bash
# Check database connectivity
docker exec qrion-app-prod ping your-db-host

# Check environment variables
docker exec qrion-app-prod env | grep DB_

# Test database connection
docker exec qrion-app-prod curl http://localhost:8080/actuator/health
```

### Out of Disk Space

```bash
# Check disk usage
df -h

# Clean up Docker
docker system prune -af
docker volume prune -f

# Clean up old logs
find /opt/qrion-prod/logs -name "*.log.*" -mtime +7 -delete
```

## 📚 Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)

## 🤝 Support

For issues or questions:
1. Check application logs: `/opt/qrion-prod/logs/`
2. Check container logs: `docker-compose logs`
3. Check GitHub Actions workflow runs
4. Contact DevOps team
