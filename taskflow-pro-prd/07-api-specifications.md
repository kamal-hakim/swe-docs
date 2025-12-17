# API Specifications

## Overview

This document provides complete REST API specifications for TaskFlow Pro, including endpoints, request/response formats, authentication requirements, error handling, and rate limiting. All APIs follow RESTful conventions and return JSON responses.

---

## API Base Information

### Base URLs

| Environment     | URL                                    | Purpose           |
| --------------- | -------------------------------------- | ----------------- |
| **Development** | `http://localhost:3000/api`            | Local development |
| **Staging**     | `https://staging-api.taskflow.pro/api` | Testing and QA    |
| **Production**  | `https://api.taskflow.pro/api`         | Live application  |

### API Versioning

Current Version:**v1**  
Version Strategy: URL versioning (`/api/v1/...`)  
Deprecation Policy: 6 months notice before removal

### Authentication

All authenticated endpoints require an `Authorization` header:

```http
Authorization: Bearer <access_token>
```

Token Format: JWT (JSON Web Token)  
Token Lifespan: 15 minutes (access), 7 days (refresh)  
Refresh Endpoint: `POST /api/auth/refresh`

### Content Type

All requests and responses use JSON:

```http
Content-Type: application/json
Accept: application/json
```

---

## Authentication Endpoints

### POST /api/auth/register

Create a new user account.

**Request Body**:

```typescript
interface RegisterRequest {
  firstName: string; // Required, max 50 chars
  lastName: string; // Required, max 50 chars
  email: string; // Required, must be unique, valid email
  password: string; // Required, min 8 chars, complexity rules
  acceptTerms: boolean; // Required, must be true
}
```

**Example Request**:

```json
{
  "firstName": "Sarah",
  "lastName": "Chen",
  "email": "sarah.chen@example.com",
  "password": "SecurePass123!",
  "acceptTerms": true
}
```

**Success Response (201 Created)**:

```typescript
interface RegisterResponse {
  user: {
    id: string;
    email: string;
    firstName: string;
    lastName: string;
    role: "member";
    verified: false;
    createdAt: string;
  };
  message: string;
}
```

**Example Response**:

```json
{
  "user": {
    "id": "usr_7f8a9b6c5d4e3f2a",
    "email": "sarah.chen@example.com",
    "firstName": "Sarah",
    "lastName": "Chen",
    "role": "member",
    "verified": false,
    "createdAt": "2024-12-16T10:30:00Z"
  },
  "message": "Account created. Please check your email to verify your account."
}
```

**Error Responses**:

```typescript
// 400 Bad Request - Validation Error
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": {
      "email": ["Email is already registered"],
      "password": ["Password must contain at least one special character"]
    }
  }
}

// 409 Conflict - Duplicate Email
{
  "error": {
    "code": "DUPLICATE_EMAIL",
    "message": "An account with this email already exists"
  }
}

// 429 Too Many Requests - Rate Limit
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many registration attempts. Please try again in 15 minutes.",
    "retryAfter": 900
  }
}
```

---

### POST /api/auth/login

Authenticate user and receive access tokens.

**Request Body**:

```typescript
interface LoginRequest {
  email: string;
  password: string;
  remember?: boolean; // Optional, default false
}
```

**Example Request**:

```json
{
  "email": "sarah.chen@example.com",
  "password": "SecurePass123!",
  "remember": true
}
```

**Success Response (200 OK)**:

```typescript
interface LoginResponse {
  tokens: {
    accessToken: string;
    refreshToken: string;
    expiresAt: number; // Unix timestamp
    expiresIn: number; // Seconds
  };
  user: {
    id: string;
    email: string;
    firstName: string;
    lastName: string;
    avatar: string | null;
    role: "admin" | "manager" | "member" | "guest";
    verified: boolean;
    preferences: {
      theme: "light" | "dark" | "auto";
      language: string;
      timezone: string;
    };
  };
}
```

**Example Response**:

```json
{
  "tokens": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresAt": 1702729800000,
    "expiresIn": 900
  },
  "user": {
    "id": "usr_7f8a9b6c5d4e3f2a",
    "email": "sarah.chen@example.com",
    "firstName": "Sarah",
    "lastName": "Chen",
    "avatar": "https://cdn.taskflow.pro/avatars/usr_7f8a9b6c5d4e3f2a.jpg",
    "role": "manager",
    "verified": true,
    "preferences": {
      "theme": "dark",
      "language": "en",
      "timezone": "America/Los_Angeles"
    }
  }
}
```

**Error Responses**:

```typescript
// 401 Unauthorized - Invalid Credentials
{
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Invalid email or password"
  }
}

// 403 Forbidden - Account Not Verified
{
  "error": {
    "code": "ACCOUNT_NOT_VERIFIED",
    "message": "Please verify your email address before logging in",
    "meta": {
      "resendVerificationUrl": "/api/auth/resend-verification"
    }
  }
}

// 423 Locked - Account Locked
{
  "error": {
    "code": "ACCOUNT_LOCKED",
    "message": "Account locked due to too many failed login attempts",
    "meta": {
      "unlockAt": "2024-12-16T11:30:00Z"
    }
  }
}
```

---

### POST /api/auth/refresh

Refresh access token using refresh token.

**Request Body**:

```typescript
interface RefreshRequest {
  refreshToken: string;
}
```

**Success Response (200 OK)**:

```typescript
interface RefreshResponse {
  accessToken: string;
  expiresAt: number;
  expiresIn: number;
}
```

---

### POST /api/auth/logout

Invalidate current session tokens.

**Headers**: `Authorization: Bearer <access_token>`

**Success Response (204 No Content)**

---

### GET /api/auth/me

Get current authenticated user information.

**Headers**: `Authorization: Bearer <access_token>`

**Success Response (200 OK)**:

```typescript
interface MeResponse {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  avatar: string | null;
  role: "admin" | "manager" | "member" | "guest";
  verified: boolean;
  preferences: UserPreferences;
  workspaceId: string;
  workspace: {
    id: string;
    name: string;
    slug: string;
  };
  createdAt: string;
  updatedAt: string;
}
```

---

## Project Endpoints

### GET /api/projects

Get list of projects accessible to the current user.

**Headers**: `Authorization: Bearer <access_token>`

**Query Parameters**:

```typescript
interface ProjectListParams {
  status?: "active" | "archived" | "completed"; // Filter by status
  search?: string; // Search in name/description
  sort?: "name" | "createdAt" | "updatedAt" | "dueDate";
  order?: "asc" | "desc";
  page?: number; // Page number (1-indexed)
  limit?: number; // Items per page (default: 20, max: 100)
}
```

**Example Request**:

```http
GET /api/projects?status=active&sort=updatedAt&order=desc&page=1&limit=20
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Success Response (200 OK)**:

```typescript
interface ProjectListResponse {
  data: Project[];
  meta: {
    total: number;
    page: number;
    limit: number;
    totalPages: number;
  };
}

interface Project {
  id: string;
  name: string;
  description: string | null;
  color: string;
  icon: string;

  startDate: string;
  endDate: string | null;

  ownerId: string;
  owner: {
    id: string;
    firstName: string;
    lastName: string;
    avatar: string | null;
  };

  visibility: "private" | "team" | "public";
  status: "active" | "archived" | "completed";

  // Statistics
  stats: {
    totalTasks: number;
    completedTasks: number;
    inProgressTasks: number;
    todoTasks: number;
    progressPercentage: number;
    memberCount: number;
  };

