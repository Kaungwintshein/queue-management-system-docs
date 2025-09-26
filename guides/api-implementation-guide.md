# API Implementation Guide

A comprehensive guide for implementing the backend API for the Queue Management System using modern REST API principles and real-time WebSocket communication.

## 🎯 Overview

The API serves as the backbone of the queue management system, handling:

- Token generation and management
- Queue operations
- Real-time updates via WebSocket
- User authentication and authorization
- Analytics and reporting

## 🛠️ Technology Stack

### Recommended Backend Technologies

- **Runtime**: Node.js 18+ with TypeScript
- **Framework**: Express.js or Fastify
- **Database**: PostgreSQL with Prisma ORM
- **Real-time**: Socket.io
- **Authentication**: JWT with bcrypt
- **Validation**: Zod or Joi
- **API Documentation**: Swagger/OpenAPI

### Dependencies

```json
{
  "express": "^4.18.0",
  "typescript": "^5.0.0",
  "prisma": "^5.0.0",
  "@prisma/client": "^5.0.0",
  "socket.io": "^4.7.0",
  "jsonwebtoken": "^9.0.0",
  "bcryptjs": "^2.4.3",
  "zod": "^3.22.0",
  "cors": "^2.8.5",
  "helmet": "^7.0.0",
  "rate-limiter-flexible": "^3.0.0"
}
```

## 🗄️ Database Schema

### Core Tables

```sql
-- Customer Types
CREATE TYPE customer_type AS ENUM ('instant', 'browser', 'retail');
CREATE TYPE token_status AS ENUM ('waiting', 'called', 'serving', 'completed', 'cancelled');
CREATE TYPE user_role AS ENUM ('staff', 'admin', 'super_admin');

-- Tokens Table
CREATE TABLE tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    number VARCHAR(10) NOT NULL UNIQUE,
    customer_type customer_type NOT NULL,
    status token_status DEFAULT 'waiting',
    priority INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    called_at TIMESTAMP,
    served_at TIMESTAMP,
    completed_at TIMESTAMP,
    served_by UUID REFERENCES users(id),
    notes TEXT
);

-- Users Table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role user_role NOT NULL,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP
);

-- Queue Settings Table
CREATE TABLE queue_settings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_type customer_type NOT NULL,
    prefix VARCHAR(5) NOT NULL,
    current_number INTEGER DEFAULT 0,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Service Sessions Table
CREATE TABLE service_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    staff_id UUID REFERENCES users(id),
    started_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ended_at TIMESTAMP,
    tokens_served INTEGER DEFAULT 0
);
```

### Prisma Schema

```prisma
// schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum CustomerType {
  instant
  browser
  retail
}

enum TokenStatus {
  waiting
  called
  serving
  completed
  cancelled
}

enum UserRole {
  staff
  admin
  super_admin
}

model Token {
  id           String       @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  number       String       @unique @db.VarChar(10)
  customerType CustomerType @map("customer_type")
  status       TokenStatus  @default(waiting)
  priority     Int          @default(0)
  createdAt    DateTime     @default(now()) @map("created_at")
  calledAt     DateTime?    @map("called_at")
  servedAt     DateTime?    @map("served_at")
  completedAt  DateTime?    @map("completed_at")
  servedBy     String?      @map("served_by") @db.Uuid
  notes        String?

  staff User? @relation(fields: [servedBy], references: [id])

  @@map("tokens")
}

model User {
  id           String    @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  username     String    @unique @db.VarChar(50)
  email        String    @unique @db.VarChar(100)
  passwordHash String    @map("password_hash") @db.VarChar(255)
  role         UserRole
  isActive     Boolean   @default(true) @map("is_active")
  createdAt    DateTime  @default(now()) @map("created_at")
  lastLogin    DateTime? @map("last_login")

  tokens           Token[]
  serviceSessions  ServiceSession[]

  @@map("users")
}

model QueueSetting {
  id            String       @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  customerType  CustomerType @map("customer_type")
  prefix        String       @db.VarChar(5)
  currentNumber Int          @default(0) @map("current_number")
  isActive      Boolean      @default(true) @map("is_active")
  createdAt     DateTime     @default(now()) @map("created_at")

  @@map("queue_settings")
}

model ServiceSession {
  id           String    @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  staffId      String    @map("staff_id") @db.Uuid
  startedAt    DateTime  @default(now()) @map("started_at")
  endedAt      DateTime? @map("ended_at")
  tokensServed Int       @default(0) @map("tokens_served")

  staff User @relation(fields: [staffId], references: [id])

  @@map("service_sessions")
}
```

