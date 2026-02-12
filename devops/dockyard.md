You are a senior staff engineer. Build a production-grade, multi-tenant “Docker Stack Management” platform: a SaaS control-plane + customer-side agent that manages Docker Swarm clusters and Docker stacks (Compose/stack deploy) with ArgoCD-like UX: desired state, releases, deployments, status, diffs, and rollback.

GOALS
- Provide a modern, developer-friendly UI + API to manage:
  - Swarm clusters (manager + workers)
  - Stacks (docker stack deploy from Compose)
  - Releases (immutable snapshots)
  - Deployments (audit + logs + events)
  - Rollbacks (redeploy prior release)
  - Service scaling
  - Secrets and environment variables with overlays per environment
- Multi-tenant SaaS: orgs, users, roles (RBAC), API tokens with scopes.
- Secure design: agent connects outbound to SaaS; no public Docker socket exposure.
- Everything runnable locally with docker compose.

NON-GOALS (for now)
- Kubernetes support
- Full autoscaling with metrics
- Full volume/data rollback (must warn clearly; support hooks later)

TECH STACK (use exactly this unless impossible)
Backend (control-plane):
- Go 1.22+
- Gin (or Chi) for HTTP
- Postgres (via sqlc + migrations; or GORM if sqlc is too heavy, but prefer sqlc)
- Redis (for job queue) + asynq (or a simple worker with pg advisory locks)
- JWT for user sessions
- API tokens: prefix + hashed secret, stored hashed (argon2id or bcrypt)
- OpenAPI spec generated and validated in tests
Agent:
- Go 1.22+
- Connect to Docker Engine via local /var/run/docker.sock (for local dev)
- Outbound gRPC stream (preferred) OR WebSocket to control-plane
Frontend:
- React + TypeScript + Vite
- UI library: minimal (Tailwind ok) but do NOT overfocus on styling
- Live updates via SSE or WebSocket
Infrastructure:
- docker-compose.dev.yml that runs: control-plane API, worker, postgres, redis, frontend, and a local Swarm-in-Docker cluster simulation (at least one manager + one worker if feasible). If full swarm-in-docker is too complex, provide instructions and a fallback.

REPO STRUCTURE
- /cmd/controlplane
- /cmd/worker
- /cmd/agent
- /internal/... (domain, db, auth, api, events, deploy)
- /web (frontend)
- /migrations
- /openapi
- /docs (architecture, threat model, API usage)
- /scripts (dev helpers)

DOMAIN MODEL (must implement)
Entities (tables):
- orgs(id, name, created_at)
- users(id, org_id, email, password_hash, role, created_at, last_login_at)
- api_tokens(id, org_id, name, token_prefix, token_hash, scopes, expires_at, created_at, revoked_at)
- clusters(id, org_id, name, agent_id, status, last_seen_at, created_at)
- cluster_nodes(id, cluster_id, node_id, hostname, role, availability, status, labels_json, resources_json, updated_at)
- stacks(id, org_id, name, description, created_at)
- environments(id, stack_id, name, cluster_id, created_at)  // dev/staging/prod
- stack_releases(id, stack_id, version, source_type, source_ref, compose_yaml, created_at, created_by)
- stack_overlays(id, environment_id, env_json, secrets_json_metadata, compose_patch_yaml, updated_at)
- deployments(id, environment_id, release_id, status, started_at, finished_at, created_by, error_message)
- deployment_events(id, deployment_id, ts, level, message, data_json)
- audit_events(id, org_id, actor_type, actor_id, action, target_type, target_id, ts, ip, user_agent, data_json)

Important behavior:
- Every deploy creates a Deployment record and event stream.
- Every stack release is immutable (compose_yaml frozen).
- Rollback creates a new Deployment referencing an older Release.
- Stack “desired state” is: Release.compose_yaml + Overlay.compose_patch_yaml + Overlay.env_json + secret references.
- Enforce org_id scoping everywhere.

SECURITY REQUIREMENTS
- Password hashing: argon2id preferred; bcrypt acceptable.
- JWT access token + refresh token (or short-lived JWT + session table).
- API token format: "stk_<prefix>_<secret>" where secret is never stored, only hash.
- RBAC:
  - ORG_ADMIN: full access
  - CLUSTER_ADMIN: manage clusters + deployments but not org billing/users
  - APP_ADMIN: manage stacks/releases/deployments within org
  - VIEWER: read-only
- Agent auth:
  - Agent registers using a one-time cluster registration token created in UI/API by ORG_ADMIN/CLUSTER_ADMIN.
  - After registration, agent gets a long-lived credential (mTLS or signed JWT) stored locally.
  - All commands signed/authorized; include org/cluster scoping.
- Secrets:
  - Store encrypted at rest (use AES-GCM with a master key from env CONTROLPLANE_MASTER_KEY). Store only ciphertext + metadata.
  - Secret values must never be returned by read APIs after creation; allow rotate/update with audit.

AGENT PROTOCOL
Implement command types sent from control-plane to agent:
- Heartbeat (agent -> control-plane): cluster status, docker info
- Inventory (agent -> control-plane): nodes, services, stacks summary
- DeployStack (control-plane -> agent): includes rendered compose yaml + stack name + env vars + secret payload references
- RollbackStack (control-plane -> agent): same as deploy but older release
- ScaleService (control-plane -> agent): service name/id + replicas
- GetLogs (control-plane -> agent): service/task logs (tail, since)
- StreamEvents (agent -> control-plane): deployment output and docker events

