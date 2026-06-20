# Jenkins Interview Questions & Answers
> **Format:** Direct, flowing answers — no robotic labels  
> **Project Context:** B2B Medpharm Platform (used in scenario-based answers)  
> **Covers:** Basics · Architecture · Pipelines · Plugins · Security · CI/CD · Scenario-Based

---

## 🟢 SECTION 1: BASICS & CORE CONCEPTS

---

### Q1. What is Jenkins and what is its primary purpose?

Jenkins is an open-source automation server written in Java that eliminates manual effort from the software delivery process. Every time a developer pushes code, Jenkins automatically picks it up, compiles it, runs tests, performs security scans, builds a Docker image, and deploys it — without anyone manually triggering anything.

Before Jenkins existed, teams would SSH into servers, run build commands manually, copy artifacts, and restart services by hand. That process was slow, inconsistent, and broke constantly across environments. Jenkins solves that by making the entire delivery pipeline repeatable, auditable, and fully automated through something called a **Jenkinsfile** — a Pipeline as Code definition that lives right inside the repository alongside the application code.

It connects with 1800+ plugins covering Git, Docker, Kubernetes, SonarQube, AWS, Slack, Terraform — essentially any tool in the DevOps ecosystem. And since it's open-source, there are no licensing costs, which makes it the most widely adopted CI/CD tool across the industry.

---

### Q2. What is the difference between CI, CD (Delivery), and CD (Deployment)?

These three terms get used interchangeably but they represent three very different levels of automation in the delivery process.

**Continuous Integration** is the practice of automatically building and testing code every time a developer commits. The goal is simple — catch bugs early before they accumulate. Jenkins pulls the latest code, compiles it, runs unit tests, and reports back within minutes.

**Continuous Delivery** goes one step further. After the build passes all tests, Jenkins automatically packages the application and pushes it to a staging environment — but the actual production deployment still requires a manual approval. This is the most common pattern in regulated industries where someone needs to sign off before anything goes live.

**Continuous Deployment** removes that manual gate entirely. Every commit that passes all automated checks goes straight to production with zero human intervention. This requires extremely mature test coverage and monitoring.

```
Developer pushes code
        ↓
CI  →  Build + Unit Tests (fully automated)
        ↓
CD Delivery  →  Package + Staging Deploy (automated) → Manual Approval → Production
        ↓
CD Deployment  →  Package + Staging + Production (fully automated, no manual step)
```

---

### Q3. What is Jenkins Architecture?

Jenkins follows a **Controller-Agent** architecture, and understanding this split is important because it directly affects performance, security, and scalability.

The **Controller** (previously called Master) is the brain — it hosts the Jenkins UI, schedules jobs, stores build history and configuration, manages plugins, and coordinates all the agents. One important thing to note here is that the Controller should never run actual build jobs in production. Running builds on the controller is a security and performance anti-pattern.

The **Agents** (also called Nodes or Workers) are where the actual work happens. They receive job assignments from the Controller and execute pipeline stages. Agents can be permanent machines always connected via SSH, or ephemeral — spun up on demand using Docker, Kubernetes pods, or EC2 instances and torn down after the build completes.

Each agent carries labels like `linux`, `docker`, or `java17`, and jobs are routed to agents based on those labels. This means you can have dedicated agents for specific workloads — one for Java builds, another for Docker operations, another for deployments.

```
Controller ──schedules──→ Agent 1 (Linux / Java builds)
           ──schedules──→ Agent 2 (Docker operations)
           ──schedules──→ Agent 3 (Kubernetes deployments)
```

---

### Q4. What is a Jenkins Job / Project?

A Jenkins Job is the fundamental unit of work in Jenkins — it defines what to build, how to build it, when to trigger it, and what to do with the result.

Over the years Jenkins has evolved several job types:

| Type | Description |
|---|---|
| **Freestyle Project** | GUI-configured builds. Legacy approach, no version control. |
| **Pipeline** | Jenkinsfile-based. Modern, recommended approach. |
| **Multibranch Pipeline** | Auto-creates pipelines for every branch with a Jenkinsfile. |
| **Organization Folder** | Scans entire GitHub/GitLab orgs for repos with Jenkinsfiles. |
| **Multi-configuration Project** | Matrix builds — test across multiple OS/JDK versions. |
| **Folder** | Organizes jobs into groups, like directories. |

**Freestyle Projects** were the original GUI-based approach — easy to set up but impossible to version control properly. **Pipeline Jobs** are the modern standard — the entire pipeline definition lives in a Jenkinsfile inside the repository. **Multibranch Pipelines** take it further, automatically creating and deleting pipeline jobs as branches are created and deleted. **Organization Folders** work at the GitHub org level, scanning all repositories automatically.

---

### Q5. What is a Jenkinsfile?

A Jenkinsfile is a text file written in Groovy DSL that contains the complete definition of a Jenkins Pipeline. It lives in the root of the source code repository, which means it travels with the code — every branch has its own version of the pipeline, every change to the pipeline goes through code review, and you always know exactly what ran when you look at a historical build.

This concept is called **Pipeline as Code**, and it solves a major problem that existed with Freestyle jobs — pipelines defined only in the Jenkins UI can't be versioned, reviewed, or audited properly.

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                sh './deploy.sh'
            }
        }
    }
}
```

---

### Q6. What is the difference between Declarative and Scripted Pipeline?

Both are ways to write a Jenkinsfile, but they have very different philosophies.

**Declarative Pipeline** was introduced in Jenkins 2.x and follows a structured, opinionated syntax. It has pre-defined blocks like `pipeline`, `stages`, `steps`, `post`, `environment`, and `when`. Because of this structure, Jenkins can validate it before execution, provide better error messages, and offer features like `Restart from Stage`. It's easier to read, easier to maintain, and the right choice for most use cases.

**Scripted Pipeline** is the original approach — it's essentially raw Groovy code running inside a `node {}` block. It gives you complete flexibility and full access to the Groovy language, but it's harder to read, validation only happens at runtime, and error messages are far less helpful.

| | Declarative | Scripted |
|---|---|---|
| Syntax | Structured | Full Groovy |
| Validation | Pre-flight | Runtime only |
| Error messages | Descriptive | Generic |
| Recommended for | Most pipelines | Complex dynamic logic |

```groovy
// Declarative (recommended)
pipeline {
    agent any
    environment { APP_VERSION = '1.0.0' }
    stages {
        stage('Build') {
            steps { sh 'make build' }
        }
    }
    post {
        always { cleanWs() }
        failure { mail to: 'team@example.com', subject: 'Build Failed' }
    }
}

