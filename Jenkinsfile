pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = "ramsai333"
        IMAGE_NAME = "springboot-app"
        IMAGE_TAG = "latest"
        FULL_IMAGE = "${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ramsai17/Springboot-project.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${FULL_IMAGE} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${FULL_IMAGE}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    # Update image in deployment
                    sed -i "s|image:.*|image: ${FULL_IMAGE}|g" k8s/deployment.yaml
                    kubectl apply -f k8s/deployment.yaml
                    kubectl rollout status deployment/springboot-app
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! Application deployed to Kubernetes."
        }
        failure {
            echo "Pipeline failed. Please check the logs."
        }
    }
}
