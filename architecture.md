# System Architecture & Request Flow

This document details the high-level architecture, directory structure, and request-flow walkthrough for the Task Management System (TMS) backend.


---

## 1. High-Level Architecture

The system follows a layered **MVC/Three-Tier Architecture** pattern consisting of:
1. **Client / API Interface Layer**: The client application (web/mobile) sending HTTP requests.
2. **Routing & Middleware Layer**: Express routers direct traffic and intercept requests for cross-cutting concerns (e.g., authentication, request validation, rate limiting).
3. **Controller Layer**: Handles HTTP-specific request parsing, input sanitization, orchestrating calls to the business logic layer, and returning the proper HTTP response codes/shapes.
4. **Service (Business Logic) Layer**: The core of the application where all business rules, authorization checks, and logic live. Completely decoupled from HTTP frameworks.
5. **Repository (Data Access) Layer**: Interacts directly with the database using Prisma Client. Enforces type-safety and abstracts database-specific operations.
6. **Database Layer**: PostgreSQL instance storing normalized relational data.
