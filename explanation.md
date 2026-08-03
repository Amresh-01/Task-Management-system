# Architecture & Database Design Decisions

This document provides the technical reasoning, indexing choices, API design rationale, and trade-offs considered during the design of the Task Management System (TMS) backend.

---

## 1. Database Model & Relationship Rationale

We chose **PostgreSQL** as the primary relational database for this application. A Task Management System is highly relational with strong constraints, making it a perfect fit for an ACID-compliant Relational Database Management System (RDBMS).

### Entities and Cardinality

1. **User (`User`)**
   - Represents registered accounts.
   - **Relationships**:
     - `1 : N` with `Project` (as owner).
     - `1 : N` with `ProjectMember` (membership details across boards).
     - `1 : N` with `Task` (as assignee).
     - `1 : N` with `Task` (as creator).
     - `1 : N` with `Comment` (as author).

2. **Project (`Project`)**
   - Represents workspaces/boards.
   - **Relationships**:
     - `N : 1` with `User` (owned by one creator/admin).
     - `1 : N` with `ProjectMember` (list of members invited to the board).
     - `1 : N` with `Task` (tasks belonging to this project).

3. **ProjectMember (`ProjectMember`)**
   - A junction table/join model representing the membership of a User in a Project, enriched with a **Role** enum (`ADMIN`, `MEMBER`, `VIEWER`).
   - Resolves the `M : N` relationship between `User` and `Project` cleanly while supporting Role-Based Access Control (RBAC).
   - **Cardinality**: `N : 1` with `Project`, `N : 1` with `User`.
   - **Constraint**: `@@unique([projectId, userId])` prevents duplicate memberships.

4. **Task (`Task`)**
   - Represents items inside a project board.
   - **Relationships**:
     - `N : 1` with `Project` (cascade deleted when a project is deleted).
     - `N : 1` with `User` (assignee, optional, nullable: set to null on assignee deletion).
     - `N : 1` with `User` (creator, required, cascades on user deletion).
     - `1 : N` with `Comment` (task discussion thread).
     - `1 : N` with `Attachment` (files uploaded to the task).

5. **Comment (`Comment`) & Attachment (`Attachment`)**
   - Relational extensions representing task discussions and uploads.
   - Both cascade delete with their parent `Task`.

---

## 2. Indexing Choices

Database indexes are critical to ensure queries remain performant as the data scales. We implemented the following indexing strategies:

- **`User.email`**: Index created because email is the primary identifier for authentication lookup (`SELECT * FROM User WHERE email = ?`).
- **`Project.ownerId`**: Index created to optimize listing all projects owned by a specific user.
- **`ProjectMember.projectId` and `ProjectMember.userId`**: Optimized to fast-track authorization checks, e.g. checking whether a specific user is authorized to edit a project board.
- **`Task.projectId`**: The most common lookup for tasks will be loading the board (`SELECT * FROM Task WHERE projectId = ?`).
- **`Task.assigneeId`**: Index created to optimize fetching tasks assigned to a specific user (e.g. for a user's personal dashboard "My Tasks").
- **`Task.status` and `Task.priority`**: Multi-column index configurations or separate indexes are added here to optimize filter queries on boards (e.g. filtering tasks by `status=IN_PROGRESS` or sorting by priority).

---

## 3. Technology Trade-offs

### PostgreSQL vs. MongoDB
* **Why PostgreSQL?** A task management application relies heavily on relational integrity. A task must belong to a project, an assignee must be a valid user, and permissions depend on a user's membership role in a project. In MongoDB, references are usually managed in application code, risking data drift or requiring complex transactions. PostgreSQL handles this at the storage engine level with foreign key constraints.
* **Prisma MongoDB note**: Although the assessment sheet mentioned Prisma's MongoDB connector in a single bullet point, the reference context, deliverables list, and the relational nature of the problem strongly favor PostgreSQL. We chose PostgreSQL for relational integrity.

### Soft Delete vs. Hard Delete
* **Decision**: We implemented **Soft Delete** (using a nullable `deletedAt` timestamp) for the core entities: `User`, `Project`, and `Task`. This prevents accidental data loss and allows recovery or auditing.
* **Cascade Delete**: For child entities like `ProjectMember`, `Comment`, and `Attachment`, we use **Hard Delete** (`onDelete: Cascade`) triggered by the parent's deletion. This ensures that when a parent resource is removed, orphaned dependent records are cleaned up automatically from the database.

### Authentication & Session Management
* **Decision**: JWT-based stateless authentication.
* **Trade-off**: While stateless JWTs are highly scalable, they cannot be easily revoked before expiration. In a production system, we would pair JWTs with a Redis-backed blacklist or use a sliding-window session token strategy.

---

## 4. API Design Rationale

- **RESTful Resource Hierarchy**: The endpoints follow standard REST structures, e.g. `/api/v1/projects/:projectId/tasks` for sub-resources.
- **Stateless Verification**: Every request requires a valid JWT in the `Authorization` header (`Bearer <token>`), preventing server-side session overhead.
- **Standardized Response Envelope**: All API endpoints return a unified response shape:
  ```json
  {
    "status": "success" | "fail" | "error",
    "data": { ... },       // Present on success/fail
    "message": "...",     // Present on error
    "errors": [ ... ]      // Validation detail arrays
  }
  ```
