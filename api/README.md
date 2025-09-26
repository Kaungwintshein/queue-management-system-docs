# API Documentation

This directory contains all API-related documentation for the Queue Management System.

## 📋 Available Documentation

### Core API Documentation

- **[Complete API Reference](./API_DOCUMENTATION.md)** - Comprehensive API documentation with all endpoints, request/response formats, and examples
- **[Authentication API](./authentication-api.md)** - Detailed authentication API documentation
- **[Admin APIs Status](./ADMIN_APIS_STATUS.md)** - Current implementation status of admin APIs

### Testing Resources

- **[Postman Admin Collection](./postman-admin-collection.json)** - Complete Postman collection for admin APIs
- **[Postman Auth Collection](./postman-auth-collection.json)** - Authentication-focused Postman collection

## 🚀 Quick API Reference

### Authentication Endpoints

- `POST /api/auth/login` - User login
- `POST /api/auth/register` - User registration (admin only)
- `GET /api/auth/me` - Get current user
- `POST /api/auth/change-password` - Change password
- `POST /api/auth/logout` - User logout

### User Management Endpoints

- `GET /api/auth/users` - List users (admin only)
- `POST /api/auth/users` - Create user (admin only)
- `GET /api/auth/users/:id` - Get user by ID (admin only)
- `PATCH /api/auth/users/:id` - Update user (admin only)
- `DELETE /api/auth/users/:id` - Delete user (admin only)
- `POST /api/auth/users/:id/ban` - Ban user (admin only)
- `POST /api/auth/users/:id/reactivate` - Reactivate user (admin only)
- `POST /api/auth/users/:id/reset-password` - Reset user password (admin only)

### Counter Management Endpoints

- `GET /api/counters` - List counters
- `POST /api/counters` - Create counter (admin only)
- `GET /api/counters/:id` - Get counter by ID
- `PUT /api/counters/:id` - Update counter (admin only)
- `DELETE /api/counters/:id` - Delete counter (admin only)
- `POST /api/counters/:id/assign` - Assign staff to counter (admin only)
- `POST /api/counters/:id/unassign` - Unassign staff from counter (admin only)

### Role Management Endpoints

- `GET /api/roles` - List available roles
- `GET /api/roles/:id` - Get role details

## 🔧 API Features

- **JWT Authentication**: Secure token-based authentication
- **Role-Based Access Control**: Admin, staff, and super_admin roles
- **Input Validation**: Zod schema validation for all endpoints
- **Error Handling**: Standardized error responses
- **Rate Limiting**: Built-in rate limiting for security
- **Swagger Documentation**: Auto-generated API documentation at `/api-docs`

## 📊 Response Format

All API responses follow a consistent format:

```json
{
  "success": true,
  "message": "Operation successful",
  "data": { ... },
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

Error responses:

```json
{
  "success": false,
  "message": "Error description",
  "error": "ERROR_CODE",
  "details": { ... },
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

## 🧪 Testing

Use the provided Postman collections to test the APIs:

1. Import the collections into Postman
2. Set up environment variables for your API URL
3. Use the authentication collection to get tokens
4. Test admin endpoints with the admin collection

## 📝 Swagger Documentation

Interactive API documentation is available at:

- **Development**: `http://localhost:3001/api-docs`
- **Production**: `https://your-domain.com/api-docs`

## 🔐 Authentication

All protected endpoints require a Bearer token in the Authorization header:

```
Authorization: Bearer <your-jwt-token>
```

## 📈 Rate Limiting

- **Login attempts**: 5 requests per 15 minutes per IP
- **General API**: 100 requests per 15 minutes per IP
- **Admin operations**: 50 requests per 15 minutes per IP
