# Jenkins Interview Questions & Answers
> Covers Basics · Architecture · Pipelines · Plugins · Security · CI/CD · Scenario-Based

---

## 🟢 SECTION 1: BASICS & CORE CONCEPTS

---

### 1. What is Jenkins and why is it used?
**Answer:**
Jenkins is an open-source automation server written in Java. It is the most widely used CI/CD tool that helps automate the parts of software development related to building, testing, and deploying applications.

**Why it's used:**
- **Continuous Integration** — Automatically builds and tests code on every commit
- **Continuous Delivery/Deployment** — Automates release pipelines to staging/production
- **Extensible** — 1800+ plugins for tools like Git, Docker, Kubernetes, Slack, AWS
- **Distributed builds** — Master-agent architecture scales to thousands of jobs
- **Pipeline as Code** — Jenkinsfile stores pipeline definition in version control
- **Free & open-source** — No licensing costs

---

### 2. What is the difference between CI, CD (Delivery), and CD (Deployment)?
**Answer:**

| Term | Full Form | Meaning |
|---|---|---|
| **CI** | Continuous Integration | Automatically build & test on every code commit |
| **CD** | Continuous Delivery | Automatically prepare code for release; manual approval to deploy |
| **CD** | Continuous Deployment | Fully automated release to production with no manual step |

```
Developer commits code
        ↓
  CI: Build + Test (automated)
        ↓
  CD Delivery: Package + Stage (automated) → Manual Approval → Deploy
        ↓
  CD Deployment: Package + Stage + Deploy (fully automated, no manual step)
```

---

### 3. What is Jenkins Architecture?
**Answer:**
Jenkins follows a **Master-Agent (Controller-Agent)** architecture:

**Jenkins Controller (Master):**
- Hosts the Jenkins web UI
- Manages and schedules jobs
- Stores configuration, build history, logs
- Coordinates agents
- Should NOT run builds itself in production (security + performance)

**Jenkins Agent (Worker/Node):**
- Executes actual build jobs assigned by the controller
- Can be permanent (always connected) or ephemeral (spun up per build)
- Each agent has labels/tags used to route jobs
- Types: SSH agents, JNLP agents, Docker agents, Kubernetes pods

```
[Controller] ──schedules──→ [Agent 1] Linux
             ──schedules──→ [Agent 2] Windows
             ──schedules──→ [Agent 3] Docker
```

---

### 4. What is a Jenkins Job / Project?
**Answer:**
A Jenkins Job (also called a Project) is a runnable task that Jenkins can execute. It defines what to do: what to build, how to build it, when to trigger it, and what to do with the result.

**Types of Jenkins Jobs:**
| Type | Description |
|---|---|
| **Freestyle Project** | GUI-configured, simple builds. Legacy approach. |
| **Pipeline** | Code-defined pipeline using Jenkinsfile. Recommended. |
| **Multibranch Pipeline** | Auto-creates pipelines for each branch in a repo |
| **Organization Folder** | Scans GitHub/GitLab orgs for repos with Jenkinsfiles |
| **Multi-configuration Project** | Matrix builds (test across multiple OS/JDK versions) |
| **Folder** | Organizes jobs into groups (like directories) |

---

### 5. What is a Jenkinsfile?
**Answer:**
A Jenkinsfile is a text file that contains the definition of a Jenkins Pipeline, written in Groovy DSL. It lives in the root of the source code repository, enabling Pipeline as Code.

**Benefits:**
- Version controlled alongside the application code
- Code review for pipeline changes
- Consistent pipeline across branches
- Can be reused and shared

```groovy
// Declarative Pipeline (recommended)
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

### 6. What is the difference between Declarative and Scripted Pipeline?
**Answer:**

| Feature | Declarative Pipeline | Scripted Pipeline |
|---|---|---|
| Syntax | Structured, opinionated | Flexible, full Groovy |
| Learning curve | Easier | Steeper |
| Error messages | Better | Generic |
| Validation | Pre-flight checks | Runtime only |
| Introduced | Jenkins 2.x | Original pipeline |
| Recommended | Yes (for most cases) | For complex logic |

```groovy
// Declarative (structured)
pipeline {
    agent any
    environment {
        APP_VERSION = '1.0.0'
    }
    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
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
        stage('Build') {
            sh 'make build'
        }
    } catch (e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        cleanWs()
    }
}
```

---

### 7. What is a Jenkins Agent and how do you configure one?
**Answer:**
A Jenkins Agent is a machine that runs build jobs. It connects to the controller and executes pipelines.

**Types of agents:**
- **Permanent Agents** — Always connected; configured via Manage Jenkins → Nodes
- **Cloud Agents** — Spun up on demand (Docker, Kubernetes, EC2)
- **SSH Agents** — Controller SSHes into the machine to run jobs

```groovy
// Use any available agent
agent any

// Run on specific node by label
agent { label 'linux && docker' }

// Run in Docker container
agent {
    docker {
        image 'maven:3.9-eclipse-temurin-17'
        args '-v /root/.m2:/root/.m2'
    }
}

// Run in Kubernetes pod
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

### 8. What are Jenkins Triggers?
**Answer:**
Triggers define when a pipeline should automatically start.

```groovy
pipeline {
    triggers {
        // Poll SCM every 5 minutes
        pollSCM('H/5 * * * *')

        // Time-based cron schedule
        cron('H 2 * * 1-5')    // Mon-Fri at 2 AM

        // Trigger when upstream job finishes
        upstream(upstreamProjects: 'build-job', threshold: hudson.model.Result.SUCCESS)
    }
}
```

**Other trigger types:**
- **Webhook (Push trigger)** — GitHub/GitLab sends a webhook on push (preferred over polling)
- **Manual trigger** — Human clicks "Build Now"
- **Remote trigger** — API call triggers the build
- **Build after other projects** — Downstream job trigger

---

### 9. What is a Jenkins Plugin?
**Answer:**
Plugins extend Jenkins' functionality. Jenkins has 1800+ plugins covering almost every tool and service.

**Essential plugins:**
| Plugin | Purpose |
|---|---|
| Git | Git SCM integration |
| Pipeline | Jenkinsfile support |
| Blue Ocean | Modern pipeline visualization UI |
| Docker Pipeline | Docker inside pipelines |
| Kubernetes | Run agents as K8s pods |
| Credentials | Secure credential storage |
| JUnit | Test result publishing |
| HTML Publisher | Publish HTML reports |
| Slack Notification | Slack alerts |
| SonarQube Scanner | Code quality analysis |
| Artifactory | Artifact management |
| AWS Steps | AWS service integration |
| Email Extension | Advanced email notifications |

