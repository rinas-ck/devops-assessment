# DevOps Intern Take-Home Assessment

Production deployment and CI/CD implementation for the provided React + TypeScript, FastAPI, PostgreSQL application.

## Architecture

Browser
  |
  +--> app.debyez.localhost --> Traefik --> Frontend (Nginx)
  |
  +--> api.debyez.localhost --> Traefik --> Backend (FastAPI)
                                             |
                                             +--> PostgreSQL
                                             |
                                             +--> LocalStack S3

## Technology Stack

- Frontend: React + TypeScript + Nginx
- Backend: FastAPI + Uvicorn
- Database: PostgreSQL 16
- Object Storage: LocalStack S3
- Reverse Proxy: Traefik v3.6.1
- Containers: Docker
- Orchestration: Docker Compose
- CI/CD: GitHub Actions
- Registry: Docker Hub

## Repository Structure

    .
    ├── backend/
    │   ├── app/
    │   ├── Dockerfile
    │   └── requirements.txt
    ├── frontend/
    │   ├── src/
    │   ├── Dockerfile
    │   └── nginx.conf
    ├── .github/
    │   └── workflows/
    │       └── ci.yml
    ├── .env.example
    ├── docker-compose.yml
    ├── docker-compose.prod.yml
    └── README.md

## Prerequisites

- Docker
- Docker Compose v2
- Git
- GitHub account
- Docker Hub account

Verify:

    docker --version
    docker compose version
    git --version

## Environment Configuration

Create the environment file:

    cp .env.example .env

Important configuration:

    DB_HOST=postgres
    DB_PORT=5432
    S3_ENDPOINT_URL=http://localstack:4566
    S3_PUBLIC_ENDPOINT_URL=http://localhost:4566
    VITE_API_BASE_URL=http://api.debyez.localhost/api
    CORS_ORIGINS=http://app.debyez.localhost

The .env file is ignored by Git.

## Development Deployment

Start the complete stack:

    docker compose up -d

Check services:

    docker compose ps

Stop the stack:

    docker compose down

Services:

- Traefik
- Frontend
- Backend
- PostgreSQL
- LocalStack

PostgreSQL and LocalStack use persistent Docker volumes.

## Application URLs

Frontend:

    http://app.debyez.localhost

Backend:

    http://api.debyez.localhost

Frontend API:

    http://api.debyez.localhost/api

Traefik handles routing between the hostnames and internal services.

## Docker Images

### Backend

The backend uses Python 3.12-slim and runs FastAPI with Uvicorn on port 8000.

### Frontend

The frontend uses a multi-stage Docker build.

Node.js builds the React application and Nginx serves the production build.

The API URL is supplied through VITE_API_BASE_URL.

## Traefik

Traefik provides hostname-based routing:

    app.debyez.localhost -> Frontend
    api.debyez.localhost -> Backend

Traefik listens on port 80.

The frontend and backend application ports are not directly exposed to the host.

## Health Checks

Available endpoints:

    /api/health
    /api/health/db
    /api/health/s3
    /api/health/full

Application health:

    curl http://api.debyez.localhost/api/health

Database health:

    curl http://api.debyez.localhost/api/health/db

S3 health:

    curl http://api.debyez.localhost/api/health/s3

Full health:

    curl http://api.debyez.localhost/api/health/full

Docker health checks are configured for PostgreSQL, LocalStack, backend, and frontend.

## S3-Compatible Storage

LocalStack provides S3-compatible object storage.

Internal endpoint:

    http://localstack:4566

Public endpoint:

    http://localhost:4566

The application supports:

- File upload
- File metadata
- File retrieval
- Presigned download URLs
- File deletion
- S3 health verification

## CI/CD

GitHub Actions workflow:

    .github/workflows/ci.yml

Trigger:

    Push or merge to main

Pipeline:

    Git Push
       |
       v
    GitHub Actions
       |
       +--> Validate Compose
       |
       +--> Build Backend
       |
       +--> Build Frontend
       |
       +--> Push Images to Docker Hub

Docker Hub repositories:

    rinasck/devops-assessment-backend
    rinasck/devops-assessment-frontend

Images are tagged with the full Git commit SHA:

    rinasck/devops-assessment-backend:<commit-sha>
    rinasck/devops-assessment-frontend:<commit-sha>

This provides traceability between a Docker image and its source commit.

## GitHub Secrets

Required GitHub Actions secrets:

    DOCKERHUB_USERNAME
    DOCKERHUB_TOKEN

The Docker Hub token is stored securely as a GitHub Secret and is not committed to the repository.

## Production Deployment

Production configuration:

    docker-compose.prod.yml

The production Compose file does not build the backend or frontend locally.

It uses published Docker Hub images.

Set the exact image versions:

    export BACKEND_IMAGE=rinasck/devops-assessment-backend:<commit-sha>
    export FRONTEND_IMAGE=rinasck/devops-assessment-frontend:<commit-sha>

Deploy:

    docker compose -f docker-compose.prod.yml up -d

Check:

    docker compose -f docker-compose.prod.yml ps

Check deployed images:

    docker compose -f docker-compose.prod.yml images

Production uses separate persistent volumes:

    postgres_data_prod
    localstack_data_prod

## Production Verification

Frontend:

    http://app.debyez.localhost

Backend:

    http://api.debyez.localhost

Health:

    curl http://api.debyez.localhost/api/health

Full health:

    curl http://api.debyez.localhost/api/health/full

Verify:

- Frontend access
- Backend access
- Database connectivity
- S3 connectivity
- File upload
- File download
- File deletion

## Rollback

Docker images are tagged with immutable Git commit SHAs.

To roll back, select a previously published SHA:

    export BACKEND_IMAGE=rinasck/devops-assessment-backend:<previous-commit-sha>
    export FRONTEND_IMAGE=rinasck/devops-assessment-frontend:<previous-commit-sha>

Redeploy:

    docker compose -f docker-compose.prod.yml up -d

Verify:

    docker compose -f docker-compose.prod.yml ps

    curl http://api.debyez.localhost/api/health/full

Then verify:

    http://app.debyez.localhost

Rollback uses the previously published Docker Hub images and does not require a local rebuild.

## Troubleshooting

Check services:

    docker compose ps

Production:

    docker compose -f docker-compose.prod.yml ps

View backend logs:

    docker compose logs backend

View frontend logs:

    docker compose logs frontend

View Traefik logs:

    docker compose logs traefik

Production backend logs:

    docker compose -f docker-compose.prod.yml logs backend

Production Traefik logs:

    docker compose -f docker-compose.prod.yml logs traefik

Validate Compose:

    docker compose config

Production validation:

    docker compose -f docker-compose.prod.yml config

Check port 80:

    sudo ss -ltnp | grep ':80'

## Verification Status

Verified:

- Docker containerization
- Production frontend with Nginx
- Docker Compose
- PostgreSQL persistence
- Container health checks
- LocalStack S3
- File upload
- File download
- File deletion
- Traefik routing
- Frontend hostname access
- Backend hostname access
- GitHub Actions CI/CD
- Docker Hub image publishing
- Commit SHA image tagging
- Production deployment
- Production health checks

## Git History

The implementation uses multiple meaningful commits:

1. Containerization and Compose infrastructure
2. LocalStack S3 storage
3. Traefik reverse proxy routing
4. CI/CD and production Compose
5. Deployment documentation

## Screen Recording

A screen recording is included with the assessment submission.

The demonstration covers:

- Application deployment
- Container health
- Traefik routing
- Health checks
- S3 file operations
- GitHub Actions
- Docker Hub images
- Production deployment
- Commit SHA image versioning
- Rollback verification
