# System Architecture Overview

A comprehensive architectural blueprint for the Queue Management System, detailing system components, data flow, security considerations, and deployment strategies.

## 🏗️ System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              FRONTEND LAYER                                 │
├─────────────────┬─────────────────┬─────────────────────────────────────────┤
│  Display UI     │   Staff UI      │        Super Admin UI                   │
│                 │                 │                                         │
│ ┌─────────────┐ │ ┌─────────────┐ │ ┌─────────────┬─────────────────────────┐ │
│ │   React     │ │ │   React     │ │ │   React     │    Dashboard            │ │
│ │   + Socket  │ │ │   + Auth    │ │ │   + Router  │    Analytics            │ │
│ │   (Read-Only)│ │ │   + Queue   │ │ │   + Auth    │    User Management      │ │
│ │             │ │ │   Mgmt      │ │ │   + Full    │    System Settings      │ │
│ └─────────────┘ │ └─────────────┘ │ │   Access    │                         │ │
│                 │                 │ └─────────────┴─────────────────────────┘ │
└─────────────────┴─────────────────┴─────────────────────────────────────────┘
                                    │
                                    │ HTTPS/WSS
                                    │
┌─────────────────────────────────────────────────────────────────────────────┐
│                              API GATEWAY                                    │
│                                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │Rate Limiting│  │Load Balancer│  │  SSL/TLS    │  │  Request Logging    │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────────────┐
│                           APPLICATION LAYER                                 │
├─────────────────┬─────────────────┬─────────────────────────────────────────┤
│   REST API      │   WebSocket     │           Background Jobs               │
│   Server        │   Server        │                                         │
│                 │                 │ ┌─────────────┬─────────────────────────┐ │
│ ┌─────────────┐ │ ┌─────────────┐ │ │Queue Cleanup│    Analytics            │ │
│ │Express.js   │ │ │Socket.io    │ │ │Report Gen   │    Cleanup Jobs         │ │
│ │+ Middleware │ │ │+ Auth       │ │ │Notifications│    Data Export          │ │
│ │+ Validation │ │ │+ Rooms      │ │ │             │                         │ │
│ │+ Auth       │ │ │+ Events     │ │ └─────────────┴─────────────────────────┘ │
│ └─────────────┘ │ └─────────────┘ │                                         │
└─────────────────┴─────────────────┴─────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────────────┐
│                             DATA LAYER                                      │
├─────────────────┬─────────────────┬─────────────────────────────────────────┤
│   PostgreSQL    │     Redis       │            File Storage                 │
│   Database      │     Cache       │                                         │
│                 │                 │ ┌─────────────┬─────────────────────────┐ │
│ ┌─────────────┐ │ ┌─────────────┐ │ │   Reports   │      Logs               │ │
│ │Tokens       │ │ │Session Data │ │ │   Exports   │      Backups            │ │
│ │Users        │ │ │Rate Limits  │ │ │   Assets    │      Uploads            │ │
│ │Queue Config │ │ │Cache Data   │ │ │             │                         │ │
│ │Analytics    │ │ │Temp Data    │ │ └─────────────┴─────────────────────────┘ │
│ └─────────────┘ │ └─────────────┘ │                                         │
└─────────────────┴─────────────────┴─────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────────────┐
│                        INFRASTRUCTURE LAYER                                 │
│                                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │ Process Mgr │  │  Load Bal.  │  │  Monitoring │  │     Security        │ │
│  │    PM2      │  │   Nginx     │  │  Grafana    │  │    Firewall         │ │
│  │             │  │   HAProxy   │  │  Prometheus │  │    Intrusion Det.   │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 🔄 Data Flow Architecture

### 1. Token Creation Flow

```mermaid
graph TD
    A[Staff/Admin UI] -->|Create Token Request| B[API Gateway]
    B --> C[Authentication Middleware]
    C --> D[Validation Middleware]
    D --> E[Token Service]
    E --> F[Database Query]
    F --> G[Queue Settings Check]
    G --> H[Generate Token Number]
    H --> I[Save to Database]
    I --> J[Update Cache]
    J --> K[WebSocket Broadcast]
    K --> L[Display UI Update]
    K --> M[Staff UI Update]
    E --> N[Return Response]
```

