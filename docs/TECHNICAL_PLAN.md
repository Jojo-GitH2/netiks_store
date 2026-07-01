# Netiks Store Technical Plan

## 1. Purpose

This document is the technical implementation guide for Netiks Store. It is intended to help the core team and future interns understand how the system should be built, how services are separated, how they communicate, and how the platform should evolve from local development to AWS deployment.

This plan assumes a microservice architecture, but it is designed to stay practical. The system should be implemented as a modular monorepo so the codebase remains easy to manage while still teaching real service boundaries, network communication, deployment concerns, and operational thinking.

## 2. Technical Objectives

- Build a realistic multi-vendor e-commerce platform using microservices.
- Keep the system understandable enough for interns to learn from.
- Support local development with Docker Compose.
- Prepare the services for cloud deployment on AWS.
- Keep costs low by using simple infrastructure first and scaling complexity gradually.
- Make service boundaries explicit so DevOps, backend, and frontend work can proceed independently.

## 3. Architecture Principles

- Each service should own a clear domain responsibility.
- Each service should be independently deployable.
- Services should communicate over HTTP for simplicity in v1.
- Asynchronous communication should be introduced only where it clearly improves decoupling.
- Shared data access across services should be avoided. Each service owns its own schema or logical data boundary.
- The frontend should never access databases directly. It should call backend APIs through a gateway.
- Authentication and authorization should be centralized.
- Observability, configuration, and deployment patterns should be standardized across services.

## 4. High-Level Architecture

The system will consist of:

- A Next.js frontend application for the user interface.
- An API Gateway/BFF layer that acts as the single backend entry point for the frontend.
- Core backend microservices for authentication, users/vendors, catalog, media, and admin functions.
- Shared infrastructure services such as PostgreSQL, Redis, object storage, reverse proxy, CI/CD, and monitoring.

At a high level, the request flow is:

1. User interacts with the Next.js frontend.
2. Frontend calls the API Gateway.
3. API Gateway routes the request to the relevant backend service.
4. Service validates auth and business rules.
5. Service reads or writes its own data store or storage backend.
6. Response returns through the API Gateway to the frontend.

## 5. Service Map

## 5.1 Frontend Web App

### Service name

- `web`

### Stack

- Next.js
- TypeScript
- Tailwind CSS
- TanStack Query

### Responsibilities

- Render public marketplace pages.
- Render authentication pages.
- Render vendor dashboard and admin dashboard.
- Manage client-side navigation and UI state.
- Call the API Gateway for data.
- Optimize images and frontend performance.

### Key routes

- `/`
- `/products`
- `/products/[slug]`
- `/stores/[slug]`
- `/login`
- `/register`
- `/dashboard`
- `/dashboard/products`
- `/dashboard/store`
- `/admin`

## 5.2 API Gateway / Backend For Frontend

### Service name

- `gateway`

### Stack

- FastAPI

### Responsibilities

- Serve as the single API entry point for the frontend.
- Route requests to backend microservices.
- Aggregate responses where needed.
- Enforce auth validation for protected endpoints.
- Normalize error handling and response shapes.
- Reduce direct service complexity in the frontend.

### Why this service matters

Without a gateway, the frontend would need to know every service location and contract. The gateway creates one stable surface for the UI and allows backend services to evolve more safely.

## 5.3 Identity Service

### Service name

- `identity-service`

### Responsibilities

- User registration
- User login
- Password hashing and verification
- JWT issuance and refresh strategy
- User role assignment
- Current-user identity lookup
- Account activation/deactivation in later phases

### Owns

- User credentials
- User roles
- Authentication tokens and rules

### Core endpoints

- `POST /auth/register`
- `POST /auth/login`
- `POST /auth/refresh`
- `GET /auth/me`

## 5.4 Vendor Service

### Service name

- `vendor-service`

### Responsibilities

- Store creation
- Store profile updates
- Vendor profile details
- Store lookup by slug
- Store status management

### Owns

- Stores
- Vendor-store relationships
- Store metadata

### Core endpoints