  createdAt: string;
  updatedAt: string;
}
```

**Example Response**:

```json
{
  "data": [
    {
      "id": "prj_a1b2c3d4e5f6",
      "name": "Website Redesign",
      "description": "Complete overhaul of company website with new branding",
      "color": "#6366F1",
      "icon": "mdi-web",
      "startDate": "2024-12-01T00:00:00Z",
      "endDate": "2025-02-28T23:59:59Z",
      "ownerId": "usr_7f8a9b6c5d4e3f2a",
      "owner": {
        "id": "usr_7f8a9b6c5d4e3f2a",
        "firstName": "Sarah",
        "lastName": "Chen",
        "avatar": "https://cdn.taskflow.pro/avatars/usr_7f8a9b6c5d4e3f2a.jpg"
      },
      "visibility": "team",
      "status": "active",
      "stats": {
        "totalTasks": 45,
        "completedTasks": 12,
        "inProgressTasks": 18,
        "todoTasks": 15,
        "progressPercentage": 27,
        "memberCount": 8
      },
      "createdAt": "2024-11-15T09:00:00Z",
      "updatedAt": "2024-12-16T10:45:00Z"
    }
  ],
  "meta": {
    "total": 12,
    "page": 1,
    "limit": 20,
    "totalPages": 1
  }
}
```

---

### POST /api/projects

Create a new project.

**Headers**: `Authorization: Bearer <access_token>`

**Request Body**:

```typescript
interface CreateProjectRequest {
  name: string; // Required, max 100 chars
  description?: string | null; // Optional, max 2000 chars
  color?: string; // Optional, hex color, default: random
  icon?: string; // Optional, MDI icon name, default: 'mdi-folder'
  startDate: string; // Required, ISO 8601 date
  endDate?: string | null; // Optional, ISO 8601 date, must be > startDate
  visibility?: "private" | "team" | "public"; // Optional, default: 'team'
  templateId?: string | null; // Optional, project template to use
  members?: {
    // Optional, initial team members
    userId: string;
    role: "admin" | "member";
  }[];
}
```

**Example Request**:

```json
{
  "name": "Mobile App Development",
  "description": "Native iOS and Android app for customer portal",
  "color": "#10B981",
  "icon": "mdi-cellphone",
  "startDate": "2024-12-20T00:00:00Z",
  "endDate": "2025-03-31T23:59:59Z",
  "visibility": "team",
  "members": [
    { "userId": "usr_abc123", "role": "admin" },
    { "userId": "usr_def456", "role": "member" }
  ]
}
```

**Success Response (201 Created)**:

```json
{
  "id": "prj_x9y8z7w6v5u4",
  "name": "Mobile App Development",
  "description": "Native iOS and Android app for customer portal",
  "color": "#10B981",
  "icon": "mdi-cellphone",
  "startDate": "2024-12-20T00:00:00Z",
  "endDate": "2025-03-31T23:59:59Z",
  "ownerId": "usr_7f8a9b6c5d4e3f2a",
  "owner": {
    "id": "usr_7f8a9b6c5d4e3f2a",
    "firstName": "Sarah",
    "lastName": "Chen",
    "avatar": "https://cdn.taskflow.pro/avatars/usr_7f8a9b6c5d4e3f2a.jpg"
  },
  "visibility": "team",
  "status": "active",
  "statuses": [
    {
      "id": "sts_001",
      "name": "To Do",
      "color": "#9CA3AF",
      "order": 0,
      "category": "todo"
    },
    {
      "id": "sts_002",
      "name": "In Progress",
      "color": "#3B82F6",
      "order": 1,
      "category": "in_progress"
    },
    {
      "id": "sts_003",
      "name": "Done",
      "color": "#22C55E",
      "order": 2,
      "category": "done"
    }
  ],
  "members": [
    {
      "id": "pm_001",
      "userId": "usr_7f8a9b6c5d4e3f2a",
      "role": "owner",
      "joinedAt": "2024-12-16T10:50:00Z"
    },
    {
      "id": "pm_002",
      "userId": "usr_abc123",
      "role": "admin",
      "joinedAt": "2024-12-16T10:50:00Z"
    },
    {
      "id": "pm_003",
      "userId": "usr_def456",
      "role": "member",
      "joinedAt": "2024-12-16T10:50:00Z"
    }
  ],
  "stats": {
    "totalTasks": 0,
    "completedTasks": 0,
    "inProgressTasks": 0,
    "todoTasks": 0,
    "progressPercentage": 0,
    "memberCount": 3
  },
  "createdAt": "2024-12-16T10:50:00Z",
  "updatedAt": "2024-12-16T10:50:00Z"
}
```

**Error Responses**:

```json
// 400 Bad Request
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": {
      "endDate": ["End date must be after start date"]
    }
  }
}

// 403 Forbidden
{
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "You don't have permission to create projects"
  }
}
```

---

### GET /api/projects/:id

Get detailed information about a specific project.

**Headers**: `Authorization: Bearer <access_token>`

**URL Parameters**:

- `id` (string, required): Project ID

**Example Request**:

```http
GET /api/projects/prj_a1b2c3d4e5f6
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Success Response (200 OK)**:

```json
{
  "id": "prj_a1b2c3d4e5f6",
  "name": "Website Redesign",
  "description": "Complete overhaul of company website with new branding",
  "color": "#6366F1",
  "icon": "mdi-web",
  "startDate": "2024-12-01T00:00:00Z",
  "endDate": "2025-02-28T23:59:59Z",
  "ownerId": "usr_7f8a9b6c5d4e3f2a",
  "owner": {
    "id": "usr_7f8a9b6c5d4e3f2a",
    "firstName": "Sarah",
    "lastName": "Chen",
    "avatar": "https://cdn.taskflow.pro/avatars/usr_7f8a9b6c5d4e3f2a.jpg"
  },
  "visibility": "team",
  "status": "active",
  "statuses": [
    {
      "id": "sts_001",
      "name": "To Do",
      "color": "#9CA3AF",
      "order": 0,
      "category": "todo"
    },
    {
      "id": "sts_002",
      "name": "In Progress",
      "color": "#3B82F6",
      "order": 1,
      "category": "in_progress"
    },
    {
      "id": "sts_003",
      "name": "In Review",
      "color": "#F59E0B",
      "order": 2,
      "category": "in_progress"
    },
    {
      "id": "sts_004",
      "name": "Done",
      "color": "#22C55E",
      "order": 3,
      "category": "done"
    }
  ],
  "members": [
    {
      "id": "pm_001",
      "userId": "usr_7f8a9b6c5d4e3f2a",
      "user": {
        "id": "usr_7f8a9b6c5d4e3f2a",
        "firstName": "Sarah",
        "lastName": "Chen",
        "avatar": "https://cdn.taskflow.pro/avatars/usr_7f8a9b6c5d4e3f2a.jpg",
        "title": "Project Manager"
      },
      "role": "owner",
      "joinedAt": "2024-11-15T09:00:00Z"
    },
    {
      "id": "pm_002",
      "userId": "usr_abc123",
      "user": {
        "id": "usr_abc123",
        "firstName": "Marcus",
        "lastName": "Rodriguez",
        "avatar": "https://cdn.taskflow.pro/avatars/usr_abc123.jpg",
        "title": "Full-Stack Developer"
      },
      "role": "member",
      "joinedAt": "2024-11-15T09:30:00Z"
    }
  ],
  "labels": [
    { "id": "lbl_001", "name": "Bug", "color": "#EF4444" },
    { "id": "lbl_002", "name": "Feature", "color": "#3B82F6" },
    { "id": "lbl_003", "name": "Documentation", "color": "#8B5CF6" }
  ],
  "stats": {
    "totalTasks": 45,
    "completedTasks": 12,
    "inProgressTasks": 18,
    "todoTasks": 15,
    "progressPercentage": 27,
    "memberCount": 8
  },
  "createdAt": "2024-11-15T09:00:00Z",
  "updatedAt": "2024-12-16T10:45:00Z"
}
```