## 🔗 API Endpoints

### Authentication Endpoints

```typescript
// POST /api/auth/login
interface LoginRequest {
  username: string;
  password: string;
}

interface LoginResponse {
  token: string;
  user: {
    id: string;
    username: string;
    role: UserRole;
  };
}

// POST /api/auth/logout
// Requires Authorization header

// GET /api/auth/me
interface UserResponse {
  id: string;
  username: string;
  email: string;
  role: UserRole;
  isActive: boolean;
}
```

### Token Management Endpoints

```typescript
// POST /api/tokens
interface CreateTokenRequest {
  customerType: CustomerType;
  priority?: number;
}

interface TokenResponse {
  id: string;
  number: string;
  customerType: CustomerType;
  status: TokenStatus;
  priority: number;
  createdAt: string;
  estimatedWaitTime?: number;
}

// GET /api/tokens
interface GetTokensQuery {
  status?: TokenStatus;
  customerType?: CustomerType;
  limit?: number;
  offset?: number;
}

// GET /api/tokens/:id
// Returns single TokenResponse

// PATCH /api/tokens/:id
interface UpdateTokenRequest {
  status?: TokenStatus;
  priority?: number;
  notes?: string;
}

// DELETE /api/tokens/:id
// Soft delete (set status to cancelled)
```

### Queue Management Endpoints

```typescript
// GET /api/queue/status
interface QueueStatusResponse {
  currentServing: TokenResponse | null;
  nextInQueue: TokenResponse[];
  recentlyServed: TokenResponse[];
  stats: {
    totalWaiting: number;
    averageWaitTime: number;
    totalServedToday: number;
  };
}

// POST /api/queue/call-next
interface CallNextRequest {
  customerType?: CustomerType;
  staffId: string;
}

// POST /api/queue/complete-service
interface CompleteServiceRequest {
  tokenId: string;
  staffId: string;
  notes?: string;
}

// GET /api/queue/settings
interface QueueSettingsResponse {
  instant: QueueTypeSetting;
  browser: QueueTypeSetting;
  retail: QueueTypeSetting;
}

interface QueueTypeSetting {
  prefix: string;
  currentNumber: number;
  isActive: boolean;
}

// PATCH /api/queue/settings
interface UpdateQueueSettingsRequest {
  customerType: CustomerType;
  prefix?: string;
  isActive?: boolean;
}
```

### Staff Management Endpoints

```typescript
// GET /api/staff
interface StaffListResponse {
  staff: UserResponse[];
  total: number;
}

// POST /api/staff
interface CreateStaffRequest {
  username: string;
  email: string;
  password: string;
  role: UserRole;
}

// PATCH /api/staff/:id
interface UpdateStaffRequest {
  username?: string;
  email?: string;
  password?: string;
  role?: UserRole;
  isActive?: boolean;
}

// GET /api/staff/:id/sessions
interface StaffSessionsResponse {
  sessions: ServiceSession[];
  stats: {
    totalSessions: number;
    totalTokensServed: number;
    averageSessionDuration: number;
  };
}
```

### Analytics Endpoints

```typescript
// GET /api/analytics/dashboard
interface DashboardAnalyticsResponse {
  today: {
    totalTokens: number;
    tokensServed: number;
    averageWaitTime: number;
    peakHour: string;
  };
  thisWeek: {
    totalTokens: number;
    dailyBreakdown: DailyStats[];
  };
  customerTypes: {
    instant: number;
    browser: number;
    retail: number;
  };
}

// GET /api/analytics/reports
interface ReportsQuery {
  startDate: string;
  endDate: string;
  customerType?: CustomerType;
  groupBy?: "day" | "week" | "month";
}

interface ReportsResponse {
  data: ReportData[];
  summary: {
    totalTokens: number;
    averageWaitTime: number;
    completionRate: number;
  };
}
```