- `POST /stores`
- `GET /stores/{slug}`
- `PATCH /stores/{id}`
- `GET /vendors/me/store`

## 5.5 Catalog Service

### Service name

- `catalog-service`

### Responsibilities

- Product creation and editing
- Product publishing and archiving
- Product listing and filtering
- Category management
- Product detail retrieval
- Store-product association

### Owns

- Products
- Product images metadata
- Categories
- Product status
- Searchable catalog metadata

### Core endpoints

- `POST /products`
- `GET /products`
- `GET /products/{slug}`
- `PATCH /products/{id}`
- `DELETE /products/{id}`
- `GET /categories`
- `POST /categories`

## 5.6 Media Service

### Service name

- `media-service`

### Responsibilities

- File upload handling
- File validation
- Media storage path generation
- Signed upload or retrieval flow in later phases
- Storage abstraction between local disk and S3

### Owns

- Upload rules
- Media object metadata
- File naming conventions
- Storage integration logic

### Core endpoints

- `POST /uploads`
- `GET /media/{id}`
- `DELETE /media/{id}`

## 5.7 Admin Service

### Service name

- `admin-service`

### Responsibilities

- Marketplace moderation
- Store and product suspension
- Admin-level reporting endpoints
- Category oversight if not kept in catalog service
- User and vendor review workflows

### Owns

- Moderation actions
- Audit events for admin activities

### Core endpoints

- `GET /admin/users`
- `GET /admin/stores`
- `GET /admin/products`
- `PATCH /admin/stores/{id}/status`
- `PATCH /admin/products/{id}/status`

## 5.8 Optional Notification Service

### Service name

- `notification-service`

### Status

- Phase 2 or later

### Responsibilities

- Send email notifications
- Handle system-triggered outbound messages
- Manage reusable notification templates

This is optional for v1 and should only be introduced when needed.

## 6. Shared Infrastructure Components

## 6.1 PostgreSQL

- Primary relational database
- Can begin as a single PostgreSQL instance with logical separation by service
- In early implementation, each service can use its own schema in the same Postgres instance
- This keeps costs low while still teaching service-level ownership

Recommended initial schemas:

- `identity`
- `vendor`
- `catalog`
- `admin`
- `media`

## 6.2 Redis

- Optional in MVP
- Useful for:
  - Rate limiting
  - Short-lived caching
  - Background job coordination later

## 6.3 Object Storage

- Development: local file storage or MinIO-compatible local setup
- Production: AWS S3

Used for:

- Product images
- Store logos
- Store banners

## 6.4 Reverse Proxy

- Nginx or Caddy

Responsibilities:

- Route traffic to frontend and gateway
- TLS termination in deployment
- Request buffering and limits for uploads
- Security headers

## 6.5 Container Orchestration For Local Development

- Docker Compose

Compose should run:

- `web`
- `gateway`
- `identity-service`
- `vendor-service`
- `catalog-service`
- `media-service`
- `admin-service`
- `postgres`
- `redis` optional

## 7. Communication Model

## 7.1 Synchronous communication

The default service communication pattern for v1 should be HTTP REST.

Examples:

- Gateway calls identity service to validate or fetch current user.
- Gateway calls vendor service to fetch store details.
- Gateway calls catalog service to list products.
- Vendor service may call media service to validate uploaded store assets.

This keeps the architecture easy to inspect and debug for interns.

## 7.2 Asynchronous communication

For later phases, asynchronous communication can be introduced for:

- Notification dispatch
- Audit event fan-out
- Search indexing
- Image processing

Recommended later tools:

- Redis queue or RabbitMQ

Do not introduce a message broker in the first build unless it becomes necessary.

## 8. Data Ownership Strategy

Each service should own its own tables and business logic.

Examples:

- Identity service owns users and credentials.
- Vendor service owns stores.
- Catalog service owns products and categories.
- Media service owns media metadata and storage rules.
- Admin service owns moderation records and audit actions.

Cross-service joins should not happen directly at the database level. If combined responses are needed, they should be composed through service APIs, usually by the gateway.

## 9. Example Request Flows

