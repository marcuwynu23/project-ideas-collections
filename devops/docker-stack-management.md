# Docker Stack Management Platform

Build a multi-tenant platform that manages Docker Swarm clusters and Docker stacks using a control-plane and a customer-side agent. The product should support desired state deployments, release history, rollbacks, live events, and auditing.

## Project Goal

Create a production-style DevOps platform where teams can deploy and operate Docker stacks across environments from a web UI and API, without exposing Docker sockets publicly.

## Core Outcomes

- Manage Swarm clusters, stacks, environments, releases, and deployments.
- Support immutable releases and rollback to previous releases.
- Provide service scaling and runtime status visibility.
- Handle secrets and environment overlays per environment.
- Enforce multi-tenant isolation with RBAC and scoped API tokens.
- Stream deployment events to the UI in real time.

## Scope

### In Scope (MVP)

- SaaS control-plane API + background worker + web frontend.
- Agent that connects outbound to control-plane and executes commands.
- Async deployment workflow with event logs and status transitions.
- YAML diff between rendered stack states.
- Local development setup with Docker Compose.

### Out of Scope (Initial Version)

- Kubernetes support.
- Advanced autoscaling policies.
- Full persistent volume rollback guarantees.

## Recommended Tech Stack

### Backend (Control-Plane)

- Go 1.22+
- Gin or Chi for HTTP
- Postgres with migrations and typed DB access
- Redis + worker queue (for async deployment jobs)
- JWT sessions and hashed API tokens
- OpenAPI spec generated and validated in tests

### Agent

- Go 1.22+
- Connects to local Docker engine
- Outbound gRPC stream or WebSocket to control-plane

### Frontend

- React + TypeScript + Vite
- Minimal UI framework (focus on function, not polish)
- SSE or WebSocket for live deployment updates

## System Design

### 1) Control-Plane API

- Handles auth, orgs, users, clusters, stacks, releases, deployments, and audit.
- Stores desired state and deployment history.
- Publishes deployment jobs to worker.

### 2) Deployment Worker

- Picks queued deployment jobs.
- Sends deploy/rollback/scale commands to agent.
- Persists deployment events and status transitions.

### 3) Customer-Side Agent

- Registers with one-time token.
- Sends heartbeat and inventory updates.
- Executes stack deploy/rollback/scale commands.
- Streams logs/events back to control-plane.

### 4) Web UI

- Login and org context.
- Cluster, stack, environment, and deployment pages.
- Release diff view and live event console.
- Token and audit management pages.

## Minimum Domain Model

Implement tables for:

- Organizations, users, roles
- API tokens (prefix + hashed secret)
- Clusters and cluster nodes
- Stacks and environments
- Stack releases (immutable)
- Environment overlays (env vars, secret metadata, compose patch)
- Deployments and deployment events
- Audit events

## Required Security Rules

- Password hashing (argon2id preferred).
- JWT access + refresh flow.
- API token format `stk_<prefix>_<secret>`, store only hash.
- RBAC roles: ORG_ADMIN, CLUSTER_ADMIN, APP_ADMIN, VIEWER.
- Secrets encrypted at rest with a master key.
- Secret values never returned after write.
- Strict tenant scoping on all queries and commands.

## Required API Areas

Implement and document endpoints for:

- Authentication (login/refresh/logout)
- Organizations and users
- API tokens
- Clusters and agent registration
- Stacks, environments, releases
- Deployments, rollbacks, and deployment event streaming
- Runtime status and service scaling
- Audit search

## Frontend Pages (MVP)

- Login
- Cluster list and cluster detail
- Stack list and stack detail
- Environment detail (deploy, rollback, status, live events, scale)
- Releases list with diff view
- API tokens page
- Audit events page

## Local Development Setup

Provide `docker-compose.dev.yml` with:

- Postgres
- Redis
- Control-plane API
- Worker
- Frontend
- Agent

Provide scripts:

- `make dev`
- `make migrate`
- `make test`

## AI-Ready Implementation Phases

1. Initialize repository structure and local compose environment.
2. Build schema, migrations, and DB access layer.
3. Implement auth, RBAC, and API token system.
4. Implement cluster registration plus heartbeat/inventory flow.
5. Implement stack, environment, and release CRUD.
6. Add async deployments, worker processing, and event streaming.
7. Implement rollback and service scaling.
8. Build frontend pages wired to live APIs.
9. Add tests and CI for core flows.

## Definition of Done

- System runs locally with one startup command.
- Multi-tenant isolation and RBAC enforced.
- Releases are immutable; rollback works through new deployment records.
- Deployment events stream live to UI.
- Secrets are encrypted and never exposed via read APIs.
- OpenAPI spec exists and tests pass for critical auth/deploy/security paths.
