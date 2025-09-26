# Authentication API Documentation

## Overview

The Queue Management System provides a comprehensive authentication API with JWT-based authentication and role-based access control (RBAC). The system supports three user roles: `staff`, `admin`, and `super_admin`.

## Base URL

```
http://localhost:3001/api/auth
```

## Authentication

Most endpoints require JWT authentication via Bearer token in the Authorization header:

```
Authorization: Bearer <jwt_token>
```

## User Roles & Permissions

### Staff

- Can access basic queue operations
- Can view their own profile
- Cannot manage other users

### Admin

- All staff permissions
- Can manage users within their organization
- Can create/update/deactivate users
- Can view organization analytics

### Super Admin

- All admin permissions
- Can manage system-wide settings
- Can access cross-organization data

## API Endpoints

### 1. User Login

```http
POST /api/auth/login
```

**Description:** Authenticate user and return JWT token.

**Request Body:**

```json
{
  "username": "string (required)",
  "password": "string (required)",
  "remember": "boolean (optional)"
}
```

**Response (200):**

```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "user": {
      "id": "uuid",
      "username": "string",
      "email": "string",
      "role": "staff|admin|super_admin",
      "organizationId": "uuid",
      "permissions": {},
      "isActive": true,
      "lastLogin": "datetime",
      "createdAt": "datetime",
      "updatedAt": "datetime"
    },
    "token": "jwt_token_string",
    "refreshToken": "refresh_token_string",
    "expiresIn": 86400
  },
  "timestamp": "2025-09-23T14:30:00.000Z"
}
```

**Error Responses:**

- `401`: Invalid credentials
- `429`: Too many login attempts (rate limited)

---

### 2. Get Current User Profile

```http
GET /api/auth/me
```

**Description:** Get the authenticated user's profile information.

**Headers:**

```
Authorization: Bearer <jwt_token>
```

**Response (200):**

```json
{
  "success": true,
  "message": "User profile retrieved",
  "data": {
    "id": "uuid",
    "username": "string",
    "email": "string",
    "role": "staff|admin|super_admin",
    "organizationId": "uuid",
    "isActive": true,
    "permissions": {},
    "lastLogin": "datetime",
    "createdAt": "datetime",
    "updatedAt": "datetime"
  }
}
```

**Error Responses:**

- `401`: Unauthorized (invalid/expired token)

---

### 3. Change Password

```http
POST /api/auth/change-password
```

**Description:** Change the authenticated user's password.

**Headers:**

```
Authorization: Bearer <jwt_token>
```

**Request Body:**

```json
{
  "currentPassword": "string (required)",
  "newPassword": "string (required)",
  "confirmNewPassword": "string (required)"
}
```

**Response (200):**

```json
{
  "success": true,
  "message": "Password changed successfully"
}
```

**Error Responses:**

