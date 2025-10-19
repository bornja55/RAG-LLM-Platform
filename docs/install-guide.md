# 📦 Installation Guide

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Quick Install (Recommended)](#quick-install-recommended)
- [Manual Installation](#manual-installation)
- [Development Setup](#development-setup)
- [Production Deployment](#production-deployment)
- [Post-Installation](#post-installation)
- [Troubleshooting](#troubleshooting)

---

## ✅ Prerequisites

### Hardware Requirements

#### Minimum (Development/Testing)
```
CPU:     4 cores
RAM:     16 GB
Storage: 100 GB SSD
Network: 100 Mbps
```

#### Recommended (Production)
```
CPU:     16+ cores (Dell R730xd: Dual Xeon E5-2640 v4)
RAM:     64-128 GB
Storage: 500 GB NVMe SSD + 10 TB HDD (for documents)
Network: 1 Gbps+ (dedicated line)
```

### Software Requirements

```bash
# Required
- Docker 24.0+
- Docker Compose 2.20+
- Git 2.40+

# Operating System
- Ubuntu 22.04 LTS (recommended)
- Debian 12
- RHEL 9 / Rocky Linux 9
- macOS 13+ (for development only)

# Optional
- Nginx (for reverse proxy)
- Certbot (for SSL certificates)
```

### External Services

```bash
# Required API Keys
- Gemini API Key (Google AI Studio)
- OpenAI API Key (optional, for fallback)
- LINE Channel Access Token
- LINE Channel Secret

# Optional
- Anthropic API Key (for Claude)
- Cloudflare account (for CDN & Workers)
```

---

## 🚀 Quick Install (Recommended)

### One-Command Installation

```bash
curl -fsSL https://install.rag-platform.com | bash
```

**หรือ:**

```bash
wget -qO- https://install.rag-platform.com | bash
```

### Installation Wizard จะถาม:

```
1. Domain name (e.g., rag.yourdomain.com)
2. Admin email
3. Gemini API key
4. LINE Channel credentials
5. Database password
6. SSL certificate (Let's Encrypt หรือ custom)
```

### ระยะเวลาติดตั้ง

```
- Download images: 5-10 นาที
- Database initialization: 2-3 นาที
- SSL certificate: 1-2 นาที
- Total: ~15-20 นาที
```

---

## 🔧 Manual Installation

### Step 1: Install Docker

#### Ubuntu/Debian

```bash
# Update package index
sudo apt update
sudo apt upgrade -y

# Install dependencies
sudo apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Set up repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Verify installation
docker --version
docker compose version
```

#### RHEL/Rocky Linux

```bash
# Install Docker
sudo dnf install -y dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

---

### Step 2: Clone Repository

```bash
# Clone the repository
git clone https://github.com/your-org/rag-platform.git
cd rag-platform

# Checkout latest stable version
git checkout v1.0.0
```

---

### Step 3: Environment Configuration

```bash
# Copy environment template
cp .env.example .env

# Edit configuration
nano .env
```

#### Minimum Required Configuration

```bash
# ============================================
# General Settings
# ============================================
DOMAIN=rag.yourdomain.com
ENVIRONMENT=production
SECRET_KEY=  # Will be generated in Step 4

# ============================================
# Database
# ============================================
POSTGRES_USER=raguser
POSTGRES_PASSWORD=  # Will be generated in Step 4
POSTGRES_DB=ragdb
DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}

# ============================================
# Redis
# ============================================
REDIS_URL=redis://redis:6379/0
REDIS_PASSWORD=  # Will be generated in Step 4

# ============================================
# MinIO
# ============================================
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=  # Will be generated in Step 4
MINIO_ENDPOINT=minio:9000
MINIO_BUCKET=rag-documents

# ============================================
# Qdrant
# ============================================
QDRANT_URL=http://qdrant:6333
QDRANT_API_KEY=  # Will be generated in Step 4

# ============================================
# LLM Providers (REQUIRED)
# ============================================
GEMINI_API_KEY=your-gemini-api-key-here

# Optional
OPENAI_API_KEY=your-openai-api-key-here
ANTHROPIC_API_KEY=your-anthropic-api-key-here

# ============================================
# Embedding Models
# ============================================
EMBEDDING_PROVIDER=openai
EMBEDDING_MODEL=text-embedding-3-large
EMBEDDING_DIMENSION=3072

# ============================================
# LINE Official Account (REQUIRED for English Mania)
# ============================================
LINE_CHANNEL_ACCESS_TOKEN=your-line-channel-access-token
LINE_CHANNEL_SECRET=your-line-channel-secret

# ============================================
# Email (SMTP)
# ============================================
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password
SMTP_FROM=noreply@yourdomain.com

# ============================================
# Security
# ============================================
CORS_ORIGINS=https://rag.yourdomain.com
JWT_SECRET_KEY=  # Will be generated in Step 4
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60

# ============================================
# English Mania Specific
# ============================================
EM_BANK_ACCOUNT=xxx-x-xxxxx-x
EM_BANK_NAME=ธนาคารไทยพาณิชย์
EM_BANK_ACCOUNT_NAME=English Mania Co., Ltd.
EM_ADMIN_LINE_IDS=Uxxxxxx,Uyyyyyy  # LINE User IDs of admins
EM_ENABLE_OCR=true
```

---

### Step 4: Generate Secrets

```bash
# Run secret generation script
./scripts/generate-secrets.sh

# This will automatically update .env with:
# - SECRET_KEY
# - POSTGRES_PASSWORD
# - REDIS_PASSWORD
# - MINIO_ROOT_PASSWORD
# - QDRANT_API_KEY
# - JWT_SECRET_KEY
```

**หรือสร้างเอง:**

```bash
# Generate random secure secrets
openssl rand -base64 32  # SECRET_KEY
openssl rand -base64 32  # POSTGRES_PASSWORD
openssl rand -base64 32  # REDIS_PASSWORD
openssl rand -base64 32  # MINIO_ROOT_PASSWORD
openssl rand -base64 32  # QDRANT_API_KEY
openssl rand -base64 64  # JWT_SECRET_KEY
```

---

### Step 5: Get API Keys

#### 1. Gemini API Key

```bash
# 1. ไปที่ Google AI Studio
https://aistudio.google.com/

# 2. คลิก "Get API Key"
# 3. Create new API key
# 4. Copy API key
# 5. Paste ลงใน .env:
GEMINI_API_KEY=AIza...
```

#### 2. OpenAI API Key (Optional)

```bash
# 1. ไปที่ OpenAI Platform
https://platform.openai.com/

# 2. คลิก "API keys"
# 3. Create new secret key
# 4. Copy API key
# 5. Paste ลงใน .env:
OPENAI_API_KEY=sk-...
```

#### 3. LINE Official Account

```bash
# 1. ไปที่ LINE Developers Console
https://developers.line.biz/console/

# 2. Create new Provider (ถ้ายังไม่มี)
# 3. Create new Messaging API channel

# 4. ใน "Messaging API" tab:
#    - Copy "Channel access token" → LINE_CHANNEL_ACCESS_TOKEN
#    - Copy "Channel secret" → LINE_CHANNEL_SECRET

# 5. ตั้งค่า Webhook URL (หลัง deploy):
#    https://your-domain.com/api/v1/webhooks/line

# 6. Enable "Use webhooks"
# 7. Disable "Auto-reply messages"
```

---

### Step 6: Start Services

```bash
# Pull Docker images
docker compose pull

# Start all services
docker compose up -d

# View logs
docker compose logs -f

# Check service status
docker compose ps
```

**Services ที่จะ start:**

```
✓ postgres   - PostgreSQL 16
✓ qdrant     - Qdrant vector database
✓ redis      - Redis cache
✓ minio      - MinIO object storage
✓ api        - FastAPI backend
✓ web        - Next.js frontend
✓ worker     - Celery worker
✓ nginx      - Reverse proxy
```

---

### Step 7: Initialize Database

```bash
# Run database migrations
docker compose exec api alembic upgrade head

# Seed initial data
docker compose exec api python scripts/seed.py

# Output:
# ✓ Created default organization
# ✓ Created default workspace
# ✓ Created admin user
```

---

### Step 8: Create Admin User

```bash
# Interactive admin creation
docker compose exec api python scripts/create-admin.py

# หรือใช้ command line arguments
docker compose exec api python scripts/create-admin.py \
  --email admin@yourdomain.com \
  --password "YourSecurePassword123!" \
  --name "Admin User"

# Output:
# ✓ Admin user created successfully!
# 
# Login credentials:
# Email: admin@yourdomain.com
# Password: YourSecurePassword123!
# 
# Access the platform at:
# https://rag.yourdomain.com
```

---

### Step 9: SSL Certificate Setup

#### Option A: Let's Encrypt (Recommended)

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx -y

# Get certificate
sudo certbot --nginx -d rag.yourdomain.com

# Auto-renewal (crontab)
sudo crontab -e

# Add this line:
0 12 * * * /usr/bin/certbot renew --quiet
```

#### Option B: Custom Certificate

```bash
# Copy your certificates
sudo mkdir -p /etc/nginx/ssl
sudo cp your-cert.crt /etc/nginx/ssl/
sudo cp your-key.key /etc/nginx/ssl/

# Update nginx.conf
server {
    listen 443 ssl;
    server_name rag.yourdomain.com;
    
    ssl_certificate /etc/nginx/ssl/your-cert.crt;
    ssl_certificate_key /etc/nginx/ssl/your-key.key;
    
    # ... rest of config
}

# Reload Nginx
docker compose restart nginx
```

---

### Step 10: Verify Installation

```bash
# Check all services are running
docker compose ps

# Expected output:
# NAME           STATUS
# postgres       Up (healthy)
# qdrant         Up
# redis          Up
# minio          Up
# api            Up (healthy)
# web            Up
# worker         Up
# nginx          Up

# Check health endpoint
curl https://rag.yourdomain.com/health

# Expected response:
{
  "status": "healthy",
  "version": "1.0.0",
  "services": {
    "database": "ok",
    "redis": "ok",
    "qdrant": "ok",
    "minio": "ok"
  }
}

# Test API
curl https://rag.yourdomain.com/api/v1/health

# Test LINE webhook (from LINE platform)
# Send "สวัสดี" to your LINE OA
# Should receive reply from bot
```

---

## 💻 Development Setup

### For Backend Development

```bash
# Clone repository
git clone https://github.com/your-org/rag-platform.git
cd rag-platform

# Create Python virtual environment
cd backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements-dev.txt

# Copy environment file
cp .env.example .env
# Edit .env with your settings

# Run database migrations
alembic upgrade head

# Start development server
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# API will be available at:
# http://localhost:8000
# API docs: http://localhost:8000/docs
```

### For Frontend Development

```bash
# Navigate to frontend directory
cd frontend

# Install dependencies
npm install
# or
pnpm install

# Copy environment file
cp .env.example .env.local
# Edit .env.local:
NEXT_PUBLIC_API_URL=http://localhost:8000

# Start development server
npm run dev
# or
pnpm dev

# Frontend will be available at:
# http://localhost:3000
```

### Using Docker Compose for Development

```bash
# Use development compose file
docker compose -f docker-compose.dev.yml up

# This will:
# - Mount source code as volumes (hot reload)
# - Enable debug mode
# - Expose additional ports
# - Use development environment variables
```

---

## 🚀 Production Deployment

### Pre-deployment Checklist

```bash
✓ Domain name configured (A record pointing to server)
✓ SSL certificate ready (Let's Encrypt or custom)
✓ All API keys obtained
✓ Firewall configured
  - Allow: 80 (HTTP)
  - Allow: 443 (HTTPS)
  - Allow: 22 (SSH)
  - Block: All other ports
✓ Database backup strategy
✓ Monitoring setup (optional but recommended)
```

### Firewall Configuration (UFW)

```bash
# Install UFW
sudo apt install ufw -y

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (change 22 if using different port)
sudo ufw allow 22/tcp

# Allow HTTP & HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status
```

### Nginx Configuration

```bash
# Create nginx config
sudo nano /etc/nginx/sites-available/rag-platform

# Add configuration:
```

```nginx
# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name rag.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

# HTTPS server
server {
    listen 443 ssl http2;
    server_name rag.yourdomain.com;

    # SSL certificates
    ssl_certificate /etc/letsencrypt/live/rag.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/rag.yourdomain.com/privkey.pem;

    # SSL configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Client max body size (for file uploads)
    client_max_body_size 100M;

    # Frontend
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Backend API
    location /api {
        proxy_pass http://localhost:8000;
        proxy_http_version 1.1;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;
        
        # Timeouts for long-running queries
        proxy_read_timeout 300s;
        proxy_connect_timeout 75s;
    }

    # WebSocket support (for streaming)
    location /ws {
        proxy_pass http://localhost:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Static files (if served by nginx)
    location /static {
        alias /var/www/rag-platform/static;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/rag-platform /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Reload nginx
sudo systemctl reload nginx
```

### Systemd Service (Auto-start on boot)

```bash
# Create systemd service
sudo nano /etc/systemd/system/rag-platform.service
```

```ini
[Unit]
Description=RAG Platform Docker Compose
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/opt/rag-platform
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```

```bash
# Enable service
sudo systemctl enable rag-platform

# Start service
sudo systemctl start rag-platform

# Check status
sudo systemctl status rag-platform
```

---

## 🔍 Post-Installation

### 1. Access the Platform

```bash
# Web interface
https://rag.yourdomain.com

# Login with admin credentials
Email: admin@yourdomain.com
Password: (created in Step 8)
```

### 2. Create First Organization

```bash
1. Login as admin
2. Go to "Organizations"
3. Click "Create Organization"
4. Fill in details:
   - Name: Your Company Name
   - Slug: your-company (URL-friendly)
   - Domain: yourcompany.com
5. Click "Create"
```

### 3. Create First Workspace

```bash
1. Select organization
2. Go to "Workspaces"
3. Click "Create Workspace"
4. Fill in details:
   - Name: Sales Department
   - Description: Sales team workspace
5. Click "Create"
```

### 4. Upload Documents

```bash
1. Select workspace
2. Go to "Documents"
3. Click "Upload"
4. Drag & drop files or browse
5. Wait for processing (shows progress)
6. Documents will be available for search
```

### 5. Connect Database

```bash
1. Go to "Database Connections"
2. Click "Add Connection"
3. Fill in details:
   - Name: Production DB
   - Type: PostgreSQL
   - Host: db.yourcompany.com
   - Port: 5432
   - Database: sales_db
   - Username: readonly_user
   - Password: ********
   - Read-only: ✓ (recommended)
4. Click "Test Connection"
5. If successful, click "Save"
```

### 6. Configure LINE OA

```bash
1. Go to LINE Developers Console
2. Navigate to your channel
3. Set Webhook URL:
   https://rag.yourdomain.com/api/v1/webhooks/line

4. Verify webhook:
   - Click "Verify"
   - Should see "Success"

5. Enable webhook
6. Test by sending message to your LINE OA
```

### 7. Invite Users

```bash
1. Go to "Users"
2. Click "Invite User"
3. Fill in:
   - Email
   - Role (Admin/Manager/Member)
   - Workspaces (select which workspaces to access)
4. Click "Send Invitation"
5. User will receive email with setup link
```

---

## 🐛 Troubleshooting

### Common Issues

#### 1. Services Won't Start

```bash
# Check Docker daemon
sudo systemctl status docker

# Check logs
docker compose logs -f

# Restart services
docker compose restart

# Rebuild if needed
docker compose up -d --build
```

#### 2. Database Connection Failed

```bash
# Check PostgreSQL is running
docker compose ps postgres

# Check database logs
docker compose logs postgres

# Test connection
docker compose exec postgres psql -U raguser -d ragdb

# If password error, check .env file
```

#### 3. Cannot Access Website

```bash
# Check nginx status
sudo systemctl status nginx

# Check nginx logs
sudo tail -f /var/log/nginx/error.log

# Test nginx config
sudo nginx -t

# Check if ports are open
sudo netstat -tlnp | grep -E '(80|443)'
```

#### 4. LINE Bot Not Responding

```bash
# Check webhook URL is correct
# LINE Console > Messaging API > Webhook URL

# Verify webhook
# Click "Verify" button - should see "Success"

# Check API logs
docker compose logs api | grep "LINE"

# Test webhook manually
curl -X POST https://rag.yourdomain.com/api/v1/webhooks/line \
  -H "Content-Type: application/json" \
  -d '{"events":[{"type":"message","message":{"type":"text","text":"test"}}]}'
```

#### 5. Slow Query Performance

```bash
# Check Qdrant is running
docker compose ps qdrant

# Check Qdrant logs
docker compose logs qdrant

# Restart Qdrant
docker compose restart qdrant

# Check database indexes
docker compose exec postgres psql -U raguser -d ragdb
\d+ documents  # Check indexes
```

#### 6. Out of Disk Space

```bash
# Check disk usage
df -h

# Clean Docker images
docker system prune -a

# Clean old logs
docker compose exec api find /var/log -name "*.log" -mtime +30 -delete

# Check MinIO storage
docker compose exec minio mc du local/rag-documents
```

---

## 📚 Additional Resources

### Documentation
- [Architecture Guide](ARCHITECTURE.md)
- [Development Tasks](DEV_TASKS.md)
- [Deployment Guide](DEPLOYMENT.md)
- [API Documentation](https://rag.yourdomain.com/docs)

### Support
- 📧 Email: support@rag-platform.com
- 💬 Discord: https://discord.gg/rag-platform
- 🐛 Issues: https://github.com/your-org/rag-platform/issues

### Updates
```bash
# Pull latest code
cd /opt/rag-platform
git pull origin main

# Pull latest images
docker compose pull

# Restart services
docker compose up -d

# Run migrations (if any)
docker compose exec api alembic upgrade head
```

---

**Installation successful! 🎉**

Next steps:
1. ✅ Login to admin panel
2. ✅ Create organization & workspace
3. ✅ Upload documents or connect database
4. ✅ Start using your AI Assistant!

---

**Last Updated:** 2025-10-19  
**Version:** 1.0.0