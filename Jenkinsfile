pipeline {
    agent any

    environment {
        registry = "333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/akannan1087/docker-spring-boot']])
            }
        }
        
        stage("Build JAR") {
            steps {
                sh "mvn clean install"
            }
        }
        
        stage("Build Docker Image") {
            steps {
                script {
                    // Build the Docker image with a unique tag
                    docker.build("${registry}:${BUILD_NUMBER}")
                }
            }
        }

        stage("Push to ECR") {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',
                    credentialsId: 'aws_cred'
                ]]) {
                    sh """
                        aws ecr get-login-password --region ap-southeast-2 | docker login --username AWS --password-stdin 333920746455.dkr.ecr.ap-southeast-2.amazonaws.com
                        docker tag helm-repo:latest 333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo:${BUILD_NUMBER}
                        docker push 333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo:${BUILD_NUMBER}
                    """
                }
            }
        }
        
        stage("Helm package") {
            steps {
                sh "helm package springboot"
            }
        }
                
        stage("Helm install") {
            steps {
                sh "helm upgrade myrelease-21 springboot-0.1.0.tgz"
            }
        }
    }
}
