# Build in Five — Docker & Application Factory Deployment

## Purpose
Use this prompt to containerise a Build in Five application and deploy it to the Application Factory on our shared AWS ECS cluster. Run this AFTER the app is working locally via docker-compose.

## How to use
1. Application working locally (`docker-compose up` passes all checks)
2. Fill in placeholders in Section 2
3. Paste into Cursor / GSD to generate deployment configuration
4. Run `./factory-provision.sh` to deploy

---

## 1. Infrastructure Standards (mandatory)

| Component | Standard |
|-----------|----------|
| Container registry | Amazon ECR, one repo per service |
| Image tags | Git SHA only. Never `:latest`. |
| Compute | AWS ECS Fargate, awsvpc network mode |
| Load balancer | Separate ALB per application |
| Database | PostgreSQL 15 on shared RDS instance, separate database per app |
| Secrets | AWS Secrets Manager: `/factory/{app-name}/{env}/` prefix |
| Logging | CloudWatch Logs: `/factory/{app-name}/{env}/` prefix |
| DNS | `{app-name}.factory.{domain}` |
| IaC | Terraform using shared Application Factory modules |
| CI/CD | GitHub Actions: push to main → build → ECR → ECS |
| Naming | `factory-{app-name}-{component}-{env}` |

---

## 2. The Prompt

```
I have a working full-stack application (FastAPI backend + React frontend + PostgreSQL) running locally via Docker Compose. I need to prepare it for deployment to our Application Factory on AWS ECS Fargate.

## APPLICATION DETAILS
- App name: [APP_NAME] (lowercase, hyphenated, e.g., "exposure-demo")
- Environment: [ENV — dev/prod]
- Domain: [APP_NAME].factory.[YOUR_DOMAIN]
- Backend port: 8000
- Frontend port: 80

## GENERATE / UPDATE THE FOLLOWING

### backend/Dockerfile (production-optimised)
```dockerfile
# Build stage
FROM python:3.11-slim as builder

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Runtime stage
FROM python:3.11-slim

WORKDIR /app

# Copy installed packages from builder
COPY --from=builder /root/.local /root/.local
ENV PATH=/root/.local/bin:$PATH

# Copy application code
COPY app/ ./app/
COPY alembic/ ./alembic/
COPY alembic.ini .

# Run as non-root user
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/api/v1/health')" || exit 1

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "2"]
```

### frontend/Dockerfile (production-optimised)
```dockerfile
# Build stage
FROM node:20-alpine as builder

WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --no-audit
COPY . .
RUN npm run build

# Runtime stage
FROM nginx:1.25-alpine

# Copy built assets
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Health check
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
  CMD wget -qO- http://localhost:80/ || exit 1

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### frontend/nginx.conf
```nginx
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml;

    # Frontend routes — serve index.html for all paths (React Router)
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API proxy — forward to backend ECS service via CloudMap or ALB
    location /api/ {
        proxy_pass http://[APP_NAME]-backend.[NAMESPACE]:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
}
```

### .github/workflows/deploy.yml
```yaml
name: Build and Deploy to ECS

on:
  push:
    branches: [main]

env:
  AWS_REGION: eu-west-1
  APP_NAME: [APP_NAME]
  ECR_REGISTRY: [AWS_ACCOUNT_ID].dkr.ecr.eu-west-1.amazonaws.com
  ECS_CLUSTER: factory-cluster
  ENVIRONMENT: [ENV]

permissions:
  id-token: write
  contents: read

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::[AWS_ACCOUNT_ID]:role/github-actions-deploy
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: ecr-login
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push backend image
        env:
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/factory/$APP_NAME/backend:$IMAGE_TAG ./backend
          docker push $ECR_REGISTRY/factory/$APP_NAME/backend:$IMAGE_TAG

      - name: Build and push frontend image
        env:
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/factory/$APP_NAME/frontend:$IMAGE_TAG ./frontend
          docker push $ECR_REGISTRY/factory/$APP_NAME/frontend:$IMAGE_TAG

      - name: Update ECS backend task definition
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        id: render-backend
        with:
          task-definition: .aws/backend-task-def.json
          container-name: ${{ env.APP_NAME }}-backend
          image: ${{ env.ECR_REGISTRY }}/factory/${{ env.APP_NAME }}/backend:${{ github.sha }}

      - name: Deploy backend to ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.render-backend.outputs.task-definition }}
          service: factory-${{ env.APP_NAME }}-backend-${{ env.ENVIRONMENT }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true

      - name: Update ECS frontend task definition
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        id: render-frontend
        with:
          task-definition: .aws/frontend-task-def.json
          container-name: ${{ env.APP_NAME }}-frontend
          image: ${{ env.ECR_REGISTRY }}/factory/${{ env.APP_NAME }}/frontend:${{ github.sha }}

      - name: Deploy frontend to ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.render-frontend.outputs.task-definition }}
          service: factory-${{ env.APP_NAME }}-frontend-${{ env.ENVIRONMENT }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true
```