## 9.1 User registration

1. Frontend sends registration request to gateway.
2. Gateway forwards request to identity service.
3. Identity service validates input, hashes password, stores user, returns token payload.
4. Gateway returns normalized auth response to frontend.

## 9.2 Vendor creates a store

1. Logged-in user opens vendor onboarding page.
2. Frontend sends create-store request to gateway with auth token.
3. Gateway validates token with identity service.
4. Gateway forwards request to vendor service.
5. Vendor service creates store record and links it to the authenticated user.
6. Vendor service returns store data to gateway.
7. Gateway returns response to frontend.

## 9.3 Vendor uploads product image

1. Frontend sends upload request to gateway.
2. Gateway validates auth.
3. Gateway forwards file request to media service.
4. Media service validates type and size, stores file, saves metadata, returns asset reference.
5. Frontend uses returned media reference when creating or updating product.

## 9.4 Public marketplace page load

1. Frontend requests products from gateway.
2. Gateway queries catalog service for product list.
3. Catalog service returns published products and category metadata.
4. Gateway returns a frontend-friendly response shape.
5. Frontend renders product grid.

## 9.5 Product detail page

1. Frontend requests product detail from gateway.
2. Gateway fetches product from catalog service.
3. Gateway fetches related store info from vendor service if needed.
4. Gateway combines data and returns a single response.

## 10. Auth And Authorization Design

### Authentication

- Identity service issues JWT access tokens.
- Refresh token support is recommended if session longevity is needed.
- The gateway validates tokens either by:
  - Verifying JWT signature directly using a shared secret or public key, or
  - Calling identity service for token introspection

Recommended v1:

- JWT validation at gateway
- Role and user ID extracted from token

### Authorization

- Gateway performs route-level authorization checks.
- Backend services also enforce ownership and role checks.

Examples:

- Vendor can only update their own store and products.
- Admin can moderate any store or product.
- Public endpoints only expose published resources.

## 11. API Design Guidelines

- Use REST endpoints with versioning support such as `/api/v1`.
- Use consistent JSON response shapes.
- Return validation errors clearly.
- Use pagination for all list endpoints.
- Use slugs for public-facing product and store routes.
- Use UUIDs internally for entity IDs where practical.

Recommended response wrapper:

```json
{
  "data": {},
  "message": "optional",
  "meta": {}
}
```

## 12. Local Development Architecture

Local development should mirror production shape as much as possible without adding unnecessary complexity.

### Local stack

- Next.js app on one container
- Each FastAPI service in its own container
- PostgreSQL container
- Optional Redis container
- Shared Docker network
- Shared environment file patterns

### Developer workflow

1. Clone repo
2. Copy example environment files
3. Start stack with Docker Compose
4. Run database migrations
5. Seed local data
6. Open frontend and test flows

## 13. AWS Deployment Strategy

The AWS design should begin simply, then evolve.

### Recommended first deployment shape

- One EC2 instance running Docker Compose or multiple containers
- Nginx or Caddy at the edge
- S3 for media
- Optional managed PostgreSQL only if budget allows

### Why this is recommended

- Low cost
- Easier for interns to understand
- Fastest route to a working cloud deployment
- Still teaches networking, containers, environment config, reverse proxying, TLS, and storage integration

### Later AWS evolution

- Split services across ECS
- Move database to RDS
- Add CloudWatch metrics and alarms
- Add load balancer
- Add Secrets Manager or SSM Parameter Store

## 14. CI/CD Plan

CI/CD should validate and package every service independently while preserving a monorepo workflow.

### CI pipeline goals

- Lint frontend and backend services
- Run unit and integration tests
- Build service images
- Check migrations
- Validate environment configuration where possible

### Suggested GitHub Actions jobs

- `web-lint-test`
- `gateway-lint-test`
- `identity-service-test`
- `vendor-service-test`
- `catalog-service-test`
- `media-service-test`
- `admin-service-test`
- `docker-build`

### CD approach

- For initial phase, manual deployment from a protected branch is acceptable.
- Later phase can push images to a registry and deploy automatically.

