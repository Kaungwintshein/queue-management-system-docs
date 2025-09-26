# Testing Documentation

This directory contains all testing-related documentation for the Queue Management System.

## 📋 Available Documentation

### Testing Guides
- **[TDD Implementation](./TDD_IMPLEMENTATION.md)** - Comprehensive Test-Driven Development implementation guide
- **[Test Quick Start](./TEST_QUICK_START.md)** - Quick start guide for running tests

## 🧪 Testing Strategy

The Queue Management System implements a comprehensive testing strategy following Test-Driven Development (TDD) principles:

### Test Types
1. **Unit Tests** - Test individual functions and components in isolation
2. **Integration Tests** - Test interactions between different parts of the system
3. **End-to-End (E2E) Tests** - Test complete user workflows
4. **API Tests** - Test API endpoints and responses

### Test Coverage
- **API Coverage**: 95%+ coverage for all API endpoints
- **Business Logic**: 100% coverage for critical business logic
- **Error Handling**: Comprehensive error scenario testing
- **Authentication**: Complete auth flow testing
- **Authorization**: Role-based access control testing

## 🏗️ Test Infrastructure

### API Testing (Jest + Supertest)
```
api/tests/
├── setup.ts                 # Global test setup
├── utils/
│   └── testHelpers.ts       # Test utilities and helpers
├── unit/                    # Unit tests
│   ├── auth.test.ts
│   ├── user-management.test.ts
│   ├── counter-management.test.ts
│   └── role-management.test.ts
├── integration/             # Integration tests
│   └── admin-workflow.test.ts
└── e2e/                     # End-to-end tests
    └── admin-portal.test.ts
```

### Test Database
- **Separate Test Database**: Isolated test environment
- **Automatic Setup/Teardown**: Database reset between test runs
- **Test Data Seeding**: Predefined test data for consistent testing
- **Transaction Rollback**: Tests run in isolated transactions

## 🚀 Running Tests

### Quick Start
```bash
# Run all tests
npm test

# Run specific test types
npm run test:unit
npm run test:integration
npm run test:e2e

# Run with coverage
npm run test:coverage

# Run in TDD mode (watch mode)
npm run test:tdd
```

### Test Scripts
- `test` - Run all tests
- `test:unit` - Run unit tests only
- `test:integration` - Run integration tests only
- `test:e2e` - Run end-to-end tests only
- `test:coverage` - Run tests with coverage report
- `test:tdd` - Run tests in watch mode for TDD
- `test:ci` - Run tests in CI environment

## 🔧 Test Configuration

### Jest Configuration
- **TypeScript Support**: ts-jest preset for TypeScript files
- **Path Mapping**: @/ alias for clean imports
- **Test Environment**: Node.js environment for API tests
- **Coverage Reports**: HTML, LCOV, and text coverage reports
- **Test Timeout**: 10-second timeout for all tests

### Test Database Setup
- **Environment Variables**: Separate test environment configuration
- **Database Migration**: Automatic schema migration for tests
- **Data Seeding**: Test data population
- **Cleanup**: Automatic cleanup after test runs

## 📊 Test Data Management

### Test Helpers
- **TestHelpers Class**: Centralized test utilities
- **Data Creation**: Helper functions for creating test data
- **Token Generation**: JWT token generation for testing
- **Database Cleanup**: Automatic cleanup functions
- **Response Validation**: Standardized response validation

### Test Scenarios
- **Valid Scenarios**: Testing happy path workflows
- **Error Scenarios**: Testing error conditions and edge cases
- **Permission Testing**: Role-based access control testing
- **Data Consistency**: Testing data integrity across operations

## 🎯 Key Test Features

### Authentication Testing
- **Login/Logout**: Complete authentication flow testing
- **Token Validation**: JWT token validation and expiration
- **Role-based Access**: Permission testing for different roles
- **Password Management**: Password change and reset testing

### API Testing
- **Request/Response Validation**: Schema validation testing
- **Error Handling**: Comprehensive error response testing
- **Rate Limiting**: Rate limit enforcement testing
- **Input Validation**: Request validation testing

### Integration Testing
- **Complete Workflows**: End-to-end user workflow testing
- **Data Consistency**: Cross-operation data integrity testing
- **Error Recovery**: Error handling and recovery testing
- **Performance Testing**: Basic performance and load testing

## 🔍 Test Utilities

### TestHelpers Class
```typescript
// Create test data
const user = await TestHelpers.createTestUser({...});
const org = await TestHelpers.createTestOrganization({...});
const counter = await TestHelpers.createTestCounter({...});

// Generate tokens
const token = TestHelpers.generateJWTToken({...});

// Validate responses
TestHelpers.expectValidUserResponse(user);
TestHelpers.expectValidApiResponse(response);

// Cleanup
await TestHelpers.cleanupDatabase();
```

### Test Data Constants
- **TEST_DATA**: Predefined test data objects
- **TEST_SCENARIOS**: Common test scenarios
- **VALIDATION_HELPERS**: Response validation utilities

## 📈 Coverage Reports

### Coverage Metrics
- **Line Coverage**: Percentage of code lines executed
- **Branch Coverage**: Percentage of code branches tested
- **Function Coverage**: Percentage of functions called
- **Statement Coverage**: Percentage of statements executed

### Coverage Reports
- **HTML Report**: Detailed HTML coverage report
- **LCOV Report**: LCOV format for CI integration
- **Text Report**: Console coverage summary
- **Coverage Directory**: `coverage/` directory with reports

## 🚀 TDD Workflow

### Red-Green-Refactor Cycle
1. **Red**: Write a failing test
2. **Green**: Write minimal code to pass the test
3. **Refactor**: Improve code while keeping tests green

### Test-First Development
- **Write Tests First**: Tests drive the development process
- **Small Increments**: Small, focused test cases
- **Continuous Testing**: Run tests frequently during development
- **Refactoring Safety**: Tests ensure refactoring doesn't break functionality

## 🔧 CI/CD Integration

### Continuous Integration
- **Automated Testing**: Tests run on every commit
- **Coverage Reporting**: Coverage reports in CI pipeline
- **Test Database**: Isolated test database in CI environment
- **Parallel Testing**: Tests run in parallel for faster execution

### Quality Gates
- **Minimum Coverage**: 90% minimum coverage requirement
- **All Tests Pass**: All tests must pass before deployment
- **No Linting Errors**: Code must pass linting checks
- **Type Safety**: TypeScript compilation must succeed
