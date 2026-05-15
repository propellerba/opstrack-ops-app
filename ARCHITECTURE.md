# OpsTrack Architectural Guide

## 1. Architecture Overview
OpsTrack is built on a **Pragmatic Layered Monolith** architecture. The system follows a classic client-server model designed to manage complex request lifecycles (Submitted -> Pending -> Approved/Rejected).

- **Single Source of Truth**: The backend manages the "Request State Machine," ensuring status transitions are validated against user roles and record states.
- **Audit-First Design**: Every mutation triggers an entry in the `audit_logs` table to ensure compliance for finance and HR workflows.
- **Separation of Concerns**: 
    - **Frontend**: Feature-based React application focusing on state synchronization via TanStack Query.
    - **Backend**: Layered logic (Controller -> Service -> Repository) to isolate transport, business logic, and data access.
    - **Database**: Strictly normalized PostgreSQL schema with enforced referential integrity.

## 2. Tech Stack Details
- **Frontend**: React 18+, TypeScript, Vite, TanStack Query v5 (Server State), Zustand (Client State), React Router v6.
- **Backend**: Node.js, Express, TypeScript, JWT (Stateless Auth).
- **Database**: PostgreSQL 15+, Knex.js or Prisma (Query Building/Migration).
- **Security**: `bcryptjs` (Hashing), `helmet` (Headers), `Zod` (Schema Validation).
- **Testing**: Vitest + React Testing Library (Frontend), Supertest + Jest/Vitest (Backend).

## 3. Folder Structure

### 3.1 Backend (Node.js/Express)
```text
src/
├── config/             # Environment variables and DB configuration
├── controllers/        # Route handlers (parsing request/returning response)
├── services/           # Core business logic (approvals, state transitions)
├── repositories/       # Data access layer (SQL queries/ORM abstraction)
├── middleware/         # Auth (JWT), RBAC, and validation wrappers
├── models/             # TypeScript interfaces and DB schemas
├── routes/             # Express router definitions (/api/v1/...)
├── utils/              # Shared helpers (date formatting, JWT signing)
└── app.ts              # Entry point
```

### 3.2 Frontend (React)
```text
src/
├── components/         # Shared UI (Button, Input) and Layout components
├── features/           # Feature-based modules
│   ├── requests/       # Logic for request management
│   │   ├── components/ # Feature-specific UI
│   │   ├── hooks/      # useRequests, useApproveRequest
│   │   ├── services/   # API calling logic
│   │   └── types/      # Domain types
│   └── auth/           # Login, Session management
├── hooks/              # Shared custom hooks
├── lib/                # API client setup (Axios interceptors)
├── stores/             # Global client state (Zustand)
└── types/              # Global TypeScript definitions
```

## 4. Design Patterns

### 4.1 Request State Machine (Backend)
All status updates must pass through a centralized service logic that checks:
1. Is the transition valid? (e.g., cannot move from `Rejected` to `Approved`).
2. Does the `user.role` have permission for this specific transition?
3. Atomicity: The status update and the `audit_log` entry must occur within a single database transaction.

### 4.2 Server State Management (Frontend)
Use **TanStack Query** for all server interactions. 
- Global loading/error states are handled via query hooks.
- Mutations (POST/PATCH) must invalidate relevant query keys to ensure the UI reflects the latest state from the PostgreSQL source of truth.

### 4.3 Repository Pattern
The `repositories/` layer abstracts SQL complexity. Controllers and Services never write raw SQL; they interact with repository methods (e.g., `RequestRepository.findByIdWithAudit(id)`).

## 5. Coding Standards

### 5.1 Naming Conventions
- **Files**: PascalCase for React components (`RequestList.tsx`), camelCase for everything else.
- **JSON Keys**: `camelCase` for all API request/response bodies.
- **Database**: `snake_case` for table names (plural) and column names.
- **Variables**: TypeScript interfaces use PascalCase; instances use camelCase.

### 5.2 TypeScript Usage
- **Strict Mode**: Mandatory. No `any` types.
- **Shared Types**: Domain models (e.g., `IRequest`) should be shared or mirrored between frontend and backend to ensure contract safety.

### 5.3 Database Standards
- **Primary Keys**: `BIGSERIAL` or `UUID`.
- **Timestamps**: `created_at` and `updated_at` (TIMESTAMPTZ) are required on every table.
- **Soft Deletes**: Use `deleted_at` instead of hard deletes for primary business entities.

## 6. API Design

### 6.1 RESTful Standards
- **Versioning**: All routes prefixed with `/api/v1`.
- **Endpoints**: Use nouns in kebab-case (e.g., `GET /api/v1/request-logs`).
- **Standard Responses**:
    - `200/201`: Success/Created.
    - `400`: Validation failure (Zod error).
    - `401/403`: Auth/Permission failure.
    - `422`: Business logic violation (State machine error).

### 6.2 Authentication & Authorization
- **JWT**: Tokens stored in HttpOnly cookies or Authorization headers.
- **RBAC Middleware**: 
  ```typescript
  router.patch('/:id/approve', authorize(['Manager', 'Admin']), RequestController.approve);
  ```

## 7. Testing Strategy

### 7.1 Priority Levels
1. **Auth Flow**: Login, Token expiration, and Role-Based Access Control.
2. **State Transitions**: Testing the "Happy Path" and "Edge Cases" of the request lifecycle.
3. **Data Integrity**: Ensuring audit logs are written for every mutation.

### 7.2 Tools
- **Unit Tests**: Focus on `services/` (Backend) and `hooks/` (Frontend).
- **Integration Tests**: Use `supertest` to hit Express endpoints and verify PostgreSQL state changes.
- **Frontend Testing**: React Testing Library to simulate user interactions on forms and approval buttons.

## 8. Database Schema & Audit
Audit tables must follow the `audit_{table_name}` pattern.
```sql
CREATE TABLE audit_requests (
    id BIGSERIAL PRIMARY KEY,
    entity_id BIGINT NOT NULL, -- ID of the request
    user_id BIGINT NOT NULL,   -- Who performed the action
    action VARCHAR(50) NOT NULL, -- 'STATUS_CHANGE', 'UPDATE', etc.
    previous_state JSONB,
    new_state JSONB,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```