// Scripted (full Groovy)
node {
    try {
        stage('Build') { sh 'make build' }
    } catch (e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        cleanWs()
    }
}
```

---

### Q7. What are Jenkins Agents and how do you configure them?

Agents are the worker machines that actually execute pipeline stages. The Controller schedules the work, but agents do the heavy lifting — compiling code, running tests, building Docker images, deploying to Kubernetes.

You can configure agents in several ways. **Permanent agents** are machines always connected to the Controller, set up via Manage Jenkins → Nodes, and accessed over SSH. **Cloud agents** are spun up on demand — using the Kubernetes plugin, each build gets a fresh pod that's destroyed after the pipeline completes, which is the cleanest and most scalable approach. **Docker agents** run stages inside containers, giving you a clean isolated environment for every build.

```groovy
// Any available agent
agent any

// Specific label
agent { label 'linux && docker' }

// Docker container
agent {
    docker {
        image 'maven:3.9-eclipse-temurin-17'
        args '-v /root/.m2:/root/.m2'
    }
}

// Kubernetes pod
agent {
    kubernetes {
        yaml '''
        spec:
          containers:
          - name: maven
            image: maven:3.9
        '''
    }
}
```

---

### Q8. What are Jenkins Triggers?

Triggers define when a Jenkins pipeline starts automatically, without anyone clicking "Build Now."

The most important and recommended trigger is the **GitHub/GitLab Webhook**. When code is pushed, GitHub immediately sends an HTTP POST request to Jenkins, which then starts the pipeline within seconds. This is far better than polling because it's instant and doesn't waste resources.

**pollSCM** is the polling alternative — Jenkins checks the repository every N minutes to see if anything changed. It works without configuring webhooks but introduces latency and unnecessary SCM load.

**cron** triggers run pipelines on a schedule regardless of code changes — useful for nightly builds, scheduled deployments, or cleanup jobs.

```groovy
triggers {
    // Webhook-based — GitHub sends POST to: https://jenkins.example.com/github-webhook/

    // Poll every 5 minutes
    pollSCM('H/5 * * * *')

    // Nightly at 2 AM on weekdays
    cron('H 2 * * 1-5')

    // Trigger after upstream job succeeds
    upstream(upstreamProjects: 'build-job', threshold: hudson.model.Result.SUCCESS)
}
```

---

### Q9. What is a Jenkins Plugin?

Plugins are extensions that add functionality to Jenkins beyond its core capabilities. Out of the box, Jenkins is fairly minimal — plugins are what make it connect with the rest of the DevOps ecosystem. There are 1800+ plugins available, and practically every tool you'd want to integrate with has one.

| Plugin | Purpose |
|---|---|
| Git | Source code checkout from GitHub/GitLab |
| Pipeline | Jenkinsfile support |
| Docker Pipeline | Build and push Docker images inside pipelines |
| Kubernetes | Run build agents as ephemeral K8s pods |
| SonarQube Scanner | Code quality analysis integration |
| Credentials | Secure secret storage |
| Slack Notification | Build status alerts to Slack |
| Email Extension | Advanced email notifications |
| Blue Ocean | Modern pipeline visualization UI |
| Role Strategy | Fine-grained RBAC |
| OWASP Dependency Check | Security vulnerability scanning |
| Trivy | Container image scanning |

The key thing to know about plugins is that they need to be kept updated — outdated plugins are one of the most common sources of security vulnerabilities and pipeline failures in Jenkins environments.

---

### Q10. What are Jenkins Build Statuses?

Jenkins uses five build statuses to communicate the outcome of every pipeline run:

| Status | Color | Meaning |
|---|---|---|
| **SUCCESS** | Green | Every stage completed without errors |
| **UNSTABLE** | Yellow | Build ran but something isn't right — test failures or quality violations |
| **FAILURE** | Red | Hard failure — compilation error, script exited non-zero, quality gate rejection |
| **ABORTED** | Gray | Someone manually stopped the build |
| **NOT_BUILT** | Gray | Stage/pipeline was skipped due to a `when` condition |

The distinction between UNSTABLE and FAILURE matters in practice — some teams allow unstable builds to proceed to staging but block them from production.

```groovy
// Manually set build status inside pipeline
currentBuild.result = 'UNSTABLE'
currentBuild.result = 'FAILURE'

// Check current status
if (currentBuild.currentResult == 'SUCCESS') {
    sh './notify-team.sh'
}
```

---

## 🔵 SECTION 2: PIPELINE DEEP DIVE

---

### Q11. What are Pipeline stages, steps, and post sections?

These three constructs form the backbone of every Declarative Pipeline.

**Stages** are logical divisions of the pipeline — Checkout, Build, Test, Security Scan, Deploy. They represent meaningful phases of your delivery process and appear as separate columns in the Jenkins UI, making it easy to see exactly where a failure occurred.

**Steps** are the actual commands that run inside a stage — shell scripts, Maven commands, Docker operations, Kubernetes deployments. Each step is a single unit of work.

**Post** is where you define actions that run after all stages complete, regardless of the result. It has conditional blocks — `always` runs no matter what, `success` only runs on green builds, `failure` triggers on red, `unstable` on yellow.

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
                junit 'target/surefire-reports/**/*.xml'
            }
        }
        stage('Deploy') {
            when { branch 'main' }
            steps { sh './deploy.sh production' }
        }
    }
    post {
        always { cleanWs() }
        success {
            slackSend message: "✅ Build succeeded: ${env.BUILD_URL}"
        }
        failure {
            mail to: 'team@company.com',
                 subject: "Build Failed: ${env.JOB_NAME}",
                 body: "Check: ${env.BUILD_URL}"
        }
    }
}
```

---

### Q12. What is the `when` directive in Jenkins Pipeline?

The `when` directive gives you conditional control over whether a stage executes at all. Without it, every stage runs on every build regardless of context. With `when`, you can make deployment stages branch-aware, environment-aware, or parameter-driven.

```groovy
// Only deploy to production from main branch
stage('Deploy to Production') {
    when { branch 'main' }
    steps { sh './deploy.sh production' }
}

// Only run if src/ directory changed
stage('Build Backend') {
    when { changeset "src/**" }
    steps { sh 'mvn clean package' }
}

// Only run if parameter is set
stage('Run Integration Tests') {
    when { expression { params.RUN_TESTS == true } }
    steps { sh 'mvn verify' }
}

// OR condition — main or any release branch
when {
    anyOf {
        branch 'main'
        branch 'release/*'
    }
}

// NOT condition
when {
    not { branch 'develop' }
}
```

---

### Q13. What are Jenkins Environment Variables?

Environment variables in Jenkins are key-value pairs available throughout the pipeline. Jenkins provides a rich set of built-in variables automatically, and you can define custom ones at the pipeline level or scoped to individual stages.

**Built-in variables:**

```groovy
env.BUILD_NUMBER    // Current build number — used for Docker image tagging
env.BUILD_URL       // Full build URL — used in Slack/email notifications
env.GIT_COMMIT      // Current commit SHA — for deployment tracking
env.GIT_BRANCH      // Current branch name
env.JOB_NAME        // Pipeline job name
env.WORKSPACE       // Working directory on the agent
env.NODE_NAME       // Agent this build is running on
```

**Custom variables:**

