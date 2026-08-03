# REST API Specifications

This document outlines the API contracts and endpoints designed for the Task Management System (TMS) backend. All request/response payloads follow standard RESTful patterns.

---

## 🔑 Global Response Envelope

All API endpoints return a standardized JSON envelope to maintain consistency:

### Success Response
```json
{
  "status": "success",
  "data": { ... }
}
```

### Validation Failure / Client Error (4xx)
```json
{
  "status": "fail",
  "data": {
    "errors": [
      {
        "field": "email",
        "message": "Invalid email address format"
      }
    ]
  }
}
```

### Server Error (500)
```json
{
  "status": "error",
  "message": "Internal server error. Please try again later."
}
```

---

## 👤 Authentication Endpoints

### 1. Register User
- **Method & Path**: `POST /api/v1/auth/register`
- **Auth Required**: None (Public)
- **Request Body**:
  ```json
  {
    "email": "user@example.com",
    "password": "SecurePassword123",
    "firstName": "John",
    "lastName": "Doe"
  }
  ```
- **Responses**:
  - `201 Created`: User successfully registered.
    ```json
    {
      "status": "success",
      "data": {
        "user": {
          "id": "u-uuid-1234",
          "email": "user@example.com",
          "firstName": "John",
          "lastName": "Doe",
          "createdAt": "2026-08-03T12:00:00Z"
        }
      }
    }
    ```
  - `400 Bad Request`: Email already in use or validation errors.

### 2. Login User
- **Method & Path**: `POST /api/v1/auth/login`
- **Auth Required**: None (Public)
- **Request Body**:
  ```json
  {
    "email": "user@example.com",
    "password": "SecurePassword123"
  }
  ```
- **Responses**:
  - `200 OK`: Successful login. Returns a stateless JWT access token.
    ```json
    {
      "status": "success",
      "data": {
        "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
        "user": {
          "id": "u-uuid-1234",
          "email": "user@example.com"
        }
      }
    }
    ```
  - `401 Unauthorized`: Invalid credentials.

---

## 📁 Project & Membership Endpoints

### 1. Create Project
- **Method & Path**: `POST /api/v1/projects`
- **Auth Required**: Yes (JWT Bearer)
- **Request Body**:
  ```json
  {
    "name": "Frontend Redesign",
    "slug": "frontend-redesign",
    "description": "Revamping client-facing screens"
  }
  ```
- **Responses**:
  - `201 Created`: Project created. Creator automatically gains `ADMIN` membership role.
    ```json
    {
      "status": "success",
      "data": {
        "project": {
          "id": "p-uuid-5678",
          "name": "Frontend Redesign",
          "slug": "frontend-redesign",
          "description": "Revamping client-facing screens",
          "status": "ACTIVE",
          "ownerId": "u-uuid-1234",
          "createdAt": "2026-08-03T12:05:00Z"
        }
      }
    }
    ```
  - `409 Conflict`: Slug already taken.

### 2. List Projects
- **Method & Path**: `GET /api/v1/projects`
- **Auth Required**: Yes (JWT Bearer)
- **Description**: Returns all projects the authenticated user owns or is a member of.
- **Responses**:
  - `200 OK`:
    ```json
    {
      "status": "success",
      "data": {
        "projects": [
          {
            "id": "p-uuid-5678",
            "name": "Frontend Redesign",
            "role": "ADMIN"
          }
        ]
      }
    }
    ```

### 3. Add Project Member
- **Method & Path**: `POST /api/v1/projects/:projectId/members`
- **Auth Required**: Yes (JWT Bearer - must have `ADMIN` role in project)
- **Request Body**:
  ```json
  {
    "userId": "u-uuid-9999",
    "role": "MEMBER" // ADMIN | MEMBER | VIEWER
  }
  ```
- **Responses**:
  - `201 Created`: Member added.
  - `403 Forbidden`: Authenticated user is not an Admin.

---

## 📋 Task Management Endpoints

### 1. Create Task
- **Method & Path**: `POST /api/v1/projects/:projectId/tasks`
- **Auth Required**: Yes (JWT Bearer - must be `ADMIN` or `MEMBER`)
- **Request Body**:
  ```json
  {
    "title": "Design authentication wireframes",
    "description": "Create layouts for login, signup, and reset passwords",
    "priority": "HIGH", // LOW | MEDIUM | HIGH | URGENT
    "dueDate": "2026-08-10T18:00:00Z",
    "assigneeId": "u-uuid-9999"
  }
  ```
- **Responses**:
  - `201 Created`:
    ```json
    {
      "status": "success",
      "data": {
        "task": {
          "id": "t-uuid-1111",
          "title": "Design authentication wireframes",
          "status": "TODO",
          "priority": "HIGH",
          "projectId": "p-uuid-5678",
          "creatorId": "u-uuid-1234",
          "assigneeId": "u-uuid-9999",
          "createdAt": "2026-08-03T12:10:00Z"
        }
      }
    }
    ```

### 2. List Tasks (with filtering)
- **Method & Path**: `GET /api/v1/projects/:projectId/tasks`
- **Auth Required**: Yes (JWT Bearer - must be workspace member)
- **Query Params**: `?status=IN_PROGRESS&priority=HIGH`
- **Responses**:
  - `200 OK`: Returns matching tasks list.

### 3. Update Task
- **Method & Path**: `PUT /api/v1/tasks/:taskId`
- **Auth Required**: Yes (JWT Bearer)
- **Request Body**: (Allows partial updates)
  ```json
  {
    "status": "IN_PROGRESS",
    "assigneeId": "u-uuid-1234"
  }
  ```
- **Responses**:
  - `200 OK`: Returns updated task object.

### 4. Delete Task (Soft Delete)
- **Method & Path**: `DELETE /api/v1/tasks/:taskId`
- **Auth Required**: Yes (JWT Bearer)
- **Responses**:
  - `200 OK`:
    ```json
    {
      "status": "success",
      "data": {
        "message": "Task successfully soft-deleted"
      }
    }
    ```