## 15. Observability Plan

Observability should be present from early development, even if lightweight.

### Logging

- Structured logs in JSON where practical
- Include request IDs
- Include service name in every log entry

### Health endpoints

Each service should expose:

- `/health/live`
- `/health/ready`

### Metrics

- Request count
- Error count
- Response latency
- Database connectivity status

### Monitoring evolution

- Local logs during development
- CloudWatch or Grafana stack later

## 16. Security Plan

- Passwords hashed with bcrypt or Argon2
- Secrets stored in environment variables
- CORS restricted by environment
- Upload size and content type validation
- Role-based access control across services
- Minimal privilege for AWS credentials
- No secrets in source control
- HTTPS required in deployed environments

## 17. Testing Strategy

### Frontend

- Unit tests for components and utilities
- Integration tests for page flows where practical
- End-to-end tests for auth and vendor workflows

### Backend

- Unit tests for business logic
- API tests for each service
- Contract tests between gateway and services
- Integration tests for auth, store creation, product CRUD, and product listing

### Infrastructure

- Docker Compose smoke test
- Health endpoint validation
- Deployment checklist for AWS environments

## 18. Recommended Repository Structure

```text
netiks-store/
  apps/
    web/
    gateway/
  services/
    identity-service/
    vendor-service/
    catalog-service/
    media-service/
    admin-service/
    notification-service/      # optional later
  packages/
    shared/
  infra/
    docker/
    nginx/
    aws/
  scripts/
  docs/
    PRD.md
    PROJECT_DOCUMENTATION.md
    TECHNICAL_PLAN.md
```

## 19. Build Sequence

The system should be built in phases so interns can understand one layer at a time.

### Phase 1: Platform foundation

- Create monorepo structure
- Scaffold frontend
- Scaffold gateway
- Scaffold all core services
- Add Docker Compose
- Add base PostgreSQL and environment configuration

### Phase 2: Identity and access

- Build identity service
- Implement registration and login
- Add JWT auth in gateway
- Protect dashboard routes in frontend

### Phase 3: Vendor management

- Build vendor service
- Implement store creation and store pages
- Connect vendor dashboard to store APIs

### Phase 4: Catalog management

- Build catalog service
- Add product CRUD
- Add categories, search, filters, and public product listing

### Phase 5: Media management

- Build media service
- Add upload flows for store and product images
- Connect media references to vendor and catalog services

### Phase 6: Admin controls

- Build admin service
- Add moderation endpoints
- Build admin dashboard basics

### Phase 7: Quality and operations

- Add tests across services
- Add logs and health checks
- Add CI workflows
- Add seed scripts and onboarding docs

### Phase 8: Deployment enablement

- Build production Docker images
- Add reverse proxy config
- Prepare AWS deployment docs and scripts
- Run end-to-end deployment rehearsal

## 20. Intern Reference Notes

Interns should use this document to answer the following questions:

- Which service owns which feature?
- Which API should a frontend or backend change target?
- Where should a new environment variable live?
- Which service should be deployed or debugged when a feature fails?
- Which data should be stored where?

When making changes, interns should avoid:

- Putting all business logic into the gateway
- Accessing another service's tables directly
- Skipping auth and ownership checks
- Hardcoding service URLs
- Mixing infrastructure configuration into application code

## 21. Recommended Decisions For This Project

To keep the architecture both educational and manageable, these are the recommended defaults:

- Use a modular monorepo
- Use one frontend app
- Use one API gateway
- Use five core backend microservices in v1:
  - identity-service
  - vendor-service
  - catalog-service
  - media-service
  - admin-service
- Use one PostgreSQL instance with separate schemas initially
- Use local storage in development and S3-compatible storage abstraction for deployment
- Use Docker Compose locally and on the first EC2 deployment

## 22. Definition Of Done For The Technical Plan

This technical plan is successful when:

- Every major feature maps to a clear service owner.
- The interaction model between services is understandable.
- The team can scaffold the repo directly from this document.
- Interns can trace a request from frontend to backend services.
- The plan supports both local development and later AWS deployment.
