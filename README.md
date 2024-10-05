# DevSecOps Project - OpenAI Chatbot UI Deployment

This project demonstrates the implementation of DevSecOps practices for deploying an OpenAI Chatbot UI using modern CI/CD tools and security integrations. The Chatbot UI is deployed on Minikube, utilizing Jenkins for automation, Docker for containerization, and Kubernetes for orchestration.

![image](https://github.com/user-attachments/assets/5d4389e0-90e6-48e8-8344-6792e3c8a835)

## Tools & Technologies Used

- **CI/CD Tool**: Jenkins
- **Security Tools**:
  - Trivy (Filesystem & Image Scanning)
  - SonarQube (Code Quality & Security Analysis)
  - OWASP Dependency Check (Vulnerability Detection)
- **Containerization**: Docker
- **Orchestration**: Kubernetes (Minikube)
- **Version Control System**: Git and GitHub

## CI/CD Workflow with Jenkins

The workflow automates the build, test, and deployment process of the OpenAI Chatbot UI application. It integrates security scanning tools to ensure the robustness and security of the deployment workflow.

![image](https://github.com/user-attachments/assets/53670879-5b5b-45ad-b875-5f2c5dc6fdfa)

### Key Workflow Stages:

1. **Build & Test**: Automated builds and tests are executed in Jenkins for the Chatbot UI.
2. **Code Quality & Security Analysis**: SonarQube checks the code for bugs, code smells, and security vulnerabilities.
3. **Filesystem & Image Scanning**: Trivy performs filesystem and Docker image scans to identify vulnerabilities.
4. **Vulnerability Detection**: OWASP Dependency Check detects known vulnerabilities in the project dependencies.
5. **Docker Build & Push**: Jenkins automates Docker image creation and pushes it to a Docker registry.
6. **Deployment to Minikube**: The Dockerized application is deployed to Minikube for local Kubernetes orchestration and testing.

## Security Tools Integration

### SonarQube Analysis

SonarQube is integrated into Jenkins to provide continuous inspection of code quality and security. It detects bugs, vulnerabilities, and code smells to ensure that only high-quality code is deployed.

![image](https://github.com/user-attachments/assets/87ae33a0-fe38-4d9a-83cf-7a7801769e56)

### Trivy File System & Image Scans

Trivy scans the file system and Docker images for vulnerabilities. Any issues found are reported, and the deployment process is paused if critical vulnerabilities are detected.

![image](https://github.com/user-attachments/assets/1788d604-27a7-4b95-b492-d90b95d4aeaf)

### OWASP Dependency Check

OWASP Dependency Check analyzes the project's dependencies to ensure no known vulnerabilities are present. If any critical issues are identified, the deployment process halts.

![image](https://github.com/user-attachments/assets/362e03a5-fd1b-4105-b035-d88a2706a28c)

## Deployment with Docker and Kubernetes (Minikube)

### Docker Build & Push

The OpenAI Chatbot UI is containerized using Docker. The Docker image is built and then pushed to a container registry (e.g., Docker Hub) using Jenkins.

![image](https://github.com/user-attachments/assets/8e1b8ef3-2f80-4f23-b410-7f31fe5b4d65)

### Kubernetes Deployment (Minikube)

The application is deployed on Minikube, a local Kubernetes cluster, which provides a lightweight and efficient environment for testing the containerized application.

![image](https://github.com/user-attachments/assets/86652e11-f0b6-4195-bd2b-5198e2659465)

## Accessing the Application

After deploying the application to Minikube, you can access it using the `kubectl port-forward` command to forward the local port to the service port on the Kubernetes cluster.

![image](https://github.com/user-attachments/assets/220594b4-f852-4e32-885d-221f27f01543)


```bash
pipeline {
    agent any
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        TRIVY_HOME = "<ADD_TRIVY_PATH>"
        NPM_HOME = "<NODE_PATH>"
        NODE = tool 'node16'
        DOCKER = "<DOCKER_PATH>"
        KUBECTL_HOME = "<KUBECTL_PATH>"
        API_KEY = "<API_KEY>"
    }

    stages {
        stage('Git') {
            steps {
                git branch: 'main', credentialsId: '371fd4b1-3e74-4a3e-8898-06c624b0ab2d', url: 'https://github.com/TanmayRao7/Chatgpt-Clone'
            }
        }
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectName=chatgpt -Dsonar.projectKey=chatgpt'
                }
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh '${TRIVY_HOME}/trivy fs . > fs_scan.txt'
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'checker'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('API KEY') {
            steps {
                sh 'echo OPENAI_API_KEY=${API_KEY} > .env.local'
            }
        }
        stage('Docker Build') {
            steps {
              sh '${DOCKER}/docker build -t tanmayrao7/chatbot:latest .'
            }
        }
        stage('Docker Push') {
            steps {
              sh '${DOCKER}/docker push tanmayrao7/chatbot:latest '
            }
        }
        stage('Trivy Image Scan') {
            steps {
              sh '${TRIVY_HOME}/trivy  image tanmayrao7/chatbot:latest > image_scan.txt'
            }
        }
        stage('Deploy K8s') {
            steps {
              sh 'cd k8s && ${KUBECTL_HOME}/kubectl apply -f chatbot-ui.yaml'
            }
        }
    }
}