```groovy
pipeline {
    environment {
        APP_NAME    = 'medpharm-api'
        IMAGE_TAG   = "${BUILD_NUMBER}"
        // Dynamic value from shell
        GIT_SHORT   = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        // From credentials store — masked in logs automatically
        DOCKER_CREDS = credentials('dockerhub-creds')
    }
    stages {
        stage('Build') {
            environment {
                MAVEN_OPTS = '-Xmx1024m'    // Stage-scoped variable
            }
            steps {
                echo "Building ${env.APP_NAME}:${env.IMAGE_TAG}"
            }
        }
    }
}
```

---

### Q14. What are Jenkins Parameters?

Parameters let you make a pipeline interactive — when someone triggers a build manually, they can provide input values that change the pipeline's behavior. This is how you build flexible deployment pipelines that work across multiple environments without maintaining separate Jenkinsfiles for each.

```groovy
pipeline {
    parameters {
        string(name: 'VERSION', defaultValue: '1.0.0', description: 'Version to deploy')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'production'], description: 'Target environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run test suite?')
        booleanParam(name: 'SKIP_SONAR', defaultValue: false, description: 'Skip SonarQube?')
        password(name: 'SECRET_TOKEN', defaultValue: '', description: 'API Token')
    }
    stages {
        stage('Deploy') {
            when {
                expression { params.ENVIRONMENT == 'production' }
            }
            steps {
                sh "./deploy.sh ${params.ENVIRONMENT} ${params.VERSION}"
            }
        }
    }
}
```

---

### Q15. What is Parallel execution in Jenkins Pipeline?

Parallel execution lets multiple stages run simultaneously on different agents, which can dramatically reduce total pipeline time. Instead of running unit tests, integration tests, and security scans one after another, you run all three at the same time.

The real-world impact is significant — a pipeline that took 45 minutes sequentially might complete in 15 minutes with proper parallelization, because the longest-running stage determines the total time rather than the sum of all stages.

```groovy
stage('Quality Checks') {
    parallel {
        stage('Unit Tests') {
            agent { label 'linux' }
            steps { sh 'mvn test' }
        }
        stage('Integration Tests') {
            agent { label 'linux' }
            steps { sh 'mvn verify -Pintegration' }
        }
        stage('OWASP Dependency Check') {
            agent { label 'linux' }
            steps { sh 'mvn dependency-check:check' }
        }
        stage('SonarQube Analysis') {
            agent { label 'linux' }
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
    }
}
```

---

### Q16. What is a Shared Library in Jenkins?

A Shared Library is a way to extract common pipeline logic into a separate Git repository and reuse it across multiple Jenkinsfiles. Without shared libraries, teams end up copying the same Docker build steps, the same SonarQube configuration, the same Slack notification logic across every single service. When something needs to change, you update it in twenty places.

With a Shared Library, you define that logic once in a centralized repo, version it, and any Jenkinsfile can call it with a single line. It's the DRY principle applied to CI/CD pipelines.

```
jenkins-shared-library/
├── vars/
│   ├── buildDockerImage.groovy
│   ├── deployToK8s.groovy
│   └── sendSlackAlert.groovy
└── src/
    └── com/company/Utils.groovy
```

```groovy
// vars/deployToK8s.groovy
def call(String environment, String image) {
    sh "kubectl set image deployment/app app=${image} -n ${environment}"
    sh "kubectl rollout status deployment/app -n ${environment}"
}
```

```groovy
// Any service's Jenkinsfile
@Library('company-shared-lib@main') _

pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                deployToK8s('staging', "myapp:${BUILD_NUMBER}")
            }
        }
    }
}
```

Configure via: **Manage Jenkins → Configure System → Global Pipeline Libraries**

---

### Q17. What are `stash` and `unstash` in Jenkins?

`stash` and `unstash` solve a specific problem — when different stages of a pipeline run on different agents, they don't share a filesystem. If you compile a JAR on a build agent and want to deploy it from a deploy agent, you need a way to pass that file between them.

`stash` saves specified files from the current workspace into temporary storage on the Controller. `unstash` retrieves them on any subsequent agent. The key distinction from `archiveArtifacts` is that stashes are temporary — they only exist for the duration of that build and are deleted when it completes.

```groovy
pipeline {
    agent none
    stages {
        stage('Build') {
            agent { label 'build-server' }
            steps {
                sh 'mvn clean package'
                stash name: 'app-jar', includes: 'target/*.jar'
            }
        }
        stage('Deploy') {
            agent { label 'deploy-server' }
            steps {
                unstash 'app-jar'    // Retrieve JAR built on different agent
                sh './deploy.sh target/*.jar'
            }
        }
    }
}
```

---

### Q18. What is the `input` step in Jenkins?

The `input` step pauses the pipeline execution and waits for a human to explicitly approve before continuing. This is your manual gate — the checkpoint before anything goes into production.

You can restrict who is allowed to approve using the `submitter` parameter, which accepts specific usernames or Jenkins groups. Combined with a `timeout`, you can auto-abort the pipeline if no one approves within a defined window — preventing builds from hanging indefinitely and consuming executor resources.

```groovy
stage('Production Approval') {
    options {
        timeout(time: 2, unit: 'HOURS')    // Auto-abort if no response in 2 hours
    }
    steps {
        input message: 'Deploy to Production?',
              ok: 'Approve Deployment',
              submitter: 'release-managers,devops-leads'
    }
}
```

---

### Q19. What is the `script` block in Declarative Pipeline?

Declarative Pipeline is intentionally restrictive — it enforces structure to keep pipelines readable and consistent. But sometimes you genuinely need dynamic logic that the Declarative syntax can't express: loops, conditionals based on runtime values, dynamic stage generation, or direct Jenkins API access.

The `script` block is the escape hatch. It lets you embed raw Groovy code inside a Declarative pipeline without switching entirely to Scripted syntax. The rule of thumb is to keep `script` blocks small and focused — if your `script` block is getting large, the logic probably belongs in a Shared Library.

```groovy
stage('Dynamic Deploy') {
    steps {
        script {
            def branch = env.BRANCH_NAME
            def targets = []

            if (branch == 'main') {
                targets = ['staging', 'production']
            } else if (branch.startsWith('release/')) {
                targets = ['staging']
            }

            for (target in targets) {
                sh "./deploy.sh ${target}"
            }

            currentBuild.displayName = "#${BUILD_NUMBER} → ${targets.join(', ')}"
        }
    }
}
```

---

### Q20. What is Blue Ocean in Jenkins?

Blue Ocean is a UI plugin that was introduced to modernize Jenkins' classic interface. It provides a much cleaner visual representation of pipelines — each stage appears as a card, parallel stages are displayed side by side, and log output is scoped per step rather than being one massive scrollable wall of text.

It also included a visual pipeline editor that lets you build Jenkinsfiles through a drag-and-drop interface, which was useful for teams newer to Pipeline as Code.

