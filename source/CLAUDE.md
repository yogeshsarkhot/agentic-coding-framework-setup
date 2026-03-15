# CLAUDE.md

# Project: Enterprise 3-Tier Application

## Azure DevOps
- Organisation: <organization_name>
- Project:      <project_name>
- Repo:         https://dev.azure.com/<organization_name>/_git/<project_name>
- Sprint field: Iteration Path = "<project_name>\\Sprint 1"
- Work item type for user stories: User Story (Agile process template)

## Tech Stack
- Frontend:  React 18 + TypeScript + Vite + Material UI
- Backend:   Java 23 + Spring Boot 3 + Maven (multi-module)
- Database:  PostgreSQL 17 + Flyway migrations
- Container: Docker (multi-stage) + ACR
- Infra:     Terraform + AKS
- CI/CD:     Azure DevOps Pipelines

## Branch Naming Convention
- Feature branches: feat/US-{id}-{short-description}
- Example: feat/US-002-spring-boot-scaffold

## Commit Convention
- Conventional Commits: feat|fix|chore|docs|test|refactor(scope): message
- Example: feat(backend): add Spring Actuator health endpoint

## Build Commands
- Backend:  cd backend && mvn clean verify
- Frontend: cd frontend && npm ci && npm run build
- Docker:   docker compose build
- Local up: docker compose up --build

## Test Commands
- Backend unit:        mvn test
- Backend integration: mvn verify -P integration-tests
- Frontend:            npm test -- --run

## Key Design Rules (enforced by ARB)
- Never hardcode secrets — environment variables or Azure Key Vault only
- All infrastructure changes via Terraform — no manual Azure portal changes
- Database migrations via Flyway only — no manual DDL
- Default-deny ABAC — absence of policy entry = access denied
- Structured JSON logging only in non-dev profiles
- traceId, spanId, correlationId, sessionId on every log line
