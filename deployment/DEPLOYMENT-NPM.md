# Queue Management System - NPM Deployment Guide

## 📋 Table of Contents

1. [Local Development Setup](#local-development-setup)
2. [Production Deployment with NPM](#production-deployment-with-npm)
3. [PM2 Process Management](#pm2-process-management)
4. [Cloud Platform Deployment](#cloud-platform-deployment)
5. [Environment Configuration](#environment-configuration)
6. [Monitoring & Maintenance](#monitoring--maintenance)

---

## 🚀 Local Development Setup

### Prerequisites

- Node.js 18+ and npm
- PostgreSQL 14+
- Redis (optional, for caching)
- Git

### Quick Start

```bash
# Clone repository
git clone <your-repo-url>
cd queue-management-system

# Setup API
cd api
npm install
cp env.example .env
# Edit .env with your database credentials
npx prisma generate
npx prisma db push
npx prisma db seed
npm run dev

# In another terminal - Setup UI
cd ui
npm install
cat > .env.local << EOF
NEXT_PUBLIC_API_URL=http://localhost:3001
NEXT_PUBLIC_WS_URL=http://localhost:3001
NEXT_PUBLIC_ENV=development
EOF
npm run dev
```

**Access URLs:**

- UI: http://localhost:3000
- API: http://localhost:3001
- API Docs: http://localhost:3001/docs

---

## 🌐 Production Deployment with NPM

### Option 1: Single VPS/Cloud Server

#### 1. Server Setup

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js 18
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PostgreSQL
sudo apt install postgresql postgresql-contrib -y
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Install Redis (optional)
sudo apt install redis-server -y
sudo systemctl start redis
sudo systemctl enable redis

# Install Nginx
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx

# Install PM2 globally
sudo npm install -g pm2
```

#### 2. Database Setup

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE USER queueuser WITH PASSWORD 'your-secure-password';
CREATE DATABASE queue_management_system OWNER queueuser;
GRANT ALL PRIVILEGES ON DATABASE queue_management_system TO queueuser;
\q
EOF
```

#### 3. Application Deployment

```bash
# Create application directory
sudo mkdir -p /var/www/queue-management
sudo chown $USER:$USER /var/www/queue-management
cd /var/www/queue-management

# Clone repository
git clone <your-repo-url> .

# Setup API
cd api
npm ci --production
cp env.example .env

# Edit .env with production values
cat > .env << EOF
DATABASE_URL="postgresql://queueuser:your-secure-password@localhost:5432/queue_management_system"
JWT_SECRET="your-super-secure-jwt-secret-min-32-characters"
JWT_EXPIRES_IN="24h"
JWT_REFRESH_EXPIRES_IN="7d"
NODE_ENV="production"
PORT=3001
FRONTEND_URL="https://yourdomain.com"
REDIS_URL="redis://localhost:6379"
LOG_LEVEL="info"
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100
EOF

# Generate Prisma client and run migrations
npx prisma generate
npx prisma db deploy
npx prisma db seed

# Build the API
npm run build

# Setup UI
cd ../ui
npm ci
cat > .env.production << EOF
NEXT_PUBLIC_API_URL=https://yourdomain.com
NEXT_PUBLIC_WS_URL=https://yourdomain.com
NEXT_PUBLIC_ENV=production
EOF

# Build the UI
npm run build
```

---

## ⚡ PM2 Process Management

### 1. Create PM2 Ecosystem File

```javascript
// /var/www/queue-management/ecosystem.config.js
module.exports = {
  apps: [
    {
      name: "queue-api",
      script: "./api/dist/app.js",
      cwd: "/var/www/queue-management",
      instances: "max", // Use all CPU cores
      exec_mode: "cluster",
      env: {
        NODE_ENV: "production",
        PORT: 3001,
      },
      // Auto-restart settings
      max_memory_restart: "1G",
      restart_delay: 4000,
      max_restarts: 10,
      min_uptime: "10s",
      // Logging
      log_file: "/var/www/queue-management/logs/api.log",
      error_file: "/var/www/queue-management/logs/api-error.log",
      out_file: "/var/www/queue-management/logs/api-out.log",
      log_date_format: "YYYY-MM-DD HH:mm:ss Z",
      // Health monitoring
      monitoring: false,
    },
    {
      name: "queue-ui",
      script: "npm",
      args: "start",
      cwd: "/var/www/queue-management/ui",
      instances: 1,
      env: {
        NODE_ENV: "production",
        PORT: 3000,
      },
      // Auto-restart settings
      max_memory_restart: "512M",
      restart_delay: 4000,
      max_restarts: 10,
      min_uptime: "10s",
      // Logging
      log_file: "/var/www/queue-management/logs/ui.log",
      error_file: "/var/www/queue-management/logs/ui-error.log",
      out_file: "/var/www/queue-management/logs/ui-out.log",
      log_date_format: "YYYY-MM-DD HH:mm:ss Z",
    },
  ],
};
```

### 2. PM2 Commands

```bash
# Create logs directory
mkdir -p /var/www/queue-management/logs

# Start applications
pm2 start ecosystem.config.js --env production

# Save PM2 configuration
pm2 save

# Setup PM2 startup script
pm2 startup
# Follow the instructions displayed

# Useful PM2 commands
pm2 list                    # List all processes
pm2 logs queue-api          # View API logs
pm2 logs queue-ui           # View UI logs
pm2 restart all             # Restart all processes
pm2 stop all                # Stop all processes
pm2 delete all              # Delete all processes
pm2 monit                   # Real-time monitoring
pm2 reload all              # Zero-downtime reload
```

### 3. Nginx Configuration

```bash
# Create Nginx configuration
sudo tee /etc/nginx/sites-available/queue-management << 'EOF'
upstream api_backend {
    server 127.0.0.1:3001;
}

upstream ui_backend {
    server 127.0.0.1:3000;
}

server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    # API routes
    location /api {
        proxy_pass http://api_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 86400;
    }

    # WebSocket routes
    location /socket.io {
        proxy_pass http://api_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 86400;
    }

    # UI routes (fallback to Next.js)
    location / {
        proxy_pass http://ui_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied expired no-cache no-store private must-revalidate auth;
    gzip_types text/plain text/css text/xml text/javascript application/x-javascript application/xml+rss application/javascript;
}
EOF

# Enable the site
sudo ln -s /etc/nginx/sites-available/queue-management /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 4. SSL with Let's Encrypt

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx -y

# Get SSL certificate
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Test automatic renewal
sudo certbot renew --dry-run

# Add auto-renewal to crontab
echo "0 12 * * * /usr/bin/certbot renew --quiet" | sudo crontab -
```

---

## ☁️ Cloud Platform Deployment

### Option 1: Vercel (UI) + Railway (API)

#### Vercel UI Deployment

```bash
# Install Vercel CLI
npm i -g vercel

# Configure vercel.json in ui directory
cat > ui/vercel.json << EOF
{
  "framework": "nextjs",
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "installCommand": "npm ci",
  "env": {
    "NEXT_PUBLIC_API_URL": "https://your-api-domain.railway.app",
    "NEXT_PUBLIC_WS_URL": "https://your-api-domain.railway.app",
    "NEXT_PUBLIC_ENV": "production"
  }
}
EOF

# Deploy to Vercel
cd ui
vercel --prod
```

#### Railway API Deployment

1. Connect your GitHub repository to Railway
2. Create a new service for the API
3. Set the root directory to `api`
4. Configure environment variables:

```bash
DATABASE_URL=postgresql://user:pass@host:port/dbname
JWT_SECRET=your-secure-jwt-secret
NODE_ENV=production
PORT=3001
```

5. Railway will automatically deploy on git push

### Option 2: Netlify (UI) + Heroku (API)

#### Netlify UI Deployment

```bash
# Install Netlify CLI
npm install -g netlify-cli

# Build configuration in ui/netlify.toml
cat > ui/netlify.toml << EOF
[build]
  base = "ui/"
  command = "npm run build"
  publish = ".next"

[build.environment]
  NEXT_PUBLIC_API_URL = "https://your-app.herokuapp.com"
  NEXT_PUBLIC_WS_URL = "https://your-app.herokuapp.com"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
EOF

# Deploy
cd ui
netlify deploy --prod
```

#### Heroku API Deployment

```bash
# Install Heroku CLI
# https://devcenter.heroku.com/articles/heroku-cli

# Create Heroku app
heroku create your-queue-api

# Add PostgreSQL addon
heroku addons:create heroku-postgresql:mini

# Set environment variables
heroku config:set NODE_ENV=production
heroku config:set JWT_SECRET=your-secure-jwt-secret

# Create Procfile in api directory
echo "web: npm start" > api/Procfile

# Deploy
cd api
git init
git add .
git commit -m "Initial commit"
heroku git:remote -a your-queue-api
git push heroku main
```

### Option 3: DigitalOcean App Platform

```yaml
# .do/app.yaml
name: queue-management-system
services:
  - name: api
    source_dir: /api
    github:
      repo: your-username/queue-management-system
      branch: main
    run_command: npm start
    build_command: npm ci && npx prisma generate && npm run build
    environment_slug: node-js
    instance_count: 1
    instance_size_slug: basic-xxs
    envs:
      - key: NODE_ENV
        value: production
      - key: JWT_SECRET
        value: your-secure-jwt-secret
        type: SECRET

  - name: ui
    source_dir: /ui
    github:
      repo: your-username/queue-management-system
      branch: main
    run_command: npm start
    build_command: npm ci && npm run build
    environment_slug: node-js
    instance_count: 1
    instance_size_slug: basic-xxs
    envs:
      - key: NEXT_PUBLIC_API_URL
        value: ${api.PUBLIC_URL}

databases:
  - name: queue-db
    engine: PG
    version: "15"
```

---

## 🔧 Environment Configuration

### Development Environment

#### API (.env)

```bash
DATABASE_URL="postgresql://username:password@localhost:5432/queue_management_system"
JWT_SECRET="dev-secret-key-change-in-production"
JWT_EXPIRES_IN="24h"
JWT_REFRESH_EXPIRES_IN="7d"
NODE_ENV="development"
PORT=3001
FRONTEND_URL="http://localhost:3000"
REDIS_URL="redis://localhost:6379"
LOG_LEVEL="debug"
```

#### UI (.env.local)

```bash
NEXT_PUBLIC_API_URL=http://localhost:3001
NEXT_PUBLIC_WS_URL=http://localhost:3001
NEXT_PUBLIC_ENV=development
```

### Production Environment

#### API (.env)

```bash
DATABASE_URL="postgresql://user:password@host:5432/queue_management_system"
JWT_SECRET="super-secure-jwt-secret-minimum-32-characters-long"
JWT_EXPIRES_IN="24h"
JWT_REFRESH_EXPIRES_IN="7d"
NODE_ENV="production"
PORT=3001
FRONTEND_URL="https://yourdomain.com"
CORS_ORIGIN="https://yourdomain.com"
REDIS_URL="redis://localhost:6379"
LOG_LEVEL="info"
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100
UPLOAD_MAX_SIZE="10mb"
UPLOAD_DEST="/var/uploads"
```

#### UI (.env.production)

```bash
NEXT_PUBLIC_API_URL=https://yourdomain.com
NEXT_PUBLIC_WS_URL=https://yourdomain.com
NEXT_PUBLIC_ENV=production
```

---

## 📊 Monitoring & Maintenance

### 1. Health Monitoring

Create health endpoint in API:

```javascript
// api/src/routes/health.ts
import { Router } from "express";
import { prisma } from "../app";

const router = Router();

router.get("/health", async (req, res) => {
  try {
    // Database health check
    await prisma.$queryRaw`SELECT 1`;

    res.json({
      status: "healthy",
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      memory: process.memoryUsage(),
      version: process.env.npm_package_version,
      database: "connected",
    });
  } catch (error) {
    res.status(503).json({
      status: "unhealthy",
      timestamp: new Date().toISOString(),
      error: error.message,
    });
  }
});

export default router;
```

### 2. Log Management

```bash
# Setup log rotation with PM2
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
pm2 set pm2-logrotate:compress true
pm2 set pm2-logrotate:dateFormat YYYY-MM-DD_HH-mm-ss
```

### 3. Database Backup Script

```bash
#!/bin/bash
# backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/var/backups/queue-management"
DB_NAME="queue_management_system"

# Create backup directory
mkdir -p $BACKUP_DIR

# Create database backup
pg_dump $DB_NAME > "$BACKUP_DIR/backup_${DATE}.sql"

# Compress backup
gzip "$BACKUP_DIR/backup_${DATE}.sql"

# Remove backups older than 7 days
find $BACKUP_DIR -name "backup_*.sql.gz" -mtime +7 -delete

echo "Backup completed: backup_${DATE}.sql.gz"
```

### 4. Deployment Script

```bash
#!/bin/bash
# deploy.sh

set -e

echo "🚀 Starting deployment..."

APP_DIR="/var/www/queue-management"
BACKUP_DIR="/var/backups/queue-management"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Create database backup
echo "📦 Creating database backup..."
mkdir -p $BACKUP_DIR
pg_dump queue_management_system > "$BACKUP_DIR/db_backup_$TIMESTAMP.sql"

# Stop PM2 processes
echo "🛑 Stopping services..."
pm2 stop all

# Update code
echo "📥 Updating code..."
cd $APP_DIR
git pull origin main

# Install dependencies and build API
echo "🔨 Building API..."
cd api
npm ci --production
npx prisma generate
npx prisma db deploy
npm run build

# Install dependencies and build UI
echo "🔨 Building UI..."
cd ../ui
npm ci
npm run build

# Start PM2 processes
echo "🚀 Starting services..."
pm2 start ecosystem.config.js --env production

# Wait for services to start
sleep 10

# Health checks
echo "🔍 Running health checks..."
if curl -f http://localhost:3001/health > /dev/null 2>&1; then
    echo "✅ API health check passed"
else
    echo "❌ API health check failed"
    exit 1
fi

if curl -f http://localhost:3000 > /dev/null 2>&1; then
    echo "✅ UI health check passed"
else
    echo "❌ UI health check failed"
    exit 1
fi

echo "✅ Deployment completed successfully!"
echo "🌐 Application is running at: https://yourdomain.com"
```

### 5. Monitoring Commands

```bash
# System monitoring
top                          # CPU and memory usage
htop                         # Enhanced system monitor
df -h                        # Disk usage
free -h                      # Memory usage
netstat -tulpn              # Network connections

# Application monitoring
pm2 list                     # PM2 process list
pm2 monit                    # Real-time monitoring
pm2 logs                     # View all logs
pm2 logs queue-api           # API logs only
pm2 logs queue-ui            # UI logs only

# Database monitoring
sudo -u postgres psql -c "SELECT * FROM pg_stat_activity;"
sudo -u postgres psql -c "SELECT datname, size FROM (SELECT datname, pg_database_size(datname) as size FROM pg_database) as sizes ORDER BY size DESC;"

# Nginx monitoring
sudo nginx -t                # Test configuration
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

---

## 🚨 Troubleshooting

### Common Issues

1. **Port already in use**

   ```bash
   # Find process using port
   lsof -i :3001
   # Kill process
   kill -9 <PID>
   ```

2. **Database connection issues**

   ```bash
   # Check PostgreSQL status
   sudo systemctl status postgresql
   # Restart PostgreSQL
   sudo systemctl restart postgresql
   ```

3. **Prisma client issues**

   ```bash
   # Regenerate Prisma client
   cd api
   npx prisma generate
   ```

4. **PM2 processes not starting**

   ```bash
   # Check PM2 logs
   pm2 logs
   # Restart PM2
   pm2 delete all
   pm2 start ecosystem.config.js
   ```

5. **Next.js build issues**
   ```bash
   # Clear Next.js cache
   cd ui
   rm -rf .next
   npm run build
   ```

### Performance Optimization

```bash
# Enable gzip in Nginx
# Add to /etc/nginx/sites-available/queue-management
gzip on;
gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

# Enable caching for static assets
location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

This deployment guide provides comprehensive instructions for deploying your Queue Management System using NPM without Docker. Choose the deployment option that best fits your infrastructure and requirements.