## 🔧 API Implementation

### Server Setup

```typescript
// src/app.ts
import express from "express";
import cors from "cors";
import helmet from "helmet";
import { createServer } from "http";
import { Server as SocketIOServer } from "socket.io";
import { PrismaClient } from "@prisma/client";
import { errorHandler } from "./middleware/errorHandler";
import { authRouter } from "./routes/auth";
import { tokensRouter } from "./routes/tokens";
import { queueRouter } from "./routes/queue";
import { staffRouter } from "./routes/staff";
import { analyticsRouter } from "./routes/analytics";

const app = express();
const server = createServer(app);
const io = new SocketIOServer(server, {
  cors: {
    origin: process.env.FRONTEND_URL || "http://localhost:3000",
    methods: ["GET", "POST"],
  },
});

export const prisma = new PrismaClient();

// Middleware
app.use(helmet());
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.use("/api/auth", authRouter);
app.use("/api/tokens", tokensRouter);
app.use("/api/queue", queueRouter);
app.use("/api/staff", staffRouter);
app.use("/api/analytics", analyticsRouter);

// Error handling
app.use(errorHandler);

// Socket.io setup
io.on("connection", (socket) => {
  console.log("Client connected:", socket.id);

  socket.on("join_room", (room) => {
    socket.join(room);
  });

  socket.on("disconnect", () => {
    console.log("Client disconnected:", socket.id);
  });
});

export { io };

const PORT = process.env.PORT || 3001;
server.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### Token Service

```typescript
// src/services/tokenService.ts
import { prisma } from "../app";
import { CustomerType, TokenStatus } from "@prisma/client";
import { io } from "../app";

export class TokenService {
  async createToken(customerType: CustomerType, priority: number = 0) {
    // Get queue settings for the customer type
    const settings = await prisma.queueSetting.findFirst({
      where: { customerType, isActive: true },
    });

    if (!settings) {
      throw new Error(`Queue not active for customer type: ${customerType}`);
    }

    // Generate token number
    const nextNumber = settings.currentNumber + 1;
    const tokenNumber = `${settings.prefix}${nextNumber
      .toString()
      .padStart(3, "0")}`;

    // Create token
    const token = await prisma.token.create({
      data: {
        number: tokenNumber,
        customerType,
        priority,
        status: "waiting",
      },
    });

    // Update queue settings
    await prisma.queueSetting.update({
      where: { id: settings.id },
      data: { currentNumber: nextNumber },
    });

    // Emit real-time update
    io.emit("token:created", token);
    io.emit("queue:updated", await this.getQueueStatus());

    return token;
  }

  async callNextToken(staffId: string, customerType?: CustomerType) {
    const whereClause = {
      status: "waiting" as TokenStatus,
      ...(customerType && { customerType }),
    };

    const nextToken = await prisma.token.findFirst({
      where: whereClause,
      orderBy: [{ priority: "desc" }, { createdAt: "asc" }],
    });

    if (!nextToken) {
      throw new Error("No tokens in queue");
    }

    const updatedToken = await prisma.token.update({
      where: { id: nextToken.id },
      data: {
        status: "called",
        calledAt: new Date(),
        servedBy: staffId,
      },
    });

    // Emit real-time update
    io.emit("token:called", updatedToken);
    io.emit("queue:updated", await this.getQueueStatus());

    return updatedToken;
  }

  async completeService(tokenId: string, staffId: string, notes?: string) {
    const token = await prisma.token.update({
      where: { id: tokenId },
      data: {
        status: "completed",
        completedAt: new Date(),
        servedBy: staffId,
        notes,
      },
    });

    // Update service session
    const activeSession = await prisma.serviceSession.findFirst({
      where: {
        staffId,
        endedAt: null,
      },
    });

    if (activeSession) {
      await prisma.serviceSession.update({
        where: { id: activeSession.id },
        data: {
          tokensServed: {
            increment: 1,
          },
        },
      });
    }

    // Emit real-time update
    io.emit("token:completed", token);
    io.emit("queue:updated", await this.getQueueStatus());

    return token;
  }

