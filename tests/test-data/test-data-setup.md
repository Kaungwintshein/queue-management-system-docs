# Test Data Setup and Management

This document outlines the comprehensive test data setup, management, and cleanup procedures for the Queue Management System E2E tests using Appium.

## 📋 Table of Contents

1. [Test Data Overview](#test-data-overview)
2. [Database Setup](#database-setup)
3. [Test Fixtures](#test-fixtures)
4. [Mobile App Test Data](#mobile-app-test-data)
5. [Web Application Test Data](#web-application-test-data)
6. [API Test Data](#api-test-data)
7. [Cleanup Procedures](#cleanup-procedures)
8. [Data Isolation](#data-isolation)
9. [Performance Test Data](#performance-test-data)

## 🎯 Test Data Overview

### Test Data Categories

- **User Data**: Test users with different roles and permissions
- **Organization Data**: Test organizations with various configurations
- **Counter Data**: Test counters in different states and assignments
- **Token Data**: Test tokens with various priorities and statuses
- **Queue Data**: Test queue states and configurations
- **Configuration Data**: System settings and configurations

### Test Data Principles

- **Isolation**: Each test suite uses independent data
- **Predictability**: Test data is consistent and known
- **Cleanup**: Automatic cleanup after test execution
- **Scalability**: Support for large datasets in performance tests

## 🗄️ Database Setup

### Test Database Configuration

```javascript
// test-database.config.js
const testDatabaseConfig = {
  host: process.env.TEST_DB_HOST || "localhost",
  port: process.env.TEST_DB_PORT || 5432,
  database: process.env.TEST_DB_NAME || "queue_management_test",
  username: process.env.TEST_DB_USER || "test_user",
  password: process.env.TEST_DB_PASSWORD || "test_password",
  ssl: false,
  logging: false,
};

module.exports = testDatabaseConfig;
```

### Database Initialization

```javascript
// database-setup.js
const { PrismaClient } = require("@prisma/client");
const testDatabaseConfig = require("./test-database.config");

class TestDatabaseSetup {
  constructor() {
    this.prisma = new PrismaClient({
      datasources: {
        db: {
          url: `postgresql://${testDatabaseConfig.username}:${testDatabaseConfig.password}@${testDatabaseConfig.host}:${testDatabaseConfig.port}/${testDatabaseConfig.database}`,
        },
      },
    });
  }

  async initialize() {
    // Run migrations
    await this.runMigrations();

    // Seed test data
    await this.seedTestData();

    // Verify setup
    await this.verifySetup();
  }

  async runMigrations() {
    const { execSync } = require("child_process");
    execSync("npx prisma migrate deploy", {
      env: { ...process.env, DATABASE_URL: this.getDatabaseUrl() },
    });
  }

  async seedTestData() {
    // Seed organizations
    await this.seedOrganizations();

    // Seed users
    await this.seedUsers();

    // Seed counters
    await this.seedCounters();

    // Seed tokens
    await this.seedTokens();

    // Seed configurations
    await this.seedConfigurations();
  }

  async verifySetup() {
    const orgCount = await this.prisma.organization.count();
    const userCount = await this.prisma.user.count();
    const counterCount = await this.prisma.counter.count();

    console.log(`Test database setup complete:
      - Organizations: ${orgCount}
      - Users: ${userCount}
      - Counters: ${counterCount}`);
  }

  getDatabaseUrl() {
    return `postgresql://${testDatabaseConfig.username}:${testDatabaseConfig.password}@${testDatabaseConfig.host}:${testDatabaseConfig.port}/${testDatabaseConfig.database}`;
  }

  async cleanup() {
    await this.prisma.$disconnect();
  }
}

module.exports = TestDatabaseSetup;
```

## 🧪 Test Fixtures

### Base Test Data

```javascript
// test-fixtures.js
const bcrypt = require("bcryptjs");
const { UserRole, TokenStatus, CounterStatus } = require("@prisma/client");

const TEST_FIXTURES = {
  organizations: [
    {
      id: "test-org-1",
      name: "Test Organization 1",
      settings: {
        maxQueueLength: 100,
        averageServiceTime: 5,
        notificationSettings: {
          email: true,
          sms: true,
          push: true,
        },
      },
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: "test-org-2",
      name: "Test Organization 2",
      settings: {
        maxQueueLength: 50,
        averageServiceTime: 3,
        notificationSettings: {
          email: false,
          sms: true,
          push: false,
        },
      },
      createdAt: new Date(),
      updatedAt: new Date(),
    },
  ],

  users: [
    {
      id: "super-admin-1",
      username: "superadmin",
      email: "superadmin@test.com",
      password: bcrypt.hashSync("superadmin123", 10),
      role: UserRole.super_admin,
      organizationId: "test-org-1",
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: "admin-1",
      username: "admin",
      email: "admin@test.com",
      password: bcrypt.hashSync("admin123", 10),
      role: UserRole.admin,
      organizationId: "test-org-1",
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: "admin-2",
      username: "admin2",
      email: "admin2@test.com",
      password: bcrypt.hashSync("admin123", 10),
      role: UserRole.admin,
      organizationId: "test-org-2",
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: "staff-1",
      username: "staff1",
      email: "staff1@test.com",
      password: bcrypt.hashSync("staff123", 10),
      role: UserRole.staff,
      organizationId: "test-org-1",
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: "staff-2",
      username: "staff2",
      email: "staff2@test.com",
      password: bcrypt.hashSync("staff123", 10),
      role: UserRole.staff,
      organizationId: "test-org-1",
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: "staff-3",
      username: "staff3",
      email: "staff3@test.com",
      password: bcrypt.hashSync("staff123", 10),
      role: UserRole.staff,
      organizationId: "test-org-2",
      isActive: false, // Inactive staff for testing
      createdAt: new Date(),
      updatedAt: new Date(),
    },
  ],

  counters: [
    {
      id: "counter-1",
      name: "Counter 1",
      description: "Primary service counter",
      isActive: true,
      organizationId: "test-org-1",
      assignedStaffId: "staff-1",
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: "counter-2",
      name: "Counter 2",
      description: "Secondary service counter",
      isActive: true,
      organizationId: "test-org-1",
      assignedStaffId: "staff-2",
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: "counter-3",
      name: "Counter 3",
      description: "Inactive counter for testing",
      isActive: false,
      organizationId: "test-org-1",
      assignedStaffId: null,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: "counter-4",
      name: "Counter 4",
      description: "Unassigned counter",
      isActive: true,
      organizationId: "test-org-1",
      assignedStaffId: null,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: "counter-5",
      name: "Counter 5",
      description: "Organization 2 counter",
      isActive: true,
      organizationId: "test-org-2",
      assignedStaffId: "staff-3",
      createdAt: new Date(),
      updatedAt: new Date(),
    },
  ],

  tokens: [
    {
      id: "token-1",
      tokenNumber: "A001",
      customerType: "instant",
      priority: 1,
      status: TokenStatus.waiting,
      position: 1,
      estimatedWaitTime: 5,
      notes: "Urgent service required",
      organizationId: "test-org-1",
      assignedCounterId: "counter-1",
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: "token-2",
      tokenNumber: "A002",
      customerType: "browser",
      priority: 2,
      status: TokenStatus.waiting,
      position: 2,
      estimatedWaitTime: 10,
      notes: null,
      organizationId: "test-org-1",
      assignedCounterId: "counter-1",
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: "token-3",
      tokenNumber: "A003",
      customerType: "retail",
      priority: 3,
      status: TokenStatus.called,
      position: 0,
      estimatedWaitTime: 0,
      notes: "Standard service",
      organizationId: "test-org-1",
      assignedCounterId: "counter-2",
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: "token-4",
      tokenNumber: "A004",
      customerType: "retail",
      priority: 3,
      status: TokenStatus.completed,
      position: 0,
      estimatedWaitTime: 0,
      notes: null,
      organizationId: "test-org-1",
      assignedCounterId: "counter-1",
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: "token-5",
      tokenNumber: "B001",
      customerType: "instant",
      priority: 1,
      status: TokenStatus.waiting,
      position: 1,
      estimatedWaitTime: 3,
      notes: "Organization 2 token",
      organizationId: "test-org-2",
      assignedCounterId: "counter-5",
      createdAt: new Date(),
      updatedAt: new Date(),
    },
  ],

  tokenConfigs: [
    {
      id: "config-1",
      name: "Standard Configuration",
      customerTypes: ["instant", "browser", "retail"],
      priorities: {
        instant: 1,
        browser: 2,
        retail: 3,
      },
      serviceTimes: {
        instant: 3,
        browser: 5,
        retail: 7,
      },
      organizationId: "test-org-1",
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: "config-2",
      name: "Fast Service Configuration",
      customerTypes: ["instant", "browser"],
      priorities: {
        instant: 1,
        browser: 1,
      },
      serviceTimes: {
        instant: 2,
        browser: 3,
      },
      organizationId: "test-org-2",
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
  ],
};

module.exports = TEST_FIXTURES;
```

### Test Data Generators

```javascript
// test-data-generators.js
const { faker } = require("@faker-js/faker");
const bcrypt = require("bcryptjs");
const { UserRole, TokenStatus } = require("@prisma/client");

class TestDataGenerator {
  static generateUser(overrides = {}) {
    const role =
      overrides.role || faker.helpers.arrayElement(Object.values(UserRole));
    const username = overrides.username || faker.internet.userName();

    return {
      id: faker.string.uuid(),
      username,
      email: overrides.email || faker.internet.email(),
      password: bcrypt.hashSync(overrides.password || "password123", 10),
      role,
      organizationId: overrides.organizationId || "test-org-1",
      isActive: overrides.isActive !== undefined ? overrides.isActive : true,
      createdAt: new Date(),
      updatedAt: new Date(),
      ...overrides,
    };
  }

  static generateCounter(overrides = {}) {
    return {
      id: faker.string.uuid(),
      name: overrides.name || faker.company.name() + " Counter",
      description: overrides.description || faker.lorem.sentence(),
      isActive: overrides.isActive !== undefined ? overrides.isActive : true,
      organizationId: overrides.organizationId || "test-org-1",
      assignedStaffId: overrides.assignedStaffId || null,
      createdAt: new Date(),
      updatedAt: new Date(),
      ...overrides,
    };
  }

  static generateToken(overrides = {}) {
    const customerType =
      overrides.customerType ||
      faker.helpers.arrayElement(["instant", "browser", "retail"]);
    const priority =
      overrides.priority || this.getPriorityForCustomerType(customerType);
    const status =
      overrides.status ||
      faker.helpers.arrayElement(Object.values(TokenStatus));

    return {
      id: faker.string.uuid(),
      tokenNumber: overrides.tokenNumber || this.generateTokenNumber(),
      customerType,
      priority,
      status,
      position: overrides.position || faker.number.int({ min: 0, max: 10 }),
      estimatedWaitTime:
        overrides.estimatedWaitTime || faker.number.int({ min: 1, max: 30 }),
      notes:
        overrides.notes ||
        (faker.datatype.boolean() ? faker.lorem.sentence() : null),
      organizationId: overrides.organizationId || "test-org-1",
      assignedCounterId: overrides.assignedCounterId || null,
      createdAt: new Date(),
      updatedAt: new Date(),
      ...overrides,
    };
  }

  static generateOrganization(overrides = {}) {
    return {
      id: faker.string.uuid(),
      name: overrides.name || faker.company.name(),
      settings: {
        maxQueueLength:
          overrides.maxQueueLength || faker.number.int({ min: 50, max: 200 }),
        averageServiceTime:
          overrides.averageServiceTime || faker.number.int({ min: 3, max: 10 }),
        notificationSettings: {
          email: faker.datatype.boolean(),
          sms: faker.datatype.boolean(),
          push: faker.datatype.boolean(),
        },
      },
      createdAt: new Date(),
      updatedAt: new Date(),
      ...overrides,
    };
  }

  static getPriorityForCustomerType(customerType) {
    const priorities = {
      instant: 1,
      browser: 2,
      retail: 3,
    };
    return priorities[customerType] || 3;
  }

  static generateTokenNumber() {
    const prefix = faker.helpers.arrayElement(["A", "B", "C"]);
    const number = faker.number
      .int({ min: 1, max: 999 })
      .toString()
      .padStart(3, "0");
    return `${prefix}${number}`;
  }

  static generateBulkUsers(count, overrides = {}) {
    return Array.from({ length: count }, () => this.generateUser(overrides));
  }

  static generateBulkCounters(count, overrides = {}) {
    return Array.from({ length: count }, () => this.generateCounter(overrides));
  }

  static generateBulkTokens(count, overrides = {}) {
    return Array.from({ length: count }, () => this.generateToken(overrides));
  }
}

module.exports = TestDataGenerator;
```

## 📱 Mobile App Test Data

### Mobile-Specific Test Data

```javascript
// mobile-test-data.js
const mobileTestData = {
  // Device configurations for testing
  devices: [
    {
      name: "iPhone 14",
      platform: "iOS",
      version: "16.0",
      capabilities: {
        platformName: "iOS",
        platformVersion: "16.0",
        deviceName: "iPhone 14",
        automationName: "XCUITest",
      },
    },
    {
      name: "iPhone 13",
      platform: "iOS",
      version: "15.0",
      capabilities: {
        platformName: "iOS",
        platformVersion: "15.0",
        deviceName: "iPhone 13",
        automationName: "XCUITest",
      },
    },
    {
      name: "Pixel 6",
      platform: "Android",
      version: "13.0",
      capabilities: {
        platformName: "Android",
        platformVersion: "13.0",
        deviceName: "Pixel 6",
        automationName: "UiAutomator2",
      },
    },
    {
      name: "Samsung Galaxy S21",
      platform: "Android",
      version: "12.0",
      capabilities: {
        platformName: "Android",
        platformVersion: "12.0",
        deviceName: "Samsung Galaxy S21",
        automationName: "UiAutomator2",
      },
    },
  ],

  // App-specific test data
  appConfigs: [
    {
      name: "iOS App",
      bundleId: "com.queueapp.mobile",
      appPath: "./apps/QueueManagement.app",
      capabilities: {
        platformName: "iOS",
        platformVersion: "16.0",
        deviceName: "iPhone 14",
        automationName: "XCUITest",
        app: "./apps/QueueManagement.app",
        bundleId: "com.queueapp.mobile",
      },
    },
    {
      name: "Android App",
      packageName: "com.queueapp.mobile",
      appPath: "./apps/QueueManagement.apk",
      capabilities: {
        platformName: "Android",
        platformVersion: "13.0",
        deviceName: "Pixel 6",
        automationName: "UiAutomator2",
        app: "./apps/QueueManagement.apk",
        appPackage: "com.queueapp.mobile",
        appActivity: "com.queueapp.mobile.MainActivity",
      },
    },
  ],

  // Mobile-specific test scenarios
  scenarios: {
    customerJourney: {
      tokenCreation: {
        instant: {
          customerType: "instant",
          priority: 1,
          notes: "Urgent service",
        },
        browser: {
          customerType: "browser",
          priority: 2,
          notes: "Online service",
        },
        retail: {
          customerType: "retail",
          priority: 3,
          notes: "In-store service",
        },
      },
      queueStatus: {
        waiting: { status: "waiting", position: 3, estimatedWait: 15 },
        called: { status: "called", position: 0, estimatedWait: 0 },
        completed: { status: "completed", position: 0, estimatedWait: 0 },
      },
    },
    staffOperations: {
      login: {
        valid: { username: "staff1", password: "staff123" },
        invalid: { username: "staff1", password: "wrongpassword" },
      },
      tokenProcessing: {
        callNext: { action: "call-next", expectedStatus: "called" },
        complete: { action: "complete", expectedStatus: "completed" },
        noShow: { action: "no-show", expectedStatus: "no-show" },
      },
    },
  },
};

module.exports = mobileTestData;
```

## 🌐 Web Application Test Data

### Web-Specific Test Data

```javascript
// web-test-data.js
const webTestData = {
  // Browser configurations
  browsers: [
    {
      name: "Chrome Mobile",
      platform: "Android",
      capabilities: {
        platformName: "Android",
        platformVersion: "13.0",
        deviceName: "Pixel 6",
        browserName: "Chrome",
        automationName: "UiAutomator2",
      },
    },
    {
      name: "Safari Mobile",
      platform: "iOS",
      capabilities: {
        platformName: "iOS",
        platformVersion: "16.0",
        deviceName: "iPhone 14",
        browserName: "Safari",
        automationName: "XCUITest",
      },
    },
    {
      name: "Firefox Mobile",
      platform: "Android",
      capabilities: {
        platformName: "Android",
        platformVersion: "13.0",
        deviceName: "Pixel 6",
        browserName: "Firefox",
        automationName: "UiAutomator2",
      },
    },
  ],

  // Viewport configurations
  viewports: [
    { name: "Mobile Portrait", width: 375, height: 667 },
    { name: "Mobile Landscape", width: 667, height: 375 },
    { name: "Tablet Portrait", width: 768, height: 1024 },
    { name: "Tablet Landscape", width: 1024, height: 768 },
  ],

  // PWA test data
  pwaFeatures: {
    offline: {
      cachedPages: ["/", "/queue-status", "/token-creation"],
      offlineMessage:
        "You are currently offline. Some features may be limited.",
    },
    notifications: {
      permission: "granted",
      types: ["queue-update", "token-called", "service-complete"],
    },
    installation: {
      prompt: "Install Queue Management App",
      manifest: {
        name: "Queue Management",
        short_name: "QueueApp",
        description: "Queue management system",
        start_url: "/",
        display: "standalone",
        theme_color: "#000000",
        background_color: "#ffffff",
      },
    },
  },

  // Responsive design test data
  responsiveElements: {
    navigation: {
      mobile: "#mobile-menu",
      desktop: "#desktop-menu",
      tablet: "#tablet-menu",
    },
    forms: {
      mobile: ".form-mobile",
      desktop: ".form-desktop",
    },
    buttons: {
      mobile: ".btn-mobile",
      desktop: ".btn-desktop",
    },
  },
};

module.exports = webTestData;
```

## 🔌 API Test Data

### API-Specific Test Data

```javascript
// api-test-data.js
const apiTestData = {
  // Authentication test data
  auth: {
    validCredentials: [
      { username: "admin", password: "admin123", role: "admin" },
      { username: "staff1", password: "staff123", role: "staff" },
      {
        username: "superadmin",
        password: "superadmin123",
        role: "super_admin",
      },
    ],
    invalidCredentials: [
      { username: "admin", password: "wrongpassword" },
      { username: "nonexistent", password: "password123" },
      { username: "", password: "admin123" },
      { username: "admin", password: "" },
    ],
    expiredTokens: [
      "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.expired.token",
      "invalid.token.format",
      "malformed.token",
    ],
  },

  // API endpoint test data
  endpoints: {
    auth: {
      login: { method: "POST", path: "/api/auth/login" },
      logout: { method: "POST", path: "/api/auth/logout" },
      me: { method: "GET", path: "/api/auth/me" },
    },
    users: {
      list: { method: "GET", path: "/api/auth/users" },
      create: { method: "POST", path: "/api/auth/users" },
      get: { method: "GET", path: "/api/auth/users/:id" },
      update: { method: "PATCH", path: "/api/auth/users/:id" },
      delete: { method: "DELETE", path: "/api/auth/users/:id" },
    },
    counters: {
      list: { method: "GET", path: "/api/counters" },
      create: { method: "POST", path: "/api/counters" },
      get: { method: "GET", path: "/api/counters/:id" },
      update: { method: "PUT", path: "/api/counters/:id" },
      delete: { method: "DELETE", path: "/api/counters/:id" },
    },
    tokens: {
      create: { method: "POST", path: "/api/tokens/public" },
      list: { method: "GET", path: "/api/tokens" },
      get: { method: "GET", path: "/api/tokens/:id" },
      update: { method: "PATCH", path: "/api/tokens/:id" },
      delete: { method: "DELETE", path: "/api/tokens/:id" },
    },
  },

  // Request/response test data
  requests: {
    createUser: {
      valid: {
        username: "testuser",
        email: "testuser@test.com",
        password: "password123",
        role: "staff",
      },
      invalid: {
        username: "",
        email: "invalid-email",
        password: "123",
        role: "invalid_role",
      },
    },
    createCounter: {
      valid: {
        name: "Test Counter",
        description: "Test counter description",
        isActive: true,
      },
      invalid: {
        name: "",
        description: null,
        isActive: "invalid",
      },
    },
    createToken: {
      valid: {
        customerType: "instant",
        priority: 1,
        notes: "Test token",
      },
      invalid: {
        customerType: "invalid",
        priority: 0,
        notes: null,
      },
    },
  },

  // WebSocket test data
  websocket: {
    events: {
      queueUpdate: {
        type: "queue_update",
        data: {
          queueLength: 5,
          averageWaitTime: 10,
          activeCounters: 3,
        },
      },
      tokenCalled: {
        type: "token_called",
        data: {
          tokenNumber: "A001",
          counterName: "Counter 1",
          position: 0,
        },
      },
      serviceComplete: {
        type: "service_complete",
        data: {
          tokenNumber: "A001",
          serviceTime: 5,
          counterName: "Counter 1",
        },
      },
    },
  },
};

module.exports = apiTestData;
```

## 🧹 Cleanup Procedures

### Test Data Cleanup

```javascript
// test-cleanup.js
const { PrismaClient } = require("@prisma/client");

class TestDataCleanup {
  constructor() {
    this.prisma = new PrismaClient({
      datasources: {
        db: {
          url: process.env.TEST_DATABASE_URL,
        },
      },
    });
  }

  async cleanupAll() {
    try {
      // Clean up in reverse dependency order
      await this.cleanupTokens();
      await this.cleanupCounters();
      await this.cleanupUsers();
      await this.cleanupOrganizations();
      await this.cleanupTokenConfigs();

      console.log("Test data cleanup completed successfully");
    } catch (error) {
      console.error("Error during test data cleanup:", error);
      throw error;
    } finally {
      await this.prisma.$disconnect();
    }
  }

  async cleanupTokens() {
    await this.prisma.token.deleteMany({
      where: {
        organizationId: {
          startsWith: "test-org-",
        },
      },
    });
    console.log("Tokens cleaned up");
  }

  async cleanupCounters() {
    await this.prisma.counter.deleteMany({
      where: {
        organizationId: {
          startsWith: "test-org-",
        },
      },
    });
    console.log("Counters cleaned up");
  }

  async cleanupUsers() {
    await this.prisma.user.deleteMany({
      where: {
        organizationId: {
          startsWith: "test-org-",
        },
      },
    });
    console.log("Users cleaned up");
  }

  async cleanupOrganizations() {
    await this.prisma.organization.deleteMany({
      where: {
        id: {
          startsWith: "test-org-",
        },
      },
    });
    console.log("Organizations cleaned up");
  }

  async cleanupTokenConfigs() {
    await this.prisma.tokenConfig.deleteMany({
      where: {
        organizationId: {
          startsWith: "test-org-",
        },
      },
    });
    console.log("Token configs cleaned up");
  }

  async cleanupByTestSuite(testSuiteId) {
    // Clean up data specific to a test suite
    await this.prisma.token.deleteMany({
      where: {
        notes: {
          contains: `test-suite-${testSuiteId}`,
        },
      },
    });
  }

  async cleanupBySession(sessionId) {
    // Clean up data specific to a test session
    await this.prisma.token.deleteMany({
      where: {
        notes: {
          contains: `session-${sessionId}`,
        },
      },
    });
  }
}

module.exports = TestDataCleanup;
```

### Appium Session Cleanup

```javascript
// appium-cleanup.js
class AppiumSessionCleanup {
  static async cleanupSession(driver) {
    try {
      // Take screenshot before cleanup
      await this.takeScreenshot(driver, "final-state");

      // Clear app data if needed
      await this.clearAppData(driver);

      // Close app
      await driver.closeApp();

      // Delete session
      await driver.deleteSession();

      console.log("Appium session cleaned up successfully");
    } catch (error) {
      console.error("Error during Appium session cleanup:", error);
      // Force cleanup
      try {
        await driver.deleteSession();
      } catch (forceError) {
        console.error("Force cleanup failed:", forceError);
      }
    }
  }

  static async takeScreenshot(driver, name) {
    try {
      const screenshot = await driver.takeScreenshot();
      const fs = require("fs");
      const path = require("path");

      const screenshotDir = path.join(__dirname, "../screenshots");
      if (!fs.existsSync(screenshotDir)) {
        fs.mkdirSync(screenshotDir, { recursive: true });
      }

      const filename = `${name}-${Date.now()}.png`;
      const filepath = path.join(screenshotDir, filename);

      fs.writeFileSync(filepath, screenshot, "base64");
      console.log(`Screenshot saved: ${filepath}`);
    } catch (error) {
      console.error("Error taking screenshot:", error);
    }
  }

  static async clearAppData(driver) {
    try {
      // Clear app data for Android
      if (driver.capabilities.platformName === "Android") {
        await driver.execute("mobile: shell", {
          command: "pm clear com.queueapp.mobile",
        });
      }

      // Reset app for iOS
      if (driver.capabilities.platformName === "iOS") {
        await driver.execute("mobile: terminateApp", {
          bundleId: "com.queueapp.mobile",
        });
      }
    } catch (error) {
      console.error("Error clearing app data:", error);
    }
  }
}

module.exports = AppiumSessionCleanup;
```

## 🔒 Data Isolation

### Test Data Isolation Strategy

```javascript
// test-isolation.js
class TestDataIsolation {
  constructor() {
    this.testSuiteId = this.generateTestSuiteId();
    this.sessionId = this.generateSessionId();
  }

  generateTestSuiteId() {
    return `test-suite-${Date.now()}-${Math.random()
      .toString(36)
      .substr(2, 9)}`;
  }

  generateSessionId() {
    return `session-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }

  // Create isolated test data
  createIsolatedUser(overrides = {}) {
    return {
      ...overrides,
      username: `${overrides.username || "testuser"}-${this.testSuiteId}`,
      email: `${overrides.email || "test@test.com"}-${this.testSuiteId}`,
      notes: `Test suite: ${this.testSuiteId}, Session: ${this.sessionId}`,
    };
  }

  createIsolatedCounter(overrides = {}) {
    return {
      ...overrides,
      name: `${overrides.name || "Test Counter"}-${this.testSuiteId}`,
      description: `Test suite: ${this.testSuiteId}, Session: ${this.sessionId}`,
    };
  }

  createIsolatedToken(overrides = {}) {
    return {
      ...overrides,
      notes: `Test suite: ${this.testSuiteId}, Session: ${this.sessionId}`,
    };
  }

  // Clean up isolated data
  async cleanupIsolatedData() {
    const cleanup = new (require("./test-cleanup"))();

    // Clean up by test suite
    await cleanup.cleanupByTestSuite(this.testSuiteId);

    // Clean up by session
    await cleanup.cleanupBySession(this.sessionId);
  }
}

module.exports = TestDataIsolation;
```

## ⚡ Performance Test Data

### Large Dataset Generation

```javascript
// performance-test-data.js
const TestDataGenerator = require("./test-data-generators");

class PerformanceTestData {
  static async generateLargeDataset() {
    console.log("Generating large dataset for performance testing...");

    // Generate 1000 users
    const users = TestDataGenerator.generateBulkUsers(1000, {
      organizationId: "perf-test-org",
    });

    // Generate 100 counters
    const counters = TestDataGenerator.generateBulkCounters(100, {
      organizationId: "perf-test-org",
    });

    // Generate 5000 tokens
    const tokens = TestDataGenerator.generateBulkTokens(5000, {
      organizationId: "perf-test-org",
    });

    console.log(`Generated:
      - ${users.length} users
      - ${counters.length} counters
      - ${tokens.length} tokens`);

    return { users, counters, tokens };
  }

  static async generateStressTestData() {
    console.log("Generating stress test data...");

    // Generate data for stress testing
    const stressData = {
      users: TestDataGenerator.generateBulkUsers(5000),
      counters: TestDataGenerator.generateBulkCounters(500),
      tokens: TestDataGenerator.generateBulkTokens(10000),
    };

    return stressData;
  }

  static async generateConcurrentTestData() {
    console.log("Generating concurrent test data...");

    // Generate data for concurrent testing
    const concurrentData = {
      organizations: Array.from({ length: 10 }, (_, i) =>
        TestDataGenerator.generateOrganization({
          id: `concurrent-org-${i}`,
          name: `Concurrent Organization ${i}`,
        })
      ),
      users: Array.from({ length: 100 }, (_, i) =>
        TestDataGenerator.generateUser({
          organizationId: `concurrent-org-${i % 10}`,
          username: `concurrent-user-${i}`,
        })
      ),
    };

    return concurrentData;
  }
}

module.exports = PerformanceTestData;
```

---

**Last Updated**: $(date)
**Version**: 1.0.0
**Maintainer**: Development Team
