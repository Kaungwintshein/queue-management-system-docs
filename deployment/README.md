# Deployment Documentation

This directory contains all deployment-related documentation for the Queue Management System.

## 📋 Available Documentation

### Deployment Guides
- **[General Deployment Guide](./DEPLOYMENT.md)** - Comprehensive deployment instructions for various environments
- **[NPM Deployment Guide](./DEPLOYMENT-NPM.md)** - NPM-specific deployment guide and package management

## 🚀 Deployment Overview

The Queue Management System consists of two main components:

1. **API Backend** - Express.js API server with PostgreSQL database
2. **UI Frontend** - Next.js React application

## 🏗️ Architecture

### Production Stack
- **API Server**: Node.js + Express.js + TypeScript
- **Database**: PostgreSQL with Prisma ORM
- **Frontend**: Next.js + React + TailwindCSS
- **Process Management**: PM2 for production process management
- **Reverse Proxy**: Nginx (optional)
- **SSL/TLS**: Let's Encrypt certificates (optional)

### Environment Requirements
- **Node.js**: v18+ (LTS recommended)
- **PostgreSQL**: v13+ (v15+ recommended)
- **PM2**: Latest version for process management
- **Nginx**: v1.18+ (optional, for reverse proxy)

## 🔧 Deployment Options

### 1. Traditional Server Deployment
- **VPS/Cloud Server**: Deploy on VPS or cloud instances
- **Dedicated Server**: Deploy on dedicated hardware
- **Shared Hosting**: Limited deployment options

### 2. Container Deployment
- **Docker**: Containerized deployment (if Docker files are available)
- **Kubernetes**: Container orchestration (advanced)

### 3. Platform-as-a-Service (PaaS)
- **Heroku**: Easy deployment with add-ons
- **Railway**: Modern deployment platform
- **Vercel**: Frontend deployment (UI only)
- **Render**: Full-stack deployment

## 📦 Package Management

### NPM Deployment
- **Package Installation**: Standard npm install process
- **Dependency Management**: Lock file for consistent installs
- **Script Management**: npm scripts for common tasks
- **Version Management**: Semantic versioning for releases

### Production Dependencies
- **API Dependencies**: Express, Prisma, JWT, etc.
- **UI Dependencies**: Next.js, React, TailwindCSS, etc.
- **Development Dependencies**: Testing, linting, building tools

## 🗄️ Database Setup

### PostgreSQL Configuration
- **Database Creation**: Create production database
- **User Setup**: Create database user with appropriate permissions
- **Connection String**: Configure DATABASE_URL environment variable
- **Migrations**: Run Prisma migrations for schema setup
- **Seeding**: Optional data seeding for initial setup

### Database Security
- **User Permissions**: Minimal required permissions
- **Connection Security**: SSL connections in production
- **Backup Strategy**: Regular database backups
- **Monitoring**: Database performance monitoring

## 🔐 Environment Configuration

### Required Environment Variables

#### API Environment Variables
```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/queue_management"

# JWT Configuration
JWT_SECRET="your-super-secret-jwt-key"
JWT_EXPIRES_IN="24h"

# Server Configuration
PORT=3001
NODE_ENV="production"

# CORS Configuration
FRONTEND_URL="https://your-frontend-domain.com"

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100
```

#### UI Environment Variables
```bash
# API Configuration
NEXT_PUBLIC_API_URL="https://your-api-domain.com"
NEXT_PUBLIC_WS_URL="wss://your-api-domain.com"

# App Configuration
NEXT_PUBLIC_APP_NAME="Queue Management System"
NEXT_PUBLIC_APP_VERSION="1.0.0"
```

## 🚀 Deployment Process

### 1. Server Preparation
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PostgreSQL
sudo apt install postgresql postgresql-contrib

# Install PM2
sudo npm install -g pm2
```

### 2. Application Deployment
```bash
# Clone repository
git clone <repository-url>
cd queue-management-system

# Install dependencies
npm install
cd api && npm install
cd ../ui && npm install

# Build applications
cd ../api && npm run build
cd ../ui && npm run build
```

### 3. Database Setup
```bash
# Create database
sudo -u postgres createdb queue_management

# Run migrations
cd api
npx prisma migrate deploy

# Seed database (optional)
npx prisma db seed
```

### 4. Process Management
```bash
# Start with PM2
pm2 start ecosystem.config.js

# Save PM2 configuration
pm2 save

# Setup PM2 startup
pm2 startup
```

## 🔧 Process Management

### PM2 Configuration
- **Ecosystem File**: `ecosystem.config.js` for PM2 configuration
- **Process Monitoring**: Automatic restart on crashes
- **Log Management**: Centralized logging with log rotation
- **Cluster Mode**: Multiple instances for load balancing
- **Environment Management**: Different environments (dev, staging, prod)

### PM2 Commands
```bash
# Start applications
pm2 start ecosystem.config.js

# Monitor processes
pm2 monit

# View logs
pm2 logs

# Restart applications
pm2 restart all

# Stop applications
pm2 stop all

# Delete applications
pm2 delete all
```

## 🌐 Reverse Proxy (Optional)

### Nginx Configuration
```nginx
server {
    listen 80;
    server_name your-domain.com;

    # API proxy
    location /api {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # Frontend proxy
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

## 🔒 Security Considerations

### Production Security
- **Environment Variables**: Secure storage of sensitive data
- **HTTPS**: SSL/TLS certificates for secure communication
- **CORS**: Proper CORS configuration for cross-origin requests
- **Rate Limiting**: API rate limiting to prevent abuse
- **Input Validation**: Comprehensive input validation and sanitization
- **Authentication**: Secure JWT token handling

### Database Security
- **Connection Security**: SSL database connections
- **User Permissions**: Minimal database user permissions
- **Backup Encryption**: Encrypted database backups
- **Access Control**: Restricted database access

## 📊 Monitoring & Logging

### Application Monitoring
- **PM2 Monitoring**: Built-in PM2 monitoring
- **Health Checks**: API health check endpoints
- **Error Tracking**: Centralized error logging
- **Performance Monitoring**: Response time and throughput monitoring

### Log Management
- **Log Rotation**: Automatic log file rotation
- **Log Levels**: Different log levels (error, warn, info, debug)
- **Centralized Logging**: Centralized log collection
- **Log Analysis**: Log analysis and alerting

## 🔄 Backup & Recovery

### Database Backups
- **Automated Backups**: Scheduled database backups
- **Backup Storage**: Secure backup storage
- **Backup Testing**: Regular backup restoration testing
- **Point-in-Time Recovery**: Database point-in-time recovery

### Application Backups
- **Code Backups**: Version control with Git
- **Configuration Backups**: Environment configuration backups
- **Deployment Backups**: Deployment artifact backups

## 🚀 Scaling Considerations

### Horizontal Scaling
- **Load Balancing**: Multiple API instances behind load balancer
- **Database Scaling**: Read replicas for database scaling
- **CDN**: Content delivery network for static assets
- **Caching**: Redis for session and data caching

### Vertical Scaling
- **Server Resources**: CPU, memory, and storage scaling
- **Database Optimization**: Query optimization and indexing
- **Application Optimization**: Code and performance optimization