### 2. Queue Status Update Flow

```mermaid
graph TD
    A[Display UI] -->|Request Queue Status| B[API Gateway]
    B --> C[Cache Check]
    C -->|Cache Hit| D[Return Cached Data]
    C -->|Cache Miss| E[Database Query]
    E --> F[Aggregate Queue Data]
    F --> G[Update Cache]
    G --> H[Return Response]

    I[WebSocket Events] --> J[Real-time Updates]
    J --> K[Display UI]
    J --> L[Staff UI]
    J --> M[Admin UI]
```

### 3. Service Completion Flow

```mermaid
graph TD
    A[Staff Action] -->|Complete Service| B[API Gateway]
    B --> C[Authorization Check]
    C --> D[Token Service]
    D --> E[Update Token Status]
    E --> F[Record Service Time]
    F --> G[Update Staff Metrics]
    G --> H[Database Transaction]
    H --> I[Cache Invalidation]
    I --> J[WebSocket Broadcast]
    J --> K[UI Updates]
    D --> L[Return Success]
```

## 🗄️ Database Schema Design

### Core Tables Structure

```sql
-- Database: queue_management_system

-- Customer Types Enum
CREATE TYPE customer_type AS ENUM ('instant', 'browser', 'retail');
CREATE TYPE token_status AS ENUM ('waiting', 'called', 'serving', 'completed', 'cancelled');
CREATE TYPE user_role AS ENUM ('staff', 'admin', 'super_admin');

-- Main Tables
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id),
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role user_role NOT NULL,
    permissions JSONB DEFAULT '{}',
    is_active BOOLEAN DEFAULT true,
    last_login TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE queue_configurations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id),
    customer_type customer_type NOT NULL,
    prefix VARCHAR(5) NOT NULL,
    current_number INTEGER DEFAULT 0,
    max_number INTEGER DEFAULT 999,
    reset_daily BOOLEAN DEFAULT true,
    reset_time TIME DEFAULT '00:00:00',
    is_active BOOLEAN DEFAULT true,
    priority_multiplier DECIMAL(3,2) DEFAULT 1.0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(organization_id, customer_type)
);

CREATE TABLE tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id),
    number VARCHAR(10) NOT NULL,
    customer_type customer_type NOT NULL,
    status token_status DEFAULT 'waiting',
    priority INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    called_at TIMESTAMP,
    served_at TIMESTAMP,
    completed_at TIMESTAMP,
    cancelled_at TIMESTAMP,
    served_by UUID REFERENCES users(id),
    estimated_wait_time INTEGER, -- in minutes
    actual_wait_time INTEGER, -- in minutes
    service_duration INTEGER, -- in minutes
    notes TEXT,
    metadata JSONB DEFAULT '{}',
    UNIQUE(organization_id, number, DATE(created_at))
);

CREATE TABLE service_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    staff_id UUID REFERENCES users(id),
    organization_id UUID REFERENCES organizations(id),
    started_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ended_at TIMESTAMP,
    tokens_served INTEGER DEFAULT 0,
    average_service_time DECIMAL(5,2),
    break_time INTEGER DEFAULT 0, -- in minutes
    notes TEXT
);

CREATE TABLE system_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id),
    user_id UUID REFERENCES users(id),
    action VARCHAR(50) NOT NULL,
    entity_type VARCHAR(50),
    entity_id UUID,
    details JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE analytics_snapshots (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id),
    snapshot_date DATE NOT NULL,
    total_tokens INTEGER DEFAULT 0,
    tokens_by_type JSONB DEFAULT '{}',
    average_wait_time DECIMAL(5,2),
    peak_hours JSONB DEFAULT '{}',
    staff_performance JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(organization_id, snapshot_date)
);
```

