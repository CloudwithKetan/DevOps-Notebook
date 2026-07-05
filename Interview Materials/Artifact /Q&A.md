# Artifacts — Interview Q&A

## 1. What is an artifact? ⭐⭐⭐⭐⭐

**Expected Answer**
An artifact is the output generated after building an application. It is the deployable file, such as a JAR, WAR, or React build folder, that is used for deployment.

## 2. What are examples of artifacts? ⭐⭐⭐⭐⭐

**Expected Answer**
- Java → `.jar`, `.war`
- Spring Boot → `.jar`
- React → `build/` folder
- Angular → `dist/` folder

## 3. How is an artifact created? ⭐⭐⭐⭐☆

**Expected Answer**
An artifact is created during the build process.

For example:

- Java:
```
mvn clean package
```
creates a `.jar` or `.war` file.

- React:
```
npm run build
```
creates a `build/` folder.

## 4. Where are artifacts stored? ⭐⭐⭐⭐☆

**Expected Answer**
Artifacts are commonly stored in artifact repositories such as:
- Nexus
- Artifactory

These repositories provide versioning, centralized storage, and make artifacts available for deployment.

## 5. Why do we store artifacts? ⭐⭐⭐⭐☆

**Expected Answer**
We store artifacts so the same tested build can be deployed to multiple environments like Dev, QA, UAT, and Production without rebuilding the application.

## 6. What happens after an artifact is created? ⭐⭐⭐⭐⭐

**Expected Answer**
After the artifact is created:

1. Jenkins uploads it to Nexus or Artifactory (if used).
2. A Docker image may be built using the artifact.
3. The Docker image is pushed to a container registry.
4. Kubernetes deploys the application.

## 7. What is the difference between source code and an artifact? ⭐⭐⭐⭐☆

**Expected Answer**
- Source code is written by developers and cannot be deployed directly.
- Artifact is the compiled or built output that is ready for deployment.

## 8. Which Jenkins stage creates the artifact? ⭐⭐⭐☆☆

**Expected Answer**
The Build stage creates the artifact.

Example:
```
stage('Build') {
    steps {
        sh 'mvn clean package'
    }
}
```

## 9. Is a Docker image an artifact? ⭐⭐⭐⭐☆

**Expected Answer**
Yes. In many organizations, a Docker image is also treated as an artifact because it is a versioned, deployable package. However, when discussing Maven or npm, the initial artifacts are typically the JAR/WAR file or the React `build` folder, which are then used to create the Docker image.

## 10. Explain the artifact flow in your project. ⭐⭐⭐⭐⭐

**Expected Answer**
The developer pushes code to Git. Jenkins triggers the pipeline and builds the application using Maven or npm. The build generates an artifact, such as a JAR file or a React build folder. The artifact is then used to build a Docker image, which is pushed to a container registry. Finally, Kubernetes deploys the Docker image.

## Scenario-Based Question

**Q. Jenkins successfully builds the project, but the JAR file is not found. What will you check?**

**Answer**
I would check:
- Whether `mvn clean package` completed successfully.
- Whether the artifact exists in the `target/` directory.
- Whether the `pom.xml` packaging configuration is correct.
- Whether the Jenkins pipeline is looking in the correct path.
- Whether the build failed during the packaging phase.

## What interviewers expect from a 9–11 month intern

They generally won't expect you to explain advanced topics like:
- Nexus repository replication
- Artifact retention policies
- Repository cleanup strategies
- Snapshot vs. release repository internals

Instead, they expect you to understand:
- What an artifact is.
- How it is created.
- Where it is stored.
- How it moves through a CI/CD pipeline.
- How it is eventually deployed.

Given your internship and DevOps learning path, being able to clearly explain the end-to-end flow from Git → Jenkins → Maven/npm → Artifact → Docker → Registry → Kubernetes will leave a strong impression in junior DevOps interviews.
