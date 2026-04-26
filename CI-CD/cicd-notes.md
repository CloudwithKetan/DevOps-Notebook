# 🔄 CI/CD Pipelines — Complete Notes



![CI/CD Pipeline Flow](images/cicd-pipeline-flow.svg)

> CI/CD stands for Continuous Integration and Continuous Delivery/Deployment. It's the practice of automatically building, testing, and deploying code every time a developer pushes a change — eliminating manual steps and catching bugs early.

---

## 1. What is CI/CD?

### Continuous Integration (CI)

**CI** means every developer merges their code into a shared branch frequently (multiple times a day). Each merge automatically triggers:

1. Build the application
2. Run unit tests
3. Run integration tests
4. Check code quality
5. Report back pass/fail

The goal: **find bugs within minutes of writing them**, not days later when multiple changes have stacked up.

### Continuous Delivery (CD)

**Continuous Delivery** means after CI passes, the build artifact is automatically deployed to a staging environment — always in a deployable state. A human still approves the final push to production.

### Continuous Deployment

**Continuous Deployment** goes one step further — every change that passes all tests is automatically deployed to production without any human approval. Companies like Netflix and Amazon do thousands of deployments per day this way.

```
Developer pushes code
         ↓
    Source Control (Git)
         ↓
    CI: Build + Test
         ↓
    Artifact (Docker image, jar, zip)
         ↓
    Deploy to Staging (CD)
         ↓
    Integration Tests on Staging
         ↓
    Manual Approval (Delivery) or Auto (Deployment)
         ↓
    Deploy to Production
```

---

## 2. GitHub Actions

GitHub Actions is GitHub's built-in CI/CD system. It's free for public repos and extremely easy to integrate since your code already lives in GitHub.

### How it works

- Workflows are YAML files stored in `.github/workflows/`
- Each workflow is triggered by **events** (push, pull_request, schedule, etc.)
- A workflow contains **jobs** that run on **runners** (GitHub-managed VMs or your own)
- Each job contains **steps** (individual commands or reusable actions)

### Anatomy of a workflow

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:                              # Trigger events
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:                             # Global environment variables
  NODE_VERSION: '20'

jobs:
  build-and-test:
    name: Build and Test
    runs-on: ubuntu-latest       # GitHub-hosted runner

    steps:
      # Step 1: Get the code
      - name: Checkout code
        uses: actions/checkout@v4

      # Step 2: Set up Node.js
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'           # Cache node_modules for speed

      # Step 3: Install dependencies
      - name: Install dependencies
        run: npm ci              # ci = clean install (faster, reproducible)

      # Step 4: Run tests
      - name: Run tests
        run: npm test

      # Step 5: Build
      - name: Build application
        run: npm run build

      # Step 6: Upload build artifact
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/
```

### Docker Build and Push to Docker Hub

```yaml
# .github/workflows/docker.yml
name: Build and Push Docker Image

on:
  push:
    branches: [main]
    tags: ['v*.*.*']        # Also trigger on version tags

jobs:
  docker:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}   # Stored as repo secret

      - name: Extract metadata for Docker
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: myusername/myapp
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha    # Use GitHub Actions cache
          cache-to: type=gha,mode=max
```

### Secrets Management in GitHub Actions

Never put passwords, tokens, or API keys in your YAML files. Store them in **GitHub Secrets**:

1. Go to: Repo → Settings → Secrets and variables → Actions
2. Add: `DOCKERHUB_TOKEN`, `AWS_ACCESS_KEY_ID`, etc.
3. Reference in workflow: `${{ secrets.DOCKERHUB_TOKEN }}`

Secrets are encrypted, never appear in logs, and are only available to workflows in that repo.

### Matrix Strategy — Test on Multiple Versions

```yaml
jobs:
  test:
    strategy:
      matrix:
        python-version: ['3.10', '3.11', '3.12']
        os: [ubuntu-latest, windows-latest]
    
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      - run: pip install pytest && pytest
```

This creates 6 jobs automatically (3 Python versions × 2 OS). If any fails, you know exactly which combination broke.

---

## 3. Jenkins

Jenkins is the older, battle-tested CI/CD system. It's self-hosted (you run your own server), highly customizable, and used by many enterprise teams. Its power comes from its massive plugin ecosystem.

### Jenkinsfile (Pipeline as Code)

Jenkins reads a `Jenkinsfile` from your repo to define the pipeline:

```groovy
pipeline {
    agent any           // Run on any available agent
    
    environment {
        APP_NAME = 'myapp'
        DOCKER_REGISTRY = 'myregistry.azurecr.io'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/myorg/myapp.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar'
                }
            }
        }

        stage('Test') {
            parallel {                // Run these test stages simultaneously
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test'
                    }
                    post {
                        always {
                            junit 'target/surefire-reports/*.xml'
                        }
                    }
                }
                stage('Code Quality') {
                    steps {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    def image = docker.build("${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}")
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'acr-credentials') {
                        image.push()
                        image.push('latest')
                    }
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                sh """
                    kubectl set image deployment/${APP_NAME} \
                    ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER} \
                    --namespace=staging
                """
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'      // Only deploy prod from main branch
            }
            input {
                message 'Deploy to production?'
                ok 'Yes, deploy!'
            }
            steps {
                sh """
                    kubectl set image deployment/${APP_NAME} \
                    ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER} \
                    --namespace=production
                """
            }
        }
    }

    post {
        success {
            slackSend channel: '#deployments',
                      message: "✅ ${APP_NAME} v${BUILD_NUMBER} deployed successfully"
        }
        failure {
            slackSend channel: '#deployments',
                      message: "❌ ${APP_NAME} build ${BUILD_NUMBER} FAILED"
            mail to: 'team@company.com',
                 subject: "Build Failed: ${APP_NAME}",
                 body: "Check Jenkins: ${BUILD_URL}"
        }
    }
}
```

---

## 4. GitLab CI/CD

GitLab has CI/CD built in — no separate tool needed. Pipelines are defined in `.gitlab-ci.yml` at the root of the repo.

```yaml
# .gitlab-ci.yml
image: docker:24         # Default Docker image for jobs