### Indexes for Performance

```sql
-- Performance Indexes
CREATE INDEX idx_tokens_org_status_created ON tokens(organization_id, status, created_at);
CREATE INDEX idx_tokens_org_type_status ON tokens(organization_id, customer_type, status);
CREATE INDEX idx_tokens_staff_served ON tokens(served_by, served_at) WHERE served_by IS NOT NULL;
CREATE INDEX idx_tokens_created_date ON tokens(DATE(created_at));
CREATE INDEX idx_users_org_role ON users(organization_id, role);
CREATE INDEX idx_service_sessions_staff_date ON service_sessions(staff_id, DATE(started_at));
CREATE INDEX idx_system_logs_org_action_date ON system_logs(organization_id, action, DATE(created_at));

-- Partial Indexes for active records
CREATE INDEX idx_tokens_active_queue ON tokens(organization_id, customer_type, priority DESC, created_at ASC)
WHERE status = 'waiting';
CREATE INDEX idx_users_active ON users(organization_id, role) WHERE is_active = true;
```

## 🔐 Security Architecture

### Authentication & Authorization

```typescript
// Security Layers
interface SecurityConfig {
  authentication: {
    strategy: "JWT";
    tokenExpiry: "24h";
    refreshTokenExpiry: "7d";
    encryption: "bcrypt";
    saltRounds: 12;
  };

  authorization: {
    roleBasedAccess: boolean;
    permissionGranularity: "action-level";
    resourceBasedAccess: boolean;
  };

  apiSecurity: {
    rateLimiting: {
      windowMs: 900000; // 15 minutes
      maxRequests: 100;
      strictEndpoints: {
        login: 5;
        tokenCreation: 20;
      };
    };
    cors: {
      origin: string[];
      credentials: boolean;
    };
    headers: {
      helmet: boolean;
      hsts: boolean;
      noSniff: boolean;
    };
  };
}
```

### Role-Based Permissions

```typescript
// Permission Matrix
interface PermissionMatrix {
  super_admin: {
    tokens: ["create", "read", "update", "delete"];
    users: ["create", "read", "update", "delete"];
    settings: ["read", "update"];
    analytics: ["read", "export"];
    system: ["configure", "monitor", "backup"];
  };

  admin: {
    tokens: ["create", "read", "update"];
    users: ["create", "read", "update"];
    settings: ["read"];
    analytics: ["read"];
    system: ["monitor"];
  };

  staff: {
    tokens: ["create", "read", "update"];
    users: ["read:self"];
    settings: ["read:limited"];
    analytics: ["read:limited"];
    system: [];
  };
}
```

### Data Encryption

```typescript
// Encryption Strategy
interface EncryptionConfig {
  atRest: {
    database: "AES-256-GCM";
    files: "AES-256-GCM";
    backups: "AES-256-GCM";
  };

  inTransit: {
    api: "TLS 1.3";
    websocket: "WSS";
    database: "SSL";
  };

  sensitive_fields: {
    passwords: "bcrypt";
    tokens: "JWT + RS256";
    personal_data: "AES-256-GCM";
  };
}
```

## 📊 Performance & Scalability

### Caching Strategy

```typescript
// Multi-Level Caching
interface CachingStrategy {
  levels: {
    L1_Browser: {
      type: "Memory";
      ttl: "5m";
      storage: "LocalStorage + SessionStorage";
    };

    L2_CDN: {
      type: "Edge";
      ttl: "1h";
      storage: "CloudFlare/AWS CloudFront";
    };

    L3_Application: {
      type: "Redis";
      ttl: "15m";
      patterns: ["queue:status:*", "user:session:*", "analytics:*"];
    };

    L4_Database: {
      type: "Query Result Cache";
      ttl: "5m";
      triggers: ["data_change", "manual_invalidation"];
    };
  };
}
```

### Load Balancing Strategy