---

## Task Endpoints

### GET /api/tasks

Get list of tasks with filtering and pagination.

**Headers**: `Authorization: Bearer <access_token>`

**Query Parameters**:

```typescript
interface TaskListParams {
  projectId?: string; // Filter by project
  statusId?: string; // Filter by status
  assigneeId?: string; // Filter by assignee
  priority?: "low" | "medium" | "high" | "critical";
  labels?: string; // Comma-separated label IDs
  search?: string; // Search in title/description
  dueBefore?: string; // ISO 8601 date
  dueAfter?: string; // ISO 8601 date
  includeCompleted?: boolean; // Default: true
  sort?: "title" | "createdAt" | "updatedAt" | "dueDate" | "priority";
  order?: "asc" | "desc";
  page?: number;
  limit?: number;
}
```

**Example Request**:

```http
GET /api/tasks?projectId=prj_a1b2c3d4e5f6&statusId=sts_002&assigneeId=me&page=1&limit=20
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Success Response (200 OK)**:

```typescript
interface TaskListResponse {
  data: Task[];
  meta: {
    total: number;
    page: number;
    limit: number;
    totalPages: number;
  };
}

interface Task {
  id: string;
  title: string;
  description: string | null;

  projectId: string;
  project: {
    id: string;
    name: string;
    color: string;
  };

  statusId: string;
  status: {
    id: string;
    name: string;
    color: string;
    category: "todo" | "in_progress" | "done";
  };

  priority: "low" | "medium" | "high" | "critical";

  assignees: {
    id: string;
    firstName: string;
    lastName: string;
    avatar: string | null;
  }[];

  labels: {
    id: string;
    name: string;
    color: string;
  }[];

  dueDate: string | null;
  startDate: string | null;
  completedAt: string | null;

  estimatedHours: number | null;
  trackedHours: number;

  commentCount: number;
  attachmentCount: number;
  subtaskCount: number;
  completedSubtaskCount: number;

  createdById: string;
  createdBy: {
    id: string;
    firstName: string;
    lastName: string;
  };

  createdAt: string;
  updatedAt: string;
}
```

**Example Response**:

```json
{
  "data": [
    {
      "id": "tsk_001",
      "title": "Implement user authentication flow",
      "description": "Create login, register, and password reset pages with form validation",
      "projectId": "prj_a1b2c3d4e5f6",
      "project": {
        "id": "prj_a1b2c3d4e5f6",
        "name": "Website Redesign",
        "color": "#6366F1"
      },
      "statusId": "sts_002",
      "status": {
        "id": "sts_002",
        "name": "In Progress",
        "color": "#3B82F6",
        "category": "in_progress"
      },
      "priority": "high",
      "assignees": [
        {
          "id": "usr_abc123",
          "firstName": "Marcus",
          "lastName": "Rodriguez",
          "avatar": "https://cdn.taskflow.pro/avatars/usr_abc123.jpg"
        }
      ],
      "labels": [{ "id": "lbl_002", "name": "Feature", "color": "#3B82F6" }],
      "dueDate": "2024-12-20T17:00:00Z",
      "startDate": "2024-12-16T09:00:00Z",
      "completedAt": null,
      "estimatedHours": 8,
      "trackedHours": 4.5,
      "commentCount": 3,
      "attachmentCount": 2,
      "subtaskCount": 5,
      "completedSubtaskCount": 2,
      "createdById": "usr_7f8a9b6c5d4e3f2a",
      "createdBy": {
        "id": "usr_7f8a9b6c5d4e3f2a",
        "firstName": "Sarah",
        "lastName": "Chen"
      },
      "createdAt": "2024-12-16T09:00:00Z",
      "updatedAt": "2024-12-16T11:30:00Z"
    }
  ],
  "meta": {
    "total": 18,
    "page": 1,
    "limit": 20,
    "totalPages": 1
  }
}
```

---

### POST /api/tasks

Create a new task.

**Headers**: `Authorization: Bearer <access_token>`

**Request Body**:

```typescript
interface CreateTaskRequest {
  title: string; // Required, max 200 chars
  description?: string | null; // Optional, markdown supported
  projectId: string; // Required, must have access
  statusId?: string; // Optional, defaults to first status
  priority?: "low" | "medium" | "high" | "critical"; // Default: 'medium'
  assigneeIds?: string[]; // Optional, user IDs
  labelIds?: string[]; // Optional, label IDs
  dueDate?: string | null; // Optional, ISO 8601
  startDate?: string | null; // Optional, ISO 8601
  estimatedHours?: number | null; // Optional, positive number
  parentTaskId?: string | null; // Optional, for subtasks
}
```

**Example Request**:

```json
{
  "title": "Design mockups for dashboard page",
  "description": "Create high-fidelity mockups in Figma with light and dark theme variants",
  "projectId": "prj_a1b2c3d4e5f6",
  "statusId": "sts_001",
  "priority": "high",
  "assigneeIds": ["usr_designer01"],
  "labelIds": ["lbl_design", "lbl_ui"],
  "dueDate": "2024-12-20T17:00:00Z",
  "estimatedHours": 6
}
```

**Success Response (201 Created)**:

```json
{
  "id": "tsk_new123",
  "title": "Design mockups for dashboard page",
  "description": "Create high-fidelity mockups in Figma with light and dark theme variants",
  "projectId": "prj_a1b2c3d4e5f6",
  "project": {
    "id": "prj_a1b2c3d4e5f6",
    "name": "Website Redesign",
    "color": "#6366F1"
  },
  "statusId": "sts_001",
  "status": {
    "id": "sts_001",
    "name": "To Do",
    "color": "#9CA3AF",
    "category": "todo"
  },
  "priority": "high",
  "assignees": [
    {
      "id": "usr_designer01",
      "firstName": "Jessica",
      "lastName": "Martinez",
      "avatar": "https://cdn.taskflow.pro/avatars/usr_designer01.jpg"
    }
  ],
  "labels": [
    { "id": "lbl_design", "name": "Design", "color": "#EC4899" },
    { "id": "lbl_ui", "name": "UI", "color": "#8B5CF6" }
  ],
  "dueDate": "2024-12-20T17:00:00Z",
  "startDate": null,
  "completedAt": null,
  "estimatedHours": 6,
  "trackedHours": 0,
  "commentCount": 0,
  "attachmentCount": 0,
  "subtaskCount": 0,
  "completedSubtaskCount": 0,
  "createdById": "usr_7f8a9b6c5d4e3f2a",
  "createdBy": {
    "id": "usr_7f8a9b6c5d4e3f2a",
    "firstName": "Sarah",
    "lastName": "Chen"
  },
  "createdAt": "2024-12-16T11:00:00Z",
  "updatedAt": "2024-12-16T11:00:00Z"
}
```

---

### PATCH /api/tasks/:id

Update an existing task (partial update).

**Headers**: `Authorization: Bearer <access_token>`

**URL Parameters**:

- `id` (string, required): Task ID

**Request Body** (all fields optional):

```typescript
interface UpdateTaskRequest {
  title?: string;
  description?: string | null;
  statusId?: string;
  priority?: "low" | "medium" | "high" | "critical";
  assigneeIds?: string[];
  labelIds?: string[];
  dueDate?: string | null;
  startDate?: string | null;
  estimatedHours?: number | null;
  completedAt?: string | null; // Set to mark complete
}
```

**Example Request**:

```json
{
  "statusId": "sts_003",
  "completedAt": "2024-12-16T11:30:00Z"
}
```

**Success Response (200 OK)**:

```json
{
  "id": "tsk_001",
  "title": "Implement user authentication flow",
  "statusId": "sts_003",
  "status": {
    "id": "sts_003",
    "name": "Done",
    "color": "#22C55E",
    "category": "done"
  },
  "completedAt": "2024-12-16T11:30:00Z",
  "updatedAt": "2024-12-16T11:30:00Z"
}
```

---

### DELETE /api/tasks/:id

Delete a task (soft delete).

**Headers**: `Authorization: Bearer <access_token>`

**URL Parameters**:

- `id` (string, required): Task ID

**Success Response (204 No Content)**

**Error Response**:

```json
// 403 Forbidden - Insufficient Permissions
{
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "You don't have permission to delete this task"
  }
}

