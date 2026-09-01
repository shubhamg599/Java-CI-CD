pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarCloud Scan') {
            steps {
                withCredentials([string(
                    credentialsId: 'sonarqube-token',
                    variable: 'SONAR_TOKEN'
                )]) {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=$SONAR_TOKEN -Dsonar.projectKey=shubhamg599_Java-CI-CD -Dsonar.organization=shubhamg599'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t jenkins-ci-cd:${BUILD_NUMBER} .'
            }
        }

        stage('Push to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS-Creds'
                ]]) {
                    sh 'aws --version'
                    sh 'aws sts get-caller-identity'

                    sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 658860694489.dkr.ecr.us-east-1.amazonaws.com'

                    sh 'docker tag jenkins-ci-cd:${BUILD_NUMBER} 658860694489.dkr.ecr.us-east-1.amazonaws.com/jenkins-ci-cd:${BUILD_NUMBER}'

                    sh 'docker push 658860694489.dkr.ecr.us-east-1.amazonaws.com/jenkins-ci-cd:${BUILD_NUMBER}'
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    set -e

                    echo "Updating kubeconfig..."
                    aws eks update-kubeconfig \
                        --region us-east-1 \
                        --name Jenkins-CI-CD-demo

                    echo "Updating image tag..."
                    sed -i "s|IMAGE_TAG|${BUILD_NUMBER}|g" k8s/deployment.yaml

                    echo "Deploying application..."
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    echo "Waiting for rollout..."
                    kubectl rollout status deployment/java-app --timeout=5m

                    echo "Pods:"
                    kubectl get pods

                    echo "Service:"
                    kubectl get service java-app
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check the console output.'
        }
    }
}