Worth knowing for interviews: **Blue Ocean is currently in maintenance mode**. The Jenkins community decided not to invest further in it and is instead improving the classic UI. So while it still works and you'll find it in many organizations, new features are going into the classic Jenkins interface, not Blue Ocean.

---

## 🟡 SECTION 3: CREDENTIALS & SECURITY

---

### Q21. How do you manage secrets in Jenkins?

Hardcoding credentials in a Jenkinsfile is one of the worst things you can do — it exposes secrets in Git history, build logs, and to anyone with read access to the repository. Jenkins solves this with the **Credentials Plugin**, which provides an encrypted store for secrets that pipelines can reference by ID without the actual value ever appearing in plain text.

**Supported credential types:**
- Username + Password pairs
- Secret Text (for API tokens)
- Secret Files (for kubeconfig files or certificates)
- SSH private keys
- AWS credentials

Jenkins automatically masks credential values in build logs — they appear as `****`.

```groovy
pipeline {
    environment {
        DOCKER_CREDS = credentials('dockerhub-creds')
        // Auto-creates DOCKER_CREDS_USR and DOCKER_CREDS_PSW
    }
    stages {
        stage('Push Image') {
            steps {
                sh 'docker login -u $DOCKER_CREDS_USR -p $DOCKER_CREDS_PSW'
                sh 'docker push myapp:latest'
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'deploy-key', keyFileVariable: 'SSH_KEY')
                ]) {
                    sh 'ssh -i $SSH_KEY deploy@server ./deploy.sh'
                }
            }
        }
    }
}
```

---

### Q22. What is Jenkins Role-Based Access Control (RBAC)?

By default, Jenkins has very coarse access control — users either have full admin access or they don't. In any real team environment, that's unacceptable. The **Role Strategy Plugin** brings proper RBAC to Jenkins, letting you define roles with specific permission sets and assign them to users or groups at different scopes.

**Three scope levels:**

- **Global roles** — apply across all of Jenkins
- **Item roles** — apply to specific jobs or folders using pattern matching
- **Node roles** — control which agents a user can manage

```
Global Roles:
  admin     → Overall/Administer (full access)
  developer → Job/Build, Job/Cancel, Job/Read
  viewer    → Job/Read only

Item Roles:
  medpharm-dev → pattern: medpharm/* → Job/Build, Job/Read
  flight-dev   → pattern: flight/*   → Job/Build, Job/Read

Assignments:
  dev-team-medpharm → medpharm-dev role
  ci-bot            → developer (globally)
  qa-team           → viewer (globally)
```

Configure via: **Manage Jenkins → Configure Global Security → Role-Based Strategy**

---

### Q23. What is Jenkins Security Realm and Authorization?

These are two separate security concerns that work together.

The **Security Realm** handles authentication — it's how Jenkins verifies who you are. Options include:
- Jenkins' own internal user database (fine for small teams)
- LDAP or Active Directory (standard for enterprises)
- GitHub OAuth (convenient for development teams)
- SAML 2.0 (for SSO with Okta, Azure AD, etc.)

**Authorization** is what comes after authentication — it determines what an authenticated user is actually allowed to do. Options include:
- Role-based Strategy (recommended — uses the RBAC plugin)
- Matrix-based security (per-user, per-permission control)
- "Anyone can do anything" — never use this in production

```
Manage Jenkins → Configure Global Security
  Security Realm:    GitHub OAuth / LDAP / Jenkins DB
  Authorization:     Role-Based Strategy
```

---

### Q24. How do you prevent Jenkins from logging sensitive data?

Jenkins automatically masks any value that comes from the Credentials store when you use `withCredentials` or bind credentials through the `environment` block — those values appear as `****` in build logs. But there are several ways secrets can still leak if you're not careful.

The most common mistake is using `echo` to print a variable that contains a secret. Another common issue is shell verbose mode — by default, bash prints every command it executes with `set -x`, which means `sh 'command $SECRET'` will print the expanded value of `$SECRET` before running it.

```groovy
// ❌ Never do this — leaks secret in logs
echo "DB Password is: ${env.DB_PASS}"

// ❌ Shell verbose mode exposes secret
sh 'connect-db --password $DB_PASS'

// ✅ Use withCredentials — auto-masked
withCredentials([string(credentialsId: 'db-pass', variable: 'DB_PASS')]) {
    sh '''
        set +x                          // Disable shell verbose mode
        connect-db --password $DB_PASS  // **** in logs
    '''
}

// ✅ MaskPasswords plugin for custom values
wrap([$class: 'MaskPasswordsBuildWrapper',
      varPasswordPairs: [[password: 'my-secret', var: 'SECRET']]]) {
    sh 'use-secret $SECRET'    // Logs: use-secret ****
}
```

---

## 🟠 SECTION 4: PLUGINS & INTEGRATIONS

---

### Q25. How does Jenkins integrate with Git/GitHub?

The integration works in both directions. Jenkins pulls code from GitHub, and GitHub notifies Jenkins when code changes.

On the GitHub side, you configure a **Webhook** — GitHub sends an HTTP POST to your Jenkins URL whenever a push or pull request event occurs. Jenkins receives it, identifies which pipeline to trigger, and starts the build immediately. This is the preferred approach over polling because it's instant and doesn't waste resources.

```groovy
// Production-grade checkout with SSH credentials and shallow clone
stage('Checkout') {
    steps {
        checkout([
            $class: 'GitSCM',
            branches: [[name: '*/main']],
            extensions: [
                [$class: 'CleanBeforeCheckout'],
                [$class: 'CloneOption', depth: 1, shallow: true]
            ],
            userRemoteConfigs: [[
                credentialsId: 'github-ssh-key',
                url: 'git@github.com:org/repo.git'
            ]]
        ])
    }
}
```

**Webhook setup:** Repository → Settings → Webhooks → Add webhook
- Payload URL: `https://jenkins.example.com/github-webhook/`
- Content type: `application/json`
- Events: Push + Pull Request

---

### Q26. How do you integrate Jenkins with Docker?

Jenkins uses the Docker Pipeline plugin to interact with Docker directly from Groovy pipeline steps. The typical flow is: build the image → scan for vulnerabilities → push to registry → deploy.

A critical detail — the Jenkins agent running Docker steps needs Docker daemon access. This is usually achieved by mounting the Docker socket (`/var/run/docker.sock`) into the agent container.

```groovy
pipeline {
    agent any
    environment {
        REGISTRY = 'your-registry.io'
        IMAGE    = "${REGISTRY}/myapp:${BUILD_NUMBER}"
    }
    stages {
        stage('Build Image') {
            steps {
                script { docker.build("${IMAGE}") }
            }
        }
        stage('Security Scan') {
            steps {
                sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE}"
            }
        }
        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry("https://${REGISTRY}", 'registry-creds') {
                        docker.image("${IMAGE}").push()
                        docker.image("${IMAGE}").push('latest')
                    }
                }
            }
        }
    }
    post {
        always {
            sh "docker rmi ${IMAGE} || true"    // Cleanup to save disk space
        }
    }
}
```

