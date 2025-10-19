# 🚢 Deployment Guide

## 📋 Table of Contents

- [Overview](#overview)
- [Phase 1: Self-hosted (Year 1)](#phase-1-self-hosted-year-1)
- [Phase 2: Hybrid (Year 2)](#phase-2-hybrid-year-2)
- [Phase 3: Full Cloud (Year 3+)](#phase-3-full-cloud-year-3)
- [Migration Strategy](#migration-strategy)
- [Monitoring & Maintenance](#monitoring--maintenance)
- [Backup & Recovery](#backup--recovery)
- [Security Hardening](#security-hardening)

---

## 🎯 Overview

### Deployment Philosophy

```
Start Small → Scale Smart → Optimize Continuously
```

**3-Phase Strategy:**
1. **Self-hosted** - ต้นทุนต่ำ, ควบคุมเต็ม, เหมาะกับ MVP
2. **Hybrid** - แบ่งเบา workload, เริ่ม scale
3. **Full Cloud** - Enterprise-ready, auto-scale, HA

---

## 📍 Phase 1: Self-hosted (Year 1)

### Infrastructure: Dell R730xd

```
Hardware Specs:
├── CPU: Dual Xeon E5-2640 v4 (20 cores / 40 threads)
├── RAM: 128 GB DDR4 ECC
├── Storage: 30 TB (ZFS RAID-Z2)
│   ├── 500 GB NVMe - OS + Databases
│   ├── 10 TB HDD - Documents
│   └── 20 TB HDD - Backups
├── Network: 1 Gbps dedicated
└── Hypervisor: Proxmox VE 8.2
```

### VM Layout

```
┌─────────────────────────────────────────┐
│         Proxmox VE 8.2                  │
├─────────────────────────────────────────┤
│                                         │
│  VM1: App Server (Ubuntu 22.04)        │
│  ├── RAM: 64 GB                         │
│  ├── CPU: 16 cores                      │
│  ├── Disk: 200 GB (NVMe)                │
│  └── Services:                          │
│      ├── Docker Engine                  │
│      ├── PostgreSQL 16 (16GB RAM)       │
│      ├── Qdrant (8GB RAM)               │
│      ├── Redis (4GB RAM)                │
│      ├── MinIO (8GB RAM)                │
│      ├── FastAPI Backend (16GB RAM)     │
│      ├── Next.js Frontend (8GB RAM)     │
│      └── Nginx (2GB RAM)                │
│                                         │
│  VM2: Monitoring (Ubuntu 22.04)        │
│  ├── RAM: 8 GB                          │
│  ├── CPU: 4 cores                       │
│  ├── Disk: 50 GB (NVMe)                 │
│  └── Services:                          │
│      ├── Prometheus                     │
│      ├── Grafana                        │
│      └── Loki (logs)                    │
│                                         │
│  VM3: Backup Server (Ubuntu 22.04)     │
│  ├── RAM: 4 GB                          │
│  ├── CPU: 2 cores                       │
│  ├── Disk: 5 TB (HDD)                   │
│  └── Services:                          │
│      └── Restic / Borg Backup           │
│                                         │
└─────────────────────────────────────────┘
```

### Docker Compose Setup

**File: `docker-compose.yml`**

```yaml
version: '3.8'

services:
  postgres:
    image: pgvector/pgvector:pg16
    restart: always
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      resources:
        limits:
          memory: 16G

  qdrant:
    image: qdrant/qdrant:latest
    restart: always
    volumes:
      - qdrant_data:/qdrant/storage
    networks:
      - backend
    deploy:
      resources:
        limits:
          memory: 8G

  redis:
    image: redis:7-alpine
    restart: always
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    networks:
      - backend
    deploy:
      resources:
        limits:
          memory: 4G

  minio:
    image: minio/minio:latest
    restart: always
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
    volumes:
      - minio_data:/data
    networks:
      - backend
    deploy:
      resources:
        limits:
          memory: 8G

  api:
    image: ragplatform/api:latest
    restart: always
    depends_on:
      - postgres
      - redis
      - qdrant
      - minio
    environment:
      DATABASE_URL: ${DATABASE_URL}
      REDIS_URL: ${REDIS_URL}
      QDRANT_URL: ${QDRANT_URL}
      GEMINI_API_KEY: ${GEMINI_API_KEY}
    volumes:
      - ./logs:/app/logs
    networks:
      - backend
      - frontend
    deploy:
      resources:
        limits:
          memory: 16G

  web:
    image: ragplatform/web:latest
    restart: always
    environment:
      NEXT_PUBLIC_API_URL: ${API_URL}
    networks:
      - frontend
    deploy:
      resources:
        limits:
          memory: 8G

  worker:
    image: ragplatform/api:latest
    restart: always
    command: celery -A app.worker worker -l info
    depends_on:
      - postgres
      - redis
    environment:
      DATABASE_URL: ${DATABASE_URL}
      REDIS_URL: ${REDIS_URL}
    networks:
      - backend
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 8G

  nginx:
    image: nginx:alpine
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    networks:
      - frontend
    deploy:
      resources:
        limits:
          memory: 2G

networks:
  backend:
  frontend:

volumes:
  postgres_data:
  qdrant_data:
  redis_data:
  minio_data:
```

### Network Configuration

```
Internet → Router → Firewall → Nginx (VM1)
                        ↓
                   Services (VM1)
```

**Firewall Rules (UFW):**
```bash
# Allow SSH (change port if needed)
ufw allow 22/tcp

# Allow HTTP/HTTPS
ufw allow 80/tcp
ufw allow 443/tcp

# Block all other incoming
ufw default deny incoming
ufw default allow outgoing

# Enable
ufw enable
```

### SSL/TLS Setup (Let's Encrypt)

```bash
# Install Certbot
apt install certbot python3-certbot-nginx

# Get certificate
certbot --nginx -d rag.yourdomain.com

# Auto renewal (crontab)
0 0 1 * * certbot renew --quiet
```

### Estimated Cost

```
Monthly Operating Cost:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Gemini API (5-10K queries/day)    $250-500
LINE Messaging API (Pro)          $80
Electricity (200W 24/7)           $30
Internet (Business 1Gbps)         $100
Domain + SSL                      $3
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Total Monthly:                    $463-713

Hardware (One-time):
Dell R730xd: $0 (already owned)
UPS: $15,000 (optional)

Target Users: 50-200 concurrent
Uptime Target: 99% (8.76h downtime/year)
```

---

## 🌐 Phase 2: Hybrid (Year 2)

### Architecture Evolution

```
On-Premise (Keep):                Cloud (New):
├── PostgreSQL (main data)        ├── Vercel (Frontend)
├── Qdrant (vectors)              ├── Cloudflare R2 (Storage)
├── Redis (cache)                 ├── Cloud Run (Workers)
└── Customer Databases            └── Cloudflare Worker (LINE)
```

### Migration Steps

**Week 1-2: Frontend to Vercel**
```bash
# 1. Push to GitHub
git push origin main

# 2. Deploy to Vercel
vercel --prod

# 3. Update DNS
# Point www.yourdomain.com to Vercel

# 4. Test thoroughly
# 5. Switch traffic gradually (10% → 50% → 100%)
```

**Week 3-4: Storage to Cloudflare R2**
```bash
# 1. Create R2 bucket
wrangler r2 bucket create rag-documents

# 2. Migrate existing files (background job)
rclone sync minio:rag-documents r2:rag-documents

# 3. Update app to use R2
# 4. Verify all uploads work
# 5. Keep MinIO as backup for 1 month
```

**Week 5-6: Workers to Cloud Run**
```bash
# 1. Containerize worker
docker build -t gcr.io/project/worker .

# 2. Deploy to Cloud Run
gcloud run deploy worker \
  --image gcr.io/project/worker \
  --platform managed \
  --region asia-southeast1

# 3. Update Redis queue to point to Cloud Run
# 4. Monitor performance
```

**Week 7-8: LINE Webhook to Cloudflare Worker**
```bash
# 1. Deploy worker
wrangler publish

# 2. Update LINE webhook URL
# 3. Test all LINE features
# 4. Monitor error rates
```

### Updated Architecture

```
                    ┌────────────────┐
                    │   Cloudflare   │
                    │   (CDN + WAF)  │
                    └────────┬───────┘
                             │
              ┌──────────────┼──────────────┐
              ↓              ↓              ↓
        ┌──────────┐  ┌──────────┐  ┌──────────┐
        │  Vercel  │  │  Worker  │  │    R2    │
        │(Frontend)│  │  (LINE)  │  │(Storage) │
        └────┬─────┘  └────┬─────┘  └──────────┘
             │             │
             └──────┬──────┘
                    ↓
            ┌───────────────┐
            │  Cloud Run    │
            │  (Workers)    │
            └───────┬───────┘
                    │
                    ↓
            ┌───────────────┐
            │  On-Premise   │
            │  Dell R730xd  │
            ├───────────────┤
            │  PostgreSQL   │
            │  Qdrant       │
            │  Redis        │
            │  FastAPI      │
            └───────────────┘
```

### Estimated Cost

```
Monthly Cost:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
On-Premise:
  Electricity + Internet           $130
  
Cloud:
  Vercel (Pro)                     $20
  Cloudflare R2 (20TB)             $300
  Cloud Run (Workers)              $300-500
  Cloudflare Worker (LINE)         $5
  Gemini API                       $500-1,000
  LINE API                         $80
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Total Monthly:                     $1,335-2,035

Target Users: 500-1,000 concurrent
Uptime Target: 99.5%
```

---

## ☁️ Phase 3: Full Cloud (Year 3+)

### Kubernetes Deployment

**Platform: GKE (Google Kubernetes Engine)**

```
Cluster Configuration:
├── Region: asia-southeast1
├── Nodes: 6 (n2-standard-8)
├── Auto-scaling: 3-12 nodes
└── Node pools:
    ├── App pool (4-8 nodes)
    ├── Worker pool (2-4 nodes)
    └── System pool (2 nodes)
```

### Services Architecture

```yaml
# Namespaces
- platform (main app)
- data (databases)
- monitoring (observability)
- ingress (load balancer)

# Deployments
platform:
  - api (3+ replicas)
  - web (2+ replicas)
  - worker (5+ replicas)

data:
  - postgresql (StatefulSet, 3 replicas)
  - qdrant (StatefulSet, 3 replicas)
  - redis (StatefulSet, 3 replicas - sentinel)
```

### Infrastructure as Code

**Terraform Structure:**
```
terraform/
├── main.tf
├── variables.tf
├── outputs.tf
├── modules/
│   ├── gke/
│   ├── database/
│   ├── storage/
│   ├── networking/
│   └── monitoring/
└── environments/
    ├── staging/
    └── production/
```

### CI/CD Pipeline

```yaml
# GitHub Actions
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - Checkout code
      - Build Docker images
      - Push to GCR
      - Run tests
      - Deploy to GKE (rolling update)
      - Health checks
      - Rollback if failed
```

### Estimated Cost

```
Monthly Cost (Production):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
GKE Cluster                          $1,500-2,000
Cloud SQL (PostgreSQL, HA)           $500-800
Qdrant Cloud (Clustered)             $400-800
Cloud Storage (40TB)                 $600-1,000
CDN (Cloudflare)                     $200-400
Gemini API (High volume)             $1,500-3,000
LINE API                             $80
Load Balancer                        $50
Monitoring (Datadog/NewRelic)        $100-200
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Total Monthly:                       $4,930-8,330

Target Users: 1,000-10,000 concurrent
Uptime Target: 99.9% (SLA)
```

---

## 🔄 Migration Strategy

### Zero-Downtime Migration

```
Phase 1 → Phase 2:
1. Deploy cloud services (parallel)
2. Sync data (continuous)
3. Route 10% traffic to cloud
4. Monitor for 1 week
5. Increase to 50%
6. Monitor for 1 week
7. Switch to 100%
8. Keep on-premise as hot backup (1 month)
9. Decommission old setup

Phase 2 → Phase 3:
1. Set up Kubernetes cluster
2. Deploy all services
3. Import data to Cloud SQL
4. Blue-green deployment
5. Switch DNS
6. Monitor
7. Decommission hybrid setup
```

### Data Migration Tools

```bash
# PostgreSQL migration
pg_dump -h old-host -U user dbname | \
  psql -h new-host -U user dbname

# Qdrant migration
qdrant-cli export old-cluster
qdrant-cli import new-cluster

# MinIO → R2 migration
rclone sync minio:bucket r2:bucket \
  --transfers 32 --checkers 16
```

---

## 📊 Monitoring & Maintenance

### Monitoring Stack

**Phase 1 (Self-hosted):**
```
Prometheus + Grafana + Loki
- Metrics collection
- Dashboards
- Log aggregation
- Alerts (Email, LINE)
```

**Phase 3 (Cloud):**
```
Google Cloud Monitoring + Datadog
- APM
- Distributed tracing
- Error tracking (Sentry)
- Uptime monitoring (UptimeRobot)
```

### Key Metrics

```yaml
Performance:
  - API response time (p50, p95, p99)
  - Database query time
  - Vector search latency
  - Queue processing time

Availability:
  - Service uptime
  - Error rate
  - Success rate

Business:
  - Active users
  - Queries per day
  - Documents processed
  - Revenue tracking
```

### Maintenance Schedule

```
Daily:
- Check error logs
- Monitor disk space
- Review alerts

Weekly:
- Review performance metrics
- Check backup integrity
- Update dependencies

Monthly:
- Security patches
- Database optimization
- Cost review
- Capacity planning

Quarterly:
- Disaster recovery drill
- Architecture review
- Performance audit
```

---

## 💾 Backup & Recovery

### Backup Strategy

**3-2-1 Rule:**
- **3** copies of data
- **2** different media
- **1** off-site

```
Daily Backups:
├── PostgreSQL (pg_dump)
│   ├── Full backup: 02:00 AM
│   ├── Retention: 7 days
│   └── Size: ~50 GB
│
├── Documents (MinIO/R2)
│   ├── Incremental sync
│   ├── Retention: 30 days
│   └── Size: ~10 TB
│
└── Qdrant (snapshots)
    ├── Daily: 03:00 AM
    ├── Retention: 7 days
    └── Size: ~200 GB

Weekly Backups:
└── Full system backup
    ├── Every Sunday 01:00 AM
    ├── Retention: 4 weeks
    └── Storage: Off-site (Backblaze B2)
```

### Backup Script

```bash
#!/bin/bash
# backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups"

# PostgreSQL
pg_dump -h localhost -U user ragdb | \
  gzip > $BACKUP_DIR/postgres_$DATE.sql.gz

# Qdrant
curl -X POST http://qdrant:6333/snapshots

# MinIO (sync to B2)
rclone sync minio:rag-documents b2:rag-backups

# Cleanup old backups (keep 7 days)
find $BACKUP_DIR -name "postgres_*.sql.gz" -mtime +7 -delete

# Verify backup
gunzip -t $BACKUP_DIR/postgres_$DATE.sql.gz
```

### Recovery Procedures

**RTO (Recovery Time Objective): 1 hour**
**RPO (Recovery Point Objective): 15 minutes**

```
Disaster Scenarios:

1. Database Corruption:
   - Restore from last backup
   - Apply WAL logs (PITR)
   - Time: 15-30 minutes

2. Full Server Failure:
   - Provision new VM
   - Restore from backup
   - Update DNS
   - Time: 30-60 minutes

3. Data Center Outage:
   - Failover to cloud (Phase 2+)
   - Restore from off-site backup
   - Time: 1-2 hours
```

---

## 🔒 Security Hardening

### Server Hardening

```bash
# Disable root login
sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

# Change SSH port
sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config

# Install fail2ban
apt install fail2ban
systemctl enable fail2ban

# Enable automatic security updates
apt install unattended-upgrades
dpkg-reconfigure -plow unattended-upgrades

# Install ClamAV (antivirus)
apt install clamav clamav-daemon
freshclam
systemctl enable clamav-daemon
```

### Application Security

```yaml
Headers:
  - Strict-Transport-Security: max-age=31536000
  - X-Frame-Options: SAMEORIGIN
  - X-Content-Type-Options: nosniff
  - X-XSS-Protection: 1; mode=block

Rate Limiting:
  - Login: 5 attempts / 15 minutes
  - API: 100 requests / minute
  - Upload: 10 files / hour

Encryption:
  - Database: AES-256
  - Transit: TLS 1.3
  - API Keys: Encrypted at rest
```

### Security Checklist

```
✓ SSL/TLS certificates up to date
✓ Firewall properly configured
✓ Services running as non-root
✓ Regular security updates
✓ Intrusion detection (Fail2ban)
✓ Log monitoring
✓ Backup encryption
✓ Strong password policy
✓ 2FA for admin accounts
✓ API rate limiting
✓ CORS properly configured
✓ SQL injection prevention
✓ XSS protection
✓ CSRF tokens
✓ Input validation
```

---

**Last Updated:** 2025-10-19  
**Version:** 1.0.0