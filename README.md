# Task Management System (Backend)

A production-ready, multi-tenant Task Management System backend built with **Node.js**, **TypeScript**, **Express**, and **Prisma** (PostgreSQL).

---

## 🏛 Database Architecture (ERD)

This project uses PostgreSQL as its primary database. Below is the Entity-Relationship Diagram (ERD) mapping out the database schema, including Users, Projects, Memberships, Tasks, Comments, and Attachments.

```mermaid
erDiagram
    User {
        string id PK
        string email UK
        string passwordHash
        string firstName
        string lastName
        datetime deletedAt
        datetime createdAt
        datetime updatedAt
    }

    Project {
        string id PK
        string name
        string slug UK
        string description
        string status
        string ownerId FK
        datetime deletedAt
        datetime createdAt
        datetime updatedAt
    }

    ProjectMember {
        string id PK
        string projectId FK
        string userId FK
        string role
        datetime createdAt
        datetime updatedAt
    }

    Task {
        string id PK
        string title
        string description
        string status
        string priority
        datetime dueDate
        string projectId FK
        string assigneeId FK
        string creatorId FK
        datetime deletedAt
        datetime createdAt
        datetime updatedAt
    }

    Comment {
        string id PK
        string content
        string taskId FK
        string userId FK
        datetime createdAt
        datetime updatedAt
    }

    Attachment {
        string id PK
        string fileName
        string fileUrl
        int fileSize
        string mimeType
        string taskId FK
        datetime createdAt
        datetime updatedAt
    }

    %% Relationships
    User ||--o{ Project : "owns"
    User ||--|{ ProjectMember : "joins"
    Project ||--|{ ProjectMember : "has"
    Project ||--o{ Task : "contains"
    
    User ||--o{ Task : "creates"
    User |o--o{ Task : "assigned_to"
    
    Task ||--o{ Comment : "has"
    User ||--o{ Comment : "writes"
    
    Task ||--o{ Attachment : "has"
```

---

## 🛠 Tech Stack

- **Runtime**: [Node.js](https://nodejs.org/)
- **Language**: [TypeScript](https://www.typescriptlang.org/)
- **Framework**: [Express.js](https://expressjs.com/)
- **ORM**: [Prisma](https://www.prisma.io/)
- **Database**: [PostgreSQL](https://www.postgresql.org/)
- **Authentication**: JWT (JSON Web Tokens) with stateless validation

---

## 🚀 Getting Started

### Prerequisites
- Node.js (v18+ recommended)
- npm or yarn
- PostgreSQL database instance

### Installation
1. Clone the repository:
   ```bash
   git clone https://github.com/Amresh-01/Task-Management-system.git
   cd Task_management_backend
   ```
2. Install dependencies:
   ```bash
   npm install
   ```

3. Configure environment variables. Create a `.env` file in the root directory:
   ```env
   DATABASE_URL="postgresql://username:password@localhost:5432/task_management_db?schema=public"
   JWT_SECRET="your_jwt_secret_key"
   PORT=5000
   ```

4. Run database migrations:
   ```bash
   npx prisma migrate dev
   ```

5. Start the development server:
   ```bash
   npm run dev
   ```

---

## 📁 Directory Structure

```text
├── prisma/             # Prisma schema and database migrations
├── src/
│   ├── controllers/    # Route handler controllers
│   ├── middlewares/    # Auth, Validation, & Error handling middleware
│   ├── models/         # TypeScript interfaces & types
│   ├── routes/         # API Endpoint routing definitions
│   ├── services/       # Business logic layer
│   └── app.ts          # Express app configuration & server entry point
├── package.json
└── tsconfig.json
```