---

### Q27. How do you integrate Jenkins with Kubernetes?

The **Kubernetes plugin** allows Jenkins to dynamically spin up a fresh Kubernetes pod as a build agent for each pipeline run, execute the stages inside that pod, and then delete it when the build completes. This is the most scalable and clean approach to Jenkins agents — no persistent agent machines to manage, no environment contamination between builds, and automatic scaling.

You can specify multiple containers within the pod — one for Maven builds, one for Docker operations, one for kubectl commands — and switch between them per stage using the `container()` directive.

```groovy
pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.9-eclipse-temurin-17
    command: ['sleep', '99d']
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
  - name: docker
    image: docker:24-dind
    securityContext:
      privileged: true
"""
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') { sh 'mvn clean package' }
            }
        }
        stage('Docker Build') {
            steps {
                container('docker') { sh 'docker build -t myapp:latest .' }
            }
        }
    }
}
```

---

### Q28. How do you integrate Jenkins with SonarQube?

SonarQube integration in Jenkins works through two components: the **SonarQube Scanner plugin** which runs the analysis, and the **Quality Gate webhook** which blocks the pipeline if the code doesn't meet defined thresholds.

Configure the SonarQube server connection in Manage Jenkins → Configure System. Then in the pipeline, wrap the scanner command in `withSonarQubeEnv()` which automatically injects the server URL and authentication token. After the scan completes, `waitForQualityGate` pauses the pipeline until SonarQube returns a PASS or FAIL verdict.

```groovy
stage('Code Quality Analysis') {
    steps {
        withSonarQubeEnv('SonarQube-Server') {
            sh '''
                mvn sonar:sonar \
                  -Dsonar.projectKey=medpharm-api \
                  -Dsonar.projectName="MedPharm API" \
                  -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
            '''
        }
    }
}

stage('Quality Gate') {
    steps {
        timeout(time: 5, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
```

---

### Q29. How do you publish test results and reports in Jenkins?

Publishing results means Jenkins reads the output files from your test frameworks and renders them in the build UI — making test failures visible directly in Jenkins without having to dig through logs.

```groovy
post {
    always {
        // JUnit test results — shows pass/fail per test case
        junit '**/target/surefire-reports/*.xml'

        // Code coverage report
        jacoco(
            execPattern: '**/target/jacoco.exec',
            classPattern: '**/target/classes',
            sourcePattern: '**/src/main/java'
        )

        // Any HTML report (OWASP, coverage, custom)
        publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'target/site/jacoco',
            reportFiles: 'index.html',
            reportName: 'Code Coverage Report'
        ])

        // Archive built artifacts for download
        archiveArtifacts artifacts: 'target/*.jar',
                         fingerprint: true,
                         onlyIfSuccessful: true
    }
}
```

---

### Q30. How do you send notifications from Jenkins?

Notifications are configured in the `post` block and fire based on build outcome. The most useful pattern is sending a Slack message to a `#deployments` channel on success and a `#alerts` channel on failure.

```groovy
post {
    success {
        slackSend channel: '#deployments',
                  color: 'good',
                  message: "✅ *${env.JOB_NAME}* #${env.BUILD_NUMBER} deployed successfully → ${env.BUILD_URL}"

        emailext(
            subject: "✅ Build Passed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "<h3>Build succeeded</h3><p>URL: <a href='${env.BUILD_URL}'>${env.BUILD_URL}</a></p>",
            mimeType: 'text/html',
            to: 'team@company.com'
        )
    }
    failure {
        slackSend channel: '#alerts',
                  color: 'danger',
                  message: "❌ *${env.JOB_NAME}* #${env.BUILD_NUMBER} FAILED → ${env.BUILD_URL}"
    }
}
```

---

## 🔴 SECTION 5: ADVANCED CONCEPTS

---

### Q31. What is a Multibranch Pipeline?

A Multibranch Pipeline is Jenkins' way of managing pipelines at the branch level automatically. Instead of manually creating a pipeline job for every branch, Jenkins scans the repository, finds every branch that contains a Jenkinsfile, and creates a separate pipeline job for each one. When a new branch is pushed, a new job appears. When a branch is deleted, its job disappears. Pull requests get their own isolated pipeline runs.

```groovy
// Branch-aware Jenkinsfile
pipeline {
    agent any
    stages {
        stage('Build & Test') {
            steps { sh 'mvn clean verify' }
        }
        stage('Deploy to Staging') {
            when { branch 'develop' }
            steps { sh './deploy.sh staging' }
        }
        stage('Deploy to Production') {
            when { branch 'main' }
            steps {
                input message: 'Approve production deployment?',
                      submitter: 'release-managers'
                sh './deploy.sh production'
            }
        }
    }
}
```

---

### Q32. What is Jenkins Configuration as Code (JCasC)?

JCasC lets you define the entire Jenkins controller configuration — plugins, credentials, security settings, agent definitions, Slack integration, everything — in a single YAML file stored in version control. Instead of clicking through the Jenkins UI to set things up, you commit a YAML file and Jenkins configures itself from it on startup.

If your Jenkins server dies, you spin up a new one, point it at your JCasC YAML, and it's fully configured in minutes. No more manual re-configuration after disasters.

```yaml
jenkins:
  systemMessage: "Managed by JCasC — do not edit manually"
  numExecutors: 0
  securityRealm:
    ldap:
      configurations:
        - server: ldap://ldap.company.com
          rootDN: dc=company,dc=com
  authorizationStrategy:
    roleBased:
      roles:
        global:
          - name: admin
            permissions: [Overall/Administer]
            assignments: [devops-team]

credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              id: github-creds
              username: ci-bot
              password: ${GITHUB_TOKEN}    # Injected from environment variable

unclassified:
  slackNotifier:
    teamDomain: yourcompany
    tokenCredentialId: slack-token
```

---

### Q33. What is Jenkins Job DSL Plugin?

The Job DSL plugin lets you create and manage Jenkins jobs programmatically using a Groovy-based DSL, rather than clicking through the UI for every single job. When you need to add a new service, you add one line to the seed script, run it, and the new pipeline job appears in Jenkins automatically.

```groovy
// seed-job.groovy — creates pipeline jobs for all services
['order-service', 'auth-service', 'pharmacy-service', 'payment-service'].each { service ->
    pipelineJob("medpharm/${service}-pipeline") {
        displayName("${service} CI/CD")
        definition {
            cpsScm {
                scm {
                    git {
                        remote {
                            url("https://github.com/medpharm/${service}.git")
                            credentials('github-creds')
                        }
                        branch('*/main')
                    }
                }
                scriptPath('Jenkinsfile')
            }
        }
        triggers { githubPush() }
    }
}
```

---

### Q34. What is the difference between `archiveArtifacts` and `stash`?

Both save files, but for completely different purposes and lifetimes.

