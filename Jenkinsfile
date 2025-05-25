pipeline {
    agent any

    environment {
        AWS_REGION = 'us-west-2'
        REPO_NAME = 'springboot'
        AWS_ACCOUNT_ID = credentials('aws-account-id')  // AWS Account ID as Secret Text
        // Set JAVA_HOME here if you want globally (optional)
        // JAVA_HOME = 'C:\\Program Files\\Java\\jdk-17.0.4'
    }

    stages {
        stage('Clone Git Repo') {
            steps {
                git 'https://github.com/nagalakshmivemparala/HelloWorld-Springboot-App.git'
            }
        }

        stage('Build with Maven') {
            steps {
                bat '''
                set JAVA_HOME=C:\\Program Files\\Java\\jdk-17.0.4
                set PATH=%JAVA_HOME%\\bin;%PATH%
                mvn clean package
                '''
                archiveArtifacts artifacts: '**/target/*.war', fingerprint: true
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t springboot .'
            }
        }

        stage('Tag Docker Image') {
            steps {
                bat "docker tag springboot:latest %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com/%REPO_NAME%:latest"
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    bat """
                    aws configure set aws_access_key_id %AWS_ACCESS_KEY_ID%
                    aws configure set aws_secret_access_key %AWS_SECRET_ACCESS_KEY%
                    aws configure set region %AWS_REGION%

                    FOR /F "tokens=*" %%i IN ('aws ecr get-login-password --region %AWS_REGION%') DO docker login --username AWS --password %%i %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com
                    """
                }
            }
        }

        stage('Push to ECR') {
            steps {
                bat "docker push %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com/%REPO_NAME%:latest"
            }
        }
    }
}
