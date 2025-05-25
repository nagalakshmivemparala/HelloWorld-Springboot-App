pipeline {
    agent any

    environment {
        AWS_REGION = 'us-west-2'
        REPO_NAME = 'springboot'
    }

    stages {
        stage('Set ECR Repo URL') {
            steps {
                withCredentials([string(credentialsId: 'aws-account-id', variable: 'AWS_ACCOUNT_ID')]) {
                    script {
                        env.ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.REPO_NAME}"
                    }
                }
            }
        }

        stage('Clone Git Repo') {
            steps {
                git 'https://github.com/nagalakshmivemparala/HelloWorld-Springboot-App.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t springboot .'
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh 'docker tag springboot:latest $ECR_REPO:latest'
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'aws-credentials', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set region $AWS_REGION

                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Push to ECR') {
            steps {
                sh 'docker push $ECR_REPO:latest'
            }
        }
    }
}