stages:
  - build
  - test
  - security
  - deploy

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

# ── Build Stage ──────────────────────────────────────────
build-image:
  stage: build
  services:
    - docker:24-dind    # Docker-in-Docker for building images
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE
  only:
    - main
    - merge_requests

# ── Test Stage ───────────────────────────────────────────
unit-tests:
  stage: test
  image: python:3.11
  before_script:
    - pip install -r requirements.txt
  script:
    - pytest tests/unit/ --junitxml=report.xml
  artifacts:
    reports:
      junit: report.xml     # GitLab parses this and shows test results in MR
    expire_in: 1 week

integration-tests:
  stage: test
  image: python:3.11
  services:
    - postgres:15           # Spin up a DB for integration tests
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: test
    POSTGRES_PASSWORD: test
    DATABASE_URL: postgresql://test:test@postgres/testdb
  script:
    - pip install -r requirements.txt
    - pytest tests/integration/

# ── Security Stage ───────────────────────────────────────
container-scan:
  stage: security
  script:
    - trivy image --exit-code 1 --severity HIGH,CRITICAL $DOCKER_IMAGE

# ── Deploy Stage ─────────────────────────────────────────
deploy-staging:
  stage: deploy
  environment:
    name: staging
    url: https://staging.myapp.com
  script:
    - kubectl set image deployment/myapp myapp=$DOCKER_IMAGE -n staging
  only:
    - main

deploy-production:
  stage: deploy
  environment:
    name: production
    url: https://myapp.com
  script:
    - kubectl set image deployment/myapp myapp=$DOCKER_IMAGE -n production
  when: manual           # Requires someone to click "Deploy" in GitLab UI
  only:
    - main
```

**GitLab predefined variables** — GitLab provides useful variables automatically:
- `$CI_REGISTRY` — GitLab Container Registry URL
- `$CI_COMMIT_SHORT_SHA` — Short git commit hash (great for image tags)
- `$CI_PROJECT_NAME` — Your project name
- `$CI_PIPELINE_ID` — Unique pipeline ID

---

## 5. ArgoCD — GitOps for Kubernetes

ArgoCD takes a different approach called **GitOps**: your Git repo is the single source of truth for what should be running in Kubernetes. ArgoCD watches your repo and automatically syncs the cluster to match.

```
Git repo (desired state)
      ↕ ArgoCD watches and syncs
Kubernetes cluster (actual state)
```

If someone manually changes something in the cluster (bypassing Git), ArgoCD detects the drift and either alerts you or automatically reverts it.

```yaml
# argocd-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-webapp
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/myorg/k8s-manifests
    targetRevision: main        # Watch the main branch
    path: apps/webapp           # Folder containing K8s YAML files

  destination:
    server: https://kubernetes.default.svc
    namespace: production

  syncPolicy:
    automated:
      prune: true               # Delete resources removed from Git
      selfHeal: true            # Revert manual changes in cluster
    syncOptions:
      - CreateNamespace=true
```

**Typical GitOps workflow with ArgoCD:**

1. Developer pushes code → CI pipeline builds a new Docker image → tags it `v1.5.2`
2. CI pipeline updates the image tag in a Kubernetes manifest YAML in a separate `k8s-manifests` Git repo
3. ArgoCD detects the change in the manifests repo
4. ArgoCD automatically applies the updated manifest to the cluster
5. New pods with image `v1.5.2` roll out

This keeps the Git repo as the audit trail for what's running where, and means no one ever needs direct `kubectl apply` access to production.

---

## 6. CI/CD Best Practices

**Fail fast** — run fast checks first (linting, unit tests) before slower ones (integration tests, builds). If linting fails, don't waste time building.

**Keep pipelines fast** — slow pipelines kill developer productivity. Target under 10 minutes for the main CI pipeline. Use caching, parallelism, and test splitting.

**Use branch protection** — require CI to pass before merging to main. Never allow direct pushes to main.

**Separate build from deploy** — build once, deploy many times. The same artifact (Docker image) should go from staging to production unchanged.

**Immutable artifacts** — never rebuild or overwrite an artifact. Tag images with git commit SHAs, not `latest`. This ensures what you test is what you deploy.

**Environment-specific config via secrets/env vars** — the image is the same in staging and production. Only config (DB URLs, API keys) changes, injected via secrets.

**Rollback strategy** — always know how to roll back. For Kubernetes: `kubectl rollout undo deployment/myapp`. For GitOps: revert the Git commit in the manifests repo.

---

## 7. GitHub Actions vs Jenkins vs GitLab CI

| Feature | GitHub Actions | Jenkins | GitLab CI |
|---------|---------------|---------|-----------|
| Hosting | Cloud (GitHub) | Self-hosted | Cloud or self-hosted |
| Setup | Zero | Heavy | Zero (built into GitLab) |
| Cost | Free for public repos | Server cost only | Free tier available |
| Config | YAML | Groovy (Jenkinsfile) | YAML |
| Ecosystem | GitHub Marketplace | 1800+ plugins | Built-in (registry, security) |
| Best for | GitHub repos | Enterprise, complex workflows | Full DevOps lifecycle |

---

*CI/CD is the heartbeat of DevOps — once you set it up, every code push is automatically built, tested, and ready to ship.*