| | `archiveArtifacts` | `stash` |
|---|---|---|
| Lifetime | Build retention period | Current build only |
| Accessible via | Jenkins UI, API | `unstash` within same build |
| Use case | Release artifacts, audit trail | Cross-agent file sharing |

```groovy
// Archive — survives after build completes
archiveArtifacts artifacts: 'target/*.jar', fingerprint: true

// Stash — temporary, cross-stage only
stash name: 'compiled-app', includes: 'target/*.jar'

// On different agent:
unstash 'compiled-app'
```

---

### Q35. How does Jenkins handle workspace cleanup?

Without cleanup, Jenkins agents accumulate gigabytes of workspace data over time. The **Workspace Cleanup plugin** provides `cleanWs()` to handle this.

```groovy
options {
    buildDiscarder(logRotator(numToKeepStr: '10'))  // Keep last 10 builds only
}

post {
    always {
        cleanWs()    // Always clean workspace after build
    }
    failure {
        // Archive logs before cleaning on failure
        archiveArtifacts artifacts: 'logs/**', allowEmptyArchive: true
        cleanWs()
    }
}
```

---

### Q36. What is Jenkins Throttle Concurrent Builds?

The **Throttle Concurrent Builds plugin** prevents multiple pipeline instances from accessing a shared resource simultaneously. Without throttling, if ten developers push code at the same time, you could have ten simultaneous deployments hitting the same staging environment — causing conflicts, test failures from interference, and corrupted state.

```groovy
options {
    throttleJobProperty(
        categories: ['staging-environment'],
        throttleEnabled: true,
        throttleOption: 'category',
        maxConcurrentTotal: 1    // Only one deployment to staging at a time
    )
}
```

---

### Q37. What is Jenkins Pipeline Durability?

Jenkins stores pipeline state to disk at regular intervals so a pipeline can survive a Controller restart mid-execution. The frequency of these disk writes is what durability settings control — a trade-off between safety and performance.

```groovy
options {
    // For performance-critical pipelines with many fast steps
    durabilityHint('PERFORMANCE_OPTIMIZED')

    // For critical deployment pipelines where losing state is unacceptable
    durabilityHint('MAX_SURVIVABILITY')

    // Balanced middle ground
    durabilityHint('SURVIVABLE_NONATOMIC')
}
```

---

### Q38. What is the `retry` and `timeout` option?

`retry` and `timeout` are defensive mechanisms that make pipelines resilient to transient failures and runaway builds.

`timeout` sets a maximum execution time — if exceeded, Jenkins aborts the build. This prevents pipelines from hanging indefinitely when a deployment gets stuck or a test suite enters an infinite loop.

`retry` automatically re-runs a failed stage or step a specified number of times before marking it as failed. It's particularly useful for flaky tests, unstable network calls, or artifact downloads that occasionally fail due to transient issues.

```groovy
pipeline {
    options {
        timeout(time: 60, unit: 'MINUTES')    // Entire pipeline must complete in 60 mins
    }
    stages {
        stage('Deploy') {
            options {
                retry(3)
                timeout(time: 15, unit: 'MINUTES')
            }
            steps {
                sh 'kubectl rollout status deployment/app --timeout=10m'
            }
        }
        stage('Pull Dependency') {
            steps {
                retry(3) {
                    sh 'wget https://artifacts.company.com/lib.zip'
                }
            }
        }
    }
}
```

---

### Q39. What is Jenkins Fingerprinting?

Fingerprinting tracks the lineage of artifacts across multiple Jenkins jobs. When Jenkins archives an artifact with fingerprinting enabled, it computes an MD5 hash of the file. Any downstream job that consumes that artifact and also has fingerprinting enabled gets linked — Jenkins can then tell you exactly which upstream build produced an artifact and which downstream builds consumed it.

This is valuable for audit trails in regulated environments — you can answer "which build of the auth-service is currently running in production, and which source commit does it correspond to?"

```groovy
// Upstream — archive with fingerprint
archiveArtifacts artifacts: 'target/*.jar', fingerprint: true

// Downstream — track consumption
fingerprint 'libs/auth-service.jar'
```

---

### Q40. What is the `lock` step in Jenkins?

The `lock` step from the **Lockable Resources plugin** is a mutex for your pipeline — it prevents multiple builds from accessing the same resource concurrently. While Throttle Concurrent Builds limits how many pipelines run at once, `lock` is more surgical — it blocks only the specific stage that needs exclusive access, letting other stages of concurrent builds continue.

```groovy
stage('Deploy to Staging') {
    steps {
        lock(resource: 'staging-environment', inversePrecedence: true) {
            // Only one build can be in this block at a time
            sh './deploy.sh staging'
            sh './run-smoke-tests.sh'
        }
        // Lock released — next queued build can proceed
    }
}
```

---

## ⚫ SECTION 6: SCENARIO-BASED QUESTIONS

> All scenarios reference the **B2B Medpharm Platform** — a microservices-based pharmacy ordering system with services for Order Management, Pharmacy Inventory, Prescription Validation, Auth, and API Gateway.

---

### Scenario 1: How did you set up CI/CD for a Java application in your project?

In the **B2B Medpharm Platform**, we had a Spring Boot backend serving pharmacy inventory, order management, and prescription validation APIs. The development team was pushing code multiple times a day, and manual deployments were becoming a bottleneck and a consistent source of errors.

I set up a **Multibranch Pipeline** connected to the GitHub repository via webhook. The moment a developer pushed code, GitHub triggered Jenkins automatically. The pipeline ran these stages in sequence:

**Checkout → Maven Build → Unit Tests → SonarQube Analysis → Quality Gate → OWASP Dependency Check → Docker Build → Trivy Image Scan → Push to DockerHub → Deploy to Kubernetes**

For the `develop` branch, deployment went straight to the staging namespace. For `main`, there was an `input` step requiring approval from the release manager before production deployment. The entire pipeline ran in under 12 minutes. We went from manual deployments taking 45 minutes with frequent human errors to a fully automated, consistent process.

