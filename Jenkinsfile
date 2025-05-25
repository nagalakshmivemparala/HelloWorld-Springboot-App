pipeline {
    agent any

    environment {
        AWS_REGION = 'us-west-2'
        REPO_NAME = 'springboot'
        AWS_ACCOUNT_ID = credentials('aws-account-id')  // AWS Account ID as Secret Text
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

        stage('Configure AWS CLI') {
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
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([
                    file(credentialsId: 'eks-kubeconfig', variable: 'KUBECONFIG_FILE')
                ]) {
                    bat '''
                    set KUBECONFIG=%KUBECONFIG_FILE%
                    kubectl apply -f deployment.yaml
                    '''
                }
            }
        }
    }
}