// 409 Conflict - Has Dependencies
{
  "error": {
    "code": "HAS_DEPENDENCIES",
    "message": "Cannot delete task with active dependencies",
    "details": {
      "dependentTasks": ["tsk_002", "tsk_003"]
    }
  }
}
```

---

## Comment Endpoints

### POST /api/tasks/:taskId/comments

Add a comment to a task.

**Headers**: `Authorization: Bearer <access_token>`

**URL Parameters**:

- `taskId` (string, required): Task ID

**Request Body**:

```typescript
interface CreateCommentRequest {
  text: string; // Required, max 5000 chars, markdown supported
  attachments?: {
    // Optional file attachments
    fileName: string;
    fileSize: number;
    fileType: string;
    fileUrl: string; // Pre-uploaded to storage
  }[];
}
```

**Example Request**:

```json
{
  "text": "@marcus Great progress on this! Can you also add error handling for the edge case we discussed? cc: @emily",
  "attachments": [
    {
      "fileName": "error-flow.png",
      "fileSize": 245680,
      "fileType": "image/png",
      "fileUrl": "https://cdn.taskflow.pro/attachments/abc123.png"
    }
  ]
}
```

**Success Response (201 Created)**:

```json
{
  "id": "cmt_xyz789",
  "taskId": "tsk_001",
  "text": "@marcus Great progress on this! Can you also add error handling for the edge case we discussed? cc: @emily",
  "authorId": "usr_7f8a9b6c5d4e3f2a",
  "author": {
    "id": "usr_7f8a9b6c5d4e3f2a",
    "firstName": "Sarah",
    "lastName": "Chen",
    "avatar": "https://cdn.taskflow.pro/avatars/usr_7f8a9b6c5d4e3f2a.jpg"
  },
  "attachments": [
    {
      "id": "att_001",
      "fileName": "error-flow.png",
      "fileSize": 245680,
      "fileType": "image/png",
      "fileUrl": "https://cdn.taskflow.pro/attachments/abc123.png",
      "thumbnailUrl": "https://cdn.taskflow.pro/thumbnails/abc123_thumb.png",
      "uploadedAt": "2024-12-16T11:35:00Z"
    }
  ],
  "mentions": [
    { "userId": "usr_abc123", "userName": "marcus" },
    { "userId": "usr_emily01", "userName": "emily" }
  ],
  "edited": false,
  "createdAt": "2024-12-16T11:35:00Z",
  "updatedAt": "2024-12-16T11:35:00Z"
}
```

---

### GET /api/tasks/:taskId/comments

Get all comments for a task.

**Headers**: `Authorization: Bearer <access_token>`

**Query Parameters**:

```typescript
interface CommentListParams {
  page?: number;
  limit?: number; // Default: 50, max: 100
  sort?: "createdAt" | "updatedAt";
  order?: "asc" | "desc"; // Default: asc (oldest first)
}
```

**Success Response (200 OK)**:

```json
{
  "data": [
    {
      "id": "cmt_001",
      "taskId": "tsk_001",
      "text": "Started working on this. Implementing the login form first.",
      "authorId": "usr_abc123",
      "author": {
        "id": "usr_abc123",
        "firstName": "Marcus",
        "lastName": "Rodriguez",
        "avatar": "https://cdn.taskflow.pro/avatars/usr_abc123.jpg"
      },
      "attachments": [],
      "mentions": [],
      "edited": false,
      "createdAt": "2024-12-16T09:15:00Z",
      "updatedAt": "2024-12-16T09:15:00Z"
    },
    {
      "id": "cmt_002",
      "taskId": "tsk_001",
      "text": "@marcus Looks good! Make sure to handle the forgot password flow as well.",
      "authorId": "usr_7f8a9b6c5d4e3f2a",
      "author": {
        "id": "usr_7f8a9b6c5d4e3f2a",
        "firstName": "Sarah",
        "lastName": "Chen",
        "avatar": "https://cdn.taskflow.pro/avatars/usr_7f8a9b6c5d4e3f2a.jpg"
      },
      "attachments": [],
      "mentions": [{ "userId": "usr_abc123", "userName": "marcus" }],
      "edited": false,
      "createdAt": "2024-12-16T10:20:00Z",
      "updatedAt": "2024-12-16T10:20:00Z"
    }
  ],
  "meta": {
    "total": 3,
    "page": 1,
    "limit": 50,
    "totalPages": 1
  }
}
```

---

## Notification Endpoints

### GET /api/notifications

Get user's notifications.

**Headers**: `Authorization: Bearer <access_token>`

**Query Parameters**:

```typescript
interface NotificationListParams {
  unreadOnly?: boolean; // Default: false
  type?: string; // Filter by notification type
  page?: number;
  limit?: number; // Default: 50, max: 100
}
```

**Success Response (200 OK)**:

```json
{
  "data": [
    {
      "id": "notif_001",
      "type": "mention",
      "title": "You were mentioned",
      "message": "@sarah mentioned you in a comment",
      "icon": "mdi-at",
      "color": "info",
      "userId": "usr_abc123",
      "taskId": "tsk_001",
      "projectId": "prj_a1b2c3d4e5f6",
      "commentId": "cmt_002",
      "read": false,
      "readAt": null,
      "actionUrl": "/projects/prj_a1b2c3d4e5f6/tasks/tsk_001",
      "actionText": "View Task",
      "createdAt": "2024-12-16T10:20:00Z"
    }
  ],
  "meta": {
    "total": 15,
    "unreadCount": 8,
    "page": 1,
    "limit": 50,
    "totalPages": 1
  }
}
```

---

### POST /api/notifications/:id/read

Mark notification as read.

**Headers**: `Authorization: Bearer <access_token>`

**Success Response (200 OK)**:

```json
{
  "id": "notif_001",
  "read": true,
  "readAt": "2024-12-16T11:40:00Z"
}
```

---

### POST /api/notifications/read-all

Mark all notifications as read.

**Headers**: `Authorization: Bearer <access_token>`

**Success Response (200 OK)**:

```json
{
  "message": "All notifications marked as read",
  "count": 8
}
```

---

## File Upload Endpoints

### POST /api/upload

Upload a file (avatar, task attachment, etc.).

**Headers**:

```http
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
```

**Request Body** (multipart/form-data):

```typescript
interface UploadRequest {
  file: File; // Required, max 10MB
  type: "avatar" | "attachment" | "export";
  relatedId?: string; // Optional, related entity ID
}
```

**Example Request (multipart form)**:

```
POST /api/upload
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="avatar.jpg"
Content-Type: image/jpeg