```groovy
pipeline {
    agent any
    tools { maven 'Maven-3.9' }
    environment {
        APP_NAME = 'medpharm-api'
        IMAGE    = "cloudwithketan/${APP_NAME}:${BUILD_NUMBER}"
    }
    stages {
        stage('Checkout')   { steps { checkout scm } }
        stage('Build')      { steps { sh 'mvn clean package -DskipTests' } }
        stage('Unit Tests') { steps { sh 'mvn test' } }
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') { sh 'mvn sonar:sonar' }
                timeout(time: 5, unit: 'MINUTES') { waitForQualityGate abortPipeline: true }
            }
        }
        stage('OWASP Scan')   { steps { sh 'mvn dependency-check:check' } }
        stage('Docker Build') { steps { sh "docker build -t ${IMAGE} ." } }
        stage('Trivy Scan')   { steps { sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE}" } }
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "docker login -u $USER -p $PASS && docker push ${IMAGE}"
                }
            }
        }
        stage('Deploy Staging') {
            when { branch 'develop' }
            steps {
                sh "kubectl set image deployment/${APP_NAME} ${APP_NAME}=${IMAGE} -n staging"
                sh "kubectl rollout status deployment/${APP_NAME} -n staging"
            }
        }
        stage('Approve Production') {
            when { branch 'main' }
            options { timeout(time: 2, unit: 'HOURS') }
            steps { input message: 'Deploy to Production?', submitter: 'release-managers' }
        }
        stage('Deploy Production') {
            when { branch 'main' }
            steps {
                sh "kubectl set image deployment/${APP_NAME} ${APP_NAME}=${IMAGE} -n production"
                sh "kubectl rollout status deployment/${APP_NAME} -n production"
            }
        }
    }
    post {
        always { cleanWs() }
        success { slackSend color: 'good', message: "✅ ${APP_NAME} #${BUILD_NUMBER} deployed → ${BUILD_URL}" }
        failure { slackSend color: 'danger', message: "❌ ${APP_NAME} #${BUILD_NUMBER} FAILED → ${BUILD_URL}" }
    }
}
```

---

### Scenario 2: Jenkins Server Running Slow — How did you handle it?

In the Medpharm project, builds that used to take 10 minutes were stretching to 25-30 minutes, and the Jenkins UI was sluggish.

First I identified where the bottleneck was. Checking system metrics on the Controller showed CPU at 85%, memory near capacity, and disk I/O consistently high. Running `df -h` revealed `/var/lib/jenkins` was at 92% capacity.

The root cause was a combination of issues: the Controller was running builds itself (bad practice), build history wasn't being pruned (hundreds of builds with large workspace data accumulated), and several unused plugins were consuming memory on startup.

Fixes I applied in order: moved all build execution to dedicated agents so the Controller only schedules and coordinates. Added `buildDiscarder(logRotator(numToKeepStr: '10'))` to all pipelines to auto-prune old builds. Added `cleanWs()` in all `post { always }` blocks. Removed 8 unused plugins.

After these changes, Controller CPU dropped to under 20%, disk usage fell to 40%, and build times returned to normal.

---

### Scenario 3: How did you handle CI/CD for Microservices in the Medpharm Platform?

The Medpharm platform had 5 independent microservices — Order Management, Pharmacy Inventory, Prescription Validation, Auth Service, and API Gateway. Each had a different release cadence and was owned by a different team.

The approach was **one Multibranch Pipeline per microservice**, each with its own Jenkinsfile in its own repository. This gave teams full autonomy — the pharmacy team could deploy their service without waiting for the order management team, and a failure in one service's pipeline didn't block any other.

To avoid duplicating the 80% of pipeline logic that was identical across all services — the Docker build steps, the SonarQube configuration, the Kubernetes deployment commands, the Slack notification format — I created a **Jenkins Shared Library** stored in a separate repository. Each service's Jenkinsfile was reduced to about 30 lines, calling shared functions like `buildDockerImage()`, `runSecurityScans()`, and `deployToK8s()`.

When we needed to update the Trivy severity threshold across all services, I updated it in one place in the shared library. All five pipelines picked it up automatically on their next run.

---

### Scenario 4: Jenkins Job Failed During Build — How do you troubleshoot it?

The first thing I always do is open the **Console Output** of the failed build. Jenkins prints every command and its output, and the failure point is usually clearly marked with a non-zero exit code or an error message. I read from the bottom up — the last few lines usually tell you exactly what failed.

Once I identify the failing stage, I ask: was this working before? If yes, what changed — did someone update the Jenkinsfile, modify a credential, change infrastructure, or update a dependency?

In one specific incident on the Medpharm platform, the pipeline started failing at the Docker push stage with a `401 Unauthorized` error. The Console Output showed Jenkins was successfully building the image but failing authentication with DockerHub. Checking the Credentials Manager revealed the DockerHub token had expired — someone had rotated it in DockerHub but hadn't updated the Jenkins credential. Updated the credential, re-ran the pipeline, it passed immediately.

**General troubleshooting order:** Console Output → Credentials → Environment Variables → Agent connectivity → External service availability (SonarQube, registry, Kubernetes API) → Recent code changes.

---

### Scenario 5: How did you integrate Jenkins with Docker in your project?

In the Medpharm platform, every microservice needed to be containerized and pushed to DockerHub as part of the CI pipeline. The goal was that every successful build produced a versioned, security-scanned Docker image ready for Kubernetes deployment.

The integration worked in three steps. First, after the Maven build, Jenkins ran `docker build` using the Docker Pipeline plugin, tagging the image with the Jenkins `BUILD_NUMBER` for traceability. Second, Trivy scanned the image for HIGH and CRITICAL vulnerabilities — if any were found, the build failed right there and nothing got pushed. Third, Jenkins authenticated with DockerHub using credentials from the Jenkins Credentials store and pushed the image.

A practical detail I handled — the Jenkins agent needed access to the Docker daemon. I mounted `/var/run/docker.sock` into the agent container, which lets it communicate with the host's Docker daemon without running Docker-in-Docker.

After the image was pushed, the deploy stage referenced it by the exact `BUILD_NUMBER` tag, ensuring we always knew which Jenkins build produced what's running in Kubernetes. This gave us full traceability from Git commit → Jenkins build → Docker image → Kubernetes pod.

---

### Scenario 6: How did you implement a Rollback Strategy in Jenkins?

In the Medpharm platform, a bad deployment to production could directly impact pharmacies placing orders. We couldn't afford extended downtime.

The rollback strategy had two layers. The first was **Kubernetes native rollback** — before every deployment, the pipeline captured the currently running image tag and stored it as `PREVIOUS_IMAGE`. If post-deployment smoke tests failed, a `catch` block automatically ran `kubectl set image` with the previous tag, reverting within 60 seconds.

```groovy
stage('Deploy') {
    steps {
        script {
            env.PREVIOUS_IMAGE = sh(
                returnStdout: true,
                script: "kubectl get deployment medpharm-api -n production -o jsonpath='{.spec.template.spec.containers[0].image}'"
            ).trim()

            sh "kubectl set image deployment/medpharm-api medpharm-api=${IMAGE} -n production"
            sh "kubectl rollout status deployment/medpharm-api -n production --timeout=5m"
        }
    }
}

stage('Smoke Tests') {
    steps {
        script {
            try {
                sh './run-smoke-tests.sh'
            } catch (e) {
                sh "kubectl set image deployment/medpharm-api medpharm-api=${env.PREVIOUS_IMAGE} -n production"
                slackSend color: 'danger',
                          message: "🔄 ROLLBACK triggered. Reverted to ${env.PREVIOUS_IMAGE}"
                error("Deployment rolled back due to smoke test failures")
            }
        }
    }
}
```

The second layer was Docker image versioning with `BUILD_NUMBER` tags — since we never used `latest` in production, rolling back manually was always possible by updating the Kubernetes deployment to a previous tag.