- `400`: Validation error (passwords don't match, weak password)
- `401`: Invalid current password
- `401`: Unauthorized

---

### 4. User Logout

```http
POST /api/auth/logout
```

**Description:** Logout the authenticated user.

**Headers:**

```
Authorization: Bearer <jwt_token>
```

**Response (200):**

```json
{
  "success": true,
  "message": "Logout successful"
}
```

**Error Responses:**

- `401`: Unauthorized

---

### 5. User Registration (Admin Only)

```http
POST /api/auth/register
```

**Description:** Register a new user (admin/super_admin only).

**Headers:**

```
Authorization: Bearer <jwt_token>
```

**Request Body:**

```json
{
  "username": "string (required)",
  "email": "string (required)",
  "password": "string (required)",
  "confirmPassword": "string (required)",
  "role": "staff|admin|super_admin (optional, defaults to staff)"
}
```

**Response (201):**

```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "id": "uuid",
    "username": "string",
    "email": "string",
    "role": "staff|admin|super_admin",
    "organizationId": "uuid",
    "isActive": true,
    "createdAt": "datetime"
  }
}
```

**Error Responses:**

- `400`: Validation error
- `403`: Insufficient permissions
- `409`: Username/email already exists

---

### 6. List Users (Admin Only)

```http
GET /api/auth/users
```

**Description:** Get list of users in the organization.

**Headers:**

```
Authorization: Bearer <jwt_token>
```

**Query Parameters:**

- `role` (optional): Filter by role (`staff`, `admin`, `super_admin`)
- `isActive` (optional): Filter by active status (`true`, `false`)
- `limit` (optional): Number of users to return (1-100, default: 20)
- `offset` (optional): Number of users to skip (default: 0)

**Response (200):**

```json
{
  "success": true,
  "message": "Users retrieved successfully",
  "data": {
    "users": [
      {
        "id": "uuid",
        "username": "string",
        "email": "string",
        "role": "staff|admin|super_admin",
        "isActive": true,
        "lastLogin": "datetime",
        "createdAt": "datetime"
      }
    ],
    "total": 50,
    "limit": 20,
    "offset": 0
  }
}
```

**Error Responses:**

- `403`: Insufficient permissions

---

### 7. Create User (Admin Only)

```http
POST /api/auth/users
```

**Description:** Create a new user (admin/super_admin only).

**Headers:**

```
Authorization: Bearer <jwt_token>
```

**Request Body:**

```json
{
  "username": "string (required)",
  "email": "string (required)",
  "password": "string (required)",
  "role": "staff|admin|super_admin (required)",
  "permissions": "object (optional)",
  "isActive": "boolean (optional, default: true)"
}
```

**Response (201):**

```json
{
  "success": true,
  "message": "User created successfully",
  "data": {
    "id": "uuid",
    "username": "string",
    "email": "string",
    "role": "staff|admin|super_admin",
    "organizationId": "uuid",
    "isActive": true,
    "createdAt": "datetime"
  }
}
```

**Error Responses:**

- `400`: Validation error
- `403`: Insufficient permissions
- `409`: Username/email already exists

---

### 8. Get User by ID

```http
GET /api/auth/users/{userId}
```

**Description:** Get user details by ID. Users can only view their own profile unless they're admin.

**Headers:**

```
Authorization: Bearer <jwt_token>
```

**Path Parameters:**

- `userId`: UUID of the user

**Response (200):**

```json
{
  "success": true,
  "message": "User retrieved successfully",
  "data": {
    "id": "uuid",
    "username": "string",
    "email": "string",
    "role": "staff|admin|super_admin",
    "organizationId": "uuid",
    "isActive": true,
    "permissions": {},
    "lastLogin": "datetime",
    "createdAt": "datetime",
    "updatedAt": "datetime"
  }
}
```

**Error Responses:**

- `403`: Insufficient permissions
- `404`: User not found

---

### 9. Update User

```http
PATCH /api/auth/users/{userId}
```

**Description:** Update user information. Admins can update any user, staff can only update themselves.

**Headers:**

```
Authorization: Bearer <jwt_token>
```

**Path Parameters:**

- `userId`: UUID of the user

**Request Body:**

```json
{
  "username": "string (optional)",
  "email": "string (optional)",
  "role": "staff|admin|super_admin (optional, admin only)",
  "permissions": "object (optional, admin only)",
  "isActive": "boolean (optional, admin only)"
}
```

**Response (200):**

```json
{
  "success": true,
  "message": "User updated successfully",
  "data": {
    "id": "uuid",
    "username": "string",
    "email": "string",
    "role": "staff|admin|super_admin",
    "organizationId": "uuid",
    "isActive": true,
    "updatedAt": "datetime"
  }
}
```

**Error Responses:**

- `400`: Validation error
- `403`: Insufficient permissions
- `404`: User not found

---

### 10. Deactivate User (Admin Only)

```http
POST /api/auth/users/{userId}/deactivate
```

**Description:** Deactivate a user account (admin/super_admin only).

**Headers:**

```
Authorization: Bearer <jwt_token>
```

**Path Parameters:**

- `userId`: UUID of the user

**Response (200):**

```json
{
  "success": true,
  "message": "User deactivated successfully"
}
```

**Error Responses:**

- `403`: Insufficient permissions
- `404`: User not found

## Error Response Format

All error responses follow this format:

```json
{
  "success": false,
  "message": "Error description",
  "error": "ERROR_CODE",
  "details": {
    "field": "Additional error details"
  }
}
```

## Rate Limiting

Authentication endpoints are rate-limited:

- Login: 5 attempts per minute per IP
- Other auth endpoints: 100 requests per minute per user

## Security Features

- **Password Hashing**: Using bcrypt with salt rounds
- **JWT Tokens**: 7-day expiration, HS256 algorithm
- **Rate Limiting**: Protection against brute force attacks
- **Input Validation**: Comprehensive validation using Zod schemas
- **Role-based Access Control**: Fine-grained permissions
- **Audit Logging**: All authentication events are logged

## Example Usage

### Login Flow

```bash
# 1. Login
curl -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "admin@example.com",
    "password": "securepassword123"
  }'

# 2. Use the returned token for authenticated requests
curl -X GET http://localhost:3001/api/auth/me \
  -H "Authorization: Bearer <jwt_token>"
```

### Create User Flow (Admin)

```bash
# Create a new staff user
curl -X POST http://localhost:3001/api/auth/users \
  -H "Authorization: Bearer <admin_jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "newstaff",
    "email": "staff@example.com",
    "password": "staffpassword123",
    "role": "staff"
  }'
```
