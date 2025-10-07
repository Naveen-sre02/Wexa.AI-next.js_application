# Wexa.AI-next.js_application
#DevOps_internship-assessment
# Next.js Containerization & Minikube Deployment

This repo demonstrates containerizing a Next.js app, building & publishing Docker images via GitHub Actions to GitHub Container Registry (GHCR), and deploying to Kubernetes (Minikube).

## What’s included
- `Dockerfile` — multi-stage production build
- `.github/workflows/ci.yml` — build & push to GHCR on push to `main`
- `k8s/deployment.yaml`, `k8s/service.yaml` — Kubernetes manifests
- `.dockerignore`

## Setup instructions

### 1. Local run (dev)
```bash
npm install
npm run dev
# open http://localhost:3000
2. Build production locally (optional)
bash
Copy code
npm ci
npm run build
npm start
3. Build and run with Docker (local)
bash
Copy code
docker build -t ghcr.io/<GH_OWNER>/<REPO>:latest .
docker run -p 3000:3000 ghcr.io/<GH_OWNER>/<REPO>:latest
# open http://localhost:3000
4. GitHub Actions / GHCR
On push to main, GitHub Actions workflow will:

build the Docker image

push it to ghcr.io/<OWNER>/<REPO>:<sha> and :latest

Secrets / permissions

The workflow uses ${{ secrets.GITHUB_TOKEN }}. Ensure package permissions are enabled so GITHUB_TOKEN can write to GHCR. Alternatively create a PAT (GHCR_PAT) with write:packages and update workflow.

5. Deploy to Minikube (two methods)
A. Pull from GHCR

Start minikube: minikube start

If your GHCR image is private, create imagePullSecret:

bash
Copy code
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=<GH_USERNAME> \
  --docker-password=<PERSONAL_ACCESS_TOKEN> \
  --docker-email=<EMAIL>
Update k8s/deployment.yaml image to ghcr.io/<OWNER>/<REPO>:latest and ensure imagePullSecrets references ghcr-secret.

kubectl apply -f k8s/

minikube service nextjs-service --url

B. Load local image into Minikube

minikube start

docker build -t ghcr.io/<OWNER>/<REPO>:latest .

minikube image load ghcr.io/<OWNER>/<REPO>:latest

kubectl apply -f k8s/

minikube service nextjs-service --url

How to access the deployed app
If using NodePort service: minikube service nextjs-service --url will show URL for access.

If using LoadBalancer type and minikube tunnel, use minikube tunnel to get the external IP.

Troubleshooting
Pod not starting: kubectl logs <pod> and kubectl describe pod <pod> for events.

ImagePullBackOff: verify image name, repository permissions, and imagePullSecret.

Next.js 404 on healthchecks: ensure readinessProbe path / returns 200. If your app routes differently, adjust probe path.

Files to submit
Source code (Next.js app)

Dockerfile, .dockerignore

.github/workflows/ci.yml

k8s/deployment.yaml, k8s/service.yaml

README.md with setup & deployment steps

markdown
Copy code

---

# 8) Extra tips & checklist for submission
- Make repo **PUBLIC** (assignment asked for public).
- Push all code and files to `main`.
- Ensure the Docker image name in `deployment.yaml` points to the image published by GHCR (replace placeholders).
- If you used a PAT anywhere, do **not** commit it — store in GitHub secrets only.
- In GitHub repo Settings → Actions → General, make sure Workflow permissions allow `packages: write` for `GITHUB_TOKEN` (or use `GHCR_PAT`).
- Add short paragraph in README describing how to access the running app (minikube service url) and the steps you used.

---

If you want, I can:
- generate a ready-to-use `ci.yml` with a PAT-based login instead of `GITHUB_TOKEN`,
- adapt Dockerfile if you use `pnpm` or monorepo,
- produce a script that automates minikube deployment steps (`deploy.sh`),
- or paste every file into a single zip-ready structure.