### terraform/main.tf
```hcl
module "factory_app" {
  source = "../modules/factory-app"  # Shared Application Factory module

  app_name    = "[APP_NAME]"
  environment = "[ENV]"
  
  # ECS configuration
  ecs_cluster_id = data.aws_ecs_cluster.factory.id
  
  backend_image  = "${var.ecr_registry}/factory/[APP_NAME]/backend:${var.image_tag}"
  frontend_image = "${var.ecr_registry}/factory/[APP_NAME]/frontend:${var.image_tag}"
  
  backend_cpu    = 512
  backend_memory = 1024
  frontend_cpu   = 256
  frontend_memory = 512

  # Database
  rds_instance_id = data.aws_db_instance.shared.id
  db_name         = "factory_[APP_NAME_UNDERSCORED]_[ENV]"

  # Networking
  vpc_id             = data.aws_vpc.main.id
  private_subnet_ids = data.aws_subnets.private.ids
  public_subnet_ids  = data.aws_subnets.public.ids

  # DNS
  domain_name    = "[APP_NAME].factory.[DOMAIN]"
  hosted_zone_id = data.aws_route53_zone.factory.zone_id

  # Secrets (created in Secrets Manager before first deploy)
  secrets = {
    DATABASE_URL     = "/factory/[APP_NAME]/[ENV]/database-url"
    ENTRA_TENANT_ID  = "/factory/[APP_NAME]/[ENV]/entra-tenant-id"
    ENTRA_CLIENT_ID  = "/factory/[APP_NAME]/[ENV]/entra-client-id"
    ENTRA_AUDIENCE   = "/factory/[APP_NAME]/[ENV]/entra-audience"
  }

  # Tags
  tags = {
    Project     = "build-in-five"
    Application = "[APP_NAME]"
    Environment = "[ENV]"
    Owner       = "[YOUR_NAME]"
    ManagedBy   = "terraform"
  }
}
```

### terraform/variables.tf
```hcl
variable "ecr_registry" {
  description = "ECR registry URL"
  type        = string
}

variable "image_tag" {
  description = "Docker image tag (Git SHA)"
  type        = string
}
```

### factory-manifest.json (update with deployment details)
- Update with actual AWS resource names after Terraform apply
- Used by factory tooling for lifecycle management

## DEPLOYMENT CHECKLIST

Before first deployment:
1. [ ] Application works locally via docker-compose
2. [ ] All tests pass
3. [ ] factory-manifest.json filled in
4. [ ] Secrets created in AWS Secrets Manager
5. [ ] Entra ID App Registration created for this app
6. [ ] Terraform plan reviewed and approved
7. [ ] GitHub repo created with deploy workflow secrets configured

Deploy:
1. [ ] Run `terraform apply` to provision infrastructure
2. [ ] Push to main to trigger first deployment
3. [ ] Verify backend health: `curl https://[APP_NAME].factory.[DOMAIN]/api/v1/health`
4. [ ] Verify frontend loads: `open https://[APP_NAME].factory.[DOMAIN]`
5. [ ] Verify auth flow: login via Entra ID, check token propagation
6. [ ] Run Alembic migrations against RDS: `alembic upgrade head`

## CRITICAL CONSTRAINTS
- Never use :latest tags
- Never hardcode AWS account IDs, secrets, or credentials in code
- Always use OIDC for GitHub Actions → AWS authentication (no access keys)
- Always run as non-root user in containers
- Always include HEALTHCHECK in Dockerfiles
- Always use multi-stage builds to minimise image size
- Terraform state must be in the shared S3 backend (never local state)
```