```yaml
# Load Balancer Configuration
load_balancer:
  type: nginx
  algorithm: least_connections
  health_checks:
    interval: 30s
    timeout: 5s
    retries: 3

  upstream_servers:
    - server: api-server-1:3001 weight=3
    - server: api-server-2:3001 weight=3
    - server: api-server-3:3001 weight=2 # backup

  session_persistence:
    method: ip_hash # for WebSocket connections

  rate_limiting:
    requests_per_second: 100
    burst: 200
```

### Database Optimization

```sql
-- Query Optimization Strategies

-- 1. Partitioning for large tables
CREATE TABLE tokens_y2024 PARTITION OF tokens
FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

-- 2. Materialized views for analytics
CREATE MATERIALIZED VIEW daily_queue_stats AS
SELECT
    DATE(created_at) as date,
    customer_type,
    COUNT(*) as total_tokens,
    AVG(actual_wait_time) as avg_wait_time,
    AVG(service_duration) as avg_service_time
FROM tokens
WHERE created_at >= CURRENT_DATE - INTERVAL '90 days'
GROUP BY DATE(created_at), customer_type;

-- 3. Refresh strategy
CREATE OR REPLACE FUNCTION refresh_daily_stats()
RETURNS void AS $$
BEGIN
    REFRESH MATERIALIZED VIEW CONCURRENTLY daily_queue_stats;
END;
$$ LANGUAGE plpgsql;

-- Schedule refresh every hour
SELECT cron.schedule('refresh-stats', '0 * * * *', 'SELECT refresh_daily_stats();');
```

## 🔄 Real-time Architecture

### WebSocket Event System

```typescript
// Event Architecture
interface EventSystem {
  namespaces: {
    "/display": {
      rooms: ["queue_updates", "system_announcements"];
      events: ["token_called", "queue_updated", "system_maintenance"];
    };

    "/staff": {
      rooms: ["staff_notifications", "queue_management"];
      events: ["token_assigned", "service_completed", "queue_alerts"];
      authentication: "required";
    };

    "/admin": {
      rooms: ["admin_dashboard", "system_monitoring"];
      events: ["user_activity", "system_metrics", "alerts"];
      authentication: "required";
      authorization: ["admin", "super_admin"];
    };
  };

  reliability: {
    heartbeat: "30s";
    reconnection: "exponential_backoff";
    message_queuing: "redis_streams";
    delivery_guarantee: "at_least_once";
  };
}
```

### Event-Driven Architecture

```typescript
// Event Bus Implementation
class EventBus {
  private handlers = new Map<string, Function[]>();

  async emit(event: string, data: any) {
    // Emit to local handlers
    const localHandlers = this.handlers.get(event) || [];
    localHandlers.forEach((handler) => handler(data));

    // Emit to WebSocket clients
    io.to(event).emit(event, data);

    // Persist critical events
    if (this.isCriticalEvent(event)) {
      await this.persistEvent(event, data);
    }
  }

  private async persistEvent(event: string, data: any) {
    await redis.xadd(
      "events:stream",
      "*",
      "event",
      event,
      "data",
      JSON.stringify(data)
    );
  }
}
```

## 🏗️ Microservices Architecture (Optional Scale-out)

### Service Decomposition

```yaml
# For larger deployments
microservices:
  api_gateway:
    responsibilities: [routing, authentication, rate_limiting]
    technologies: [nginx, kong, envoy]

  queue_service:
    responsibilities: [token_management, queue_operations]
    database: postgresql
    cache: redis

  user_service:
    responsibilities: [authentication, user_management]
    database: postgresql
    cache: redis

  analytics_service:
    responsibilities: [reporting, metrics, insights]
    database: postgresql + timescaledb
    cache: redis

  notification_service:
    responsibilities: [real_time_updates, alerts]
    technologies: [socket.io, redis_streams]

  file_service:
    responsibilities: [reports, exports, uploads]
    storage: s3_compatible
```

## 📈 Monitoring & Observability

### Metrics Collection

