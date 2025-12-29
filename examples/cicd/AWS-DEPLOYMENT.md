# AWS Deployment Configuration

**Last Updated**: 29/12/2025
**Version**: 1.3.1
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Overview

Comprehensive AWS deployment configurations for GitHub Actions across all supported stacks. Includes ECS (containerised), S3/CloudFront (static), and Lambda (serverless) deployment patterns optimised for production use.

All examples follow best practices for security, caching, and performance in the AWS ecosystem.

## Metadata

| Property | Value |
|----------|-------|
| **Example Version** | 2.0.0 |
| **Last Updated** | 2025-12 |
| **AWS Actions** | v4 |
| **Stacks** | TALL (Laravel 12.x/PHP 8.4), Django 6.x/Python 3.14, Next.js 16.x |

---

## Table of Contents

- [AWS Deployment Configuration](#aws-deployment-configuration)
  - [Overview](#overview)
  - [Metadata](#metadata)
  - [Table of Contents](#table-of-contents)
  - [ECS Deployment](#ecs-deployment)
    - [.github/workflows/deploy-ecs.yml](#githubworkflowsdeploy-ecsyml)
    - [Dockerfile for ECS](#dockerfile-for-ecs)
  - [S3 + CloudFront (Static Sites)](#s3--cloudfront-static-sites)
    - [.github/workflows/deploy-s3.yml](#githubworkflowsdeploy-s3yml)
  - [Lambda Deployment](#lambda-deployment)
    - [.github/workflows/deploy-lambda.yml](#githubworkflowsdeploy-lambdayml)
  - [Required AWS Secrets](#required-aws-secrets)
    - [GitHub Repository Secrets](#github-repository-secrets)
    - [GitHub Environment Variables](#github-environment-variables)
    - [Required IAM Permissions](#required-iam-permissions)


## ECS Deployment

### .github/workflows/deploy-ecs.yml

```yaml
# ECS Deployment
# Builds Docker image, pushes to ECR, and deploys to ECS

name: Deploy to ECS

on:
  push:
    branches: [main]
  workflow_dispatch:

env:
  AWS_REGION: eu-west-2
  ECR_REPOSITORY: my-app
  ECS_SERVICE: my-app-service
  ECS_CLUSTER: my-app-cluster
  CONTAINER_NAME: my-app

jobs:
  deploy:
    name: Deploy to ECS
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build, tag, and push image to Amazon ECR
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

      - name: Download task definition
        run: |
          aws ecs describe-task-definition \
            --task-definition ${{ env.ECS_SERVICE }} \
            --query taskDefinition > task-definition.json

      - name: Fill in the new image ID in the Amazon ECS task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-definition.json
          container-name: ${{ env.CONTAINER_NAME }}
          image: ${{ steps.build-image.outputs.image }}

      - name: Deploy Amazon ECS task definition
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true
```

### Dockerfile for ECS

```dockerfile
# Multi-stage Dockerfile for ECS deployment
# Optimised for production with minimal image size

# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Production stage
FROM node:20-alpine AS production
WORKDIR /app

# Create non-root user for security
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

# Copy built application
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./

USER nodejs

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "dist/main.js"]
```

---

## S3 + CloudFront (Static Sites)

### .github/workflows/deploy-s3.yml

```yaml
# S3 + CloudFront Deployment
# Deploys static site to S3 and invalidates CloudFront cache

name: Deploy to S3

on:
  push:
    branches: [main]
  workflow_dispatch:

env:
  AWS_REGION: eu-west-2

jobs:
  deploy:
    name: Deploy to S3
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build static site
        run: npm run build
        env:
          NODE_ENV: production
          VITE_API_URL: ${{ vars.API_URL }}

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Deploy to S3
        run: |
          aws s3 sync ./dist s3://${{ vars.S3_BUCKET }} \
            --delete \
            --cache-control "public, max-age=31536000, immutable" \
            --exclude "index.html" \
            --exclude "*.json"

          # Upload index.html and JSON with no-cache
          aws s3 cp ./dist/index.html s3://${{ vars.S3_BUCKET }}/index.html \
            --cache-control "no-cache, no-store, must-revalidate"

      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ vars.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

---

## Lambda Deployment

### .github/workflows/deploy-lambda.yml

```yaml
# Lambda Deployment
# Packages and deploys serverless function to AWS Lambda

name: Deploy to Lambda

on:
  push:
    branches: [main]
  workflow_dispatch:

env:
  AWS_REGION: eu-west-2
  LAMBDA_FUNCTION_NAME: my-lambda-function

jobs:
  deploy:
    name: Deploy to Lambda
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci --only=production

      - name: Build application
        run: npm run build

      - name: Package Lambda function
        run: |
          cd dist
          zip -r ../function.zip .
          cd ..
          zip -ur function.zip node_modules

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Deploy to Lambda
        run: |
          aws lambda update-function-code \
            --function-name ${{ env.LAMBDA_FUNCTION_NAME }} \
            --zip-file fileb://function.zip

      - name: Wait for update to complete
        run: |
          aws lambda wait function-updated \
            --function-name ${{ env.LAMBDA_FUNCTION_NAME }}

      - name: Publish new version
        run: |
          aws lambda publish-version \
            --function-name ${{ env.LAMBDA_FUNCTION_NAME }} \
            --description "Deployed from commit ${{ github.sha }}"
```

---

## Required AWS Secrets

### GitHub Repository Secrets

| Secret Name | Description | How to Get |
|-------------|-------------|------------|
| `AWS_ACCESS_KEY_ID` | AWS IAM access key ID | AWS IAM Console > Users > Security credentials |
| `AWS_SECRET_ACCESS_KEY` | AWS IAM secret access key | AWS IAM Console > Users > Security credentials |

### GitHub Environment Variables

| Variable Name | Description | Example |
|---------------|-------------|---------|
| `S3_BUCKET` | S3 bucket name for static hosting | `my-app-production` |
| `CLOUDFRONT_DISTRIBUTION_ID` | CloudFront distribution ID | `E1234567890ABC` |
| `API_URL` | Backend API URL | `https://api.example.com` |

### Required IAM Permissions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:PutImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ecs:UpdateService",
        "ecs:DescribeServices",
        "ecs:DescribeTaskDefinition",
        "ecs:RegisterTaskDefinition"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "cloudfront:CreateInvalidation"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "lambda:UpdateFunctionCode",
        "lambda:PublishVersion",
        "lambda:GetFunction"
      ],
      "Resource": "arn:aws:lambda:*:*:function:my-function"
    }
  ]
}
```
