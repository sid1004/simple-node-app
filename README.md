# Simple Node App – CI Pipeline with Jenkins & SonarQube

This project demonstrates a complete CI (Continuous Integration) pipeline using **Jenkins** and **SonarQube**, running on local Docker containers. The pipeline clones source code from GitHub, performs static code analysis using SonarQube, and builds a Docker image of the application.

---

## ✅ What We Did (Step-by-Step Summary)

### 1) Setup Jenkins on Local
- Jenkins was started using a Docker container mapped on port **8080**.
- A persistent volume was used (`jenkins_home`) to store Jenkins configurations.
- Jenkins serves as the CI tool to automate code build and testing workflows.

### 2) Setup SonarQube on Local
- SonarQube LTS was run via Docker on port **9000**.
- A **SonarQube token** was generated for Jenkins authentication.
- SonarQube is used to analyze code quality, detect bugs, vulnerabilities, and maintainability issues.

### 3) Application Source Code (This Repository)
- A simple **Node.js** sample application is stored in this repository.
- Code was pushed to GitHub to act as the source for Jenkins.
- The application contains a `Dockerfile` for container image creation.

### 4) Connect GitHub Repository with Jenkins
- A Jenkins Pipeline job was created and configured to pull code from this repository’s `main` branch.
- The pipeline automatically fetches the latest code on every build run.

### 5) Integrate SonarQube with Jenkins
- SonarQube server details were configured in Jenkins under **Manage Jenkins → System**.
- **SonarScanner** was installed via **Global Tool Configuration**.
- The generated SonarQube token was stored securely in **Jenkins Credentials** and referenced in the pipeline.

### 6) Create and Execute the Jenkins Declarative Pipeline
The pipeline contains **3 stages**:

| Stage | Purpose |
|------|---------|
| **Clone Code** | Pull latest application code from GitHub |
| **Sonar Scan** | Analyze code quality using SonarQube |
| **Build Docker Image** | Build container image for the Node app |

---

## 🧱 Jenkins Pipeline (Jenkinsfile)

```groovy
pipeline {
  agent any

  stages {
    stage("Clone Code") {
      steps {
        git url: "https://github.com/sidharth1004/simple-node-app.git", branch: "main"
      }
    }

    stage("Sonar Scan") {
      steps {
        withSonarQubeEnv('sonarqube') {
          withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
            withEnv(["PATH+SCANNER=${tool 'sonar-scanner'}/bin"]) {
              sh """
                sonar-scanner \
                  -Dsonar.projectKey=simple-node-app \
                  -Dsonar.sources=. \
                  -Dsonar.login=$SONAR_AUTH_TOKEN
              """
            }
          }
        }
      }
    }

    stage("Build Docker Image") {
      steps {
        sh "docker build -t simple-node-app ."
      }
    }

    stage("Deploy Container") {
      steps {
        sh '''
        docker rm -f simple-node-app || true
        docker run -d --name simple-node-app -p 3000:3000 simple-node-app
        '''
      }
    }
  }
}