[binary data]
------WebKitFormBoundary
Content-Disposition: form-data; name="type"

avatar
------WebKitFormBoundary--
```

**Success Response (200 OK)**:

```json
{
  "id": "att_abc123",
  "fileName": "avatar.jpg",
  "fileSize": 245680,
  "fileType": "image/jpeg",
  "url": "https://cdn.taskflow.pro/uploads/avatar_abc123.jpg",
  "thumbnailUrl": "https://cdn.taskflow.pro/uploads/avatar_abc123_thumb.jpg",
  "uploadedAt": "2024-12-16T11:45:00Z",
  "expiresAt": null
}
```

**Error Response**:

```json
// 413 Payload Too Large
{
  "error": {
    "code": "FILE_TOO_LARGE",
    "message": "File size exceeds 10MB limit",
    "details": {
      "maxSize": 10485760,
      "actualSize": 15728640
    }
  }
}

// 415 Unsupported Media Type
{
  "error": {
    "code": "UNSUPPORTED_FILE_TYPE",
    "message": "File type not allowed",
    "details": {
      "allowedTypes": ["image/jpeg", "image/png", "image/gif", "application/pdf"]
    }
  }
}
```

---

## Error Handling

### Standard Error Response Format

All API errors follow this consistent structure:

```typescript
interface ApiError {
  error: {
    code: string; // Machine-readable error code
    message: string; // Human-readable error message
    details?: any; // Additional error context
    timestamp?: string; // ISO 8601 timestamp
    requestId?: string; // Unique request ID for tracing
  };
}
```

### HTTP Status Codes

| Code    | Meaning                | Usage                                           |
| ------- | ---------------------- | ----------------------------------------------- |
| **200** | OK                     | Successful GET, PATCH requests                  |
| **201** | Created                | Successful POST creating resource               |
| **204** | No Content             | Successful DELETE                               |
| **400** | Bad Request            | Validation errors, malformed requests           |
| **401** | Unauthorized           | Missing or invalid authentication               |
| **403** | Forbidden              | Authenticated but not authorized                |
| **404** | Not Found              | Resource doesn't exist                          |
| **409** | Conflict               | Resource conflict (duplicate, version mismatch) |
| **413** | Payload Too Large      | Request body or file too large                  |
| **415** | Unsupported Media Type | Invalid content type                            |
| **422** | Unprocessable Entity   | Valid syntax but semantic errors                |
| **429** | Too Many Requests      | Rate limit exceeded                             |
| **500** | Internal Server Error  | Server-side error                               |
| **503** | Service Unavailable    | Temporary service disruption                    |

### Error Codes

| Code                       | Description                      | HTTP Status |
| -------------------------- | -------------------------------- | ----------- |
| `VALIDATION_ERROR`         | Input validation failed          | 400         |
| `INVALID_CREDENTIALS`      | Wrong email/password             | 401         |
| `TOKEN_EXPIRED`            | Access token expired             | 401         |
| `TOKEN_INVALID`            | Malformed or invalid token       | 401         |
| `INSUFFICIENT_PERMISSIONS` | User lacks required permission   | 403         |
| `ACCOUNT_NOT_VERIFIED`     | Email not verified               | 403         |
| `ACCOUNT_LOCKED`           | Account temporarily locked       | 423         |
| `RESOURCE_NOT_FOUND`       | Requested resource doesn't exist | 404         |
| `DUPLICATE_EMAIL`          | Email already registered         | 409         |
| `DUPLICATE_RESOURCE`       | Resource already exists          | 409         |
| `VERSION_CONFLICT`         | Optimistic locking conflict      | 409         |
| `FILE_TOO_LARGE`           | File exceeds size limit          | 413         |
| `UNSUPPORTED_FILE_TYPE`    | File type not allowed            | 415         |
| `RATE_LIMIT_EXCEEDED`      | Too many requests                | 429         |
| `INTERNAL_ERROR`           | Unexpected server error          | 500         |
| `SERVICE_UNAVAILABLE`      | Service temporarily down         | 503         |

---

## Rate Limiting

### Rate Limit Headers

All API responses include rate limit information:

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1702730400
Retry-After: 60
```

### Rate Limit Policies

| Endpoint Category                 | Limit        | Window     | Scope    |
| --------------------------------- | ------------ | ---------- | -------- |
| **Authentication**                | 5 requests   | 15 minutes | Per IP   |
| **Read Operations (GET)**         | 100 requests | 1 minute   | Per user |
| **Write Operations (POST/PATCH)** | 60 requests  | 1 minute   | Per user |
| **File Uploads**                  | 10 requests  | 1 minute   | Per user |
| **Search**                        | 30 requests  | 1 minute   | Per user |

### Rate Limit Exceeded Response

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again later.",
    "details": {
      "limit": 100,
      "window": "1 minute",
      "retryAfter": 45
    }
  }
}
```

---

## Pagination

### Pagination Parameters

All list endpoints support pagination:

| Parameter | Type   | Default | Max |
| --------- | ------ | ------- | --- |
| `page`    | number | 1       | N/A |
| `limit`   | number | 20      | 100 |

### Pagination Response Metadata

```typescript
interface PaginationMeta {
  total: number; // Total number of items
  page: number; // Current page (1-indexed)
  limit: number; // Items per page
  totalPages: number; // Total number of pages
  hasNextPage: boolean; // Whether there's a next page
  hasPrevPage: boolean; // Whether there's a previous page
}
```

### Pagination Links (Optional)

```json
{
  "data": [...],
  "meta": {
    "total": 156,
    "page": 2,
    "limit": 20,
    "totalPages": 8,
    "hasNextPage": true,
    "hasPrevPage": true
  },
  "links": {
    "first": "/api/tasks?page=1&limit=20",
    "prev": "/api/tasks?page=1&limit=20",
    "next": "/api/tasks?page=3&limit=20",
    "last": "/api/tasks?page=8&limit=20"
  }
}
```

---

## Webhooks (Future Enhancement)

### Webhook Event Types

| Event             | Trigger                 | Payload               |
| ----------------- | ----------------------- | --------------------- |
| `task.created`    | New task created        | Task object           |
| `task.updated`    | Task updated            | Task object + changes |
| `task.deleted`    | Task deleted            | Task ID               |
| `task.completed`  | Task marked complete    | Task object           |
| `comment.created` | New comment             | Comment object        |
| `project.created` | New project             | Project object        |
| `member.joined`   | Member joined workspace | User + workspace      |

### Webhook Delivery

```json
POST https://customer-webhook-url.com/webhook
Content-Type: application/json
X-TaskFlow-Signature: sha256=abc123...
X-TaskFlow-Event: task.created

{
  "event": "task.created",
  "timestamp": "2024-12-16T11:50:00Z",
  "workspaceId": "ws_001",
  "data": {
    "id": "tsk_new123",
    "title": "New task title",
    ...
  }
}
```

---

## API Client Examples

### TypeScript/JavaScript Client

```typescript
// utils/api-client.ts
import { useRuntimeConfig } from "#app";

class TaskFlowApiClient {
  private baseUrl: string;
  private accessToken: string | null = null;

  constructor() {
    const config = useRuntimeConfig();
    this.baseUrl = config.public.apiBaseUrl;
  }

  setAccessToken(token: string) {
    this.accessToken = token;
  }

  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const headers = new Headers(options.headers);
    headers.set("Content-Type", "application/json");