  async getQueueStatus() {
    const [currentServing, nextInQueue, recentlyServed] = await Promise.all([
      // Current serving token
      prisma.token.findFirst({
        where: { status: "called" },
        orderBy: { calledAt: "desc" },
      }),

      // Next 5 tokens in queue
      prisma.token.findMany({
        where: { status: "waiting" },
        orderBy: [{ priority: "desc" }, { createdAt: "asc" }],
        take: 5,
      }),

      // Recently served tokens
      prisma.token.findMany({
        where: { status: "completed" },
        orderBy: { completedAt: "desc" },
        take: 5,
      }),
    ]);

    const stats = await this.getQueueStats();

    return {
      currentServing,
      nextInQueue,
      recentlyServed,
      stats,
    };
  }

  private async getQueueStats() {
    const today = new Date();
    today.setHours(0, 0, 0, 0);

    const [totalWaiting, totalServedToday, waitTimes] = await Promise.all([
      prisma.token.count({
        where: { status: "waiting" },
      }),

      prisma.token.count({
        where: {
          status: "completed",
          completedAt: { gte: today },
        },
      }),

      prisma.token.findMany({
        where: {
          status: "completed",
          completedAt: { gte: today },
          calledAt: { not: null },
          completedAt: { not: null },
        },
        select: {
          calledAt: true,
          completedAt: true,
          createdAt: true,
        },
      }),
    ]);

    const averageWaitTime =
      waitTimes.length > 0
        ? waitTimes.reduce((sum, token) => {
            const waitTime =
              token.calledAt!.getTime() - token.createdAt.getTime();
            return sum + waitTime;
          }, 0) /
          waitTimes.length /
          1000 /
          60 // Convert to minutes
        : 0;

    return {
      totalWaiting,
      averageWaitTime: Math.round(averageWaitTime),
      totalServedToday,
    };
  }
}
```

### Authentication Middleware

```typescript
// src/middleware/auth.ts
import jwt from "jsonwebtoken";
import { Request, Response, NextFunction } from "express";
import { prisma } from "../app";

interface AuthRequest extends Request {
  user?: {
    id: string;
    username: string;
    role: string;
  };
}

export const authenticate = async (
  req: AuthRequest,
  res: Response,
  next: NextFunction
) => {
  try {
    const token = req.header("Authorization")?.replace("Bearer ", "");

    if (!token) {
      return res.status(401).json({ error: "No token provided" });
    }

    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as any;

    const user = await prisma.user.findUnique({
      where: { id: decoded.userId },
      select: { id: true, username: true, role: true, isActive: true },
    });

    if (!user || !user.isActive) {
      return res.status(401).json({ error: "Invalid token" });
    }

    req.user = user;
    next();
  } catch (error) {
    res.status(401).json({ error: "Invalid token" });
  }
};

export const authorize = (roles: string[]) => {
  return (req: AuthRequest, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: "Unauthorized" });
    }

    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: "Insufficient permissions" });
    }

    next();
  };
};
```

## 🔄 Real-time Events

### WebSocket Event Types

```typescript
// Socket Events
interface SocketEvents {
  // Queue updates
  "queue:updated": (queueStatus: QueueStatusResponse) => void;
  "token:created": (token: TokenResponse) => void;
  "token:called": (token: TokenResponse) => void;
  "token:completed": (token: TokenResponse) => void;

  // System events
  "system:maintenance": (message: string) => void;
  "queue:paused": (customerType: CustomerType) => void;
  "queue:resumed": (customerType: CustomerType) => void;

  // Client events
  join_room: (room: string) => void;
  leave_room: (room: string) => void;
}
```

## 🛡️ Security Implementation

### Rate Limiting

```typescript
// src/middleware/rateLimiter.ts
import { RateLimiterRedis } from "rate-limiter-flexible";
import Redis from "ioredis";

const redis = new Redis(process.env.REDIS_URL);

export const rateLimiter = new RateLimiterRedis({
  storeClient: redis,
  keyGenerator: (req) => req.ip,
  points: 100, // Number of requests
  duration: 60, // Per 60 seconds
});

export const strictRateLimiter = new RateLimiterRedis({
  storeClient: redis,
  keyGenerator: (req) => req.ip,
  points: 10,
  duration: 60,
});
```

### Input Validation

```typescript
// src/schemas/tokenSchemas.ts
import { z } from "zod";

