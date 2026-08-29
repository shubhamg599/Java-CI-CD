pipeline {
agent any

```
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
                sh '''
                    mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                    -Dsonar.host.url=https://sonarcloud.io \
                    -Dsonar.token=$SONAR_TOKEN \
                    -Dsonar.projectKey=shubhamg599_Java-CI-CD \
                    -Dsonar.organization=shubhamg599
                '''
            }
        }
    }

    stage('Docker Build') {
        steps {
            sh '''
                docker build -t jenkins-ci-cd:${BUILD_NUMBER} .
            '''
        }
    }

    stage('Push to ECR') {
        steps {
            withCredentials([
                [$class: 'AmazonWebServicesCredentialsBinding',
                 credentialsId: 'aws-ecr-credentials']
            ]) {
                sh '''
                    set -eux

                    echo "Checking AWS CLI..."
                    aws --version

                    echo "Checking AWS identity..."
                    aws sts get-caller-identity

                    echo "Logging in to Amazon ECR..."
                    aws ecr get-login-password --region us-east-1 | \
                    docker login --username AWS --password-stdin \
                    658860694489.dkr.ecr.us-east-1.amazonaws.com

                    echo "Tagging Docker image..."
                    docker tag \
                    jenkins-ci-cd:${BUILD_NUMBER} \
                    658860694489.dkr.ecr.us-east-1.amazonaws.com/jenkins-ci-cd:${BUILD_NUMBER}

                    echo "Pushing Docker image to ECR..."
                    docker push \
                    658860694489.dkr.ecr.us-east-1.amazonaws.com/jenkins-ci-cd:${BUILD_NUMBER}
                '''
            }
        }
    }
}

post {
    success {
        echo 'CI/CD Pipeline completed successfully!'
    }

    failure {
        echo 'CI/CD Pipeline failed. Check the stage logs above for the exact error.'
    }
}
```

}
