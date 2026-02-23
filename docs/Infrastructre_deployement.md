# Infrastructure & Deployment Specification
**Version:** 1.0  
**Status:** Draft  
**Date:** February 2026  

---

## Overview

This document defines the GCP infrastructure configuration, deployment procedures, environment strategy, and operational runbooks for the AgriField platform.

---

## 1. GCP Services Used

| Service | Purpose | Tier |
|---|---|---|
| Cloud Run | All backend services | Serverless |
| Cloud SQL (PostgreSQL 15) | Primary relational database | Enterprise (prod), Basic (dev/staging) |
| Cloud Storage | Photos, help content, exports | Standard |
| Cloud Tasks | AI dispatch queue, async processing | Standard |
| Pub/Sub | Event-driven messaging between services | Standard |
| Cloud Scheduler | Cron jobs (visit generation, reports) | Standard |
| Memorystore (Redis 7) | Caching — master data, sessions | Basic (prod) |
| Firebase Auth | SSO bridge, JWT issuance | Spark/Blaze |
| Firebase Cloud Messaging | Push notifications | Free tier |
| API Gateway | Public ingress, auth, rate limiting | Standard |
| Cloud Armor | WAF, DDoS protection | Standard |
| Cloud Build | CI/CD pipeline | Standard |
| Artifact Registry | Docker image storage | Standard |
| Secret Manager | Credentials and API keys | Standard |
| Cloud Logging | Centralised logging | Standard |
| Cloud Monitoring | Metrics and alerting | Standard |
| Cloud Trace | Distributed tracing | Standard |

---

## 2. Environment Configuration

### 2.1 Development (`agrifield-dev`)

| Resource | Configuration |
|---|---|
| Cloud SQL | db-f1-micro, 10GB, no HA |
| Cloud Run | Max 5 instances per service |
| Cloud Storage | Single bucket with env prefix |
| Memorystore | Not provisioned (in-memory mock in dev) |
| Cloud Armor | Not enabled |
| Firebase | Separate dev Firebase project |

### 2.2 Staging (`agrifield-staging`)

| Resource | Configuration |
|---|---|
| Cloud SQL | db-g1-small, 50GB, no HA |
| Cloud Run | Max 10 instances per service |
| Cloud Storage | Separate staging buckets |
| Memorystore | Basic tier, 1GB |
| Cloud Armor | Enabled (permissive rules) |
| Firebase | Separate staging Firebase project |

### 2.3 Production (`agrifield-prod`)

| Resource | Configuration |
|---|---|
| Cloud SQL | db-custom-2-7680, 200GB, HA enabled, 1 read replica |
| Cloud Run | Max 50 instances (api-core), per-service limits |
| Cloud Storage | Multi-region, versioning enabled |
| Memorystore | Standard tier, 5GB |
| Cloud Armor | Strict rules, geo-blocking enabled |
| Firebase | Production Firebase project |

---

## 3. Cloud Run Service Deployments

### 3.1 Deployment Configuration (api-core example)

```yaml
# cloud-run/api-core.yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: api-core
  annotations:
    run.googleapis.com/ingress: internal-and-cloud-load-balancing
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/minScale: "0"
        autoscaling.knative.dev/maxScale: "50"
        run.googleapis.com/cpu-throttling: "true"
        run.googleapis.com/startup-cpu-boost: "true"
        run.googleapis.com/cloudsql-instances: agrifield-prod:asia-southeast1:agrifield-db
    spec:
      containerConcurrency: 80
      timeoutSeconds: 60
      serviceAccountName: api-core-sa@agrifield-prod.iam.gserviceaccount.com
      containers:
        - image: asia-southeast1-docker.pkg.dev/agrifield-prod/services/api-core:latest
          ports:
            - containerPort: 8080
          env:
            - name: NODE_ENV
              value: production
            - name: DB_CONNECTION_NAME
              value: agrifield-prod:asia-southeast1:agrifield-db
            - name: FIREBASE_PROJECT_ID
              value: agrifield-prod
          resources:
            limits:
              cpu: "1"
              memory: 512Mi
          volumeMounts:
            - name: secrets
              mountPath: /secrets
      volumes:
        - name: secrets
          secret:
            secretName: api-core-secrets
```

---

### 3.2 All Service Deploy Commands