```typescript
// Monitoring Stack
interface MonitoringConfig {
  metrics: {
    application: {
      library: "prometheus-client";
      custom_metrics: [
        "queue_length_by_type",
        "average_wait_time",
        "service_completion_rate",
        "api_response_time",
        "websocket_connections"
      ];
    };

    infrastructure: {
      collection: "node_exporter";
      metrics: ["cpu", "memory", "disk", "network"];
    };

    database: {
      collection: "postgres_exporter";
      metrics: ["connection_count", "query_performance", "cache_hit_ratio"];
    };
  };

  logging: {
    structured: true;
    format: "json";
    levels: ["error", "warn", "info", "debug"];
    centralized: "elasticsearch + kibana";
  };

  tracing: {
    library: "opentelemetry";
    sampling_rate: 0.1;
    backend: "jaeger";
  };

  alerting: {
    channels: ["slack", "email", "webhook"];
    rules: [
      "queue_length > 50",
      "average_wait_time > 30min",
      "api_error_rate > 5%",
      "database_connection_failure"
    ];
  };
}
```

## 🚀 Deployment Architecture

### Process Management Strategy

```yaml
# PM2 Ecosystem Configuration
# ecosystem.config.js
module.exports = {
  apps: [
    {
      name: 'queue-api',
      script: 'dist/app.js',
      instances: 'max',
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
        PORT: 3001,
        DATABASE_URL: 'postgresql://user:pass@localhost:5432/queue_system',
        REDIS_URL: 'redis://localhost:6379'
      },
      log_file: '/var/log/pm2/queue-api.log',
      error_file: '/var/log/pm2/queue-api-error.log',
      out_file: '/var/log/pm2/queue-api-out.log',
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z'
    },
    {
      name: 'queue-ui',
      script: 'npm',
      args: 'start',
      cwd: '/path/to/ui',
      env: {
        NODE_ENV: 'production',
        PORT: 3000,
        NEXT_PUBLIC_API_URL: 'http://localhost:3001'
      },
      log_file: '/var/log/pm2/queue-ui.log',
      error_file: '/var/log/pm2/queue-ui-error.log',
      out_file: '/var/log/pm2/queue-ui-out.log'
    }
  ]
}
```

### Production Deployment

```yaml
# Kubernetes Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: queue-system-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: queue-system-api
  template:
    metadata:
      labels:
        app: queue-system-api
    spec:
      containers:
        - name: api
          image: queue-system:latest
          ports:
            - containerPort: 3001
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: database-secret
                  key: url
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
```

## 🔧 DevOps Pipeline

### CI/CD Configuration

```yaml
# GitHub Actions
name: CI/CD Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"
      - name: Install dependencies
        run: npm ci
      - name: Run tests
        run: npm test
      - name: Run integration tests
        run: npm run test:integration

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"
      - name: Build application
        run: |
          npm ci
          npm run build

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to production
        run: kubectl set image deployment/queue-system-api api=queue-system:${{ github.sha }}
```

## 📋 System Requirements

### Minimum Requirements

```yaml
development:
  cpu: 2 cores
  memory: 4GB RAM
  storage: 20GB SSD
  network: 10 Mbps

production_small: # <100 concurrent users
  cpu: 4 cores
  memory: 8GB RAM
  storage: 100GB SSD
  network: 100 Mbps
  database: 2 cores, 4GB RAM

production_medium: # <500 concurrent users
  cpu: 8 cores
  memory: 16GB RAM
  storage: 500GB SSD
  network: 1 Gbps
  database: 4 cores, 8GB RAM
  cache: 2GB Redis

production_large: # <2000 concurrent users
  cpu: 16 cores
  memory: 32GB RAM
  storage: 1TB SSD
  network: 10 Gbps
  database: 8 cores, 16GB RAM (with read replicas)
  cache: 4GB Redis Cluster
  load_balancer: dedicated
```

---

_This system architecture document provides a comprehensive blueprint for implementing a scalable, secure, and maintainable queue management system that can grow with your business needs._
