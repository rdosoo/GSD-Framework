# Build in Five — Prompt Library

## Overview

These prompts standardise application development across the Build in Five programme. Every application built by the team — whether for a live demo, a rapid prototype, or an Application Factory deployment — must use these prompts to ensure consistency with our approved tech stack and AWS deployment patterns.

## Approved Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | FastAPI 0.115.x + Uvicorn + Python 3.11 |
| ORM | SQLAlchemy 2.x (async, asyncpg driver) |
| Migrations | Alembic |
| Database | PostgreSQL 15+ (RDS in production) |
| Frontend | React 18.x (functional components, hooks only) |
| Build | Vite |
| CSS | Tailwind CSS 3.x |
| Auth | Microsoft Entra ID (MSAL React + python-jose/authlib) |
| AI (optional) | OpenAI API |
| Containers | Docker multi-stage builds |
| Reverse proxy | Nginx (frontend) |
| CI/CD | GitHub Actions → ECR → ECS |
| Infrastructure | Terraform (Application Factory modules) |
| Cloud | AWS ECS Fargate (shared cluster, separate ALB per app) |

**Do not substitute any technology without explicit approval from the technical lead.**

## Prompt Sequence

Use these prompts in order when building a new application:

| # | Prompt | Purpose | When to use |
|---|--------|---------|-------------|
| 01 | `01-skeleton-app-scaffold.md` | Generate the complete application skeleton with auth, API, frontend, Docker, CI/CD, and Terraform | **Always first.** Every new app starts here. |
| 02 | `02-add-backend-api.md` | Add domain-specific API endpoints, models, schemas, services, and tests | After skeleton is generated. Use once per entity/resource. |
| 03 | `03-add-frontend-pages.md` | Add domain-specific pages, tables, forms, and UI components | After backend endpoints exist. Use once per page. |
| 04 | `04-docker-and-deployment.md` | Production Docker configuration, GitHub Actions CI/CD, Terraform, and deployment checklist | When ready to deploy to Application Factory. |

## Workflow

```
1. /gsd/new-project → paste 01-skeleton-app-scaffold.md
2. /gsd/execute → scaffold generated
3. docker-compose up → verify skeleton works locally
4. Paste 02-add-backend-api.md → add your domain models and endpoints
5. Paste 03-add-frontend-pages.md → add your UI
6. docker-compose up → verify full app works locally
7. Run tests → all passing
8. Paste 04-docker-and-deployment.md → prepare for AWS deployment
9. ./factory-provision.sh → deploy to Application Factory
```

## Customisation Rules

- **You CAN** add: models, endpoints, pages, components, services, tests
- **You CAN** add: additional Python or npm packages that don't conflict with the stack
- **You CANNOT** replace: the backend framework, frontend framework, CSS framework, auth provider, or database
- **You CANNOT** add: Redux, styled-components, CSS modules, SQLite, or any tech explicitly prohibited in the prompts

## Questions?

Contact the Build in Five technical lead (Advisory Services) or raise in the project Slack channel.