    if (this.accessToken) {
      headers.set("Authorization", `Bearer ${this.accessToken}`);
    }

    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      ...options,
      headers,
    });

    if (!response.ok) {
      const error = await response.json();
      throw new ApiError(
        error.error.code,
        error.error.message,
        response.status
      );
    }

    if (response.status === 204) {
      return null as T;
    }

    return response.json();
  }

  // Auth methods
  async register(data: RegisterRequest): Promise<RegisterResponse> {
    return this.request<RegisterResponse>("/auth/register", {
      method: "POST",
      body: JSON.stringify(data),
    });
  }

  async login(credentials: LoginRequest): Promise<LoginResponse> {
    const response = await this.request<LoginResponse>("/auth/login", {
      method: "POST",
      body: JSON.stringify(credentials),
    });

    this.setAccessToken(response.tokens.accessToken);
    return response;
  }

  async logout(): Promise<void> {
    await this.request<void>("/auth/logout", { method: "POST" });
    this.accessToken = null;
  }

  // Project methods
  async getProjects(params?: ProjectListParams): Promise<ProjectListResponse> {
    const query = new URLSearchParams(params as any).toString();
    return this.request<ProjectListResponse>(`/projects?${query}`);
  }

  async getProject(id: string): Promise<Project> {
    return this.request<Project>(`/projects/${id}`);
  }

  async createProject(data: CreateProjectRequest): Promise<Project> {
    return this.request<Project>("/projects", {
      method: "POST",
      body: JSON.stringify(data),
    });
  }

  async updateProject(
    id: string,
    data: UpdateProjectRequest
  ): Promise<Project> {
    return this.request<Project>(`/projects/${id}`, {
      method: "PATCH",
      body: JSON.stringify(data),
    });
  }

  async deleteProject(id: string): Promise<void> {
    return this.request<void>(`/projects/${id}`, { method: "DELETE" });
  }

  // Task methods
  async getTasks(params?: TaskListParams): Promise<TaskListResponse> {
    const query = new URLSearchParams(params as any).toString();
    return this.request<TaskListResponse>(`/tasks?${query}`);
  }

  async createTask(data: CreateTaskRequest): Promise<Task> {
    return this.request<Task>("/tasks", {
      method: "POST",
      body: JSON.stringify(data),
    });
  }

  async updateTask(id: string, data: UpdateTaskRequest): Promise<Task> {
    return this.request<Task>(`/tasks/${id}`, {
      method: "PATCH",
      body: JSON.stringify(data),
    });
  }

  // Comment methods
  async addComment(
    taskId: string,
    data: CreateCommentRequest
  ): Promise<Comment> {
    return this.request<Comment>(`/tasks/${taskId}/comments`, {
      method: "POST",
      body: JSON.stringify(data),
    });
  }
}

// API Error class
class ApiError extends Error {
  constructor(public code: string, message: string, public status: number) {
    super(message);
    this.name = "ApiError";
  }
}

export const api = new TaskFlowApiClient();
```

### Usage in Composable

```typescript
// composables/api/useProjects.ts
import { api } from "~/utils/api-client";

export function useProjects() {
  const items = ref<Project[]>([]);
  const loading = ref(false);
  const error = ref<Error | null>(null);

  async function fetchProjects(params?: ProjectListParams) {
    loading.value = true;
    error.value = null;

    try {
      const response = await api.getProjects(params);
      items.value = response.data;
      return response;
    } catch (err) {
      error.value = err as Error;
      throw err;
    } finally {
      loading.value = false;
    }
  }

  async function createProject(data: CreateProjectRequest) {
    loading.value = true;
    error.value = null;

    try {
      const project = await api.createProject(data);
      items.value.push(project);
      return project;
    } catch (err) {
      error.value = err as Error;
      throw err;
    } finally {
      loading.value = false;
    }
  }

  return {
    items: readonly(items),
    loading: readonly(loading),
    error: readonly(error),
    fetchProjects,
    createProject,
  };
}
```

---

## API Implementation with Nuxt Server Routes

### Example: Auth Login Endpoint

```typescript
// server/api/auth/login.post.ts
import { z } from "zod";
import { hash, verify } from "argon2";
import { sign } from "jsonwebtoken";

// Validation schema
const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  remember: z.boolean().optional().default(false),
});

export default defineEventHandler(async (event) => {
  // Parse and validate request body
  const body = await readBody(event);
  const validatedData = loginSchema.parse(body);

  // Find user by email
  const user = await db.user.findUnique({
    where: { email: validatedData.email },
    include: { workspace: true },
  });

  if (!user) {
    throw createError({
      statusCode: 401,
      data: {
        error: {
          code: "INVALID_CREDENTIALS",
          message: "Invalid email or password",
        },
      },
    });
  }

  // Verify password
  const isValidPassword = await verify(
    user.passwordHash,
    validatedData.password
  );

  if (!isValidPassword) {
    // Increment failed login attempts
    await db.user.update({
      where: { id: user.id },
      data: { failedLoginAttempts: { increment: 1 } },
    });

    throw createError({
      statusCode: 401,
      data: {
        error: {
          code: "INVALID_CREDENTIALS",
          message: "Invalid email or password",
        },
      },
    });
  }

  // Check if account is verified
  if (!user.verified) {
    throw createError({
      statusCode: 403,
      data: {
        error: {
          code: "ACCOUNT_NOT_VERIFIED",
          message: "Please verify your email address",
          details: {
            resendVerificationUrl: "/api/auth/resend-verification",
          },
        },
      },
    });
  }

  // Generate tokens
  const config = useRuntimeConfig();

  const accessToken = sign(
    { userId: user.id, role: user.role },
    config.jwtSecret,
    { expiresIn: "15m" }
  );

  const refreshToken = sign(
    { userId: user.id, type: "refresh" },
    config.jwtSecret,
    { expiresIn: validatedData.remember ? "30d" : "7d" }
  );

  const expiresIn = 15 * 60; // 15 minutes in seconds
  const expiresAt = Date.now() + expiresIn * 1000;

  // Reset failed login attempts
  await db.user.update({
    where: { id: user.id },
    data: {
      failedLoginAttempts: 0,
      lastLoginAt: new Date(),
    },
  });

  // Return response
  return {
    tokens: {
      accessToken,
      refreshToken,
      expiresAt,
      expiresIn,
    },
    user: {
      id: user.id,
      email: user.email,
      firstName: user.firstName,
      lastName: user.lastName,
      avatar: user.avatar,
      role: user.role,
      verified: user.verified,
      preferences: {
        theme: user.preferences.theme,
        language: user.preferences.language,
        timezone: user.preferences.timezone,
      },
    },
  };
});
```

---

### Example: Create Task Endpoint

```typescript
// server/api/tasks.post.ts
import { z } from 'zod';

const createTaskSchema = z.object({
  title: z.string().min(1).max(200),
  description: z.string().max(10000).optional().nullable(),
  projectId: z.string().uuid(),
  statusId: z.string().uuid().optional(),
  priority: z.enum(['low', 'medium', 'high', 'critical']).optional().default('medium'),
  assigneeIds: z.array(z.string().uuid()).optional().default([]),
  labelIds: z.array(z.string().uuid()).optional().default([]),
  dueDate: z.string().datetime().optional().nullable(),
  estimatedHours: z.number().positive().optional().nullable(),
  parentTaskId: z.string().uuid().optional().nullable()
});