```bash
# Install via CLI
jenkins-plugin-cli --plugins git:latest pipeline:latest
```

---

### 10. What are Jenkins Build Statuses?
**Answer:**

| Status | Color | Meaning |
|---|---|---|
| **SUCCESS** | Blue/Green | Build completed successfully |
| **UNSTABLE** | Yellow | Build succeeded but tests failed or warnings |
| **FAILURE** | Red | Build failed (compilation error, test failure, script error) |
| **ABORTED** | Gray | Build was manually stopped |
| **NOT_BUILT** | Gray | Stage or build was skipped |

```groovy
// Set build result manually
currentBuild.result = 'UNSTABLE'
currentBuild.result = 'FAILURE'

// Check build result
if (currentBuild.currentResult == 'SUCCESS') {
    // ...
}
```

---

## 🔵 SECTION 2: PIPELINE DEEP DIVE

---

### 11. What are Pipeline stages, steps, and post sections?
**Answer:**

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {           // Logical division of the pipeline
            steps {                    // Actual commands to execute
                git 'https://github.com/org/repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
                junit 'target/surefire-reports/**/*.xml'   // Publish test results
            }
        }

        stage('Deploy') {
            when {
                branch 'main'         // Conditional execution
            }
            steps {
                sh './deploy.sh production'
            }
        }
    }

    post {                            // Runs after all stages
        always {
            cleanWs()                 // Always clean workspace
        }
        success {
            slackSend message: "Build succeeded: ${env.BUILD_URL}"
        }
        failure {
            mail to: 'team@company.com',
                 subject: "Build Failed: ${env.JOB_NAME}",
                 body: "Check: ${env.BUILD_URL}"
        }
        unstable {
            echo 'Build is unstable!'
        }
    }
}
```

---

### 12. What is the `when` directive in Jenkins Pipeline?
**Answer:**
The `when` directive allows conditional execution of stages based on criteria.

```groovy
stage('Deploy to Production') {
    when {
        // Run only on main branch
        branch 'main'
    }
}

stage('Deploy to Staging') {
    when {
        // Multiple conditions (AND by default)
        branch 'develop'
        environment name: 'DEPLOY_ENV', value: 'staging'
    }
}

stage('Run if tag') {
    when {
        tag "release-*"    // Run when tag matches pattern
    }
}

stage('Conditional on param') {
    when {
        expression { params.DEPLOY == true }
    }
}

stage('Run on change') {
    when {
        changeset "src/**"    // Only if src/ changed
    }
}

// OR condition
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

### 13. What are Jenkins Environment Variables?
**Answer:**

**Built-in variables:**
```groovy
env.BUILD_NUMBER        // Current build number (e.g., "42")
env.BUILD_ID            // Build ID
env.BUILD_URL           // Full URL of this build
env.JOB_NAME            // Name of the job
env.JOB_BASE_NAME       // Job name without folder path
env.WORKSPACE           // Absolute path of the workspace
env.GIT_BRANCH          // Current git branch
env.GIT_COMMIT          // Current git commit SHA
env.NODE_NAME           // Agent the build is running on
env.BRANCH_NAME         // For multibranch pipelines
```

**Defining custom environment variables:**
```groovy
pipeline {
    environment {
        APP_NAME = 'my-app'
        VERSION = sh(returnStdout: true, script: 'git describe --tags').trim()
        // From credentials
        AWS_ACCESS_KEY = credentials('aws-access-key')
    }
    stages {
        stage('Build') {
            environment {
                STAGE_VAR = 'only-in-this-stage'
            }
            steps {
                echo "Building ${env.APP_NAME} v${env.VERSION}"
            }
        }
    }
}
```

---

### 14. What are Jenkins Parameters?
**Answer:**
Parameters allow users to provide input when triggering a build manually.

```groovy
pipeline {
    parameters {
        string(name: 'VERSION', defaultValue: '1.0.0', description: 'Version to deploy')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'production'], description: 'Target env')
        password(name: 'SECRET_TOKEN', defaultValue: '', description: 'API Token')
        text(name: 'CHANGELOG', defaultValue: '', description: 'Release notes')
        file(name: 'CONFIG_FILE', description: 'Upload config file')
    }
    stages {
        stage('Deploy') {
            when {
                expression { params.ENVIRONMENT == 'production' }
            }
            steps {
                echo "Deploying version ${params.VERSION} to ${params.ENVIRONMENT}"
                sh "./deploy.sh ${params.ENVIRONMENT} ${params.VERSION}"
            }
        }
    }
}
```

---

### 15. What is Parallel execution in Jenkins Pipeline?
**Answer:**
Parallel stages run simultaneously, reducing total pipeline time.

```groovy
pipeline {
    agent any
    stages {
        stage('Test in Parallel') {
            parallel {
                stage('Unit Tests') {
                    agent { label 'linux' }
                    steps {
                        sh 'npm run test:unit'
                    }
                }
                stage('Integration Tests') {
                    agent { label 'linux' }
                    steps {
                        sh 'npm run test:integration'
                    }
                }
                stage('E2E Tests') {
                    agent { label 'chrome' }
                    steps {
                        sh 'npm run test:e2e'
                    }
                }
            }
        }

        stage('Build for Multiple Platforms') {
            parallel {
                stage('Build Linux') {
                    steps { sh 'make build-linux' }
                }
                stage('Build Windows') {
                    steps { bat 'make build-windows' }
                }
                stage('Build Mac') {
                    steps { sh 'make build-mac' }
                }
            }
        }
    }
}
```

---

### 16. What is a Shared Library in Jenkins?
**Answer:**
A Shared Library is a way to define reusable Groovy code (functions, classes, pipeline steps) that can be shared across multiple Jenkinsfiles. It avoids code duplication.

**Directory structure:**
```
jenkins-shared-library/
├── vars/
│   ├── buildApp.groovy      # Global variables / custom steps
│   ├── deployToK8s.groovy
│   └── sendSlackAlert.groovy
├── src/
│   └── com/company/
│       └── Utils.groovy     # Groovy classes
└── resources/
    └── templates/
        └── email.html
```

