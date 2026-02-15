# Build in Five — Add Backend API Endpoints

## Purpose
Use this prompt AFTER generating the skeleton app (01-skeleton-app-scaffold.md) to add domain-specific API endpoints for your demo scenario. This ensures consistent API patterns across all Build in Five applications.

## How to use
1. Generate the skeleton app first using `01-skeleton-app-scaffold.md`
2. Fill in the placeholders in Section 2 below
3. Paste into Cursor / GSD

---

## 1. API Conventions (mandatory)

All endpoints must follow these patterns:

| Convention | Standard |
|-----------|----------|
| URL prefix | `/api/v1/` |
| Naming | Plural nouns, kebab-case: `/api/v1/exposure-records` |
| Methods | GET (list/read), POST (create), PUT (full update), PATCH (partial update), DELETE (soft delete) |
| Pagination | `?skip=0&limit=20` query params; response includes `total`, `items`, `skip`, `limit` |
| Filtering | Query params: `?status=active&region=EMEA` |
| Sorting | `?sort_by=created_at&sort_order=desc` |
| Error responses | `{"detail": "Human-readable message", "code": "MACHINE_CODE"}` |
| Status codes | 200 (OK), 201 (Created), 204 (No Content/Deleted), 400 (Bad Request), 401 (Unauthorized), 403 (Forbidden), 404 (Not Found), 422 (Validation Error) |
| Auth | All endpoints require authentication by default |
| RBAC | Use `require_role()` dependency for role-gated endpoints |
| Timestamps | ISO 8601 UTC: `2025-02-11T14:30:00Z` |
| IDs | UUID v4 |

---

## 2. The Prompt

```
I have an existing FastAPI application scaffolded with the Build in Five skeleton (FastAPI, SQLAlchemy async, PostgreSQL, Entra ID auth, RBAC). I need to add domain-specific API endpoints.

## DOMAIN MODEL

Entity name: [ENTITY_NAME]
Description: [WHAT_THIS_ENTITY_REPRESENTS]

Fields:
- id: UUID (auto-generated)
- [FIELD_1]: [TYPE] — [DESCRIPTION]
- [FIELD_2]: [TYPE] — [DESCRIPTION]
- [FIELD_3]: [TYPE] — [DESCRIPTION]
- created_at: datetime (auto-generated)
- updated_at: datetime (auto-generated)
- created_by: str (from JWT, auto-populated)

Relationships:
- [DESCRIBE_ANY_RELATIONSHIPS_TO_OTHER_ENTITIES]

## ENDPOINTS NEEDED

1. `GET /api/v1/[ENTITIES]` — List all with pagination, filtering by [FILTER_FIELDS], sorting
2. `GET /api/v1/[ENTITIES]/{id}` — Get single by ID
3. `POST /api/v1/[ENTITIES]` — Create new (roles: [ALLOWED_ROLES])
4. `PUT /api/v1/[ENTITIES]/{id}` — Update (roles: [ALLOWED_ROLES])
5. `DELETE /api/v1/[ENTITIES]/{id}` — Soft delete (roles: [ALLOWED_ROLES])
[ADD OR REMOVE ENDPOINTS AS NEEDED]

## GENERATE THE FOLLOWING FILES

### backend/app/models/[entity].py
- SQLAlchemy model inheriting from Base
- All fields with correct types (UUID, String, Integer, DateTime, Boolean, etc.)
- Soft delete via `is_deleted: bool = False` and `deleted_at: datetime | None`
- Relationships defined with `relationship()` if applicable

### backend/app/schemas/[entity].py
- Pydantic v2 models:
  - `[Entity]Create` — fields required for creation (exclude id, timestamps, created_by)
  - `[Entity]Update` — all fields optional (for PATCH-style updates)
  - `[Entity]Response` — full entity including id, timestamps, created_by
  - `[Entity]ListResponse` — paginated: `{"items": [...], "total": int, "skip": int, "limit": int}`

### backend/app/services/[entity]_service.py
- Async service class with methods: list, get_by_id, create, update, soft_delete
- All database operations via SQLAlchemy async session
- Filtering and sorting applied at query level (not in Python)
- Raise HTTPException(404) if entity not found
- Raise HTTPException(403) if user lacks permission

### backend/app/api/v1/[entities].py
- FastAPI APIRouter with prefix="/api/v1/[entities]" and tags=["[Entities]"]
- Each endpoint uses dependencies: get_db, get_current_user, require_role (where needed)
- Request validation via Pydantic schemas
- Response model specified on each endpoint

### backend/app/api/router.py
- Update the main router to include the new entity router

### Alembic migration
- Generate migration: `alembic revision --autogenerate -m "add [entity] table"`
- Include up and down migration

### backend/tests/test_[entities].py
- Async tests using httpx AsyncClient
- Test: list (empty), create, get by id, update, delete, list (with data)
- Test: 401 without token, 403 with wrong role
- Use fixtures from conftest.py

## CRITICAL CONSTRAINTS
- Use async SQLAlchemy throughout (no sync operations)
- All responses must use Pydantic response models (no raw dicts)
- Pagination must be handled at the database level (LIMIT/OFFSET), not in Python
- Soft deletes only — never hard delete. Filter out soft-deleted records in all list/get queries.
- created_by must be auto-populated from the authenticated user's JWT (never from request body)
- All string fields that accept user input must have max_length constraints in the Pydantic schema
- No raw SQL — use SQLAlchemy query builder
```

---

## 3. Example: Exposure Records

Here's a filled-in example for the data ingestion demo scenario:

```
Entity name: ExposureRecord
Description: An individual exposure data record ingested from a client file upload

Fields:
- id: UUID (auto-generated)
- source_file: str — Original filename of the uploaded file
- client_name: str — Name of the client or prospect
- location_id: str — Location identifier from the source data
- geography: str — Geographic region (e.g., "US-FL", "EU-DE")
- peril: str — Peril type (e.g., "Hurricane", "Flood", "Earthquake")
- line_of_business: str — LoB category (e.g., "Property", "Casualty")
- tiv: float — Total Insured Value
- currency: str — ISO currency code (e.g., "USD", "GBP")
- validation_status: str — "valid", "warning", "error"
- validation_messages: list[str] — Any validation issues found
- created_at: datetime (auto-generated)
- updated_at: datetime (auto-generated)
- created_by: str (from JWT, auto-populated)

Relationships:
- Belongs to an IngestionJob (many ExposureRecords per IngestionJob)
```