export default defineEventHandler(async (event) => {
  // Authenticate
  const user = await requireAuth(event);

  // Validate request
  const body = await readBody(event);
  const data = createTaskSchema.parse(body);

  // Check project access
  const project = await db.project.findUnique({
    where: { id: data.projectId },
    include: { members: true }
  });

  if (!project) {
    throw createError({
      statusCode: 404,
      data: {
        error: {
          code: 'RESOURCE_NOT_FOUND',
          message: 'Project not found'
        }
      }
    });
  }

  const hasAccess = project.members.some(m => m.userId === user.id);
  if (!hasAccess) {
    throw createError({
      statusCode: 403,
      data: {
        error: {
          code: 'INSUFFICIENT_PERMISSIONS',
          message: 'You don't have access to this project'
        }
      }
    });
  }

  // Use default status if not provided
  const statusId = data.statusId || project.statuses[0].id;

  // Create task
  const task = await db.task.create({
    data: {
      title: data.title,
      description: data.description,
      projectId: data.projectId,
      statusId,
      priority: data.priority,
      dueDate: data.dueDate,
      estimatedHours: data.estimatedHours,
      parentTaskId: data.parentTaskId,
      createdById: user.id,
      assignees: {
        connect: data.assigneeIds.map(id => ({ id }))
      },
      labels: {
        connect: data.labelIds.map(id => ({ id }))
      }
    },
    include: {
      project: { select: { id: true, name: true, color: true } },
      status: true,
      assignees: { select: { id: true, firstName: true, lastName: true, avatar: true } },
      labels: true,
      createdBy: { select: { id: true, firstName: true, lastName: true } }
    }
  });

  // Send notifications to assignees
  for (const assigneeId of data.assigneeIds) {
    await createNotification({
      userId: assigneeId,
      type: 'assignment',
      title: 'New task assigned',
      message: `${user.firstName} assigned you to "${task.title}"`,
      taskId: task.id,
      projectId: task.projectId
    });
  }

  // Return created task
  setResponseStatus(event, 201);
  return transformTaskResponse(task);
});

// Helper to transform database task to API response
function transformTaskResponse(task: any): Task {
  return {
    id: task.id,
    title: task.title,
    description: task.description,
    projectId: task.projectId,
    project: {
      id: task.project.id,
      name: task.project.name,
      color: task.project.color
    },
    statusId: task.statusId,
    status: {
      id: task.status.id,
      name: task.status.name,
      color: task.status.color,
      category: task.status.category
    },
    priority: task.priority,
    assignees: task.assignees.map((a: any) => ({
      id: a.id,
      firstName: a.firstName,
      lastName: a.lastName,
      avatar: a.avatar
    })),
    labels: task.labels,
    dueDate: task.dueDate?.toISOString() || null,
    startDate: task.startDate?.toISOString() || null,
    completedAt: task.completedAt?.toISOString() || null,
    estimatedHours: task.estimatedHours,
    trackedHours: calculateTrackedHours(task.timeEntries),
    commentCount: task._count?.comments || 0,
    attachmentCount: task._count?.attachments || 0,
    subtaskCount: task._count?.subtasks || 0,
    completedSubtaskCount: task.subtasks?.filter(s => s.completedAt).length || 0,
    createdById: task.createdById,
    createdBy: {
      id: task.createdBy.id,
      firstName: task.createdBy.firstName,
      lastName: task.createdBy.lastName
    },
    createdAt: task.createdAt.toISOString(),
    updatedAt: task.updatedAt.toISOString()
  };
}
```

---

## API Security

### Authentication Middleware

```typescript
// server/middleware/auth.ts
import { verify } from "jsonwebtoken";

export default defineEventHandler(async (event) => {
  // Skip auth for public routes
  const publicRoutes = ["/api/auth/login", "/api/auth/register", "/api/health"];
  if (publicRoutes.some((route) => event.path.startsWith(route))) {
    return;
  }

  // Extract token
  const authHeader = getHeader(event, "authorization");
  if (!authHeader || !authHeader.startsWith("Bearer ")) {
    throw createError({
      statusCode: 401,
      data: {
        error: {
          code: "MISSING_TOKEN",
          message: "Authentication required",
        },
      },
    });
  }

  const token = authHeader.substring(7);

  // Verify token
  try {
    const config = useRuntimeConfig();
    const decoded = verify(token, config.jwtSecret);

    // Attach user to event context
    event.context.user = decoded;
  } catch (error) {
    if (error.name === "TokenExpiredError") {
      throw createError({
        statusCode: 401,
        data: {
          error: {
            code: "TOKEN_EXPIRED",
            message: "Access token expired. Please refresh.",
          },
        },
      });
    }

    throw createError({
      statusCode: 401,
      data: {
        error: {
          code: "TOKEN_INVALID",
          message: "Invalid access token",
        },
      },
    });
  }
});

// Helper function to require authentication
export async function requireAuth(event: H3Event): Promise<JWTPayload> {
  if (!event.context.user) {
    throw createError({
      statusCode: 401,
      data: {
        error: {
          code: "UNAUTHORIZED",
          message: "Authentication required",
        },
      },
    });
  }

  return event.context.user;
}
```

---

### CSRF Protection

```typescript
// server/middleware/csrf.ts
export default defineEventHandler(async (event) => {
  const method = event.method;

  // Only check state-changing methods
  if (!["POST", "PATCH", "PUT", "DELETE"].includes(method)) {
    return;
  }

  // Skip API routes (using token auth)
  if (event.path.startsWith("/api/")) {
    return;
  }

  // Verify CSRF token
  const csrfToken = getHeader(event, "x-csrf-token");
  const csrfCookie = getCookie(event, "csrf_token");

  if (!csrfToken || csrfToken !== csrfCookie) {
    throw createError({
      statusCode: 403,
      data: {
        error: {
          code: "INVALID_CSRF_TOKEN",
          message: "Invalid CSRF token",
        },
      },
    });
  }
});
```

---

## API Testing

### Unit Test Example

```typescript
// server/api/auth/login.post.test.ts
import { describe, it, expect, beforeEach, vi } from "vitest";
import { createEvent } from "h3";
import loginHandler from "./login.post";