```groovy
// vars/buildApp.groovy
def call(String language, String version) {
    stage('Build') {
        if (language == 'java') {
            sh "mvn clean package -Dapp.version=${version}"
        } else if (language == 'node') {
            sh "npm ci && npm run build"
        }
    }
}

// vars/deployToK8s.groovy
def call(String environment, String image) {
    stage("Deploy to ${environment}") {
        sh "kubectl set image deployment/app app=${image} -n ${environment}"
        sh "kubectl rollout status deployment/app -n ${environment}"
    }
}
```

```groovy
// Jenkinsfile using shared library
@Library('my-shared-library@main') _

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                buildApp('java', '2.1.0')
            }
        }
        stage('Deploy') {
            steps {
                deployToK8s('staging', 'myapp:2.1.0')
            }
        }
    }
}
```

**Configure:** Manage Jenkins → Configure System → Global Pipeline Libraries

---

### 17. What are `stash` and `unstash` in Jenkins?
**Answer:**
`stash` saves files from the workspace for later use in another stage or agent. `unstash` retrieves the saved files.

Useful when different stages run on different agents and need to share build artifacts.

```groovy
pipeline {
    agent none
    stages {
        stage('Build') {
            agent { label 'build-server' }
            steps {
                sh 'mvn clean package'
                stash name: 'compiled-jar', includes: 'target/*.jar'
            }
        }
        stage('Test') {
            agent { label 'test-server' }
            steps {
                unstash 'compiled-jar'    // Retrieve jar from build stage
                sh 'java -jar target/*.jar &'
                sh 'mvn verify'
            }
        }
        stage('Deploy') {
            agent { label 'deploy-server' }
            steps {
                unstash 'compiled-jar'
                sh './deploy.sh target/*.jar'
            }
        }
    }
}
```

---

### 18. What is `input` step in Jenkins?
**Answer:**
The `input` step pauses the pipeline and waits for human approval before continuing. Used for manual approval gates (e.g., before deploying to production).

```groovy
stage('Deploy to Production') {
    steps {
        input message: 'Deploy to production?',
              ok: 'Yes, Deploy!',
              submitter: 'admin,release-team',   // Only these users can approve
              parameters: [
                  choice(choices: ['us-east-1', 'eu-west-1'], name: 'REGION'),
                  booleanParam(name: 'NOTIFY_USERS', defaultValue: true)
              ]
    }
}

// With timeout — auto-abort if no response in 1 hour
stage('Approve') {
    options {
        timeout(time: 1, unit: 'HOURS')
    }
    steps {
        input 'Approve deployment to production?'
    }
}
```

---

### 19. What is the `script` block in Declarative Pipeline?
**Answer:**
The `script` block allows you to run Scripted Pipeline (Groovy) code inside a Declarative Pipeline. Used for logic that Declarative syntax can't express.

```groovy
pipeline {
    agent any
    stages {
        stage('Smart Deploy') {
            steps {
                script {
                    // Complex Groovy logic
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

                    // Use Jenkins API
                    currentBuild.displayName = "#${BUILD_NUMBER} - ${branch}"
                    currentBuild.description = "Deployed to: ${targets.join(', ')}"
                }
            }
        }
    }
}
```

---

### 20. What is Blue Ocean in Jenkins?
**Answer:**
Blue Ocean is a modern UI plugin for Jenkins that provides a better visualization of pipelines. It offers:
- Visual pipeline editor (drag-and-drop pipeline creation)
- Intuitive pipeline visualization with stage-by-stage view
- Better parallel stage visualization
- Integrated log viewing per step
- Pull Request pipeline support
- Git repository browsing

Blue Ocean is still available but is in maintenance mode. Jenkins is investing in its classic UI improvements instead.

---

## 🟡 SECTION 3: CREDENTIALS & SECURITY

---

### 21. How do you manage secrets in Jenkins?
**Answer:**
Jenkins Credentials plugin provides a secure store for secrets. Never hardcode credentials in Jenkinsfiles.

**Credential types:**
- Username + Password
- Secret text (API token, secret string)
- Secret file (certificates, kubeconfig)
- SSH username + private key
- AWS credentials
- Certificate

```groovy
// Use credentials in pipeline
pipeline {
    environment {
        // Bind credentials to env vars
        DOCKER_HUB = credentials('docker-hub-creds')  // Sets DOCKER_HUB_USR and DOCKER_HUB_PSW
        API_TOKEN = credentials('api-token-secret')    // Sets API_TOKEN directly
    }
    stages {
        stage('Docker Push') {
            steps {
                sh 'docker login -u $DOCKER_HUB_USR -p $DOCKER_HUB_PSW'
                sh 'docker push myapp:latest'
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'aws-creds',
                                     usernameVariable: 'AWS_KEY',
                                     passwordVariable: 'AWS_SECRET'),
                    sshUserPrivateKey(credentialsId: 'deploy-key',
                                       keyFileVariable: 'SSH_KEY')
                ]) {
                    sh 'ssh -i $SSH_KEY deploy@server ./deploy.sh'
                }
            }
        }
    }
}
```

---

### 22. What is Jenkins Role-Based Access Control (RBAC)?
**Answer:**
Jenkins uses the **Role Strategy Plugin** for fine-grained RBAC. It allows assigning roles to users/groups at different scopes.

**Role types:**
- **Global roles** — Apply to Jenkins as a whole (admin, developer, viewer)
- **Item (project) roles** — Apply to specific jobs/folders
- **Node roles** — Apply to specific agents

**Setup:** Manage Jenkins → Configure Global Security → Role-Based Strategy

```
Roles defined:
  - admin: All permissions
  - developer: Build, Cancel, Read
  - viewer: Read only

Assignments:
  - john@company.com → developer (on folder: team-alpha/*)
  - ci-bot → developer (globally)
  - external-reviewer → viewer
```

---

### 23. What is Jenkins Security Realm and Authorization?
**Answer:**

**Security Realm** — How Jenkins authenticates users (who you are):
- Jenkins Own Database
- LDAP / Active Directory
- GitHub OAuth
- SAML 2.0

**Authorization** — What authenticated users can do:
- Anyone can do anything (not for production)
- Logged-in users can do anything
- **Matrix-based security** — Fine-grained per user
- **Role-based strategy** (plugin) — Recommended for teams

