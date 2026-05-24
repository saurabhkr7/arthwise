# GCP Deployment Checklist — arthwise-api (Cloud Run)

This file documents exact steps, commands, and variables to deploy `arthwise-api` to Cloud Run using the provided service account and credentials. Keep this file updated with deployment notes and secret names.

Prerequisites (local machine or CI)
- `gcloud` CLI installed and authenticated.
- `docker` installed (for local build/testing).
- `gh` CLI installed and authenticated (for PR operations).
- Service account JSON: `arthhwisee-7d94451f655f.json` (placed locally, DO NOT COMMIT).
- Upstash Redis credentials: store as `UPSTASH_REDIS_URL` in Secret Manager.
- MongoDB Atlas URI: stored as `MONGO_URI` in Secret Manager.

Step 0 — Authenticate with GCP using service account (local)
```bash
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/arthhwisee-7d94451f655f.json"
gcloud auth activate-service-account --key-file="$GOOGLE_APPLICATION_CREDENTIALS"
gcloud config set project arthhwisee
```

Step 1 — Enable required APIs
```bash
gcloud services enable run.googleapis.com cloudbuild.googleapis.com artifactregistry.googleapis.com secretmanager.googleapis.com iam.googleapis.com compute.googleapis.com
```

Step 2 — Create Artifact Registry repository (one-time)
```bash
gcloud artifacts repositories create arth-repo --repository-format=docker --location=asia-south1 --description="Docker images for arthwise" --project=arthhwisee
```

Step 3 — Build and push image with Cloud Build (recommended)
```bash
# From repo root
gcloud builds submit --config=cloudbuild.yaml --substitutions=_REGION=asia-south1 --project=arthhwisee
```

Step 4 — Create secrets in Secret Manager (one-time)
```bash
# Example: create secret and add value
echo -n "$MONGO_URI" | gcloud secrets create MONGO_URI --data-file=- --project=arthhwisee
echo -n "$UPSTASH_REDIS_URL" | gcloud secrets create UPSTASH_REDIS_URL --data-file=- --project=arthhwisee
# Repeat for JWT_SECRET, GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET
```

Step 5 — Deploy to Cloud Run (example using gcloud deploy)
```bash
gcloud run deploy arthwise-api \
  --image=asia-south1-docker.pkg.dev/arthhwisee/arth-repo/arthwise-api:latest \
  --region=asia-south1 \
  --project=arthhwisee \
  --platform=managed \
  --allow-unauthenticated \
  --concurrency=10 \
  --max-instances=1 \
  --set-secrets=MONGO_URI=projects/arthhwisee/secrets/MONGO_URI:latest,UPSTASH_REDIS_URL=projects/arthhwisee/secrets/UPSTASH_REDIS_URL:latest \
  --set-env-vars=USE_CACHED_NSE=false
```

Step 6 — Health check & traffic
- Verify `Service URL` returned by `gcloud run deploy`.
- Curl `/healthz` endpoint and login endpoints to verify.

Step 7 — Local smoke test (docker)
```bash
# Build locally
docker build -t arthwise-api:local -f ArthwiseServices/Dockerfile .
# Run with env-file containing MONGO_URI and UPSTASH_REDIS_URL
docker run --env-file ./local.env -p 3000:3000 arthwise-api:local
```

Step 8 — DNS (Cloudflare) & domain
- Once Cloud Run URL is stable, map custom domain `www.arthhwise.com` to Cloud Run service via Cloud Console or `gcloud` domain-mapping commands.
- Update Cloudflare DNS records to point to the Cloud Run mapping as instructed by the Console.

Step 9 — Rollback
- Redeploy the previous image tag or route traffic back to the PM2-hosted API by updating DNS or Cloud Run traffic splits.

Checklist for PR
- [ ] Add `cloudrun/arthwise-api.yaml` with example settings
- [ ] Add `cloudbuild.yaml` (optional) to build & push image
- [ ] Add cache-fallback code changes + unit tests
- [ ] Add `nse-service` snapshot endpoint example and docs for PM2 cron
- [ ] Add instructions for creating secrets and required IAM roles

Security notes
- Never commit `arthhwisee-7d94451f655f.json`, Upstash keys, Mongo URIs, or OAuth secrets.
- Use Secret Manager and grant least privilege to Cloud Run service account.

