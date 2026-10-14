# test-manager-library-3145555-infra

Infrastructure for *Manager Library*: Docker Compose services, network, persistent PostgreSQL storage, and an empty database engine.

## Scope and SDD traceability

The mandatory stack is PHP/Laravel, React, PostgreSQL 16, and Docker Compose. The Python and .NET directories in the
[specification repository](https://github.com/code-sena/test-manager-library-3145555-docs) are references; their business rules,
data model, HTTP contract, and acceptance criteria remain mandatory.

| Specification source | Infrastructure outcome |
|---|---|
| `spec.md` §5, RNF-5 | Compose defines one command for the system and starts PostgreSQL without application tables. |
| `tasks.md`, T-20 | Compose declares the database, Laravel API, and React frontend services. Completion still depends on the API and frontend Dockerfiles and their integration validation. |
| `constitution.md`, Article V; ADR-001 | Infrastructure creates only the database engine, user, volume, and network. Laravel migrations own DDL, constraints, extensions, and category seed data. |
| `data-model.md` §§3–4 | No SQL initialization is included here. Laravel must create the `library` schema, six tables, indexes, constraints, and fixed categories through migrations. |
| `data-model.md` §6; `spec.md`, RN-6 | PostgreSQL is explicitly configured for UTC. |

## Prerequisites

- Docker Engine with Docker Compose v2.
- Sibling repositories with these exact directory names next to this repository:
  - `test-manager-library-3145555-api`
  - `test-manager-library-3145555-app`
- A root-level `Dockerfile` in each sibling repository. They are not present at the time this infrastructure was created.

## Configuration

Create a local environment file from the example and replace every placeholder with local, non-production values:

```sh
cp .env.example .env
```

`.env` is ignored by Git. Do not commit application keys, JWT secrets, database passwords, or administrator credentials.

The PostgreSQL container creates an empty `library` database and its database user only. It does not create the `library` schema,
application tables, indexes, constraints, extensions, or seed data.

## Start the services

After the API and frontend repositories provide their Dockerfiles, run from this repository:

```sh
docker compose up --build
```

Service endpoints use the local ports defined in `.env`:

| Service | Default local endpoint |
|---|---|
| PostgreSQL 16 | `localhost:5432` |
| Laravel API | `http://localhost:8000` |
| React frontend | `http://localhost:5173` |

The API waits for PostgreSQL health, and the frontend starts after the API service starts. The API repository is responsible for
running Laravel migrations and creating the initial administrator from environment variables. Do not add DDL or seed scripts to this repository.

## Validate the Compose definition

```sh
docker compose --env-file .env.example config
```

This validates Compose interpolation only. A successful `docker compose up --build` additionally requires the missing API and frontend
Dockerfiles, Laravel runtime configuration, React runtime configuration, and integration tests. Therefore, T-20 is not yet complete.

## Repository responsibilities

| Repository | Responsibility |
|---|---|
| `test-manager-library-3145555-infra` | Containers, network, PostgreSQL volume, and empty database engine |
| `test-manager-library-3145555-api` | Laravel application, migrations, schema `library`, seed categories, initial administrator, API contract, and business rules |
| `test-manager-library-3145555-app` | React frontend |
| `test-manager-library-3145555-docs` | Authoritative specifications |