```
Manage Jenkins → Configure Global Security
  Security Realm: LDAP
  Authorization: Role-Based Strategy
```

---

### 24. How do you prevent Jenkins from logging sensitive data?
**Answer:**

```groovy
// Jenkins masks credentials bound via withCredentials or environment
// Credentials appear as **** in logs

// Mask custom sensitive values
wrap([$class: 'MaskPasswordsBuildWrapper',
      varPasswordPairs: [[password: 'my-secret', var: 'SECRET']]]) {
    sh 'echo using $SECRET'   // Logs: echo using ****
}

// Use set +x to prevent shell from echoing commands
steps {
    sh '''
        set +x
        echo "Sensitive: $API_TOKEN"
    '''
}

// Never use echo with secrets
// BAD:
echo "Password is: ${env.DB_PASS}"  // Appears in logs

// GOOD: Use withCredentials which auto-masks
withCredentials([string(credentialsId: 'db-pass', variable: 'DB_PASS')]) {
    sh 'connect-db --password $DB_PASS'  // **** in logs
}
```

---

## 🟠 SECTION 4: PLUGINS & INTEGRATIONS

---

### 25. How do Jenkins integrate with Git/GitHub?
**Answer:**

```groovy
// Basic Git checkout
stage('Checkout') {
    steps {
        git branch: 'main',
            credentialsId: 'github-creds',
            url: 'https://github.com/org/repo.git'
    }
}

// Full checkout with options
stage('Checkout') {
    steps {
        checkout([
            $class: 'GitSCM',
            branches: [[name: '*/main']],
            extensions: [
                [$class: 'CleanBeforeCheckout'],
                [$class: 'CloneOption', depth: 1, shallow: true]  // Shallow clone
            ],
            userRemoteConfigs: [[
                credentialsId: 'github-ssh-key',
                url: 'git@github.com:org/repo.git'
            ]]
        ])
    }
}
```

**GitHub webhook setup:**
1. GitHub repo → Settings → Webhooks → Add webhook
2. Payload URL: `https://jenkins.example.com/github-webhook/`
3. Content type: `application/json`
4. Events: Push, Pull Request

---

### 26. How do you integrate Jenkins with Docker?
**Answer:**

```groovy
pipeline {
    agent any
    environment {
        REGISTRY = 'registry.example.com'
        IMAGE = "${REGISTRY}/myapp:${env.BUILD_NUMBER}"
    }
    stages {
        stage('Build Image') {
            steps {
                script {
                    docker.build("${IMAGE}")
                }
            }
        }
        stage('Test in Container') {
            steps {
                script {
                    docker.image('maven:3.9').inside('-v /root/.m2:/root/.m2') {
                        sh 'mvn test'
                    }
                }
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
            sh "docker rmi ${IMAGE} || true"    // Cleanup
        }
    }
}
```

---

### 27. How do you integrate Jenkins with Kubernetes?
**Answer:**

```groovy
// Using Kubernetes plugin — runs each build in a fresh K8s pod
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
    volumeMounts:
    - name: docker-storage
      mountPath: /var/lib/docker
  volumes:
  - name: docker-storage
    emptyDir: {}
"""
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
        stage('Docker Build') {
            steps {
                container('docker') {
                    sh 'docker build -t myapp:latest .'
                }
            }
        }
    }
}
```

---

### 28. How do you integrate Jenkins with SonarQube?
**Answer:**

```groovy
pipeline {
    agent any
    tools {
        maven 'Maven-3.9'
    }
    stages {
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {  // Configured in Jenkins settings
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=my-project \
                          -Dsonar.projectName="My Project" \
                          -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                    '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true   // Fail build if gate fails
                }
            }
        }
    }
}
```

---

### 29. How do you publish test results and reports in Jenkins?
**Answer:**

```groovy
post {
    always {
        // JUnit test results
        junit '**/target/surefire-reports/*.xml'
        junit '**/test-results/**/*.xml'

        // Code coverage (JaCoCo)
        jacoco(execPattern: '**/target/jacoco.exec',
               classPattern: '**/target/classes',
               sourcePattern: '**/src/main/java')

        // HTML report
        publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'target/site/jacoco',
            reportFiles: 'index.html',
            reportName: 'Coverage Report'
        ])

        // Archive build artifacts
        archiveArtifacts artifacts: 'target/*.jar,target/*.war',
                         fingerprint: true,
                         onlyIfSuccessful: true
    }
}
```

---

### 30. How do you send notifications from Jenkins?
**Answer:**

```groovy
post {
    success {
        // Slack notification
        slackSend channel: '#deployments',
                  color: 'good',
                  message: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${env.BUILD_URL}"

        // Email notification
        emailext(
            subject: "Jenkins Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
                <h2>Build Succeeded</h2>
                <p>Job: ${env.JOB_NAME}</p>
                <p>Build: #${env.BUILD_NUMBER}</p>
                <p>URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
            """,
            mimeType: 'text/html',
            to: 'team@company.com'
        )
    }
    failure {
        slackSend channel: '#alerts',
                  color: 'danger',
                  message: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${env.BUILD_URL}"
    }
}
```

---

## 🔴 SECTION 5: ADVANCED CONCEPTS

---

### 31. What is a Multibranch Pipeline?
**Answer:**
A Multibranch Pipeline automatically discovers, creates, and manages pipelines for all branches and PRs in a repository that contain a Jenkinsfile.

**How it works:**
1. Jenkins scans the repository for branches
2. Any branch with a Jenkinsfile gets its own pipeline job
3. New branches auto-create jobs; deleted branches auto-remove jobs
4. Pull Requests get their own pipelines

**Configuration:**
- Manage Jenkins → New Item → Multibranch Pipeline
- Set repository URL and credentials
- Set scan schedule or webhook trigger

```groovy
// Jenkinsfile — branch-aware logic
pipeline {
    agent any
    stages {
        stage('Deploy') {
            when { branch 'main' }
            steps { sh './deploy.sh production' }
        }
        stage('Deploy Staging') {
            when { branch 'develop' }
            steps { sh './deploy.sh staging' }
        }
    }
}
```

---

### 32. What is Jenkins Configuration as Code (JCasC)?
**Answer:**
JCasC (Jenkins Configuration as Code) plugin allows you to configure the entire Jenkins controller from a YAML file — eliminating manual GUI configuration. The config is version-controlled and reproducible.