```bash
# Deploy all services to a given environment
# Usage: ./scripts/deploy.sh [dev|staging|prod] [service|all]

# Deploy single service
gcloud run deploy api-core \
  --image asia-southeast1-docker.pkg.dev/agrifield-prod/services/api-core:$TAG \
  --region asia-southeast1 \
  --project agrifield-prod \
  --platform managed \
  --no-traffic  # Deploy without sending traffic (use for canary)

# Shift traffic after validation
gcloud run services update-traffic api-core \
  --to-latest \
  --region asia-southeast1 \
  --project agrifield-prod
```

---

## 4. Cloud SQL Configuration

### 4.1 Production Database Setup

```bash
# Create production Cloud SQL instance
gcloud sql instances create agrifield-db \
  --database-version POSTGRES_15 \
  --tier db-custom-2-7680 \
  --region asia-southeast1 \
  --availability-type REGIONAL \    # High availability
  --backup-start-time 02:00 \
  --enable-point-in-time-recovery \
  --storage-auto-increase \
  --storage-size 200 \
  --no-assign-ip \                   # Private IP only
  --network default \
  --project agrifield-prod

# Create database
gcloud sql databases create agrifield \
  --instance agrifield-db \
  --project agrifield-prod

# Create read replica
gcloud sql instances create agrifield-db-replica \
  --master-instance-name agrifield-db \
  --region asia-southeast2 \         # Different region for DR
  --project agrifield-prod
```

### 4.2 Database Migrations

Migrations are managed with **Flyway** (or equivalent):

```
db/
└── migrations/
    ├── V1__initial_schema.sql
    ├── V2__add_seasons_table.sql
    ├── V3__add_price_predictions.sql
    └── ...
```

Migrations run automatically in CI/CD pipeline before deploying new service versions:

```bash
# Run migrations against target environment
flyway -url=jdbc:postgresql://... -user=... -password=... migrate
```

**Rules:**
- Migrations are additive only — never modify or delete existing columns in production
- Breaking changes (column removal, type changes) require multi-phase migrations
- Every migration has a corresponding rollback script in `db/rollbacks/`

---

## 5. Cloud Storage Buckets

```bash
# Create production buckets
# Photos — raw uploads from mobile
gcloud storage buckets create gs://agrifield-prod-photos-raw \
  --location asia-southeast1 \
  --uniform-bucket-level-access \
  --no-public-access-prevention

# Photos — post-processing
gcloud storage buckets create gs://agrifield-prod-photos-processed \
  --location asia-southeast1 \
  --uniform-bucket-level-access

# Help content (videos, PDFs)
gcloud storage buckets create gs://agrifield-prod-help-content \
  --location asia-southeast1

# Report exports
gcloud storage buckets create gs://agrifield-prod-exports \
  --location asia-southeast1
```

### Lifecycle Policies

```json
// agrifield-prod-photos-raw lifecycle
{
  "rule": [{
    "action": { "type": "SetStorageClass", "storageClass": "NEARLINE" },
    "condition": { "age": 90 }         // Move to Nearline after 90 days
  }, {
    "action": { "type": "SetStorageClass", "storageClass": "COLDLINE" },
    "condition": { "age": 365 }        // Move to Coldline after 1 year
  }]
}

// agrifield-prod-exports lifecycle
{
  "rule": [{
    "action": { "type": "Delete" },
    "condition": { "age": 7 }          // Auto-delete exports after 7 days
  }]
}
```

---

## 6. Cloud Tasks Queues

```bash
# AI analysis dispatch queue
gcloud tasks queues create ai-dispatch \
  --location asia-southeast1 \
  --max-dispatches-per-second 10 \    # Respect AI API rate limits
  --max-concurrent-dispatches 20 \
  --max-attempts 5 \
  --min-backoff 10s \
  --max-backoff 300s \
  --max-doublings 3 \
  --project agrifield-prod

# Notification dispatch queue
gcloud tasks queues create notification-dispatch \
  --location asia-southeast1 \
  --max-dispatches-per-second 50 \
  --max-concurrent-dispatches 20 \
  --max-attempts 3 \
  --project agrifield-prod
```

---

## 7. Cloud Scheduler Jobs

