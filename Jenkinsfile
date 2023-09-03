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
                script {
                    // Set the ECR repository URL and image name
                    def ecrRepoUrl = '333920746455.dkr.ecr.ap-southeast-2.amazonaws.com'
                    def imageName = "${ecrRepoUrl}/helm-repo:${BUILD_NUMBER}"

                    // Authenticate with AWS ECR using Jenkins credentials
                    withCredentials([aws(credentials: 'YourAWSAccessKeyCredentialsId', region: 'ap-southeast-2')]) {
                        // Authenticate Docker with ECR
                        sh "aws ecr get-login-password --region ap-southeast-2 | docker login --username AWS --password-stdin ${ecrRepoUrl}"

                        // Tag your Docker image with the ECR repository URL
                        sh "docker tag ${registry}:${BUILD_NUMBER} ${imageName}"

                        // Push the Docker image to ECR
                        sh "docker push ${imageName}"
                    }
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
