# ElderPing Health Service

## Overview

The Health Service is a Node.js-based microservice responsible for tracking and managing health data for elderly users in the ElderPing platform. It handles health metrics recording, medication tracking, and health data retrieval.

## Technology Stack

- **Runtime**: Node.js 18
- **Framework**: Express.js
- **Database**: PostgreSQL
- **Container**: Docker (multi-stage build with Alpine Linux)
- **Security**: Non-root user execution (USER node)

## Features

- Health metrics recording (blood pressure, heart rate, weight, etc.)
- Medication tracking and adherence monitoring
- Health data history and trends
- Health alerts and notifications
- Health check endpoint for monitoring

## API Endpoints

### Health Metrics
- `POST /api/health/metrics` - Record health metrics
- `GET /api/health/metrics/:userId` - Get health metrics for user
- `GET /api/health/metrics/:userId/history` - Get health history

### Medication
- `POST /api/health/medication` - Add medication record
- `GET /api/health/medication/:userId` - Get medications for user
- `PUT /api/health/medication/:id` - Update medication
- `DELETE /api/health/medication/:id` - Delete medication

### Health
- `GET /health` - Health check endpoint

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `DB_HOST` | PostgreSQL database host | Yes |
| `DB_PORT` | PostgreSQL database port | Yes |
| `DB_USER` | PostgreSQL username | Yes |
| `DB_PASSWORD` | PostgreSQL password | Yes |
| `DB_NAME` | Database name (health_db) | Yes |
| `PORT` | Service port (default: 3000) | No |

## Database Schema

### Health Metrics Table
```sql
CREATE TABLE health_metrics (
  id         SERIAL PRIMARY KEY,
  user_id    INT NOT NULL,
  metric_type VARCHAR(50) NOT NULL,
  value      DECIMAL(10,2) NOT NULL,
  unit       VARCHAR(20),
  recorded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Medications Table
```sql
CREATE TABLE medications (
  id          SERIAL PRIMARY KEY,
  user_id     INT NOT NULL,
  name        VARCHAR(100) NOT NULL,
  dosage      VARCHAR(50),
  frequency   VARCHAR(50),
  start_date  DATE,
  end_date    DATE,
  created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Docker Image

- **Repository**: `arunnsimon/elderpinq-health-service`
- **Tags**: 
  - `dev-latest` - Development builds from develop branch
  - `prod-latest` - Production builds from main branch
  - `<version>` - Release tags

## CI/CD Pipeline

The service uses GitHub Actions for continuous integration and deployment:

1. **Security Scanning**
   - SAST (Static Application Security Testing)
   - SCA (Software Composition Analysis)
   - Trivy vulnerability scanning

2. **Docker Build & Publish**
   - Multi-stage Docker build
   - Push to Docker Hub
   - Tagged based on branch (dev-latest/prod-latest)

3. **GitOps Deployment**
   - Updates Helm chart image tag in elderping-k8s-charts
   - ArgoCD automatically syncs changes

## Kubernetes Deployment

### Helm Chart
Located in `elderping-k8s-charts/microservices/health-service/`

**Resources:**
- Deployment with 2 replicas
- Service (ClusterIP on port 3000)
- HorizontalPodAutoscaler (2-5 replicas, 80% CPU target)

**Configuration:**
- Namespace: elderping-dev (dev) / elderping-prod (prod)
- Resource requests: 100m CPU, 128Mi memory
- Resource limits: 500m CPU, 256Mi memory
- Liveness/Readiness probes on /health endpoint

## Security Features

- **Non-root container**: Runs as `node` user (not root)
- **Environment variables**: Sensitive data via Kubernetes Secrets
- **Network policies**: Restricts ingress/egress traffic (when enabled)

## Development

### Local Setup
```bash
# Install dependencies
npm install

# Set environment variables
cp .env.example .env
# Edit .env with your values

# Run development server
npm run dev

# Run tests
npm test

# Build for production
npm run build
```

### Docker Build
```bash
# Build image
docker build -t elderping-health-service .

# Run container
docker run -p 3000:3000 --env-file .env elderping-health-service
```

## Monitoring

- **Health Check**: `GET /health` returns service status
- **Metrics**: Exposed for Prometheus scraping
- **Logs**: Collected by Loki
- **Dashboards**: Grafana dashboards for monitoring

## Troubleshooting

### Common Issues

**Database Connection Failed**
- Verify DB_HOST, DB_PORT, DB_USER, DB_PASSWORD, DB_NAME
- Check PostgreSQL is accessible from the pod
- Verify network policies allow database access

**Container Not Starting**
- Check pod logs: `kubectl logs <pod-name> -n elderping-dev`
- Verify resource limits are sufficient
- Check liveness probe configuration

## Contributing

1. Create feature branch from develop
2. Make changes and test locally
3. Commit with descriptive message
4. Push to feature branch
5. Create pull request to develop

## License

Proprietary - ElderPing Platform
