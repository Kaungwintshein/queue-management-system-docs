# Queue Management System Documentation

Welcome to the comprehensive documentation for the Queue Management System. This documentation is organized into logical sections to help you find the information you need quickly.

## 📁 Documentation Structure

### 🚀 [Getting Started](./guides/)

- [System Architecture](./guides/system-architecture.md) - Overview of the system design and components
- [API Implementation Guide](./guides/api-implementation-guide.md) - How to implement and extend the API
- [API Integration Guide](./guides/api-integration-guide.md) - How to integrate with the API

### 🔌 [API Documentation](./api/)

- [Complete API Reference](./api/API_DOCUMENTATION.md) - Full API documentation with endpoints, requests, and responses
- [Authentication API](./api/authentication-api.md) - Authentication-specific API documentation
- [Admin APIs Status](./api/ADMIN_APIS_STATUS.md) - Current status of admin API implementation
- [Postman Collections](./api/) - Ready-to-use Postman collections for testing

### 🎨 [UI Documentation](./ui/)

- [UI Implementation Guide v1](./ui/ui-implementation-guide.md) - Initial UI implementation guide
- [UI Implementation Guide v2](./ui/ui-implementation-guide-v2.md) - Updated UI implementation guide
- [UI Implementation Guide v3](./ui/ui-implementation-guide-v3.md) - Latest UI implementation guide

### 🚀 [Deployment](./deployment/)

- [Deployment Guide](./deployment/DEPLOYMENT.md) - General deployment instructions
- [NPM Deployment](./deployment/DEPLOYMENT-NPM.md) - NPM-specific deployment guide

### 🧪 [Testing](./testing/)

- [TDD Implementation](./testing/TDD_IMPLEMENTATION.md) - Test-Driven Development implementation guide
- [Test Quick Start](./testing/TEST_QUICK_START.md) - Quick start guide for running tests

## 🏗️ Project Structure

```
queue-management-system/
├── api/                    # Backend API (Express.js + TypeScript)
│   ├── src/
│   │   ├── routes/        # API route handlers
│   │   ├── services/      # Business logic services
│   │   ├── middleware/    # Express middleware
│   │   └── utils/         # Utility functions
│   ├── tests/             # API test suites
│   └── prisma/            # Database schema and migrations
├── ui/                     # Frontend UI (Next.js + React)
│   ├── app/               # Next.js App Router pages
│   ├── components/        # React components
│   ├── lib/               # Utility libraries and hooks
│   └── styles/            # CSS and styling
└── docs/                  # Documentation (this directory)
```

## 🚀 Quick Start

1. **API Setup**: Follow the [API Implementation Guide](./guides/api-implementation-guide.md)
2. **UI Setup**: Follow the [UI Implementation Guide v3](./ui/ui-implementation-guide-v3.md)
3. **Testing**: Use the [Test Quick Start](./testing/TEST_QUICK_START.md) guide
4. **Deployment**: Check the [Deployment Guide](./deployment/DEPLOYMENT.md)

## 🔧 Key Features

- **Authentication & Authorization**: JWT-based auth with role-based access control
- **User Management**: Complete CRUD operations for users
- **Counter Management**: Manage service counters and staff assignments
- **Role Management**: Handle user roles and permissions
- **Real-time Updates**: WebSocket integration for live updates
- **Comprehensive Testing**: Unit, integration, and E2E tests
- **API Documentation**: Swagger/OpenAPI documentation
- **Postman Collections**: Ready-to-use API testing collections

## 📚 Additional Resources

- **API Reference**: [Complete API Documentation](./api/API_DOCUMENTATION.md)
- **Postman Collections**: Available in [API Documentation](./api/) folder
- **Test Coverage**: See [TDD Implementation](./testing/TDD_IMPLEMENTATION.md)
- **System Architecture**: [Architecture Overview](./guides/system-architecture.md)

## 🤝 Contributing

When contributing to this project:

1. Follow the existing code structure and patterns
2. Add tests for new features (see [TDD Implementation](./testing/TDD_IMPLEMENTATION.md))
3. Update relevant documentation
4. Use the provided Postman collections for API testing

## 📞 Support

For questions or issues:

1. Check the relevant documentation section
2. Review the API documentation and examples
3. Run the test suites to verify functionality
4. Check the deployment guides for setup issues

---

_Last updated: $(date)_
