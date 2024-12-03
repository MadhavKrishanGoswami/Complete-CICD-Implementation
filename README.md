# Jenkins Pipeline for Java-Based Application  

This project demonstrates a **CI/CD pipeline** for a Java-based application using **Jenkins**, **Maven**, **SonarQube**, **Argo CD**, **Helm**, and **Kubernetes**. The pipeline ensures automated building, testing, static code analysis, Docker image building, and deployment to Kubernetes.  

---

## **Features**  

1. **Automated Build and Test**:  
   - Utilizes Maven for dependency management, compilation, and running tests.  

2. **Static Code Analysis**:  
   - Integrates with **SonarQube** for code quality and security checks.  

3. **Containerization**:  
   - Builds a Docker image of the application.  

4. **Deployment to Kubernetes**:  
   - Uses **Helm** for Kubernetes manifest management.  
   - Deploys to a cluster managed by **Argo CD**.  

5. **Integration with Jenkins**:  
   - Defines stages for end-to-end automation.  

---

## **Technologies Used**  

| Technology  | Purpose                                                   |
|-------------|-----------------------------------------------------------|
| Jenkins     | CI/CD orchestration.                                      |
| Maven       | Build tool for Java projects.                             |
| SonarQube   | Static code analysis and code quality checks.             |
| Docker      | Containerization of the application.                      |
| Kubernetes  | Orchestrates deployment and scaling of the application.   |
| Helm        | Package manager for Kubernetes manifests.                 |
| Argo CD     | GitOps tool for continuous delivery to Kubernetes.        |  

---

## **Pipeline Workflow**  

### **Stages**  

1. **Checkout Code**  
   - Clones the repository from version control.  

2. **Build and Test**  
   - Uses Maven to clean, compile, and test the application.  

3. **Static Code Analysis**  
   - Runs SonarQube analysis to assess code quality and identify vulnerabilities.  

4. **Build and Push Docker Image**  
   - Builds a Docker image for the application.  
   - Pushes the image to a Docker registry (e.g., DockerHub).  

5. **Update Deployment Configuration**  
   - Updates Kubernetes deployment manifests with the new image tag.  

6. **Deploy to Kubernetes**  
   - Uses Helm and Argo CD to deploy the updated application to the Kubernetes cluster.  

---

## **Prerequisites**  

1. **Infrastructure Requirements**  
   - Jenkins installed and configured with Docker support.  
   - SonarQube server running and accessible.  
   - Kubernetes cluster with Argo CD installed.  

2. **Tools Setup**  
   - Jenkins plugins: `Pipeline`, `Git`, `SonarQube Scanner`.  
   - Docker CLI installed and accessible.  
   - Helm CLI configured to interact with the Kubernetes cluster.  

3. **Credentials**  
   - Docker Hub credentials stored in Jenkins (`docker-cred`).  
   - GitHub token stored in Jenkins (`github-token`).  
   - Argo CD credentials stored securely.  

---

## **Jenkins Pipeline Configuration**  

Below is an outline of the Jenkinsfile:  

```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.6.3-openjdk-17'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    environment {
        SONAR_URL = 'http://<sonarqube-server>:9000/'
        DOCKER_IMAGE = 'your-dockerhub-repo/java-app:${BUILD_NUMBER}'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build and Test') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Static Code Analysis') {
            steps {
                sh '''
                    mvn sonar:sonar \
                    -Dsonar.host.url=$SONAR_URL \
                    -Dsonar.login=<your-sonar-token>
                '''
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                sh '''
                    docker build -t $DOCKER_IMAGE .
                    echo <your-dockerhub-password> | docker login -u <your-dockerhub-username> --password-stdin
                    docker push $DOCKER_IMAGE
                '''
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    helm upgrade --install java-app ./helm-chart \
                    --set image.repository=$DOCKER_IMAGE \
                    --namespace app-namespace
                '''
                sh '''
                    argocd app sync java-app
                '''
            }
        }
    }
}
```  

---

## **How to Run the Pipeline**  

1. **Clone the Repository**:  
   ```bash
   git clone <repo-url>
   cd <project-directory>
   ```  

2. **Setup Jenkins**:  
   - Configure the pipeline using the provided Jenkinsfile.  
   - Ensure all required credentials are added to Jenkins.  

3. **Run the Pipeline**:  
   - Trigger the pipeline manually or via webhooks.  

---

## **Future Enhancements**  

- Add unit test coverage reports in SonarQube.  
- Integrate monitoring tools like Prometheus and Grafana.  
- Implement Canary or Blue/Green deployments using Kubernetes.  

---

## **License**  
This project is licensed under the MIT License.  

---  

Feel free to contribute and enhance the project! 🎉  
