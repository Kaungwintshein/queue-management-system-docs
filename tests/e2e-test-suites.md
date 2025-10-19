# End-to-End Test Suites Documentation

This document provides comprehensive documentation for all End-to-End (E2E) test suites in the Queue Management System. E2E tests simulate real user interactions across the entire system, ensuring that all components work together correctly.

## 🚀 Testing Framework: Appium

This project uses **Appium** for comprehensive E2E testing, providing:

- **Cross-platform testing**: Web, iOS, and Android applications
- **Real device testing**: Physical devices and simulators/emulators
- **Native app testing**: Full access to device capabilities
- **Web testing**: Mobile web browsers and responsive design
- **API integration**: Combined mobile and backend testing

## 📋 Table of Contents

1. [Test Suite Overview](#test-suite-overview)
2. [Appium Configuration](#appium-configuration)
3. [Mobile App Test Suites](#mobile-app-test-suites)
4. [Web Application Test Suites](#web-application-test-suites)
5. [Cross-Platform Test Suites](#cross-platform-test-suites)
6. [Authentication & Authorization Tests](#authentication--authorization-tests)
7. [Admin Portal Test Suite](#admin-portal-test-suite)
8. [Customer Journey Test Suite](#customer-journey-test-suite)
9. [Staff Interface Test Suite](#staff-interface-test-suite)
10. [Queue Management Test Suite](#queue-management-test-suite)
11. [API Integration Test Suite](#api-integration-test-suite)
12. [Performance & Load Test Suite](#performance--load-test-suite)
13. [Error Handling Test Suite](#error-handling-test-suite)
14. [Test Data Management](#test-data-management)
15. [Test Execution Guidelines](#test-execution-guidelines)

## 🎯 Test Suite Overview

### Test Categories

- **Functional Tests**: Core business logic and user workflows
- **Integration Tests**: Component interaction and data flow
- **Security Tests**: Authentication, authorization, and data protection
- **Performance Tests**: Load, stress, and scalability testing
- **Usability Tests**: User experience and interface validation

### Test Environment

- **Database**: Isolated test database with predictable data
- **Services**: Mock external services and dependencies
- **Cleanup**: Automatic test data cleanup after execution
- **Isolation**: Each test suite runs independently
- **Devices**: iOS simulators, Android emulators, and physical devices
- **Browsers**: Chrome, Safari, Firefox on mobile and desktop

## ⚙️ Appium Configuration

### Appium Server Setup

```javascript
// appium.config.js
const { config } = require("./wdio.conf");

exports.config = {
  ...config,
  services: [
    [
      "appium",
      {
        args: {
          address: "localhost",
          port: 4723,
          log: "./logs/appium.log",
        },
      },
    ],
  ],
  capabilities: [
    // iOS Configuration
    {
      platformName: "iOS",
      platformVersion: "16.0",
      deviceName: "iPhone 14",
      automationName: "XCUITest",
      app: "./apps/QueueManagement.app",
      bundleId: "com.queueapp.mobile",
      noReset: false,
      fullReset: true,
    },
    // Android Configuration
    {
      platformName: "Android",
      platformVersion: "13.0",
      deviceName: "Pixel 6",
      automationName: "UiAutomator2",
      app: "./apps/QueueManagement.apk",
      appPackage: "com.queueapp.mobile",
      appActivity: "com.queueapp.mobile.MainActivity",
      noReset: false,
      fullReset: true,
    },
    // Web Configuration
    {
      platformName: "iOS",
      platformVersion: "16.0",
      deviceName: "iPhone 14",
      browserName: "Safari",
      automationName: "XCUITest",
    },
    {
      platformName: "Android",
      platformVersion: "13.0",
      deviceName: "Pixel 6",
      browserName: "Chrome",
      automationName: "UiAutomator2",
    },
  ],
};
```

### Test Configuration

```javascript
// wdio.conf.js
exports.config = {
  runner: "local",
  specs: [
    "./tests/e2e/mobile/**/*.test.js",
    "./tests/e2e/web/**/*.test.js",
    "./tests/e2e/cross-platform/**/*.test.js",
  ],
  maxInstances: 3,
  capabilities: [],
  logLevel: "info",
  bail: 0,
  baseUrl: "http://localhost:3000",
  waitforTimeout: 10000,
  connectionRetryTimeout: 120000,
  connectionRetryCount: 3,
  framework: "mocha",
  reporters: [
    "spec",
    [
      "allure",
      {
        outputDir: "allure-results",
        disableWebdriverScreenshots: false,
      },
    ],
  ],
  mochaOpts: {
    ui: "bdd",
    timeout: 60000,
  },
  beforeSession: function (config, capabilities, specs) {
    // Setup before each session
  },
  before: function (capabilities, specs) {
    // Setup before each test
  },
  after: function (result, capabilities, specs) {
    // Cleanup after each test
  },
  afterSession: function (config, capabilities, specs) {
    // Cleanup after each session
  },
};
```

## 📱 Mobile App Test Suites

### Test Suite: `mobile-app-e2e.test.js`

#### iOS App Tests

```javascript
describe("iOS Mobile App Tests", () => {
  let driver;

  before(async () => {
    driver = await remote({
      hostname: "localhost",
      port: 4723,
      path: "/wd/hub",
      capabilities: {
        platformName: "iOS",
        platformVersion: "16.0",
        deviceName: "iPhone 14",
        automationName: "XCUITest",
        app: "./apps/QueueManagement.app",
        bundleId: "com.queueapp.mobile",
      },
    });
  });

  after(async () => {
    await driver.deleteSession();
  });

  describe("Customer Mobile App", () => {
    it("should allow customer to create token", async () => {
      // Navigate to token creation screen
      await driver.$("~token-creation-button").click();

      // Select service type
      await driver.$("~instant-service").click();

      // Add notes if needed
      await driver.$("~notes-input").setValue("Need urgent service");

      // Submit token creation
      await driver.$("~create-token-button").click();

      // Verify token creation success
      const successMessage = await driver.$("~success-message");
      await expect(successMessage).toBeDisplayed();

      // Verify token number is displayed
      const tokenNumber = await driver.$("~token-number");
      await expect(tokenNumber).toBeDisplayed();
    });

    it("should display queue status", async () => {
      // Navigate to queue status
      await driver.$("~queue-status-tab").click();

      // Verify current position
      const position = await driver.$("~current-position");
      await expect(position).toBeDisplayed();

      // Verify estimated wait time
      const waitTime = await driver.$("~estimated-wait-time");
      await expect(waitTime).toBeDisplayed();

      // Verify counter availability
      const counters = await driver.$$("~counter-item");
      await expect(counters.length).toBeGreaterThan(0);
    });

    it("should receive push notifications", async () => {
      // Enable notifications
      await driver.$("~enable-notifications").click();

      // Verify notification permission dialog
      const permissionDialog = await driver.$("~notification-permission");
      await expect(permissionDialog).toBeDisplayed();

      // Accept notifications
      await driver.$("~allow-notifications").click();

      // Verify notification settings updated
      const notificationStatus = await driver.$("~notification-status");
      await expect(notificationStatus).toHaveText("Enabled");
    });

    it("should handle offline mode", async () => {
      // Simulate network disconnection
      await driver.toggleAirplaneMode();

      // Try to create token
      await driver.$("~token-creation-button").click();

      // Verify offline message
      const offlineMessage = await driver.$("~offline-message");
      await expect(offlineMessage).toBeDisplayed();

      // Reconnect network
      await driver.toggleAirplaneMode();

      // Verify reconnection
      const onlineStatus = await driver.$("~connection-status");
      await expect(onlineStatus).toHaveText("Online");
    });
  });

  describe("Staff Mobile App", () => {
    it("should allow staff login", async () => {
      // Enter credentials
      await driver.$("~username-input").setValue("staff");
      await driver.$("~password-input").setValue("staff123");

      // Login
      await driver.$("~login-button").click();

      // Verify dashboard
      const dashboard = await driver.$("~staff-dashboard");
      await expect(dashboard).toBeDisplayed();
    });

    it("should display assigned counter", async () => {
      // Verify counter assignment
      const assignedCounter = await driver.$("~assigned-counter");
      await expect(assignedCounter).toBeDisplayed();

      // Verify queue information
      const queueInfo = await driver.$("~queue-information");
      await expect(queueInfo).toBeDisplayed();
    });

    it("should call next token", async () => {
      // Call next token
      await driver.$("~call-next-button").click();

      // Verify token called
      const calledToken = await driver.$("~called-token");
      await expect(calledToken).toBeDisplayed();

      // Verify customer notification sent
      const notificationSent = await driver.$("~notification-sent");
      await expect(notificationSent).toBeDisplayed();
    });

    it("should complete service", async () => {
      // Complete current service
      await driver.$("~complete-service-button").click();

      // Verify completion
      const completionMessage = await driver.$("~completion-message");
      await expect(completionMessage).toBeDisplayed();

      // Verify statistics updated
      const stats = await driver.$("~service-statistics");
      await expect(stats).toBeDisplayed();
    });
  });
});
```

#### Android App Tests

```javascript
describe("Android Mobile App Tests", () => {
  let driver;

  before(async () => {
    driver = await remote({
      hostname: "localhost",
      port: 4723,
      path: "/wd/hub",
      capabilities: {
        platformName: "Android",
        platformVersion: "13.0",
        deviceName: "Pixel 6",
        automationName: "UiAutomator2",
        app: "./apps/QueueManagement.apk",
        appPackage: "com.queueapp.mobile",
        appActivity: "com.queueapp.mobile.MainActivity",
      },
    });
  });

  after(async () => {
    await driver.deleteSession();
  });

  describe("Customer Android App", () => {
    it("should handle different screen sizes", async () => {
      // Test on different device orientations
      await driver.setOrientation("landscape");

      // Verify layout adaptation
      const landscapeLayout = await driver.$("id=main-container");
      await expect(landscapeLayout).toBeDisplayed();

      // Return to portrait
      await driver.setOrientation("portrait");

      // Verify portrait layout
      const portraitLayout = await driver.$("id=main-container");
      await expect(portraitLayout).toBeDisplayed();
    });

    it("should handle Android back button", async () => {
      // Navigate to token creation
      await driver.$("id=token-creation-button").click();

      // Use Android back button
      await driver.back();

      // Verify return to main screen
      const mainScreen = await driver.$("id=main-screen");
      await expect(mainScreen).toBeDisplayed();
    });

    it("should handle Android permissions", async () => {
      // Request location permission
      await driver.$("id=location-permission-button").click();

      // Handle permission dialog
      const permissionDialog = await driver.$("id=permission-dialog");
      if (await permissionDialog.isDisplayed()) {
        await driver.$("id=allow-button").click();
      }

      // Verify permission granted
      const locationStatus = await driver.$("id=location-status");
      await expect(locationStatus).toHaveText("Enabled");
    });
  });
});
```

## 🌐 Web Application Test Suites

### Test Suite: `web-app-e2e.test.js`

#### Mobile Web Tests

```javascript
describe("Mobile Web Application Tests", () => {
  let driver;

  before(async () => {
    driver = await remote({
      hostname: "localhost",
      port: 4723,
      path: "/wd/hub",
      capabilities: {
        platformName: "iOS",
        platformVersion: "16.0",
        deviceName: "iPhone 14",
        browserName: "Safari",
        automationName: "XCUITest",
      },
    });

    await driver.url("http://localhost:3000");
  });

  after(async () => {
    await driver.deleteSession();
  });

  describe("Responsive Design Tests", () => {
    it("should adapt to mobile screen size", async () => {
      // Verify mobile layout
      const mobileMenu = await driver.$("#mobile-menu");
      await expect(mobileMenu).toBeDisplayed();

      // Verify touch-friendly buttons
      const buttons = await driver.$$(".btn-mobile");
      await expect(buttons.length).toBeGreaterThan(0);
    });

    it("should handle touch interactions", async () => {
      // Test touch scrolling
      await driver.touchAction([
        { action: "press", x: 200, y: 400 },
        { action: "moveTo", x: 200, y: 200 },
        { action: "release" },
      ]);

      // Verify scroll worked
      const scrolledElement = await driver.$("#scrolled-content");
      await expect(scrolledElement).toBeDisplayed();
    });

    it("should handle mobile gestures", async () => {
      // Test swipe gesture
      await driver.touchAction([
        { action: "press", x: 300, y: 300 },
        { action: "moveTo", x: 100, y: 300 },
        { action: "release" },
      ]);

      // Verify swipe action
      const swipedContent = await driver.$("#swiped-content");
      await expect(swipedContent).toBeDisplayed();
    });
  });

  describe("PWA Features", () => {
    it("should support offline functionality", async () => {
      // Enable offline mode
      await driver.execute("mobile: setNetworkConnection", {
        type: "airplane",
      });

      // Verify offline indicator
      const offlineIndicator = await driver.$("#offline-indicator");
      await expect(offlineIndicator).toBeDisplayed();

      // Test cached content
      const cachedContent = await driver.$("#cached-content");
      await expect(cachedContent).toBeDisplayed();
    });

    it("should support push notifications", async () => {
      // Request notification permission
      await driver.$("#enable-notifications").click();

      // Handle browser permission dialog
      const permissionDialog = await driver.$("#notification-permission");
      if (await permissionDialog.isDisplayed()) {
        await driver.$("#allow-notifications").click();
      }

      // Verify notification enabled
      const notificationStatus = await driver.$("#notification-status");
      await expect(notificationStatus).toHaveText("Enabled");
    });

    it("should support app installation", async () => {
      // Check for install prompt
      const installPrompt = await driver.$("#install-prompt");
      if (await installPrompt.isDisplayed()) {
        await driver.$("#install-button").click();

        // Verify installation
        const installedIndicator = await driver.$("#installed-indicator");
        await expect(installedIndicator).toBeDisplayed();
      }
    });
  });
});
```

## 🔄 Cross-Platform Test Suites

### Test Suite: `cross-platform-e2e.test.js`

#### Unified Test Scenarios

```javascript
describe("Cross-Platform E2E Tests", () => {
  const platforms = [
    {
      name: "iOS App",
      capabilities: {
        platformName: "iOS",
        platformVersion: "16.0",
        deviceName: "iPhone 14",
        automationName: "XCUITest",
        app: "./apps/QueueManagement.app",
      },
    },
    {
      name: "Android App",
      capabilities: {
        platformName: "Android",
        platformVersion: "13.0",
        deviceName: "Pixel 6",
        automationName: "UiAutomator2",
        app: "./apps/QueueManagement.apk",
      },
    },
    {
      name: "iOS Web",
      capabilities: {
        platformName: "iOS",
        platformVersion: "16.0",
        deviceName: "iPhone 14",
        browserName: "Safari",
        automationName: "XCUITest",
      },
    },
    {
      name: "Android Web",
      capabilities: {
        platformName: "Android",
        platformVersion: "13.0",
        deviceName: "Pixel 6",
        browserName: "Chrome",
        automationName: "UiAutomator2",
      },
    },
  ];

  platforms.forEach((platform) => {
    describe(`${platform.name} Tests`, () => {
      let driver;

      before(async () => {
        driver = await remote({
          hostname: "localhost",
          port: 4723,
          path: "/wd/hub",
          capabilities: platform.capabilities,
        });

        if (platform.name.includes("Web")) {
          await driver.url("http://localhost:3000");
        }
      });

      after(async () => {
        await driver.deleteSession();
      });

      it("should complete customer journey", async () => {
        // Navigate to token creation
        const createTokenBtn = await driver.$("#create-token-button");
        await createTokenBtn.click();

        // Select service type
        const instantService = await driver.$("#instant-service");
        await instantService.click();

        // Create token
        const submitBtn = await driver.$("#submit-token");
        await submitBtn.click();

        // Verify success
        const successMessage = await driver.$("#success-message");
        await expect(successMessage).toBeDisplayed();

        // Check queue position
        const queuePosition = await driver.$("#queue-position");
        await expect(queuePosition).toBeDisplayed();
      });

      it("should handle authentication", async () => {
        // Navigate to login
        const loginBtn = await driver.$("#login-button");
        await loginBtn.click();

        // Enter credentials
        const usernameInput = await driver.$("#username");
        await usernameInput.setValue("testuser");

        const passwordInput = await driver.$("#password");
        await passwordInput.setValue("testpass");

        // Submit login
        const submitLogin = await driver.$("#submit-login");
        await submitLogin.click();

        // Verify dashboard
        const dashboard = await driver.$("#dashboard");
        await expect(dashboard).toBeDisplayed();
      });

      it("should handle real-time updates", async () => {
        // Wait for WebSocket connection
        const connectionStatus = await driver.$("#connection-status");
        await expect(connectionStatus).toHaveText("Connected");

        // Verify real-time updates
        const queueUpdates = await driver.$("#queue-updates");
        await expect(queueUpdates).toBeDisplayed();
      });
    });
  });
});
```

#### Platform-Specific Adaptations

```javascript
describe("Platform-Specific Adaptations", () => {
  it("should handle iOS-specific features", async () => {
    const driver = await remote({
      hostname: "localhost",
      port: 4723,
      path: "/wd/hub",
      capabilities: {
        platformName: "iOS",
        platformVersion: "16.0",
        deviceName: "iPhone 14",
        automationName: "XCUITest",
        app: "./apps/QueueManagement.app",
      },
    });

    try {
      // Test iOS-specific gestures
      await driver.touchAction([
        { action: "press", x: 200, y: 400 },
        { action: "moveTo", x: 200, y: 200 },
        { action: "release" },
      ]);

      // Test iOS notifications
      await driver.$("#enable-notifications").click();

      // Handle iOS permission dialog
      const permissionAlert = await driver.$("~Allow");
      if (await permissionAlert.isDisplayed()) {
        await permissionAlert.click();
      }
    } finally {
      await driver.deleteSession();
    }
  });

  it("should handle Android-specific features", async () => {
    const driver = await remote({
      hostname: "localhost",
      port: 4723,
      path: "/wd/hub",
      capabilities: {
        platformName: "Android",
        platformVersion: "13.0",
        deviceName: "Pixel 6",
        automationName: "UiAutomator2",
        app: "./apps/QueueManagement.apk",
      },
    });

    try {
      // Test Android back button
      await driver.$("id=some-screen").click();
      await driver.back();

      // Test Android permissions
      await driver.$("id=request-permission").click();

      // Handle Android permission dialog
      const allowButton = await driver.$(
        "id=com.android.packageinstaller:id/permission_allow_button"
      );
      if (await allowButton.isDisplayed()) {
        await allowButton.click();
      }
    } finally {
      await driver.deleteSession();
    }
  });
});
```

## 🔐 Authentication & Authorization Tests

### Test Suite: `auth-e2e.test.ts`

#### Test Scenarios

**1. User Login Flow**

```typescript
describe("User Login Flow", () => {
  it("should allow valid user login", async () => {
    // Test valid credentials
    // Verify JWT token generation
    // Check user session creation
  });

  it("should reject invalid credentials", async () => {
    // Test invalid username/password
    // Verify error responses
    // Check security measures
  });

  it("should handle account lockout", async () => {
    // Test multiple failed login attempts
    // Verify account lockout mechanism
    // Test unlock procedures
  });
});
```

**2. Role-Based Access Control**

```typescript
describe("Role-Based Access Control", () => {
  it("should enforce super admin permissions", async () => {
    // Test super admin access to all features
    // Verify unrestricted permissions
  });

  it("should enforce admin permissions", async () => {
    // Test admin access to management features
    // Verify restricted access to system settings
  });

  it("should enforce staff permissions", async () => {
    // Test staff access to operational features
    // Verify restricted access to admin features
  });
});
```

**3. Session Management**

```typescript
describe("Session Management", () => {
  it("should handle token expiration", async () => {
    // Test token refresh mechanism
    // Verify automatic logout on expiration
  });

  it("should handle concurrent sessions", async () => {
    // Test multiple device login
    // Verify session conflict resolution
  });

  it("should handle logout", async () => {
    // Test proper session termination
    // Verify token invalidation
  });
});
```

### Test Data Requirements

- Test users with different roles (super_admin, admin, staff)
- Valid and invalid credential combinations
- Expired and valid JWT tokens
- Session data for concurrent testing

## 🏢 Admin Portal Test Suite

### Test Suite: `admin-portal-e2e.test.ts`

#### Test Scenarios

**1. Complete Admin Workflow**

```typescript
describe("Complete Admin Workflow", () => {
  it("should simulate complete admin portal usage", async () => {
    // Step 1: Admin login
    // Step 2: View dashboard data
    // Step 3: Create new staff user
    // Step 4: Create new counter
    // Step 5: Assign staff to counter
    // Step 6: Update user information
    // Step 7: Update counter information
    // Step 8: Manage user status (ban/reactivate)
    // Step 9: Reset user password
    // Step 10: Unassign staff from counter
    // Step 11: Delete counter
    // Step 12: Delete user
    // Step 13: Logout
  });
});
```

**2. User Management Operations**

```typescript
describe("User Management Operations", () => {
  it("should create new users", async () => {
    // Test user creation with valid data
    // Verify user appears in user list
    // Check role assignment
  });

  it("should update user information", async () => {
    // Test user profile updates
    // Verify data persistence
    // Check validation rules
  });

  it("should manage user status", async () => {
    // Test user ban/unban
    // Test user activation/deactivation
    // Verify status changes
  });

  it("should reset user passwords", async () => {
    // Test password reset functionality
    // Verify temporary password generation
    // Check password change requirements
  });

  it("should delete users", async () => {
    // Test user deletion
    // Verify cascade deletion rules
    // Check data cleanup
  });
});
```

**3. Counter Management Operations**

```typescript
describe("Counter Management Operations", () => {
  it("should create new counters", async () => {
    // Test counter creation
    // Verify counter appears in list
    // Check default settings
  });

  it("should assign staff to counters", async () => {
    // Test staff assignment
    // Verify assignment persistence
    // Check conflict resolution
  });

  it("should update counter information", async () => {
    // Test counter updates
    // Verify data persistence
    // Check validation rules
  });

  it("should manage counter status", async () => {
    // Test counter activation/deactivation
    // Verify status changes
    // Check impact on queue
  });

  it("should delete counters", async () => {
    // Test counter deletion
    // Verify cleanup of assignments
    // Check impact on active tokens
  });
});
```

**4. Role Management Operations**

```typescript
describe("Role Management Operations", () => {
  it("should view role information", async () => {
    // Test role listing
    // Verify permission details
    // Check user counts per role
  });

  it("should manage role assignments", async () => {
    // Test role changes for users
    // Verify permission updates
    // Check access control changes
  });
});
```

**5. Multi-User Admin Scenarios**

```typescript
describe("Multi-User Admin Scenarios", () => {
  it("should handle multiple admins working simultaneously", async () => {
    // Test concurrent admin operations
    // Verify data consistency
    // Check conflict resolution
  });

  it("should handle admin permission changes", async () => {
    // Test role escalation/de-escalation
    // Verify immediate permission changes
    // Check session updates
  });
});
```

### Test Data Requirements

- Multiple admin users with different roles
- Test counters in various states
- Test users with different roles and statuses
- Sample organization data

## 👥 Customer Journey Test Suite

### Test Suite: `customer-journey-e2e.test.ts`

#### Test Scenarios

**1. Token Creation Flow**

```typescript
describe("Token Creation Flow", () => {
  it("should create instant service token", async () => {
    // Test instant token creation
    // Verify queue position assignment
    // Check estimated wait time
  });

  it("should create browser service token", async () => {
    // Test browser token creation
    // Verify priority handling
    // Check queue management
  });

  it("should create retail service token", async () => {
    // Test retail token creation
    // Verify standard priority
    // Check queue integration
  });

  it("should handle token creation with notes", async () => {
    // Test token creation with additional notes
    // Verify note persistence
    // Check display in staff interface
  });
});
```

**2. Queue Status Monitoring**

```typescript
describe("Queue Status Monitoring", () => {
  it("should display current queue status", async () => {
    // Test queue status retrieval
    // Verify real-time updates
    // Check position tracking
  });

  it("should show estimated wait times", async () => {
    // Test wait time calculation
    // Verify accuracy of estimates
    // Check dynamic updates
  });

  it("should display counter availability", async () => {
    // Test counter status display
    // Verify staff assignments
    // Check service availability
  });
});
```

**3. Counter Selection Flow**

```typescript
describe("Counter Selection Flow", () => {
  it("should display available counters", async () => {
    // Test counter listing
    // Verify active counter display
    // Check staff assignments
  });

  it("should allow counter selection", async () => {
    // Test counter selection process
    // Verify selection persistence
    // Check queue assignment
  });

  it("should handle counter unavailability", async () => {
    // Test inactive counter handling
    // Verify error messages
    // Check alternative options
  });
});
```

**4. Token Status Updates**

```typescript
describe("Token Status Updates", () => {
  it("should receive real-time status updates", async () => {
    // Test WebSocket connections
    // Verify status change notifications
    // Check update frequency
  });

  it("should handle token completion", async () => {
    // Test service completion flow
    // Verify final status updates
    // Check completion notifications
  });

  it("should handle token cancellation", async () => {
    // Test token cancellation
    // Verify queue position updates
    // Check cleanup procedures
  });
});
```

### Test Data Requirements

- Active counters with assigned staff
- Various token types and priorities
- Queue with multiple tokens
- WebSocket connection test data

## 👨‍💼 Staff Interface Test Suite

### Test Suite: `staff-interface-e2e.test.ts`

#### Test Scenarios

**1. Staff Login and Dashboard**

```typescript
describe("Staff Login and Dashboard", () => {
  it("should allow staff login", async () => {
    // Test staff authentication
    // Verify dashboard access
    // Check assigned counter display
  });

  it("should display assigned counter information", async () => {
    // Test counter assignment display
    // Verify queue information
    // Check current token status
  });

  it("should show queue statistics", async () => {
    // Test queue metrics display
    // Verify real-time updates
    // Check performance indicators
  });
});
```

**2. Token Processing Operations**

```typescript
describe("Token Processing Operations", () => {
  it("should call next token in queue", async () => {
    // Test next token calling
    // Verify queue position updates
    // Check notification system
  });

  it("should complete token service", async () => {
    // Test service completion
    // Verify token status update
    // Check completion metrics
  });

  it("should mark token as no-show", async () => {
    // Test no-show handling
    // Verify queue cleanup
    // Check statistics updates
  });

  it("should recall previous token", async () => {
    // Test token recall functionality
    // Verify queue management
    // Check customer notifications
  });
});
```

**3. Queue Management**

```typescript
describe("Queue Management", () => {
  it("should handle queue prioritization", async () => {
    // Test priority token handling
    // Verify queue order maintenance
    // Check fairness algorithms
  });

  it("should manage queue overflow", async () => {
    // Test high queue volume
    // Verify system stability
    // Check performance metrics
  });

  it("should handle counter switching", async () => {
    // Test staff counter changes
    // Verify queue reassignment
    // Check continuity maintenance
  });
});
```

**4. Service Analytics**

```typescript
describe("Service Analytics", () => {
  it("should track service times", async () => {
    // Test service duration tracking
    // Verify analytics accuracy
    // Check performance metrics
  });

  it("should generate service reports", async () => {
    // Test report generation
    // Verify data accuracy
    // Check export functionality
  });

  it("should display performance metrics", async () => {
    // Test metrics display
    // Verify real-time updates
    // Check trend analysis
  });
});
```

### Test Data Requirements

- Staff users with assigned counters
- Active queue with multiple tokens
- Various token statuses and priorities
- Historical service data

## 🔄 Queue Management Test Suite

### Test Suite: `queue-management-e2e.test.ts`

#### Test Scenarios

**1. Token Lifecycle Management**

```typescript
describe("Token Lifecycle Management", () => {
  it("should handle complete token lifecycle", async () => {
    // Test token creation → queuing → calling → service → completion
    // Verify status transitions
    // Check data consistency
  });

  it("should handle token cancellation", async () => {
    // Test token cancellation at various stages
    // Verify queue position updates
    // Check cleanup procedures
  });

  it("should handle token expiration", async () => {
    // Test token timeout handling
    // Verify automatic cleanup
    // Check queue rebalancing
  });
});
```

**2. Priority Management**

```typescript
describe("Priority Management", () => {
  it("should handle instant priority tokens", async () => {
    // Test instant service priority
    // Verify queue jumping
    // Check fairness maintenance
  });

  it("should handle browser priority tokens", async () => {
    // Test browser service priority
    // Verify queue position
    // Check service order
  });

  it("should handle retail priority tokens", async () => {
    // Test standard retail priority
    // Verify normal queue flow
    // Check service fairness
  });
});
```

**3. Queue Balancing**

```typescript
describe("Queue Balancing", () => {
  it("should balance load across counters", async () => {
    // Test automatic load balancing
    // Verify counter utilization
    // Check efficiency metrics
  });

  it("should handle counter failures", async () => {
    // Test counter unavailability
    // Verify queue redistribution
    // Check service continuity
  });

  it("should handle staff changes", async () => {
    // Test staff reassignment
    // Verify queue continuity
    // Check service disruption
  });
});
```

**4. Real-time Updates**

```typescript
describe("Real-time Updates", () => {
  it("should broadcast queue updates", async () => {
    // Test WebSocket broadcasting
    // Verify update delivery
    // Check client synchronization
  });

  it("should handle connection failures", async () => {
    // Test WebSocket reconnection
    // Verify data recovery
    // Check state synchronization
  });

  it("should manage update frequency", async () => {
    // Test update throttling
    // Verify performance optimization
    // Check data freshness
  });
});
```

### Test Data Requirements

- Multiple active counters
- Various token types and priorities
- WebSocket connection test scenarios
- Load balancing test data

## 🔌 API Integration Test Suite

### Test Suite: `api-integration-e2e.test.ts`

#### Test Scenarios

**1. REST API Endpoints**

```typescript
describe("REST API Endpoints", () => {
  it("should test all authentication endpoints", async () => {
    // POST /api/auth/login
    // POST /api/auth/logout
    // GET /api/auth/me
    // POST /api/auth/register
    // POST /api/auth/change-password
  });

  it("should test all user management endpoints", async () => {
    // GET /api/auth/users
    // POST /api/auth/users
    // GET /api/auth/users/:id
    // PATCH /api/auth/users/:id
    // DELETE /api/auth/users/:id
    // POST /api/auth/users/:id/ban
    // POST /api/auth/users/:id/reactivate
    // POST /api/auth/users/:id/reset-password
  });

  it("should test all counter management endpoints", async () => {
    // GET /api/counters
    // POST /api/counters
    // GET /api/counters/:id
    // PUT /api/counters/:id
    // DELETE /api/counters/:id
    // POST /api/counters/:id/assign
    // POST /api/counters/:id/unassign
  });

  it("should test all token management endpoints", async () => {
    // POST /api/tokens/public
    // GET /api/tokens
    // GET /api/tokens/:id
    // PATCH /api/tokens/:id
    // DELETE /api/tokens/:id
    // POST /api/tokens/:id/cancel
  });

  it("should test all queue management endpoints", async () => {
    // GET /api/queue/status
    // POST /api/queue/call-next
    // POST /api/queue/complete
    // POST /api/queue/no-show
    // POST /api/queue/recall
  });

  it("should test all role management endpoints", async () => {
    // GET /api/roles
    // GET /api/roles/:id
  });

  it("should test all analytics endpoints", async () => {
    // GET /api/analytics/dashboard
    // GET /api/analytics/performance
    // GET /api/analytics/reports
  });
});
```

**2. WebSocket Integration**

```typescript
describe("WebSocket Integration", () => {
  it("should handle real-time queue updates", async () => {
    // Test WebSocket connection
    // Verify queue status updates
    // Check message delivery
  });

  it("should handle token status updates", async () => {
    // Test token status broadcasting
    // Verify client notifications
    // Check update accuracy
  });

  it("should handle counter status updates", async () => {
    // Test counter status changes
    // Verify staff notifications
    // Check availability updates
  });
});
```

**3. Error Handling**

```typescript
describe("API Error Handling", () => {
  it("should handle authentication errors", async () => {
    // Test invalid tokens
    // Test expired sessions
    // Test unauthorized access
  });

  it("should handle validation errors", async () => {
    // Test invalid request data
    // Test missing required fields
    // Test data type mismatches
  });

  it("should handle server errors", async () => {
    // Test database connection failures
    // Test service unavailability
    // Test timeout scenarios
  });
});
```

### Test Data Requirements

- Valid and invalid API credentials
- Test data for all endpoint scenarios
- WebSocket connection test data
- Error condition test data

## ⚡ Performance & Load Test Suite

### Test Suite: `performance-e2e.test.ts`

#### Test Scenarios

**1. Concurrent User Testing**

```typescript
describe("Concurrent User Testing", () => {
  it("should handle multiple simultaneous logins", async () => {
    // Test 100+ concurrent logins
    // Verify system stability
    // Check response times
  });

  it("should handle concurrent token creation", async () => {
    // Test 50+ simultaneous token creation
    // Verify queue integrity
    // Check data consistency
  });

  it("should handle concurrent admin operations", async () => {
    // Test multiple admin users
    // Verify data consistency
    // Check conflict resolution
  });
});
```

**2. Load Testing**

```typescript
describe("Load Testing", () => {
  it("should handle high queue volume", async () => {
    // Test 1000+ tokens in queue
    // Verify system performance
    // Check memory usage
  });

  it("should handle rapid API requests", async () => {
    // Test 100+ requests per second
    // Verify response times
    // Check error rates
  });

  it("should handle database load", async () => {
    // Test high database activity
    // Verify query performance
    // Check connection pooling
  });
});
```

**3. Stress Testing**

```typescript
describe("Stress Testing", () => {
  it("should handle system overload", async () => {
    // Test beyond normal capacity
    // Verify graceful degradation
    // Check recovery mechanisms
  });

  it("should handle memory pressure", async () => {
    // Test high memory usage
    // Verify garbage collection
    // Check memory leaks
  });

  it("should handle network issues", async () => {
    // Test network latency
    // Test connection drops
    // Verify retry mechanisms
  });
});
```

### Test Data Requirements

- Large dataset for load testing
- Performance monitoring tools
- Stress test scenarios
- Benchmark data

## 🚨 Error Handling Test Suite

### Test Suite: `error-handling-e2e.test.ts`

#### Test Scenarios

**1. Input Validation Errors**

```typescript
describe("Input Validation Errors", () => {
  it("should handle invalid user input", async () => {
    // Test malformed requests
    // Verify error responses
    // Check data sanitization
  });

  it("should handle missing required fields", async () => {
    // Test incomplete requests
    // Verify validation messages
    // Check error handling
  });

  it("should handle data type mismatches", async () => {
    // Test wrong data types
    // Verify type conversion
    // Check error responses
  });
});
```

**2. System Error Recovery**

```typescript
describe("System Error Recovery", () => {
  it("should handle database connection failures", async () => {
    // Test database unavailability
    // Verify error handling
    // Check recovery procedures
  });

  it("should handle service unavailability", async () => {
    // Test external service failures
    // Verify fallback mechanisms
    // Check error notifications
  });

  it("should handle network failures", async () => {
    // Test network issues
    // Verify retry mechanisms
    // Check timeout handling
  });
});
```

**3. Business Logic Errors**

```typescript
describe("Business Logic Errors", () => {
  it("should handle invalid business operations", async () => {
    // Test impossible operations
    // Verify error responses
    // Check business rule enforcement
  });

  it("should handle data consistency errors", async () => {
    // Test data conflicts
    // Verify conflict resolution
    // Check data integrity
  });

  it("should handle permission violations", async () => {
    // Test unauthorized operations
    // Verify access control
    // Check security measures
  });
});
```

### Test Data Requirements

- Invalid input test cases
- Error condition scenarios
- Recovery test data
- Security test cases

## 📊 Test Data Management

### Test Data Setup

```typescript
// Test data structure
const TEST_DATA = {
  organizations: [
    {
      id: "test-org-1",
      name: "Test Organization",
      settings: {
        /* org settings */
      },
    },
  ],
  users: [
    {
      id: "super-admin-1",
      username: "superadmin",
      email: "superadmin@test.com",
      role: "super_admin",
      organizationId: "test-org-1",
    },
    {
      id: "admin-1",
      username: "admin",
      email: "admin@test.com",
      role: "admin",
      organizationId: "test-org-1",
    },
    {
      id: "staff-1",
      username: "staff",
      email: "staff@test.com",
      role: "staff",
      organizationId: "test-org-1",
    },
  ],
  counters: [
    {
      id: "counter-1",
      name: "Counter 1",
      isActive: true,
      organizationId: "test-org-1",
    },
    {
      id: "counter-2",
      name: "Counter 2",
      isActive: true,
      organizationId: "test-org-1",
    },
  ],
  tokens: [
    {
      id: "token-1",
      customerType: "instant",
      priority: 1,
      status: "waiting",
      organizationId: "test-org-1",
    },
  ],
};
```

### Test Data Cleanup

```typescript
// Cleanup procedures
const cleanupProcedures = {
  afterEach: [
    "Clear test tokens",
    "Reset counter assignments",
    "Clean user sessions",
  ],
  afterAll: [
    "Delete test users",
    "Delete test counters",
    "Delete test organization",
    "Close database connections",
  ],
};
```

## 🚀 Test Execution Guidelines

### Prerequisites

1. **Environment Setup**

   - Node.js 18+ installed
   - PostgreSQL test database running
   - Redis server running
   - All dependencies installed

2. **Test Database Setup**

   ```bash
   # Create test database
   createdb queue_management_test

   # Run migrations
   npm run db:migrate:test

   # Seed test data
   npm run db:seed:test
   ```

3. **Environment Variables**
   ```bash
   NODE_ENV=test
   DATABASE_URL=postgresql://user:pass@localhost:5432/queue_management_test
   REDIS_URL=redis://localhost:6379
   JWT_SECRET=test-secret-key
   ```

### Running Tests

```bash
# Run all E2E tests
npm run test:e2e

# Run specific test suite
npm run test:e2e -- --grep "Admin Portal"

# Run tests with coverage
npm run test:e2e:coverage

# Run tests in watch mode
npm run test:e2e:watch

# Run tests with detailed output
npm run test:e2e -- --verbose
```

### Test Reporting

- **Coverage Reports**: Generated in `coverage/` directory
- **Test Results**: Available in `test-results/` directory
- **Performance Metrics**: Logged to `performance-logs/` directory
- **Error Logs**: Captured in `error-logs/` directory

### Best Practices

1. **Test Isolation**: Each test should be independent
2. **Data Cleanup**: Always clean up test data
3. **Error Handling**: Test both success and failure scenarios
4. **Performance**: Monitor test execution time
5. **Documentation**: Keep test documentation updated

---

**Last Updated**: $(date)
**Version**: 1.0.0
**Maintainer**: Development Team
