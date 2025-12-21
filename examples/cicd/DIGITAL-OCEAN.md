# Digital Ocean Deployment Configuration

## Overview

Digital Ocean deployment configurations for GitHub Actions. Includes App Platform, Droplet (SSH), and Kubernetes (DOKS) deployment patterns.

## Metadata

| Property | Value |
|----------|-------|
| **Example Version** | 1.0.0 |
| **Last Updated** | 2025-01 |
| **doctl** | Latest |
| **Stacks** | All (TALL, Django, React, Mobile, Shared-Lib) |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [App Platform Configuration](#app-platform-configuration)
- [Droplet Deployment (SSH)](#droplet-deployment-ssh)
- [Kubernetes (DOKS) Deployment](#kubernetes-doks-deployment)
- [Required Secrets](#required-secrets)


## App Platform Configuration

### .do/app.yaml

```yaml
# Digital Ocean App Platform Configuration
# Defines the application structure for automated deployments

name: my-app

# Web service (main application)
services:
  - name: web
    github:
      repo: username/repo
      branch: main
      deploy_on_push: true
    build_command: npm run build
    run_command: npm start
    http_port: 3000
    instance_count: 2
    instance_size_slug: basic-xxs
    routes:
      - path: /
    envs:
      - key: NODE_ENV
        value: production
      - key: DATABASE_URL
        type: SECRET
        value: ${db.DATABASE_URL}
      - key: REDIS_URL
        type: SECRET
        value: ${redis.REDIS_URL}
    health_check:
      http_path: /health
      initial_delay_seconds: 10
      period_seconds: 10

# Background worker
workers:
  - name: worker
    github:
      repo: username/repo
      branch: main
      deploy_on_push: true
    build_command: npm run build
    run_command: npm run worker
    instance_count: 1
    instance_size_slug: basic-xxs
    envs:
      - key: NODE_ENV
        value: production
      - key: DATABASE_URL
        type: SECRET
        value: ${db.DATABASE_URL}

# Database
databases:
  - name: db
    engine: PG
    version: "15"
    size: db-s-1vcpu-1gb
    num_nodes: 1

# Static site (if separate frontend)
static_sites:
  - name: frontend
    github:
      repo: username/frontend-repo
      branch: main
      deploy_on_push: true
    build_command: npm run build
    output_dir: dist
    routes:
      - path: /app
    envs:
      - key: VITE_API_URL
        value: ${web.PUBLIC_URL}
```

### .github/workflows/deploy-do-app.yml

```yaml
# App Platform Deployment via doctl
# For more control than automatic GitHub deploys

name: Deploy to App Platform

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    name: Deploy to App Platform
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install doctl
        uses: digitalocean/action-doctl@v2
        with:
          token: ${{ secrets.DIGITALOCEAN_ACCESS_TOKEN }}

      - name: Deploy to App Platform
        run: |
          doctl apps create-deployment ${{ vars.APP_ID }} --wait
```

---

## Droplet Deployment (SSH)

### .github/workflows/deploy-droplet.yml

```yaml
# Droplet Deployment via SSH
# Deploys to a Digital Ocean Droplet using SSH

name: Deploy to Droplet

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    name: Deploy to Droplet
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.DROPLET_HOST }}
          username: ${{ secrets.DROPLET_USER }}
          key: ${{ secrets.DROPLET_SSH_KEY }}
          script: |
            cd /var/www/app

            # Put application in maintenance mode
            php artisan down || true

            # Pull latest changes
            git fetch origin main
            git reset --hard origin/main

            # Install dependencies
            composer install --no-dev --optimize-autoloader

            # Run migrations
            php artisan migrate --force

            # Clear and cache configurations
            php artisan config:cache
            php artisan route:cache
            php artisan view:cache

            # Build frontend assets
            npm ci
            npm run build

            # Restart queue workers
            php artisan queue:restart

            # Bring application back up
            php artisan up

            echo "Deployment complete!"
```

### .github/workflows/deploy-droplet-docker.yml

```yaml
# Droplet Docker Deployment
# Deploys Docker containers to a Droplet

name: Deploy Docker to Droplet

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    name: Deploy Docker to Droplet
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.DROPLET_HOST }}
          username: ${{ secrets.DROPLET_USER }}
          key: ${{ secrets.DROPLET_SSH_KEY }}
          script: |
            cd /var/www/app

            # Pull latest changes
            git fetch origin main
            git reset --hard origin/main

            # Pull new Docker images
            docker-compose pull

            # Restart containers with zero downtime
            docker-compose up -d --remove-orphans

            # Clean up old images
            docker image prune -f

            echo "Docker deployment complete!"
```

---

## Kubernetes (DOKS) Deployment

### .github/workflows/deploy-doks.yml

```yaml
# DOKS (Digital Ocean Kubernetes) Deployment
# Deploys to Digital Ocean Kubernetes cluster

name: Deploy to DOKS

on:
  push:
    branches: [main]
  workflow_dispatch:

env:
  REGISTRY: registry.digitalocean.com
  IMAGE_NAME: my-app

jobs:
  build:
    name: Build and Push Image
    runs-on: ubuntu-latest
    outputs:
      image: ${{ steps.build.outputs.image }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install doctl
        uses: digitalocean/action-doctl@v2
        with:
          token: ${{ secrets.DIGITALOCEAN_ACCESS_TOKEN }}

      - name: Login to DO Container Registry
        run: doctl registry login --expiry-seconds 600

      - name: Build and push image
        id: build
        run: |
          IMAGE_TAG=${{ env.REGISTRY }}/${{ vars.REGISTRY_NAME }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          docker build -t $IMAGE_TAG .
          docker push $IMAGE_TAG
          echo "image=$IMAGE_TAG" >> $GITHUB_OUTPUT

  deploy:
    name: Deploy to Kubernetes
    runs-on: ubuntu-latest
    needs: build
    environment: production

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install doctl
        uses: digitalocean/action-doctl@v2
        with:
          token: ${{ secrets.DIGITALOCEAN_ACCESS_TOKEN }}

      - name: Save DigitalOcean kubeconfig
        run: doctl kubernetes cluster kubeconfig save ${{ vars.CLUSTER_NAME }}

      - name: Update Kubernetes manifests
        run: |
          sed -i "s|IMAGE_TAG|${{ needs.build.outputs.image }}|g" k8s/deployment.yaml

      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f k8s/
          kubectl rollout status deployment/my-app --timeout=300s

      - name: Verify deployment
        run: |
          kubectl get pods -l app=my-app
          kubectl get services -l app=my-app
```

### k8s/deployment.yaml

```yaml
# Kubernetes Deployment for DOKS

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: IMAGE_TAG
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: production
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: database-url
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "200m"
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 3000
  selector:
    app: my-app
```

---

## Required Secrets

### GitHub Repository Secrets

| Secret Name | Description | How to Get |
|-------------|-------------|------------|
| `DIGITALOCEAN_ACCESS_TOKEN` | DO API token | DO Console > API > Tokens |
| `DROPLET_SSH_KEY` | SSH private key | Generate with `ssh-keygen` |
| `DROPLET_HOST` | Droplet IP or hostname | DO Console > Droplets |
| `DROPLET_USER` | SSH username | Usually `root` or custom user |

### GitHub Environment Variables

| Variable Name | Description | Example |
|---------------|-------------|---------|
| `APP_ID` | App Platform app ID | `12345678-1234-1234-1234-123456789012` |
| `CLUSTER_NAME` | DOKS cluster name | `my-cluster-production` |
| `REGISTRY_NAME` | Container registry name | `my-registry` |