```yaml
# jenkins.yaml
jenkins:
  systemMessage: "Jenkins managed by JCasC"
  numExecutors: 0    # No builds on controller
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
            assignments: [admin-group]

credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              id: github-creds
              username: ci-bot
              password: ${GITHUB_TOKEN}   # From env var

unclassified:
  location:
    url: https://jenkins.company.com/
  slackNotifier:
    teamDomain: mycompany
    tokenCredentialId: slack-token
```

---

### 33. What is Jenkins DSL (Job DSL Plugin)?
**Answer:**
The Job DSL plugin lets you define Jenkins jobs programmatically using Groovy scripts instead of the GUI. It's used to create and manage many jobs consistently.

```groovy
// seed-job.groovy — Creates multiple jobs from code
['service-a', 'service-b', 'service-c'].each { service ->
    pipelineJob("${service}-pipeline") {
        displayName("${service} CI/CD Pipeline")
        definition {
            cpsScm {
                scm {
                    git {
                        remote {
                            url("https://github.com/org/${service}.git")
                            credentials('github-creds')
                        }
                        branch('*/main')
                    }
                }
                scriptPath('Jenkinsfile')
            }
        }
        triggers {
            githubPush()
        }
    }
}
```

---

### 34. What is the difference between `archiveArtifacts` and `stash`?
**Answer:**

| Feature | `archiveArtifacts` | `stash` |
|---|---|---|
| Stored in | Jenkins server (long-term) | Jenkins (temporary) |
| Accessible | Browser download, API | Only within same build via `unstash` |
| Retention | Based on build retention policy | Deleted when build completes |
| Use case | Release artifacts, audit trails | Passing files between pipeline stages |

```groovy
// Archive — persists permanently
archiveArtifacts artifacts: 'target/*.jar', fingerprint: true

// Stash — temporary, cross-stage sharing
stash name: 'test-data', includes: 'reports/**'
// Later in another agent:
unstash 'test-data'
```

---

### 35. How does Jenkins handle workspace cleanup?
**Answer:**

```groovy
// Clean before build (Workspace Cleanup plugin)
options {
    skipDefaultCheckout()    // Don't auto-checkout
}
stages {
    stage('Clean') {
        steps {
            cleanWs()            // Clean entire workspace
        }
    }
}

// Clean specific patterns
cleanWs(patterns: [[pattern: 'target/', type: 'INCLUDE']])

// Always clean after build
post {
    always {
        cleanWs()
    }
}

// Delete workspace on failure only
post {
    failure {
        cleanWs()
    }
}
```

---

### 36. What is Jenkins Throttle Concurrent Builds?
**Answer:**
The Throttle Concurrent Builds plugin limits how many instances of a job (or a group of jobs) can run at the same time. Prevents overloading resources like a shared test environment or a deploy target.

```groovy
options {
    throttleJobProperty(
        categories: ['deploy-production'],  // Throttle category
        throttleEnabled: true,
        throttleOption: 'category',
        maxConcurrentPerNode: 1,
        maxConcurrentTotal: 1               // Only 1 at a time globally
    )
}
```

---

### 37. What is Jenkins Pipeline Durability?
**Answer:**
Jenkins Pipeline stores pipeline state to disk periodically so it can survive controller restarts mid-build. Durability settings balance performance vs. reliability.

```groovy
// Set in Jenkinsfile
options {
    durabilityHint('PERFORMANCE_OPTIMIZED')   // Fastest, but loses data on crash
    // durabilityHint('SURVIVABLE_NONATOMIC')  // Balanced
    // durabilityHint('MAX_SURVIVABILITY')     // Slowest, most durable (default)
}
```

Use `PERFORMANCE_OPTIMIZED` for long-running pipelines with many fast steps to reduce I/O. Use `MAX_SURVIVABILITY` for deployments where losing state is critical.

---

### 38. What is the `retry` and `timeout` option?
**Answer:**

```groovy
pipeline {
    options {
        timeout(time: 60, unit: 'MINUTES')    // Pipeline-level timeout
        retry(3)                               // Retry entire pipeline 3 times
    }
    stages {
        stage('Flaky Test') {
            options {
                retry(2)                       // Retry this stage 2 times
                timeout(time: 10, unit: 'MINUTES')
            }
            steps {
                sh 'npm run test:flaky'
            }
        }
        stage('Download Artifact') {
            steps {
                retry(3) {                     // Retry just this block
                    sh 'wget https://artifacts.example.com/file.zip'
                }
            }
        }
    }
}
```

---

### 39. What is Jenkins Fingerprinting?
**Answer:**
Fingerprinting tracks where files (artifacts) are used across different jobs and builds. Jenkins computes an MD5 hash of the file and associates it with jobs.

```groovy
// Enable fingerprinting
archiveArtifacts artifacts: '*.jar', fingerprint: true

// Track a specific file
fingerprint 'mylib.jar'
```

Use case: Know which downstream jobs consumed a specific build's artifact. Useful for audit trails.

---

### 40. What is the `lock` step in Jenkins?
**Answer:**
The `lock` step (from Lockable Resources plugin) prevents concurrent access to a shared resource (like a test environment, deploy target, or hardware).

```groovy
stage('Deploy to Staging') {
    steps {
        lock(resource: 'staging-environment', inversePrecedence: true) {
            sh './deploy.sh staging'
            sh './run-smoke-tests.sh'
        }
        // Lock released here — next queued build can proceed
    }
}
```

---

## ⚫ SECTION 6: SCENARIO-BASED QUESTIONS

---

### 41. SCENARIO: Jenkins build is stuck in queue and never starts. What do you check?

**Answer:**
```bash
# Step 1: Check the queue reason in Jenkins UI
# Build Queue → hover over job → shows "Waiting for next available executor"

# Step 2: Check available agents
# Manage Jenkins → Nodes → see which are online/offline

# Step 3: Check if agent labels match
# Job requires: label 'linux && docker'
# Available agents: label 'linux'    ← mismatch!

# Step 4: Check executor availability
# If all executors are busy, builds queue up
# Manage Jenkins → Nodes → check # of executors vs running builds

# Step 5: Check for offline agents
# Manage Jenkins → Nodes → Launch agent / check logs

# Step 6: Check for resource constraints (Throttle plugin)
# Job may be throttled — only 1 at a time, current one still running

# Step 7: Check for lock waiting (Lockable Resources)
# Resource may be held by another build
```