```bash
# Daily visit schedule generation (6 AM local time)
gcloud scheduler jobs create http generate-scheduled-visits \
  --location asia-southeast1 \
  --schedule "0 6 * * *" \
  --time-zone "Asia/Kuala_Lumpur" \
  --uri https://scheduler-service-xxx.run.app/internal/generate-visits \
  --http-method POST \
  --oidc-service-account-email scheduler-sa@agrifield-prod.iam.gserviceaccount.com \
  --project agrifield-prod

# Weekly agent performance report (Monday 8 AM)
gcloud scheduler jobs create http weekly-performance-report \
  --location asia-southeast1 \
  --schedule "0 8 * * 1" \
  --time-zone "Asia/Kuala_Lumpur" \
  --uri https://scheduler-service-xxx.run.app/internal/weekly-report \
  --http-method POST \
  --oidc-service-account-email scheduler-sa@agrifield-prod.iam.gserviceaccount.com \
  --project agrifield-prod

# Hourly sync queue health check
gcloud scheduler jobs create http sync-health-check \
  --location asia-southeast1 \
  --schedule "0 * * * *" \
  --uri https://api-core-xxx.run.app/internal/sync-health \
  --http-method GET \
  --oidc-service-account-email scheduler-sa@agrifield-prod.iam.gserviceaccount.com \
  --project agrifield-prod
```

---

## 8. Secret Manager

```bash
# Store secrets (example — api-core)
echo -n "connection_string" | \
  gcloud secrets create db-connection-string \
  --data-file=- \
  --replication-policy automatic \
  --project agrifield-prod

# Grant Cloud Run service account access
gcloud secrets add-iam-policy-binding db-connection-string \
  --member serviceAccount:api-core-sa@agrifield-prod.iam.gserviceaccount.com \
  --role roles/secretmanager.secretAccessor \
  --project agrifield-prod
```

### Secrets Inventory

| Secret Name | Used By | Rotation |
|---|---|---|
| `firebase-service-account` | All services | Annual |
| `db-connection-string` | api-core, api-sync, admin-api | Annual |
| `ai-api-key` | ai-dispatcher | Quarterly |
| `redis-connection-string` | api-core, api-sync | Annual |
| `fcm-server-key` | notification-service | Annual |

---

## 9. Deployment Runbook

### Standard Deployment (staging → prod)

```bash
# 1. Merge to main — auto-deploys to staging
# 2. QA runs on staging
# 3. Tag release

git tag v1.2.0
git push origin v1.2.0

# 4. Cloud Build trigger fires for release tag
# 5. Images built and pushed to Artifact Registry
# 6. Deploy services to prod with zero traffic

gcloud run deploy api-core \
  --image .../api-core:v1.2.0 \
  --no-traffic \
  --tag v1-2-0

# 7. Run smoke tests against new revision
curl https://v1-2-0---api-core-xxx.run.app/health

# 8. Gradually shift traffic
gcloud run services update-traffic api-core \
  --to-revisions v1-2-0=10  # 10% canary

# 9. Monitor error rates for 15 minutes
# 10. If healthy, shift 100%
gcloud run services update-traffic api-core --to-latest
```

### Rollback

```bash
# Instant rollback to previous revision
gcloud run services update-traffic api-core \
  --to-revisions PREVIOUS_REVISION=100 \
  --region asia-southeast1 \
  --project agrifield-prod
```

---

## 10. Mobile App Deployment

### iOS

```bash
# Build and submit via Expo EAS
eas build --platform ios --profile production
eas submit --platform ios
```

### Android

```bash
eas build --platform android --profile production
eas submit --platform android
```

### OTA Updates (Expo Updates)

For non-native changes (JS bundle only), use Expo OTA updates — no app store submission required:

```bash
eas update --branch production --message "v1.2.0 — sync engine improvements"
```

---

## 11. Infrastructure as Code

All infrastructure is defined as code using **Terraform**:

```
infra/
├── modules/
│   ├── cloud-run/
│   ├── cloud-sql/
│   ├── cloud-storage/
│   └── networking/
├── environments/
│   ├── dev/
│   │   └── main.tf
│   ├── staging/
│   │   └── main.tf
│   └── prod/
│       └── main.tf
└── README.md
```

Terraform state stored in GCS bucket per environment. Applied via Cloud Build on infrastructure changes.

---

*AgriField — Commodity Intelligence Platform | Confidential — Internal Use Only*