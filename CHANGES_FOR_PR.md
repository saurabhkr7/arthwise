Changes prepared for PR branch `deploy/cloudrun-arthwise-api`:

Modified:
- ArthwiseServices/nse_service/dashboardData/index.js (persist snapshot to Redis + snapshot fallback)
- ArthwiseServices/index.js (wire nse snapshot endpoint)

Added:
- ArthwiseServices/routes/nseSnapshot.js (POST /api/nse-snapshot to save snapshot to Redis)
- cloudrun/arthwise-api.yaml (Cloud Run manifest example)
- cloudbuild.yaml (Cloud Build pipeline to build & push image)
- DEPLOYMENT_GCP.md (detailed deployment steps and commands)

Next steps to create PR locally (run in repo root):

```bash
# initialize git if needed
git init
git remote add origin <your-remote-url>
# create branch and commit
git checkout -b deploy/cloudrun-arthwise-api
git add .
git commit -m "chore: cloud run config, cloudbuild, snapshot endpoint, cache fallback"
# push branch
git push -u origin deploy/cloudrun-arthwise-api
# create PR using gh
gh pr create --title "Deploy: Cloud Run arthwise-api + cache fallback" --body "Adds Cloud Run config, Cloud Build pipeline, cache-fallback in dashboardData, and snapshot endpoint for NSE snapshots. See DEPLOYMENT_GCP.md for deployment steps." --base main
```

If your default branch is `master` replace `--base main` with `--base master`.

Notes:
- Do NOT commit secrets.
- Ensure `GOOGLE_APPLICATION_CREDENTIALS` is set for deployments and run `gcloud auth activate-service-account` as needed.