**Common fixes:**
- Add more agents or increase executor count
- Fix label mismatch in agent or job config
- Scale agents dynamically with Kubernetes/EC2 plugin

---

### 42. SCENARIO: Pipeline fails with "No such DSL method" error. How do you fix it?

**Answer:**
```
Error: java.lang.NoSuchMethodError: No such DSL method 'xyz' found among steps
```

**Causes and fixes:**

```groovy
// Cause 1: Plugin not installed
// Fix: Install the required plugin (e.g., 'slack' step needs Slack plugin)

// Cause 2: Using Scripted syntax inside Declarative
// BAD — can't use node{} directly in declarative pipeline
pipeline {
    stages {
        stage('Test') {
            node('linux') {   // ← ERROR in declarative!
                sh 'test'
            }
        }
    }
}

// FIX — use script{} block or agent
pipeline {
    stages {
        stage('Test') {
            agent { label 'linux' }    // Correct declarative syntax
            steps {
                sh 'test'
            }
        }
    }
}

// Cause 3: Sandbox restriction (in-process script approval needed)
// Fix: Manage Jenkins → In-process Script Approval → approve the method
```

---

### 43. SCENARIO: Jenkins is running out of disk space. How do you fix it?

**Answer:**
```bash
# Step 1: Find what's consuming disk
df -h /var/lib/jenkins
du -sh /var/lib/jenkins/jobs/*  | sort -rh | head -20
du -sh /var/lib/jenkins/workspace/*  | sort -rh | head -20

# Step 2: Configure build discard policy per job
// In Jenkinsfile
options {
    buildDiscarder(logRotator(
        numToKeepStr: '10',          // Keep last 10 builds
        daysToKeepStr: '30',         // Or builds in last 30 days
        artifactNumToKeepStr: '5',   // Keep artifacts for last 5 builds
        artifactDaysToKeepStr: '7'
    ))
}

# Step 3: Clean up old workspaces
# Workspace Cleanup plugin post-build
post {
    always { cleanWs() }
}

# Step 4: Manually delete old builds
# Jenkins UI → Job → Build History → Delete specific builds

# Step 5: Remove unused plugins
# Manage Jenkins → Plugin Manager → Installed → uninstall unused

# Step 6: Move Jenkins home to larger disk
# Edit JENKINS_HOME in /etc/default/jenkins
# Symlink or migrate /var/lib/jenkins to larger storage
```

---

### 44. SCENARIO: How do you migrate Jenkins jobs from one server to another?

**Answer:**

**Option 1: Copy Jenkins home directory (simplest)**
```bash
# Stop old Jenkins
sudo systemctl stop jenkins

# Copy Jenkins home
rsync -avz /var/lib/jenkins/ user@new-server:/var/lib/jenkins/

# Or tar + transfer
tar -czf jenkins-backup.tar.gz /var/lib/jenkins/
scp jenkins-backup.tar.gz user@new-server:/tmp/
ssh new-server "tar -xzf /tmp/jenkins-backup.tar.gz -C /"

# Start new Jenkins
sudo systemctl start jenkins
```

**Option 2: Job Import Plugin**
```bash
# Export specific jobs from old server
curl -u admin:token http://old-jenkins/job/my-job/config.xml > my-job.xml

# Import to new server
curl -X POST -u admin:token \
  -H "Content-Type: application/xml" \
  --data-binary @my-job.xml \
  http://new-jenkins/createItem?name=my-job
```

**Option 3: JCasC + Shared Library (GitOps approach)**
```bash
# Store all config in Git (JCasC YAML)
# New Jenkins just loads from Git on startup
# Jobs defined as Multibranch or Job DSL — self-recovering
```

---

### 45. SCENARIO: How do you set up a full CI/CD pipeline for a Java application?

**Answer:**
```groovy
pipeline {
    agent any

    tools {
        maven 'Maven-3.9'
        jdk 'OpenJDK-17'
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        APP_NAME = 'my-java-app'
        DOCKER_REGISTRY = 'registry.company.com'
        IMAGE = "${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}"
        SONAR_PROJECT = 'my-java-app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                }
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean verify -Pcoverage'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                    jacoco execPattern: '**/target/jacoco.exec'
                }
            }
        }

        stage('Code Quality') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=${SONAR_PROJECT}'
                }
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE}")
                }
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
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'registry-creds') {
                        docker.image("${IMAGE}").push()
                        docker.image("${IMAGE}").push("latest")
                    }
                }
            }
        }

        stage('Deploy to Staging') {
            when { branch 'develop' }
            steps {
                sh "kubectl set image deployment/${APP_NAME} ${APP_NAME}=${IMAGE} -n staging"
                sh "kubectl rollout status deployment/${APP_NAME} -n staging"
            }
        }

        stage('Approve Production') {
            when { branch 'main' }
            options { timeout(time: 2, unit: 'HOURS') }
            steps {
                input message: "Deploy ${IMAGE} to production?",
                      submitter: 'release-managers'
            }
        }

        stage('Deploy to Production') {
            when { branch 'main' }
            steps {
                sh "kubectl set image deployment/${APP_NAME} ${APP_NAME}=${IMAGE} -n production"
                sh "kubectl rollout status deployment/${APP_NAME} -n production"
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            slackSend color: 'good', message: "✅ ${APP_NAME} #${BUILD_NUMBER} deployed - ${BUILD_URL}"
        }
        failure {
            slackSend color: 'danger', message: "❌ ${APP_NAME} #${BUILD_NUMBER} FAILED - ${BUILD_URL}"
        }
    }
}
```

---

### 46. SCENARIO: How do you prevent a pull request from merging if Jenkins build fails?

**Answer:**

**GitHub Setup:**
1. Install **GitHub Branch Source** plugin in Jenkins
2. Create a Multibranch Pipeline connected to the repo
3. Jenkins will auto-create pipeline jobs for PRs
4. In GitHub: Settings → Branches → Branch protection rules:
   - Enable "Require status checks to pass before merging"
   - Add Jenkins status check (e.g., `continuous-integration/jenkins/pr-merge`)

**GitLab Setup:**
1. Use **GitLab plugin** in Jenkins
2. In GitLab: Settings → Integrations → Jenkins CI
3. Enable "Merge requests events"
4. GitLab MR shows Jenkins build status; can block merge if failing