describe("POST /api/auth/login", () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it("returns tokens for valid credentials", async () => {
    // Arrange
    const event = createEvent({
      method: "POST",
      body: {
        email: "test@example.com",
        password: "ValidPass123!",
      },
    });

    // Mock database
    vi.mock("~/server/utils/db", () => ({
      user: {
        findUnique: vi.fn().mockResolvedValue({
          id: "usr_001",
          email: "test@example.com",
          passwordHash: "$argon2...",
          verified: true,
        }),
      },
    }));

    // Act
    const response = await loginHandler(event);

    // Assert
    expect(response.tokens).toBeDefined();
    expect(response.tokens.accessToken).toBeTruthy();
    expect(response.user.email).toBe("test@example.com");
  });

  it("rejects invalid credentials", async () => {
    // Arrange
    const event = createEvent({
      method: "POST",
      body: {
        email: "test@example.com",
        password: "WrongPassword",
      },
    });

    // Act & Assert
    await expect(loginHandler(event)).rejects.toThrow(
      "Invalid email or password"
    );
  });
});
```

---

## Complete API Endpoint List

### Authentication

| Method | Endpoint                        | Description                 | Auth Required       |
| ------ | ------------------------------- | --------------------------- | ------------------- |
| POST   | `/api/auth/register`            | Create new account          | No                  |
| POST   | `/api/auth/login`               | Login with credentials      | No                  |
| POST   | `/api/auth/logout`              | Logout current session      | Yes                 |
| POST   | `/api/auth/refresh`             | Refresh access token        | Yes (refresh token) |
| GET    | `/api/auth/me`                  | Get current user            | Yes                 |
| POST   | `/api/auth/verify-email`        | Verify email address        | No (token in URL)   |
| POST   | `/api/auth/resend-verification` | Resend verification email   | No                  |
| POST   | `/api/auth/forgot-password`     | Request password reset      | No                  |
| POST   | `/api/auth/reset-password`      | Reset password with token   | No (token in body)  |
| POST   | `/api/auth/change-password`     | Change password (logged in) | Yes                 |

### Projects

| Method | Endpoint                            | Description            | Auth Required |
| ------ | ----------------------------------- | ---------------------- | ------------- |
| GET    | `/api/projects`                     | List projects          | Yes           |
| GET    | `/api/projects/:id`                 | Get project details    | Yes           |
| POST   | `/api/projects`                     | Create project         | Yes           |
| PATCH  | `/api/projects/:id`                 | Update project         | Yes (member)  |
| DELETE | `/api/projects/:id`                 | Delete project         | Yes (admin)   |
| GET    | `/api/projects/:id/members`         | Get project members    | Yes           |
| POST   | `/api/projects/:id/members`         | Add member to project  | Yes (admin)   |
| DELETE | `/api/projects/:id/members/:userId` | Remove member          | Yes (admin)   |
| PATCH  | `/api/projects/:id/members/:userId` | Update member role     | Yes (admin)   |
| GET    | `/api/projects/:id/stats`           | Get project statistics | Yes           |

### Tasks

| Method | Endpoint                      | Description               | Auth Required          |
| ------ | ----------------------------- | ------------------------- | ---------------------- |
| GET    | `/api/tasks`                  | List tasks (filtered)     | Yes                    |
| GET    | `/api/tasks/:id`              | Get task details          | Yes                    |
| POST   | `/api/tasks`                  | Create task               | Yes                    |
| PATCH  | `/api/tasks/:id`              | Update task               | Yes (assignee/creator) |
| DELETE | `/api/tasks/:id`              | Delete task               | Yes (creator/admin)    |
| POST   | `/api/tasks/:id/duplicate`    | Duplicate task            | Yes                    |
| POST   | `/api/tasks/:id/move`         | Move to different project | Yes                    |
| GET    | `/api/tasks/:id/subtasks`     | Get subtasks              | Yes                    |
| POST   | `/api/tasks/:id/subtasks`     | Create subtask            | Yes                    |
| GET    | `/api/tasks/:id/dependencies` | Get task dependencies     | Yes                    |
| POST   | `/api/tasks/:id/dependencies` | Add dependency            | Yes                    |

### Comments

| Method | Endpoint                      | Description    | Auth Required      |
| ------ | ----------------------------- | -------------- | ------------------ |
| GET    | `/api/tasks/:taskId/comments` | List comments  | Yes                |
| POST   | `/api/tasks/:taskId/comments` | Create comment | Yes                |
| PATCH  | `/api/comments/:id`           | Update comment | Yes (author)       |
| DELETE | `/api/comments/:id`           | Delete comment | Yes (author/admin) |

### Notifications

| Method | Endpoint                          | Description         | Auth Required |
| ------ | --------------------------------- | ------------------- | ------------- |
| GET    | `/api/notifications`              | List notifications  | Yes           |
| GET    | `/api/notifications/unread-count` | Get unread count    | Yes           |
| POST   | `/api/notifications/:id/read`     | Mark as read        | Yes           |
| POST   | `/api/notifications/read-all`     | Mark all as read    | Yes           |
| DELETE | `/api/notifications/:id`          | Delete notification | Yes           |

### Users & Team

| Method | Endpoint               | Description              | Auth Required |
| ------ | ---------------------- | ------------------------ | ------------- |
| GET    | `/api/users`           | List workspace users     | Yes           |
| GET    | `/api/users/:id`       | Get user profile         | Yes           |
| PATCH  | `/api/users/:id`       | Update user profile      | Yes (self)    |
| POST   | `/api/users/invite`    | Invite user to workspace | Yes (admin)   |
| GET    | `/api/users/me/tasks`  | Get my tasks             | Yes           |
| GET    | `/api/users/:id/stats` | Get user statistics      | Yes           |

### File Management

| Method | Endpoint                        | Description             | Auth Required        |
| ------ | ------------------------------- | ----------------------- | -------------------- |
| POST   | `/api/upload`                   | Upload file             | Yes                  |
| GET    | `/api/attachments/:id`          | Get attachment metadata | Yes                  |
| GET    | `/api/attachments/:id/download` | Download attachment     | Yes                  |
| DELETE | `/api/attachments/:id`          | Delete attachment       | Yes (uploader/admin) |

### Reports

| Method | Endpoint                     | Description             | Auth Required |
| ------ | ---------------------------- | ----------------------- | ------------- |
| GET    | `/api/reports/dashboard`     | Dashboard statistics    | Yes           |
| GET    | `/api/reports/project/:id`   | Project analytics       | Yes (member)  |
| GET    | `/api/reports/team`          | Team performance        | Yes (manager) |
| GET    | `/api/reports/time-tracking` | Time tracking report    | Yes           |
| POST   | `/api/reports/export`        | Export report (PDF/CSV) | Yes           |

### Settings

| Method | Endpoint                    | Description               | Auth Required |
| ------ | --------------------------- | ------------------------- | ------------- |
| GET    | `/api/settings/workspace`   | Get workspace settings    | Yes           |
| PATCH  | `/api/settings/workspace`   | Update workspace settings | Yes (admin)   |
| GET    | `/api/settings/preferences` | Get user preferences      | Yes           |
| PATCH  | `/api/settings/preferences` | Update user preferences   | Yes           |

### System

| Method | Endpoint       | Description      | Auth Required |
| ------ | -------------- | ---------------- | ------------- |
| GET    | `/api/health`  | Health check     | No            |
| GET    | `/api/version` | API version info | No            |

---

## API Response Caching

### Cache Headers

**Static/Rarely Changing Data**:

```http
Cache-Control: public, max-age=3600, stale-while-revalidate=86400
```

**User-Specific Data**:

```http
Cache-Control: private, max-age=60
```

**Frequently Changing Data**:

```http
Cache-Control: private, no-cache, must-revalidate
```

### ETag Support

```http
GET /api/projects/prj_001
If-None-Match: "abc123hash"

# If resource unchanged:
HTTP/1.1 304 Not Modified
ETag: "abc123hash"

# If resource changed:
HTTP/1.1 200 OK
ETag: "xyz789hash"
Content-Type: application/json
{...}
```

---

## API Monitoring & Logging

### Request Logging Format

```json
{
  "timestamp": "2024-12-16T12:00:00Z",
  "requestId": "req_abc123",
  "userId": "usr_7f8a9b6c5d4e3f2a",
  "method": "POST",
  "path": "/api/tasks",
  "statusCode": 201,
  "duration": 145,
  "ip": "192.168.1.100",
  "userAgent": "Mozilla/5.0..."
}
```

### Error Logging Format

```json
{
  "timestamp": "2024-12-16T12:00:00Z",
  "requestId": "req_abc123",
  "userId": "usr_7f8a9b6c5d4e3f2a",
  "method": "POST",
  "path": "/api/tasks",
  "statusCode": 400,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "stack": "..."
  },
  "duration": 45
}
```

---

**Next Document**: [Data Models & Schema](./08-data-models-schema.md) →
