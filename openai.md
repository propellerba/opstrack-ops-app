# OpenAI Profile

## Purpose
- Provide model-specific operating rules for implementation work in this repository.
- Keep all generated code aligned with the approved architecture and stack.

## Project Context
- Project: OpsTrack
- Tech Stack: React, TypeScript, Node.js, Express, PostgreSQL, JWT
- Architecture Summary: A classic client-server architecture featuring a React frontend and a Node.js/Express REST API. The system uses a PostgreSQL relational database to manage request lifecycles and audit logs, secured by JWT-based authentication for role-based access control.
- Brief Snapshot: Project Name: OpsTrack

Build a lightweight operations web app for small companies to manage internal requests and approvals.

Core features:

Submit and track internal requests (IT, finance, HR)
Multi-step approval workflow with status updates
Comments and activity history on each request
Due dates, priorities, and assignees
Responsive dashboard for desktop and mobile
Tech stack:

Frontend: React + TypeScript
Backend: Node.js + Express
Database: PostgreSQL
Auth: JWT
Goal:
Deliver an MVP that supports the full request lifecycle from submission to approval/rejection with clear audit history.

## Repo Rules
- Use monorepo layout with `frontend/` and `backend/`.
- Mobile segment is in scope: include `mobile/` rules and deploy checks.
- Keep shared contracts explicit; do not break segment boundaries.
- Follow architecture decisions from `ARCHITECTURE.md` before introducing new patterns.

## Segment Rules
- Frontend work stays under `frontend/` with typed service calls and route-aware modules.
- Backend work stays under `backend/` with modular route/controller/service layers.
- Deployment logic is segment-based and triggered via `.github/workflows/deploy-segments.yml`.

## Do
- Ship small, reviewable patches tied to one clear behavior change.
- Update tests and run validation for touched segments.
- Call out assumptions and edge cases before high-impact changes.

## Don't
- Do not mix unrelated frontend/backend changes in a single patch unless required.
- Do not replace architecture conventions without explicit rationale.
- Do not leave deployment or environment assumptions undocumented.

## PR Checklist
- Scope limited to intended segment(s): frontend/backend/mobile.
- Tests updated or validation plan documented.
- Config/workflow impact reviewed (`.github/workflows/deploy-segments.yml`).
- Documentation/comments updated where behavior changed.

## Prompting Style
- Be concise and execution-focused; optimize for small, testable patches.
- Prioritize fast verification steps after each change.
- End with explicit next actions and any residual risks to verify.