```groovy
// In Jenkinsfile — report status back to GitHub
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                githubNotify context: 'Jenkins CI',
                             description: 'Running tests...',
                             status: 'PENDING'
                sh 'npm test'
            }
        }
    }
    post {
        success {
            githubNotify context: 'Jenkins CI',
                         description: 'All tests passed',
                         status: 'SUCCESS'
        }
        failure {
            githubNotify context: 'Jenkins CI',
                         description: 'Tests failed',
                         status: 'FAILURE'
        }
    }
}
```

---

### 47. SCENARIO: A Jenkins pipeline takes 45 minutes. How do you optimize it?

**Answer:**

```groovy
// Optimization 1: Run tests in parallel
stage('Tests') {
    parallel {
        stage('Unit Tests')       { steps { sh 'npm run test:unit' } }
        stage('Integration Tests') { steps { sh 'npm run test:integration' } }
        stage('Lint')             { steps { sh 'npm run lint' } }
    }
}

// Optimization 2: Use proper caching (Maven, npm, pip)
agent {
    docker {
        image 'node:18'
        args '-v /jenkins-cache/npm:/root/.npm'   // Cache npm modules
    }
}

// Optimization 3: Shallow git clone
checkout([
    $class: 'GitSCM',
    extensions: [[$class: 'CloneOption', depth: 1, shallow: true]]
])

// Optimization 4: Skip stages that haven't changed
stage('Build Frontend') {
    when {
        changeset 'frontend/**'    // Only if frontend changed
    }
    steps { sh 'npm run build' }
}

// Optimization 5: Use faster agents
agent { label 'high-cpu' }

// Optimization 6: Skip tests on docs-only commits
stage('Test') {
    when {
        not { changeset '**/*.md' }
    }
}

// Optimization 7: Cache Docker layers (build cache)
sh '''
  docker build \
    --cache-from ${REGISTRY}/myapp:cache \
    --build-arg BUILDKIT_INLINE_CACHE=1 \
    -t ${IMAGE} .
'''
```

---

### 48. SCENARIO: How do you restart a failed stage without re-running the whole pipeline?

**Answer:**
Jenkins supports **Restart from Stage** (since Jenkins 2.x):

1. Open the failed build in Jenkins UI
2. Click "Restart from Stage"
3. Select which stage to restart from
4. Click "Run"

**Requirements:**
- Pipeline must use Declarative syntax
- Build must not have been cleaned up
- Stage must have been reached in the original run

```groovy
// Make stages restartable — avoid side effects in early stages
// Don't rely on workspace data from previous stages (use stash/unstash)
// Use input step at critical points so restarts are meaningful

options {
    preserveStashes(buildCount: 5)   // Keep stashes for restarted builds
}
```

**CLI approach:**
```bash
curl -X POST "http://jenkins/job/my-pipeline/42/restart?startStage=Deploy"
```

---

### 49. SCENARIO: How do you handle a scenario where one agent goes offline mid-build?

**Answer:**
```groovy
// Prevention 1: Use Kubernetes agents — auto-replaceable pods
agent {
    kubernetes {
        yaml "..."
    }
}

// Prevention 2: Set retry and timeout options
options {
    retry(2)
    timeout(time: 30, unit: 'MINUTES')
}

// Prevention 3: Use `stash` to save work before risky operations
stages {
    stage('Build') {
        steps {
            sh 'mvn package'
            stash name: 'jar', includes: 'target/*.jar'
        }
    }
    stage('Deploy') {
        agent { label 'deploy' }   // Different agent
        steps {
            unstash 'jar'          // Retrieve from stash
            sh './deploy.sh'
        }
    }
}

// Recovery: If agent goes offline
// 1. Jenkins marks build as FAILURE after timeout
// 2. Check Manage Jenkins → Nodes → reconnect or re-provision
// 3. Use "Restart from Stage" if declarative pipeline
// 4. For cloud agents (K8s/EC2): plugin auto-terminates and relaunches
```

---

### 50. SCENARIO: How do you dynamically generate pipeline stages based on a configuration file?

**Answer:**
```groovy
// config/services.json
// ["auth-service", "user-service", "order-service", "payment-service"]

pipeline {
    agent any
    stages {
        stage('Load Config') {
            steps {
                script {
                    def config = readJSON file: 'config/services.json'
                    env.SERVICES = config.join(',')
                }
            }
        }

        stage('Build Services') {
            steps {
                script {
                    def services = env.SERVICES.split(',')
                    def parallelStages = [:]

                    services.each { service ->
                        def svc = service.trim()   // Capture for closure
                        parallelStages["Build ${svc}"] = {
                            stage("Build ${svc}") {
                                dir(svc) {
                                    sh 'mvn clean package -DskipTests'
                                    sh "docker build -t ${svc}:${BUILD_NUMBER} ."
                                }
                            }
                        }
                    }

                    parallel parallelStages
                }
            }
        }

        stage('Deploy Services') {
            steps {
                script {
                    def services = env.SERVICES.split(',')
                    services.each { service ->
                        sh "kubectl set image deployment/${service.trim()} app=${service.trim()}:${BUILD_NUMBER} -n production"
                    }
                }
            }
        }
    }
}
```

---

### 51. SCENARIO: How do you implement a rollback mechanism in Jenkins pipeline?

**Answer:**
```groovy
pipeline {
    agent any
    environment {
        APP = 'my-app'
        NAMESPACE = 'production'
    }
    stages {
        stage('Deploy') {
            steps {
                script {
                    // Save current image for rollback
                    env.PREVIOUS_IMAGE = sh(
                        returnStdout: true,
                        script: "kubectl get deployment ${APP} -n ${NAMESPACE} -o jsonpath='{.spec.template.spec.containers[0].image}'"
                    ).trim()
                    echo "Previous image: ${env.PREVIOUS_IMAGE}"

                    // Deploy new version
                    sh "kubectl set image deployment/${APP} ${APP}=${IMAGE} -n ${NAMESPACE}"
                    sh "kubectl rollout status deployment/${APP} -n ${NAMESPACE} --timeout=5m"
                }
            }
        }

        stage('Smoke Tests') {
            steps {
                script {
                    try {
                        sh './run-smoke-tests.sh'
                    } catch (e) {
                        echo "Smoke tests failed! Rolling back to ${env.PREVIOUS_IMAGE}"
                        sh "kubectl set image deployment/${APP} ${APP}=${env.PREVIOUS_IMAGE} -n ${NAMESPACE}"
                        sh "kubectl rollout status deployment/${APP} -n ${NAMESPACE}"
                        error("Deployment rolled back due to failed smoke tests")
                    }
                }
            }
        }
    }

    post {
        failure {
            slackSend color: 'danger',
                      message: "🔄 ROLLBACK triggered for ${APP}. Reverted to ${env.PREVIOUS_IMAGE}"
        }
    }
}
```

