pipeline {
    agent any

    environment {
        // ===== CREDENTIALS =====
        GIT_CREDENTIALS        = 'github-pat'
        DOCKERHUB_CREDENTIALS = 'docker-hub-creds'
        KUBE_CREDENTIALS      = 'kubeconfig-cred'

        // ===== PROJECT CONFIG =====
        DOCKER_IMAGE  = 'borgimoatez123/student-management'
        K8S_NAMESPACE = 'devops'
    }

    stages {

        // =========================
        // 📥 CHECKOUT SOURCE CODE
        // =========================
        stage('Checkout') {
            steps {
                git url: 'https://github.com/borgimoatez123/borgimotez_-4selam2_devops.git',
                    branch: 'main',
                    credentialsId: GIT_CREDENTIALS
            }
        }

        // =========================
        // 🧪 MAVEN BUILD
        // =========================
        stage('Maven Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }

        // =========================
        // 🔎 SONARQUBE ANALYSIS
        // (Community Edition compatible)
        // =========================
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=student-management \
                        -Dsonar.projectName="Student Management"
                    '''
                }
            }
        }

        // =========================
        // 🐳 DOCKER BUILD
        // =========================
        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:latest .
                '''
            }
        }

        // =========================
        // 🚀 DOCKER PUSH
        // =========================
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKERHUB_CREDENTIALS,
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_PASS'
                )]) {
                    sh '''
                        echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        // =========================
        // ☸️ DEPLOY TO KUBERNETES
        // =========================
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(
                    credentialsId: KUBE_CREDENTIALS,
                    variable: 'KUBECONFIG'
                )]) {
                    sh '''
                        echo "🚀 Deploying to Kubernetes namespace: devops"

                        kubectl apply -f k8s/springboot-deployment.yaml -n devops
                        kubectl apply -f k8s/springboot-service.yaml -n devops

                        kubectl rollout status deployment/springboot-app -n devops

                        echo "✅ Deployment completed successfully"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ FULL PIPELINE SUCCESS"
        }
        failure {
            echo "❌ PIPELINE FAILED"
        }
    }
}