---

### Scenario 7: How did you integrate Terraform with Jenkins?

In the Medpharm project, we needed to provision AWS infrastructure — EKS cluster, VPC, security groups, RDS instance, and S3 buckets — before deploying the application. I integrated Terraform into Jenkins so infrastructure and application deployments were part of the same automated pipeline.

The infrastructure pipeline had four Terraform stages: **Init → Validate → Plan → Apply**. The key design decision was running `terraform plan` and using an `input` step to require DevOps team approval before `terraform apply` ran. This prevented accidental infrastructure changes.

Terraform state was stored in an S3 backend with DynamoDB state locking. AWS credentials were stored in Jenkins Credentials Manager and injected using `withCredentials` — never hardcoded.

```groovy
stage('Terraform Plan') {
    steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                          credentialsId: 'aws-creds']]) {
            sh 'terraform init'
            sh 'terraform validate'
            sh 'terraform plan -out=tfplan'
        }
    }
}

stage('Approve Infrastructure Changes') {
    options { timeout(time: 1, unit: 'HOURS') }
    steps {
        input message: 'Review Terraform plan and approve?',
              submitter: 'devops-leads'
    }
}

stage('Terraform Apply') {
    steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                          credentialsId: 'aws-creds']]) {
            sh 'terraform apply tfplan'
        }
    }
}
```

---

### Scenario 8: Jenkins Build Stuck in Queue — How do you debug it?

This happened during a sprint in the Medpharm project when five developers pushed code simultaneously and builds started queuing with no indication of when they'd run.

The first place I checked was the **Build Queue** in the Jenkins UI — hovering over each queued item shows the reason it's waiting. It showed "Waiting for next available executor on agent with label `docker-build`."

Going to Manage Jenkins → Nodes, I could see we had only two agents with the `docker-build` label, and both were busy with running builds.

Short-term fix: I manually added a third agent to handle the queue. Long-term fix: I switched the project to use the **Kubernetes plugin** with ephemeral pod agents. Instead of a fixed pool of two agents, Jenkins now spins up a fresh pod per build in our EKS cluster, limited only by cluster capacity. Queue issues disappeared entirely.

**Other things to check in this situation:**
- Label mismatch between what the job requires and what agents have
- Throttle Concurrent Builds limiting job concurrency
- A Lockable Resource being held by a stuck build

---

### Scenario 9: How did you migrate from Freestyle Jobs to Jenkins Pipelines?

At Hisan Labs, we inherited several Freestyle jobs handling deployments. The problem was clear — nobody could tell what these jobs were actually doing without clicking into each one individually, changes couldn't go through code review, and there was no consistency across jobs. One job had a post-build step that nobody remembered setting up.

The migration approach: first I documented what each Freestyle job was doing — every build step, every post-build action, every environment variable. Then I converted each one to a Declarative Pipeline Jenkinsfile, stored in the application's repository. I ran both the old Freestyle job and the new pipeline in parallel for a week, comparing results. Once confident they were equivalent, I disabled the Freestyle job.

The immediate benefits were visible — pipeline stages showed up clearly in the Jenkins UI, failures were pinpointed to specific stages, pipeline changes went through pull requests, and new team members could understand what CI/CD was doing just by reading the Jenkinsfile. No more tribal knowledge locked in Jenkins UI configurations.

---

### Scenario 10: How do you secure Jenkins from external attacks?

Security hardening Jenkins is non-negotiable in any production setup. Here's the layered approach I follow.

**Authentication:** Enable security from day one. At Hisan Labs, we configured GitHub OAuth so developers used their existing credentials — no separate Jenkins password management.

**Network level:** Jenkins should never be exposed directly to the internet. We ran it behind an nginx reverse proxy with TLS termination, and restricted access by IP so only VPN-connected machines could reach the Jenkins UI. The GitHub webhook was the only external-facing endpoint, restricted to GitHub's published IP ranges.

**Authorization:** Role Strategy Plugin with defined roles — DevOps team got admin access, developers got Build/Read on their own folders only, read-only viewer role for managers. No anonymous access.

**Credentials:** Everything through the Credentials plugin. Zero hardcoded secrets in Jenkinsfiles. Regular audit of stored credentials to remove stale ones.

**Plugin hygiene:** Keep Jenkins core and all plugins updated. Subscribe to Jenkins security advisories. Remove unused plugins — they're unnecessary attack surface.

**Groovy sandbox:** All Jenkinsfiles run in a sandboxed Groovy environment. Any method escaping the sandbox requires explicit admin approval via Manage Jenkins → In-process Script Approval.

**Audit Trail plugin:** Logs every configuration change — who changed what and when. Essential for compliance and incident investigation.

---

## 📋 QUICK REFERENCE

### Jenkins Pipeline Template — Full Production Pipeline

```groovy
pipeline {
    agent any
    tools { maven 'Maven-3.9' }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        APP_NAME = 'your-app'
        IMAGE    = "your-registry/${APP_NAME}:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout')   { steps { checkout scm } }
        stage('Build')      { steps { sh 'mvn clean package -DskipTests' } }
        stage('Unit Tests') {
            steps { sh 'mvn test' }
            post { always { junit '**/target/surefire-reports/*.xml' } }
        }
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') { sh 'mvn sonar:sonar' }
                timeout(time: 5, unit: 'MINUTES') { waitForQualityGate abortPipeline: true }
            }
        }
        stage('OWASP Scan')   { steps { sh 'mvn dependency-check:check' } }
        stage('Docker Build') { steps { sh "docker build -t ${IMAGE} ." } }
        stage('Trivy Scan')   { steps { sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE}" } }
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "docker login -u $USER -p $PASS && docker push ${IMAGE}"
                }
            }
        }
        stage('Deploy Staging') {
            when { branch 'develop' }
            steps {
                sh "kubectl set image deployment/${APP_NAME} ${APP_NAME}=${IMAGE} -n staging"
                sh "kubectl rollout status deployment/${APP_NAME} -n staging"
            }
        }
        stage('Approve Production') {
            when { branch 'main' }
            options { timeout(time: 2, unit: 'HOURS') }
            steps { input message: 'Deploy to Production?', submitter: 'release-managers' }
        }
        stage('Deploy Production') {
            when { branch 'main' }
            steps {
                sh "kubectl set image deployment/${APP_NAME} ${APP_NAME}=${IMAGE} -n production"
                sh "kubectl rollout status deployment/${APP_NAME} -n production"
            }
        }
    }

    post {
        always { cleanWs() }
        success { slackSend color: 'good',   message: "✅ ${APP_NAME} #${BUILD_NUMBER} deployed → ${BUILD_URL}" }
        failure { slackSend color: 'danger', message: "❌ ${APP_NAME} #${BUILD_NUMBER} FAILED → ${BUILD_URL}" }
    }
}
```

---

*Prepared by Ketan Dhadve | DevOps Engineer | github.com/CloudwithKetan*
