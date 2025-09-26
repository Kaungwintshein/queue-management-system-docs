# Queue Management System - Deployment Guide

## 📋 Table of Contents

1. [Local Development Setup](#local-development-setup)
2. [Production Deployment](#production-deployment)
3. [CI/CD Pipeline](#cicd-pipeline)
4. [Environment Variables](#environment-variables)
5. [Monitoring & Maintenance](#monitoring--maintenance)

---

## 🚀 Local Development Setup

### Prerequisites

- Node.js 18+ and npm
- PostgreSQL 14+
- Redis (optional, for caching)
- Git

### 1. Clone and Setup

```bash
# Clone repository
git clone <your-repo-url>
cd queue-management-system

# Install dependencies for both API and UI
cd api && npm install
cd ../ui && npm install
cd ..
```

### 2. Database Setup

```bash
# Start PostgreSQL (macOS with Homebrew)
brew services start postgresql

# Create database
createdb queue_management_system

# Or using psql
psql -c "CREATE DATABASE queue_management_system;"
```

### 3. API Setup

```bash
cd api

# Copy environment file
cp env.example .env

# Edit .env with your local database credentials
# DATABASE_URL="postgresql://username:password@localhost:5432/queue_management_system"

# Generate Prisma client
npx prisma generate

# Run database migrations
npx prisma db push

# Seed database with initial data
npx prisma db seed

# Start API server
npm run dev
```

The API will be available at `http://localhost:3001`

### 4. UI Setup

```bash
cd ui

# Create environment file
cat > .env.local << EOF
NEXT_PUBLIC_API_URL=http://localhost:3001
NEXT_PUBLIC_WS_URL=http://localhost:3001
NEXT_PUBLIC_ENV=development
EOF

# Start development server
npm run dev
```

The UI will be available at `http://localhost:3000`

### 5. Development Workflow

```bash
# Terminal 1: API
cd api && npm run dev

# Terminal 2: UI
cd ui && npm run dev

# Terminal 3: Database tools (optional)
cd api && npx prisma studio
```

---

## 🌐 Production Deployment

### Architecture Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Load Balancer │    │   Reverse Proxy │    │   Application   │
│    (Cloudflare) │────│     (Nginx)     │────│   Containers    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                        │
                                               ┌─────────────────┐
                                               │    Database     │
                                               │  (PostgreSQL)   │
                                               └─────────────────┘
                                                        │
                                               ┌─────────────────┐
                                               │     Cache       │
                                               │    (Redis)      │
                                               └─────────────────┘
```

### Option 1: VPS/Cloud Server Deployment

#### Server Requirements

- **Minimum**: 2 CPU cores, 4GB RAM, 40GB SSD
- **Recommended**: 4 CPU cores, 8GB RAM, 80GB SSD
- Ubuntu 20.04+ or CentOS 8+

#### 1. Server Setup

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js 18
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PostgreSQL
sudo apt install postgresql postgresql-contrib -y

# Install Redis
sudo apt install redis-server -y

# Install Nginx
sudo apt install nginx -y

# Install PM2 for process management
sudo npm install -g pm2
```

#### 2. Database Setup

```bash
# Configure PostgreSQL
sudo -u postgres createuser --interactive --pwprompt queueuser
sudo -u postgres createdb -O queueuser queue_management_system

# Secure PostgreSQL
sudo -u postgres psql -c "ALTER USER queueuser PASSWORD 'your-secure-password';"
```

#### 3. Application Deployment

```bash
# Create application directory
sudo mkdir -p /var/www/queue-management
sudo chown $USER:$USER /var/www/queue-management
cd /var/www/queue-management

# Clone and build
git clone <your-repo-url> .
cd api && npm ci --production
cd ../ui && npm ci && npm run build

# Setup environment
cp api/env.example api/.env
# Edit api/.env with production values

# Generate Prisma client and run migrations
cd api
npx prisma generate
npx prisma db deploy
```

#### 4. Process Management with PM2

```bash
# Create PM2 ecosystem file
cat > ecosystem.config.js << EOF
module.exports = {
  apps: [
    {
      name: 'queue-api',
      script: './api/dist/app.js',
      cwd: '/var/www/queue-management',
      instances: 2,
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
        PORT: 3001
      }
    },
    {
      name: 'queue-ui',
      script: 'npm',
      args: 'start',
      cwd: '/var/www/queue-management/ui',
      instances: 1,
      env: {
        NODE_ENV: 'production',
        PORT: 3000
      }
    }
  ]
};
EOF

# Start applications
pm2 start ecosystem.config.js
pm2 save
pm2 startup
```

#### 5. Nginx Configuration

```bash
# Create Nginx configuration
sudo tee /etc/nginx/sites-available/queue-management << EOF
upstream api_backend {
    server 127.0.0.1:3001;
}

upstream ui_backend {
    server 127.0.0.1:3000;
}

server {
    listen 80;
    server_name your-domain.com www.your-domain.com;

    # API routes
    location /api {
        proxy_pass http://api_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;
        proxy_cache_bypass \$http_upgrade;
    }

    # WebSocket routes
    location /socket.io {
        proxy_pass http://api_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;
    }

    # UI routes
    location / {
        proxy_pass http://ui_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;
        proxy_cache_bypass \$http_upgrade;
    }
}
EOF

# Enable site
sudo ln -s /etc/nginx/sites-available/queue-management /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

#### 6. SSL with Let's Encrypt

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx -y

# Get SSL certificate
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Auto-renewal
sudo crontab -e
# Add: 0 12 * * * /usr/bin/certbot renew --quiet
```

### Option 2: Platform as a Service (PaaS)

#### Vercel (UI) + Railway/Render (API)

**UI Deployment on Vercel:**

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy from UI directory
cd ui
vercel --prod
```

**API Deployment on Railway:**

1. Connect GitHub repository to Railway
2. Set environment variables in Railway dashboard
3. Railway will auto-deploy on git push

---

## 🔄 CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy Queue Management System

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"
          cache: "npm"
          cache-dependency-path: |
            api/package-lock.json
            ui/package-lock.json

      - name: Install API dependencies
        run: cd api && npm ci

      - name: Install UI dependencies
        run: cd ui && npm ci

      - name: Run API tests
        run: cd api && npm test
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db

      - name: Run UI tests
        run: cd ui && npm test

      - name: Build API
        run: cd api && npm run build

      - name: Build UI
        run: cd ui && npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v3

      - name: Deploy to production
        uses: appleboy/ssh-action@v0.1.5
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.PRIVATE_KEY }}
          script: |
            cd /var/www/queue-management
            git pull origin main
            cd api && npm ci --production
            cd ../ui && npm ci && npm run build
            pm2 restart all
```

---

## 🔧 Environment Variables

### Development (.env.local)

#### API (.env)

```bash
# Database
DATABASE_URL="postgresql://username:password@localhost:5432/queue_management_system"

# JWT
JWT_SECRET="dev-secret-key-change-in-production"
JWT_EXPIRES_IN="24h"
JWT_REFRESH_EXPIRES_IN="7d"

# Server
NODE_ENV="development"
PORT=3001
FRONTEND_URL="http://localhost:3000"

# Optional
REDIS_URL="redis://localhost:6379"
LOG_LEVEL="debug"
```

#### UI (.env.local)

```bash
NEXT_PUBLIC_API_URL=http://localhost:3001
NEXT_PUBLIC_WS_URL=http://localhost:3001
NEXT_PUBLIC_ENV=development
```

### Production

#### API (.env)

```bash
# Database
DATABASE_URL="postgresql://user:password@db-host:5432/queue_management_system"

# JWT
JWT_SECRET="super-secure-jwt-secret-min-32-chars"
JWT_EXPIRES_IN="24h"
JWT_REFRESH_EXPIRES_IN="7d"

# Server
NODE_ENV="production"
PORT=3001
FRONTEND_URL="https://yourdomain.com"

# Security
CORS_ORIGIN="https://yourdomain.com"
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100

# Performance
REDIS_URL="redis://redis-host:6379"
LOG_LEVEL="info"

# File uploads
UPLOAD_MAX_SIZE="10mb"
UPLOAD_DEST="/var/uploads"
```

#### UI (.env.production)

```bash
NEXT_PUBLIC_API_URL=https://api.yourdomain.com
NEXT_PUBLIC_WS_URL=https://api.yourdomain.com
NEXT_PUBLIC_ENV=production
```

---

## 📊 Monitoring & Maintenance

### Health Checks

#### API Health Endpoint

```javascript
// Add to api/src/routes/health.ts
app.get("/health", (req, res) => {
  res.json({
    status: "healthy",
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    version: process.env.npm_package_version,
  });
});
```

### Monitoring Setup

```bash
# Install monitoring tools
npm install -g pm2-logrotate
pm2 install pm2-logrotate

# Setup log rotation
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
pm2 set pm2-logrotate:compress true
```

### Database Maintenance

```bash
# Database backup script
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
pg_dump queue_management_system > "backup_${DATE}.sql"
gzip "backup_${DATE}.sql"

# Clean old backups (keep 7 days)
find /backups -name "backup_*.sql.gz" -mtime +7 -delete
```

### Security Updates

```bash
# Regular updates
sudo apt update && sudo apt upgrade -y
npm audit fix
npm update

# SSL certificate renewal check
sudo certbot renew --dry-run
```

---

## 🚨 Troubleshooting

### Common Issues

1. **Port conflicts**: Change ports in .env files
2. **Database connection**: Check DATABASE_URL and PostgreSQL service
3. **WebSocket issues**: Verify proxy configuration for Socket.io
4. **Build failures**: Clear node_modules and reinstall
5. **Permission errors**: Check file ownership and nginx user permissions

### Logs

```bash
# Application logs
pm2 logs queue-api
pm2 logs queue-ui

# System logs
sudo journalctl -u nginx -f
sudo tail -f /var/log/postgresql/postgresql-*.log
```

---

## 📋 Deployment Checklist

### Pre-deployment

- [ ] Environment variables configured
- [ ] Database migrations tested
- [ ] SSL certificates ready
- [ ] Backup strategy in place
- [ ] Monitoring setup
- [ ] Load testing completed

### Post-deployment

- [ ] Health checks passing
- [ ] WebSocket connections working
- [ ] Database queries performing well
- [ ] SSL certificate valid
- [ ] Monitoring alerts configured
- [ ] Backup restoration tested

---

This deployment guide covers both simple and advanced deployment scenarios. Choose the approach that best fits your infrastructure requirements and technical expertise.
