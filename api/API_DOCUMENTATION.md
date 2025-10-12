# Queue Management System API Documentation

## Base URL

```
http://localhost:3001/api
```

## Authentication

All endpoints (except login/register) require a Bearer token in the Authorization header:

```
Authorization: Bearer <your-jwt-token>
```

## Response Format

All API responses follow this standardized format:

### Success Response

```json
{
  "success": true,
  "message": "Operation completed successfully",
  "data": { ... },
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

### Error Response

```json
{
  "success": false,
  "message": "Error description",
  "error": "ERROR_CODE",
  "details": { ... },
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

---

## Authentication Endpoints

### Login

**POST** `/auth/login`

**Request Body:**

```json
{
  "username": "admin",
  "password": "admin123"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "user": {
      "id": "uuid",
      "organizationId": "uuid",
      "username": "admin",
      "email": "admin@example.com",
      "role": "super_admin",
      "permissions": {},
      "isActive": true,
      "lastLogin": "2024-01-01T00:00:00.000Z",
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    },
    "token": "jwt-token",
    "refreshToken": "refresh-token",
    "expiresIn": 3600
  }
}
```

### Register

**POST** `/auth/register`

**Request Body:**

```json
{
  "username": "newuser",
  "email": "user@example.com",
  "password": "password123",
  "role": "staff"
}
```

### Get Current User

**GET** `/auth/me`

**Response:**

```json
{
  "success": true,
  "message": "User retrieved successfully",
  "data": {
    "id": "uuid",
    "organizationId": "uuid",
    "username": "admin",
    "email": "admin@example.com",
    "role": "super_admin",
    "permissions": {},
    "isActive": true,
    "lastLogin": "2024-01-01T00:00:00.000Z",
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

### Logout

**POST** `/auth/logout`

---

## User Management Endpoints

### Get All Users

**GET** `/auth/users`

**Query Parameters:**

- `role` (optional): Filter by role (`staff`, `admin`, `super_admin`)
- `isActive` (optional): Filter by active status (`true`/`false`)
- `limit` (optional): Number of users to return (default: 20)
- `offset` (optional): Number of users to skip (default: 0)

**Response:**

```json
{
  "success": true,
  "message": "Users retrieved successfully",
  "data": {
    "users": [
      {
        "id": "uuid",
        "organizationId": "uuid",
        "username": "admin",
        "email": "admin@example.com",
        "role": "super_admin",
        "permissions": {},
        "isActive": true,
        "lastLogin": "2024-01-01T00:00:00.000Z",
        "createdAt": "2024-01-01T00:00:00.000Z",
        "updatedAt": "2024-01-01T00:00:00.000Z"
      }
    ],
    "total": 1,
    "limit": 20,
    "offset": 0
  }
}
```

### Create User

**POST** `/auth/users`

**Request Body:**

```json
{
  "username": "newuser",
  "email": "user@example.com",
  "password": "password123",
  "role": "staff",
  "permissions": {},
  "isActive": true
}
```

**Response:**

```json
{
  "success": true,
  "message": "User created successfully",
  "data": {
    "id": "uuid",
    "organizationId": "uuid",
    "username": "newuser",
    "email": "user@example.com",
    "role": "staff",
    "permissions": {},
    "isActive": true,
    "lastLogin": null,
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

### Get User by ID

**GET** `/auth/users/{userId}`

**Response:**

```json
{
  "success": true,
  "message": "User retrieved successfully",
  "data": {
    "id": "uuid",
    "organizationId": "uuid",
    "username": "admin",
    "email": "admin@example.com",
    "role": "super_admin",
    "permissions": {},
    "isActive": true,
    "lastLogin": "2024-01-01T00:00:00.000Z",
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

### Update User

**PATCH** `/auth/users/{userId}`

**Request Body:**

```json
{
  "username": "updateduser",
  "email": "updated@example.com",
  "role": "admin",
  "isActive": true
}
```

### Ban User

**POST** `/auth/users/{userId}/ban`

**Response:**

```json
{
  "success": true,
  "message": "User banned successfully",
  "data": null
}
```

### Reactivate User

**POST** `/auth/users/{userId}/reactivate`

**Response:**

```json
{
  "success": true,
  "message": "User reactivated successfully",
  "data": null
}
```

### Reset User Password

**POST** `/auth/users/{userId}/reset-password`

**Response:**

```json
{
  "success": true,
  "message": "Password reset email sent successfully",
  "data": null
}
```

### Delete User

**DELETE** `/auth/users/{userId}`

**Response:**

```json
{
  "success": true,
  "message": "User deleted successfully",
  "data": null
}
```

---

## Counter Management Endpoints

### Get All Counters

**GET** `/counters`

**Query Parameters:**

- `active` (optional): Filter by active status (`true`/`false`)
- `assigned` (optional): Filter by assignment status (`true`/`false`)

**Response:**

```json
{
  "success": true,
  "message": "Counters retrieved successfully",
  "data": [
    {
      "id": "uuid",
      "organizationId": "uuid",
      "name": "Counter 1",
      "isActive": true,
      "assignedStaffId": "uuid",
      "assignedStaff": {
        "id": "uuid",
        "username": "staff1",
        "role": "staff"
      }
    }
  ]
}
```

### Create Counter

**POST** `/counters`

**Request Body:**

```json
{
  "name": "Counter 1",
  "isActive": true,
  "assignedStaffId": "uuid"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Counter created successfully",
  "data": {
    "id": "uuid",
    "organizationId": "uuid",
    "name": "Counter 1",
    "isActive": true,
    "assignedStaffId": "uuid",
    "assignedStaff": {
      "id": "uuid",
      "username": "staff1",
      "role": "staff"
    }
  }
}
```

### Get Counter by ID

**GET** `/counters/{counterId}`

**Response:**

```json
{
  "success": true,
  "message": "Counter retrieved successfully",
  "data": {
    "id": "uuid",
    "organizationId": "uuid",
    "name": "Counter 1",
    "isActive": true,
    "assignedStaffId": "uuid",
    "assignedStaff": {
      "id": "uuid",
      "username": "staff1",
      "role": "staff"
    }
  }
}
```

### Update Counter

**PUT** `/counters/{counterId}`

**Request Body:**

```json
{
  "name": "Updated Counter 1",
  "isActive": false,
  "assignedStaffId": "uuid"
}
```

### Delete Counter

**DELETE** `/counters/{counterId}`

**Response:**

```json
{
  "success": true,
  "message": "Counter deleted successfully"
}
```

### Assign Staff to Counter

**POST** `/counters/{counterId}/assign`

**Request Body:**

```json
{
  "staffId": "uuid"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Staff assigned successfully",
  "data": {
    "id": "uuid",
    "organizationId": "uuid",
    "name": "Counter 1",
    "isActive": true,
    "assignedStaffId": "uuid",
    "assignedStaff": {
      "id": "uuid",
      "username": "staff1",
      "role": "staff"
    }
  }
}
```

### Unassign Staff from Counter

**POST** `/counters/{counterId}/unassign`

**Response:**

```json
{
  "success": true,
  "message": "Staff unassigned successfully",
  "data": {
    "id": "uuid",
    "organizationId": "uuid",
    "name": "Counter 1",
    "isActive": true,
    "assignedStaffId": null,
    "assignedStaff": null
  }
}
```

### Get Available Counters for Staff Selection

**GET** `/counters/available`

Gets all active counters that are not assigned to any staff member. This endpoint is specifically for staff to see which counters they can select.

**Headers:**

- `Authorization: Bearer <token>` (Staff only)

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": "counter-uuid-1",
      "name": "Counter 1",
      "isActive": true,
      "assignedStaffId": null
    },
    {
      "id": "counter-uuid-2",
      "name": "Counter 2",
      "isActive": true,
      "assignedStaffId": null
    }
  ]
}
```

### Select Counter for Staff

**POST** `/counters/select`

Allows a staff member to select an available counter. Only one counter can be assigned per staff member.

**Headers:**

- `Authorization: Bearer <token>` (Staff only)

**Request Body:**

```json
{
  "counterId": "counter-uuid"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Counter selected successfully",
  "data": {
    "id": "counter-uuid",
    "name": "Counter 1",
    "isActive": true,
    "assignedStaffId": "staff-uuid",
    "assignedStaff": {
      "id": "staff-uuid",
      "username": "staff1",
      "role": "staff"
    }
  }
}
```

**Error Responses:**

- `409`: Staff already has a counter assigned
- `404`: Counter not found or already assigned
- `400`: Invalid counter ID

### Release Current Counter Assignment

**POST** `/counters/release`

Allows a staff member to release their current counter assignment.

**Headers:**

- `Authorization: Bearer <token>` (Staff only)

**Response:**

```json
{
  "success": true,
  "message": "Counter released successfully"
}
```

**Error Responses:**

- `404`: No counter assigned to the staff member

---

## Role Management Endpoints

### Get All Roles

**GET** `/roles`

**Response:**

```json
{
  "success": true,
  "message": "Roles retrieved successfully",
  "data": [
    {
      "id": "super_admin",
      "name": "Super Admin",
      "description": "Full system access with all permissions",
      "permissions": [
        "user.create",
        "user.read",
        "user.update",
        "user.delete",
        "user.ban",
        "user.reset_password",
        "counter.create",
        "counter.read",
        "counter.update",
        "counter.delete",
        "counter.assign",
        "token.create",
        "token.read",
        "token.update",
        "analytics.read",
        "settings.read",
        "settings.update"
      ],
      "userCount": 1
    },
    {
      "id": "admin",
      "name": "Admin",
      "description": "Administrative access with most permissions",
      "permissions": [
        "user.create",
        "user.read",
        "user.update",
        "user.delete",
        "user.ban",
        "user.reset_password",
        "counter.create",
        "counter.read",
        "counter.update",
        "counter.delete",
        "counter.assign",
        "token.create",
        "token.read",
        "token.update",
        "analytics.read"
      ],
      "userCount": 2
    },
    {
      "id": "staff",
      "name": "Staff",
      "description": "Basic staff access for counter operations",
      "permissions": [
        "counter.read",
        "token.create",
        "token.read",
        "token.update"
      ],
      "userCount": 5
    }
  ]
}
```

### Create Role

**POST** `/roles`

**Request Body:**

```json
{
  "name": "Manager",
  "description": "Manager role with specific permissions",
  "permissions": [
    "user.read",
    "user.update",
    "counter.read",
    "counter.update",
    "token.read",
    "token.update"
  ]
}
```

**Response:**

```json
{
  "success": true,
  "message": "Role created successfully",
  "data": {
    "id": "uuid",
    "name": "Manager",
    "description": "Manager role with specific permissions",
    "permissions": [
      "user.read",
      "user.update",
      "counter.read",
      "counter.update",
      "token.read",
      "token.update"
    ],
    "organizationId": "uuid",
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

### Get Role by ID

**GET** `/roles/{roleId}`

**Response:**

```json
{
  "success": true,
  "message": "Role retrieved successfully",
  "data": {
    "id": "super_admin",
    "name": "Super Admin",
    "description": "Full system access with all permissions",
    "permissions": [
      "user.create",
      "user.read",
      "user.update",
      "user.delete",
      "user.ban",
      "user.reset_password",
      "counter.create",
      "counter.read",
      "counter.update",
      "counter.delete",
      "counter.assign",
      "token.create",
      "token.read",
      "token.update",
      "analytics.read",
      "settings.read",
      "settings.update"
    ],
    "userCount": 1
  }
}
```

### Update Role

**PUT** `/roles/{roleId}`

**Request Body:**

```json
{
  "name": "Updated Manager",
  "description": "Updated manager role",
  "permissions": [
    "user.read",
    "user.update",
    "user.create",
    "counter.read",
    "counter.update",
    "counter.create"
  ]
}
```

### Delete Role

**DELETE** `/roles/{roleId}`

**Response:**

```json
{
  "success": true,
  "message": "Role deleted successfully",
  "data": null
}
```

---

## Queue Management Endpoints

### Get Queue Status

**GET** `/queue/status`

Get the current queue status for the organization.

**Response:**

```json
{
  "success": true,
  "message": "Queue status retrieved successfully",
  "data": {
    "counters": [
      {
        "counter": {
          "id": "counter-id",
          "name": "Counter 1",
          "isActive": true,
          "assignedStaffId": "staff-id"
        },
        "currentToken": {
          "id": "token-id",
          "number": "1001",
          "status": "called",
          "priority": 5,
          "tokenConfigId": "config-id",
          "tokenConfig": {
            "serviceName": "Express Service",
            "description": "Quick service"
          }
        },
        "nextTokens": [
          {
            "id": "token-id-2",
            "number": "1002",
            "status": "waiting",
            "priority": 3,
            "tokenConfigId": "config-id-2",
            "tokenConfig": {
              "serviceName": "Standard Service",
              "description": "Regular service"
            }
          }
        ],
        "noShowTokens": []
      }
    ],
    "summary": {
      "totalWaiting": 15,
      "totalServing": 3,
      "totalCompleted": 42
    }
  }
}
```

### Create Token

**POST** `/queue/tokens`

Create a new token in the queue.

**Request Body:**

```json
{
  "tokenConfigId": "config-id",
  "customerType": "walk-in",
  "metadata": {
    "source": "kiosk",
    "location": "lobby"
  }
}
```

**Response:**

```json
{
  "success": true,
  "message": "Token created successfully",
  "data": {
    "id": "token-id",
    "number": "1001",
    "status": "waiting",
    "priority": 5,
    "tokenConfigId": "config-id",
    "tokenConfig": {
      "serviceName": "Express Service",
      "description": "Quick service"
    },
    "createdAt": "2024-12-01T10:00:00.000Z"
  }
}
```

### Call Next Token

**POST** `/queue/call-next-token`

Call the next available token for a counter.

**Request Body:**

```json
{
  "counterId": "counter-id"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Token called successfully",
  "data": {
    "id": "token-id",
    "number": "1001",
    "status": "called",
    "priority": 5,
    "tokenConfigId": "config-id",
    "tokenConfig": {
      "serviceName": "Express Service",
      "description": "Quick service"
    },
    "calledAt": "2024-12-01T10:05:00.000Z",
    "servedBy": "staff-id",
    "counterId": "counter-id"
  }
}
```

### Call Specific Token

**POST** `/queue/call-next-token/{tokenId}`

Call a specific token by its ID.

**Request Body:**

```json
{
  "counterId": "counter-id"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Token called successfully",
  "data": {
    "id": "token-id",
    "number": "1001",
    "status": "called",
    "priority": 5,
    "tokenConfigId": "config-id",
    "tokenConfig": {
      "serviceName": "Express Service",
      "description": "Quick service"
    },
    "calledAt": "2024-12-01T10:05:00.000Z",
    "servedBy": "staff-id",
    "counterId": "counter-id"
  }
}
```

### Start Serving Token

**POST** `/queue/tokens/{tokenId}/start-serving`

Mark a token as being served.

**Response:**

```json
{
  "success": true,
  "message": "Service started successfully",
  "data": {
    "id": "token-id",
    "number": "1001",
    "status": "serving",
    "servingStartedAt": "2024-12-01T10:10:00.000Z"
  }
}
```

### Complete Service

**POST** `/queue/tokens/{tokenId}/complete`

Mark a token as completed.

**Response:**

```json
{
  "success": true,
  "message": "Service completed successfully",
  "data": {
    "token": {
      "id": "token-id",
      "number": "1001",
      "status": "completed",
      "completedAt": "2024-12-01T10:15:00.000Z",
      "serviceDuration": 5
    }
  }
}
```

### Mark Token as No-Show

**POST** `/queue/tokens/{tokenId}/no-show`

Mark a token as no-show.

**Response:**

```json
{
  "success": true,
  "message": "Token marked as no-show",
  "data": {
    "id": "token-id",
    "number": "1001",
    "status": "no_show",
    "noShowAt": "2024-12-01T10:20:00.000Z"
  }
}
```

### Recall Token

**POST** `/queue/tokens/{tokenId}/recall`

Recall a no-show token back to the queue.

**Request Body:**

```json
{
  "counterId": "counter-id"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Token recalled successfully",
  "data": {
    "id": "token-id",
    "number": "1001",
    "status": "waiting",
    "recalledAt": "2024-12-01T10:25:00.000Z"
  }
}
```

### Repeat Token Announcement

**POST** `/queue/tokens/{tokenId}/repeat-announcement`

Repeat the announcement for a called token.

**Request Body:**

```json
{
  "counterId": "counter-id"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Announcement repeated successfully",
  "data": {
    "id": "token-id",
    "number": "1001",
    "announcedAt": "2024-12-01T10:30:00.000Z"
  }
}
```

---

## Token Configuration Endpoints

### Get Active Token Configurations

**GET** `/token-configs/active`

Get all active token configurations.

**Response:**

```json
{
  "success": true,
  "message": "Active token configurations retrieved successfully",
  "data": [
    {
      "id": "config-id",
      "serviceName": "Express Service",
      "description": "Quick service for urgent requests",
      "priority": 10,
      "isActive": true,
      "createdAt": "2024-12-01T09:00:00.000Z"
    }
  ]
}
```

### Get All Token Configurations

**GET** `/token-configs`

Get all token configurations (active and inactive).

**Query Parameters:**

- `page` (optional): Page number for pagination
- `limit` (optional): Number of items per page
- `search` (optional): Search term for service name

**Response:**

```json
{
  "success": true,
  "message": "Token configurations retrieved successfully",
  "data": {
    "configs": [
      {
        "id": "config-id",
        "serviceName": "Express Service",
        "description": "Quick service for urgent requests",
        "priority": 10,
        "isActive": true,
        "createdAt": "2024-12-01T09:00:00.000Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 1,
      "totalPages": 1
    }
  }
}
```

### Create Token Configuration

**POST** `/token-configs`

Create a new token configuration.

**Request Body:**

```json
{
  "serviceName": "Premium Service",
  "description": "High-priority service for VIP customers",
  "priority": 10,
  "isActive": true
}
```

**Response:**

```json
{
  "success": true,
  "message": "Token configuration created successfully",
  "data": {
    "id": "new-config-id",
    "serviceName": "Premium Service",
    "description": "High-priority service for VIP customers",
    "priority": 10,
    "isActive": true,
    "createdAt": "2024-12-01T11:00:00.000Z"
  }
}
```

### Update Token Configuration

**PUT** `/token-configs/{configId}`

Update an existing token configuration.

**Request Body:**

```json
{
  "serviceName": "Updated Premium Service",
  "description": "Updated description",
  "priority": 8,
  "isActive": true
}
```

**Response:**

```json
{
  "success": true,
  "message": "Token configuration updated successfully",
  "data": {
    "id": "config-id",
    "serviceName": "Updated Premium Service",
    "description": "Updated description",
    "priority": 8,
    "isActive": true,
    "updatedAt": "2024-12-01T11:30:00.000Z"
  }
}
```

### Delete Token Configuration

**DELETE** `/token-configs/{configId}`

Delete a token configuration.

**Response:**

```json
{
  "success": true,
  "message": "Token configuration deleted successfully",
  "data": null
}
```

---

## Available Permissions

The following permissions are available for role assignment:

### User Management

- `user.create` - Create new users
- `user.read` - View user information
- `user.update` - Edit user details
- `user.delete` - Delete users
- `user.ban` - Ban/unban users
- `user.reset_password` - Reset user passwords

### Counter Management

- `counter.create` - Create new counters
- `counter.read` - View counter information
- `counter.update` - Edit counter details
- `counter.delete` - Delete counters
- `counter.assign` - Assign staff to counters

### Token Management

- `token.create` - Create queue tokens
- `token.read` - View token information
- `token.update` - Update token status
- `token.call` - Call tokens (next or specific)
- `token.serve` - Start serving tokens
- `token.complete` - Complete token service
- `token.no_show` - Mark tokens as no-show
- `token.recall` - Recall no-show tokens
- `token.announce` - Repeat token announcements

### Queue Management

- `queue.read` - View queue status
- `queue.manage` - Manage queue operations

### Token Configuration Management

- `token_config.create` - Create token configurations
- `token_config.read` - View token configurations
- `token_config.update` - Update token configurations
- `token_config.delete` - Delete token configurations

### Analytics

- `analytics.read` - View system analytics

### Settings

- `settings.read` - View system settings
- `settings.update` - Update system settings

---

## Error Codes

| Code | Description                                                |
| ---- | ---------------------------------------------------------- |
| 400  | Bad Request - Invalid input data                           |
| 401  | Unauthorized - Invalid or missing token                    |
| 403  | Forbidden - Insufficient permissions                       |
| 404  | Not Found - Resource not found                             |
| 409  | Conflict - Resource already exists or constraint violation |
| 500  | Internal Server Error - Server error                       |

---

## Rate Limiting

The API implements rate limiting to prevent abuse:

- **Authentication endpoints**: 5 requests per minute per IP
- **Other endpoints**: 100 requests per minute per IP

---

## Demo Credentials

For testing purposes, use these credentials:

| Role        | Username | Password |
| ----------- | -------- | -------- |
| Super Admin | admin    | admin123 |
| Admin       | manager  | admin456 |
| Staff       | staff1   | staff123 |