---

### 52. SCENARIO: Jenkins controller is down. How do you recover it?

**Answer:**
```bash
# Step 1: Check Jenkins service status
sudo systemctl status jenkins
journalctl -u jenkins -n 100 --no-pager

# Step 2: Check disk space (common cause)
df -h
du -sh /var/lib/jenkins/

# Step 3: Check Java version
java -version    # Jenkins requires Java 11 or 17

# Step 4: Check Jenkins logs
tail -200 /var/log/jenkins/jenkins.log

# Step 5: Start Jenkins in safe mode (disables plugins)
# Edit /etc/default/jenkins
JAVA_ARGS="-Djava.awt.headless=true -Dhudson.lifecycle=hudson.lifecycle.ExitLifecycle"
# Then add startup flag:
# JENKINS_ARGS="--argumentsRealm.passwd.admin=admin --argumentsRealm.roles.admin=admin -Djava.net.preferIPv4Stack=true"

# Safe mode (disable all plugins on startup)
# Visit: https://jenkins.example.com/safeRestart

# Step 6: Restore from backup
tar -xzf jenkins-backup.tar.gz -C /var/lib/jenkins/
chown -R jenkins:jenkins /var/lib/jenkins/
systemctl start jenkins

# Prevention:
# - Regular backups: ThinBackup plugin or scripted tar of JENKINS_HOME
# - Monitor disk, memory, CPU with alerts
# - Run Jenkins in Docker/K8s for easier recovery
# - Use JCasC so config is in Git (controller is stateless)
```

---

### 53. SCENARIO: How do you trigger a Jenkins pipeline from another pipeline?

**Answer:**
```groovy
// Method 1: build step — synchronous (waits for result)
stage('Trigger Downstream') {
    steps {
        script {
            def result = build job: 'deploy-pipeline',
                parameters: [
                    string(name: 'VERSION', value: env.BUILD_NUMBER),
                    string(name: 'ENVIRONMENT', value: 'staging'),
                    booleanParam(name: 'RUN_SMOKE', value: true)
                ],
                wait: true,
                propagate: true    // Fail upstream if downstream fails

            echo "Downstream build: ${result.absoluteUrl}"
        }
    }
}

// Method 2: build step — async (fire and forget)
stage('Trigger and Continue') {
    steps {
        build job: 'notification-job',
              wait: false    // Don't wait for result
    }
}

// Method 3: Trigger multiple downstream jobs in parallel
stage('Trigger Deployments') {
    steps {
        parallel(
            'Deploy EU': { build job: 'deploy-eu', parameters: [string(name: 'VERSION', value: env.VERSION)] },
            'Deploy US': { build job: 'deploy-us', parameters: [string(name: 'VERSION', value: env.VERSION)] },
            'Deploy APAC': { build job: 'deploy-apac', parameters: [string(name: 'VERSION', value: env.VERSION)] }
        )
    }
}
```

---

### 54. SCENARIO: How do you implement feature flags in your CI/CD pipeline?

**Answer:**
```groovy
pipeline {
    agent any
    parameters {
        booleanParam(name: 'ENABLE_NEW_FEATURE', defaultValue: false)
        booleanParam(name: 'SKIP_INTEGRATION_TESTS', defaultValue: false)
        choice(name: 'DEPLOYMENT_STRATEGY', choices: ['rolling', 'blue-green', 'canary'])
    }
    stages {
        stage('Build') {
            steps {
                sh """
                    mvn package \
                      -Dfeature.new_checkout=${params.ENABLE_NEW_FEATURE} \
                      -Dfeature.experiment_v2=${params.ENABLE_NEW_FEATURE}
                """
            }
        }
        stage('Integration Tests') {
            when {
                expression { !params.SKIP_INTEGRATION_TESTS }
            }
            steps {
                sh 'mvn verify -Pintegration'
            }
        }
        stage('Deploy') {
            steps {
                script {
                    if (params.DEPLOYMENT_STRATEGY == 'canary') {
                        sh './deploy-canary.sh 10'   // 10% traffic
                    } else if (params.DEPLOYMENT_STRATEGY == 'blue-green') {
                        sh './deploy-blue-green.sh'
                    } else {
                        sh './deploy-rolling.sh'
                    }
                }
            }
        }
    }
}
```

---

### 55. SCENARIO: How do you secure Jenkins from external attacks?

**Answer:**

```bash
# 1. Enable authentication (never run with security disabled)
# Manage Jenkins → Configure Global Security → Enable security

# 2. Use HTTPS (TLS)
# Run Jenkins behind nginx/Apache with SSL termination
# Or configure Jenkins keystore directly

# 3. Configure Content Security Policy
# Prevent XSS in build artifacts
System.setProperty("hudson.model.DirectoryBrowserSupport.CSP",
    "default-src 'self'; script-src 'self'")

# 4. Keep Jenkins and plugins updated
# Manage Jenkins → Plugin Manager → check for updates regularly
# Subscribe to Jenkins security advisories

# 5. Disable CLI over remoting
# Manage Jenkins → Configure Global Security → Disable CLI

# 6. Limit build permissions
# Don't allow anonymous access
# Use RBAC to limit what developers can do

# 7. Use Credentials plugin (never plaintext in jobs)

# 8. Restrict Jenkinsfile capabilities (Groovy sandbox)
# Manage Jenkins → In-process Script Approval

# 9. Network security
# Don't expose Jenkins directly to internet
# Use VPN or bastion host for access
# Restrict access by IP if possible

# 10. Audit logs
# Audit Trail plugin logs all configuration changes
```

---

*© Jenkins Interview Q&A — 55 Questions*
*Sections: Basics · Pipelines · Credentials · Plugins/Integrations · Advanced · 15 Scenarios*
