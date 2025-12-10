pipeline {
    agent any

    environment {
        GIT_CREDENTIALS = 'github-pat'
        DOCKERHUB_CREDENTIALS = 'docker-hub-creds'
        DOCKER_IMAGE = "borgimoatez123/student-management"
        KUBE_CREDENTIALS = "kubeconfig-cred"
        K8S_NAMESPACE = "devops"
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/borgimoatez123/borgimotez_-4selam2_devops.git',
                    branch: 'main',
                    credentialsId: GIT_CREDENTIALS
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:latest .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS,
                usernameVariable: 'DOCKERHUB_USER',
                passwordVariable: 'DOCKERHUB_PASS')]) {

                    sh '''
                        echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push $DOCKER_IMAGE:latest
                    '''
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        /* -------------- NEW ADDED STAGE ---------------- */
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: KUBE_CREDENTIALS]) {

                    sh '''
                        echo "📌 Applying Kubernetes resources..."

                        # Deploy MySQL (optional if already deployed)
                        kubectl apply -f k8s/mysql-secret.yaml -n devops || true
                        kubectl apply -f k8s/mysql-pv-pvc.yaml -n devops || true
                        kubectl apply -f k8s/mysql-deployment.yaml -n devops || true
                        kubectl apply -f k8s/mysql-service.yaml -n devops || true

                        # Deploy Spring Boot App
                        kubectl apply -f k8s/springboot-deployment.yaml -n devops
                        kubectl apply -f k8s/springboot-service.yaml -n devops

                        echo "🚀 Kubernetes deployment DONE"
                    '''
                }
            }
        }
        /* ----------------------------------------------- */

    }

    post {
        success { echo "✔ BUILD + DOCKER PUSH + K8S DEPLOY SUCCESSFUL" }
        failure { echo "❌ PIPELINE FAILED" }
    }
}