Agent execution rules:
- For deploy: use docker CLI or Docker API; prefer invoking `docker stack deploy -c - <stack>` with stdin for simplicity.
- Must capture stdout/stderr and stream as deployment events.
- After deploy, poll service status: desired vs running replicas and report health.
- Implement timeout and cancellation (control-plane can cancel a deployment).

API REQUIREMENTS (must implement + OpenAPI)
Auth:
- POST /v1/auth/login
- POST /v1/auth/refresh
- POST /v1/auth/logout
Orgs/Users:
- GET/POST /v1/orgs (create requires system bootstrap admin or first-run)
- GET/POST /v1/orgs/{orgId}/users
Tokens:
- POST /v1/orgs/{orgId}/tokens
- GET /v1/orgs/{orgId}/tokens
- DELETE /v1/orgs/{orgId}/tokens/{tokenId}
Clusters:
- POST /v1/orgs/{orgId}/clusters (creates registration token)
- POST /v1/agent/register (agent uses registration token)
- GET /v1/orgs/{orgId}/clusters
- GET /v1/orgs/{orgId}/clusters/{clusterId}/nodes
Stacks:
- POST /v1/orgs/{orgId}/stacks
- GET /v1/orgs/{orgId}/stacks
- GET /v1/orgs/{orgId}/stacks/{stackId}
Environments:
- POST /v1/stacks/{stackId}/environments
- GET /v1/stacks/{stackId}/environments
Releases:
- POST /v1/stacks/{stackId}/releases  (from YAML upload or git ref stub)
- GET /v1/stacks/{stackId}/releases
Deployments:
- POST /v1/environments/{envId}/deployments  (deploy a release)
- POST /v1/environments/{envId}/rollback   (to a prior release)
- GET /v1/environments/{envId}/deployments
- GET /v1/deployments/{deploymentId}
- GET /v1/deployments/{deploymentId}/events (SSE stream)
Runtime:
- POST /v1/environments/{envId}/services/{service}/scale
- GET /v1/environments/{envId}/status (stacks/services/tasks summary)
Audit:
- GET /v1/orgs/{orgId}/audit?from&to&actor&action

FRONTEND REQUIREMENTS (minimal but complete)
Pages:
- Login
- Org switch (if multiple)
- Clusters list + cluster detail (nodes, last seen, status)
- Stacks list + stack detail
- Environment detail:
  - current desired release
  - deploy button (choose release)
  - rollback button (choose previous)
  - live deployment events console
  - services table with scale action
- Releases list + “diff” view (YAML diff; and optionally derived services diff)
- Tokens page (create/revoke; show token once)
- Audit page (read-only)

DIFF/PLAN REQUIREMENT
Implement a basic diff:
- Compare release.compose_yaml + overlay patches (rendered YAML) between two releases (or two rendered states).
- Present unified diff in UI.
No need to perfectly diff Docker service specs; YAML diff is enough for MVP.

JOB/WORKER REQUIREMENTS
Deployments should be executed asynchronously:
- POST deploy returns deployment_id immediately.
- Worker requests agent to execute and streams events into deployment_events table.
- SSE endpoint reads deployment_events and streams to UI.
- Deployment state transitions: PENDING -> RUNNING -> SUCCEEDED/FAILED/CANCELED.
- Implement idempotency keys on deploy endpoint to avoid duplicate deployments.

LOCAL DEV ENVIRONMENT
- Provide docker-compose.dev.yml:
  - postgres
  - redis
  - controlplane API
  - worker
  - frontend
  - agent (connected to a local docker engine; for dev you can mount /var/run/docker.sock)
- Provide scripts:
  - `make dev` to start everything
  - `make migrate`
  - `make test` runs all backend + agent unit tests, plus minimal frontend tests if feasible.

TEST REQUIREMENTS (mandatory, exhaustive)
Backend tests:
- Auth: login, refresh, token expiry, invalid creds
- API tokens: create, scope enforcement, revoke
- RBAC: ensure forbidden actions blocked
- Multi-tenant isolation: cross-org access denied
- Releases immutable: update should fail
- Deploy endpoint: creates deployment, idempotency works, permission checks
- SSE events: streams events in order
- Secrets: encryption at rest (ciphertext != plaintext), cannot read secret value after creation
Agent tests:
- Command handling: deploy/rollback/scale parsing
- Docker execution adapter mocked (do not require real docker for unit tests)
- Event streaming: ensures output becomes deployment events
Integration test (optional but ideal):
- Spin up postgres+redis in testcontainers and run controlplane endpoints.
Frontend tests (minimal):
- Login flow
- Deployment events page renders stream

QUALITY BARS
- No placeholders. Everything must run.
- Good error messages, proper HTTP status codes.
- Structured logging.
- Context cancellation for deployments.
- Secure defaults.

DELIVERABLES
1) Full source code
2) OpenAPI spec (checked in)
3) README with:
   - architecture overview
   - how to run locally
   - security notes (docker socket risk, secret encryption)
   - API examples (curl) including token usage
4) A minimal threat model doc in /docs/threat-model.md

IMPLEMENTATION PLAN (must follow)
- Step 1: Initialize repo, modules, compose dev environment
- Step 2: DB schema + migrations + db access layer
- Step 3: Auth + RBAC + API token system
- Step 4: Agent registration + heartbeat/inventory
- Step 5: Stack/release/env CRUD
- Step 6: Deployments worker + agent DeployStack + events storage + SSE
- Step 7: Rollback + scale endpoints
- Step 8: Frontend pages wired to API
- Step 9: Tests + CI (GitHub Actions) running `make test`

When uncertain, choose the safest general approach and document decisions in README.
Proceed to implement now. Start by generating the repository layout, migration files, and docker-compose.dev.yml, then implement backend APIs with tests.