export const createTokenSchema = z.object({
  customerType: z.enum(["instant", "browser", "retail"]),
  priority: z.number().min(0).max(10).optional(),
});

export const updateTokenSchema = z.object({
  status: z
    .enum(["waiting", "called", "serving", "completed", "cancelled"])
    .optional(),
  priority: z.number().min(0).max(10).optional(),
  notes: z.string().max(500).optional(),
});
```

## 📊 Performance Optimization

### Database Indexing

```sql
-- Performance indexes
CREATE INDEX idx_tokens_status_priority_created ON tokens(status, priority DESC, created_at ASC);
CREATE INDEX idx_tokens_customer_type_status ON tokens(customer_type, status);
CREATE INDEX idx_tokens_created_at ON tokens(created_at);
CREATE INDEX idx_tokens_served_by ON tokens(served_by);
CREATE INDEX idx_service_sessions_staff_ended ON service_sessions(staff_id, ended_at);
```

### Caching Strategy

```typescript
// src/services/cacheService.ts
import Redis from "ioredis";

const redis = new Redis(process.env.REDIS_URL);

export class CacheService {
  async get<T>(key: string): Promise<T | null> {
    const value = await redis.get(key);
    return value ? JSON.parse(value) : null;
  }

  async set(key: string, value: any, ttl: number = 300) {
    await redis.setex(key, ttl, JSON.stringify(value));
  }

  async del(key: string) {
    await redis.del(key);
  }

  async invalidatePattern(pattern: string) {
    const keys = await redis.keys(pattern);
    if (keys.length > 0) {
      await redis.del(...keys);
    }
  }
}
```

## 📚 API Documentation

### Swagger Configuration

```typescript
// src/swagger.ts
import swaggerJsdoc from "swagger-jsdoc";

const options = {
  definition: {
    openapi: "3.0.0",
    info: {
      title: "Queue Management API",
      version: "1.0.0",
      description: "API for managing queue system operations",
    },
    servers: [
      {
        url: process.env.API_URL || "http://localhost:3001",
        description: "Development server",
      },
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: "http",
          scheme: "bearer",
          bearerFormat: "JWT",
        },
      },
    },
    security: [
      {
        bearerAuth: [],
      },
    ],
  },
  apis: ["./src/routes/*.ts"],
};

export const specs = swaggerJsdoc(options);
```

## 🧪 Testing Strategy

### Unit Tests

```typescript
// __tests__/services/tokenService.test.ts
import { TokenService } from "../../src/services/tokenService";
import { prisma } from "../../src/app";

describe("TokenService", () => {
  let tokenService: TokenService;

  beforeAll(() => {
    tokenService = new TokenService();
  });

  beforeEach(async () => {
    await prisma.token.deleteMany();
    await prisma.queueSetting.deleteMany();
  });

  describe("createToken", () => {
    it("should create a token with correct number format", async () => {
      // Test implementation
    });

    it("should increment queue number correctly", async () => {
      // Test implementation
    });
  });
});
```

## 🚀 Deployment

### Environment Variables

```bash
# .env
DATABASE_URL="postgresql://user:password@localhost:5432/queue_system"
JWT_SECRET="your-super-secret-jwt-key"
REDIS_URL="redis://localhost:6379"
NODE_ENV="production"
PORT=3001
FRONTEND_URL="https://your-frontend-domain.com"
```

### PM2 Process Management

```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: "queue-management-api",
      script: "dist/app.js",
      instances: "max",
      exec_mode: "cluster",
      env: {
        NODE_ENV: "production",
        PORT: 3001,
      },
      log_file: "/var/log/pm2/api.log",
      error_file: "/var/log/pm2/api-error.log",
      out_file: "/var/log/pm2/api-out.log",
    },
  ],
};
```

## 🔗 Integration Points

- **Frontend Integration**: See [API Integration Guide](./api-integration-guide.md)
- **Real-time Updates**: WebSocket events for live UI updates
- **Authentication**: JWT-based auth for role-based access
- **Database**: PostgreSQL with optimized queries

---

_This guide provides a complete backend implementation for a scalable, secure, and efficient queue management system._
