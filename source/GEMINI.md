# SemanticLexiForge - Project Context & Guidelines

This document provides essential context for Gemini CLI interactions within the `<project_name>` workspace. It defines the project architecture, development standards, and operational commands.

## Project Overview
SemanticLexiForge is an enterprise-grade 3-tier application designed for Azure deployment. It features a React-based frontend, a Java Spring Boot backend, and a PostgreSQL database.

- **Architecture**: 
  - **Frontend**: React 18 (TypeScript, Vite, Material UI).
  - **Backend**: Java 23 (Spring Boot 3.4.3, Maven multi-module: `domain`, `repository`, `service`, `api`).
  - **Database**: PostgreSQL 17 (Flyway for migrations).
  - **Infrastructure**: Azure-native (AKS, ACR, Key Vault) managed via Terraform.
  - **CI/CD**: Azure DevOps Pipelines.
- **Key Design Principles**:
  - **Zero-Secret Hardcoding**: Use Environment Variables or Azure Key Vault.
  - **IaC-First**: All infrastructure changes via Terraform.
  - **ABAC Security**: Default-deny Access Based Access Control.
  - **Observability**: Structured JSON logging (non-dev), ELK stack integration.

---

## Technical Stack & Prerequisites
- **Java**: JDK 23+ (Maven 3.9+)
- **Node.js**: Node 20+ (NPM 10+)
- **Containerization**: Docker Desktop (WSL2 backend on Windows)
- **Infrastructure**: Terraform, Azure CLI

---

## Core Commands

### Full Stack (Local Development)
```bash
# Start all services (Postgres, Backend, Frontend)
docker compose up --build
```

### Backend (Java/Spring Boot)
Located in `/backend`.
- **Build & Verify**: `mvn clean verify`
- **Unit Tests**: `mvn test`
- **Integration Tests** (Requires Docker): `mvn verify -P integration-tests`
- **Profiles**: `dev`, `prod`, `dev-pg`. Set via `SPRING_PROFILES_ACTIVE`.

### Frontend (React/Vite)
Located in `/frontend`.
- **Install**: `npm ci`
- **Dev Server**: `npm run dev`
- **Build**: `npm run build`
- **Test**: `npm test`
- **Lint**: `npm run lint`
- **Format Check**: `npm run format:check`

### Infrastructure (Terraform)
Located in `/infra`.
- Modules for `aks`, `acr`, `keyvault`, and `postgres` are in `/infra/modules`.
- Kubernetes manifests in `/infra/k8s`.

---

## Development Conventions

### Branching & Commits
- **Branch Naming**: `feat/US-{id}-{short-description}` (e.g., `feat/US-002-backend-scaffold`).
- **Commit Messages**: [Conventional Commits](https://www.conventionalcommits.org/) (`feat|fix|chore|docs|test|refactor(scope): message`).
- **Enforcement**: Husky + Commitlint block non-compliant commits.

### Repository Structure
- `backend/`: Maven multi-module structure.
  - `api/`: Controllers, DTOs, OpenAPI.
  - `service/`: Business logic, ABAC.
  - `repository/`: Persistence layer, Flyway migrations.
  - `domain/`: Entities and domain models.
- `frontend/`: React application (Vite).
- `infra/`: Terraform modules and K8s manifests.
- `docs/`: Design documents (`US-xxx/DESIGN.md`, `TASKS.md`).

### Coding Standards
- **Java**: Follow standard Spring Boot idioms. Use `@Tag("testcontainers")` for integration tests requiring Docker.
- **Frontend**: Airbnb TypeScript style guide (enforced via ESLint).
- **Security**: Database migrations must go through Flyway; no manual DDL.
- **Logging**: Include `traceId`, `spanId`, `correlationId`, and `sessionId` on every log line in non-dev environments.

---

## Important Files
- `CLAUDE.md`: High-level developer quick-start and rules.
- `docker-compose.yml`: Local multi-container orchestration.
- `backend/pom.xml`: Root Maven configuration.
- `frontend/package.json`: Frontend dependency and script management.
- `infra/modules/postgres/main.tf`: Database infrastructure definition.
