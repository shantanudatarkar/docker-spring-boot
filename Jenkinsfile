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
        
        stage ("Build JAR") {
            steps {
                sh "mvn clean install"
            }
        }
        
        stage ("Build Image") {
            steps {
                script {
                    docker.build("${registry}:${BUILD_NUMBER}")
                }
            }
        }
        
        stage ("Push to ECR") {
            steps {
                script {
                        withCredentials([string(credentialsId: 'aws_cred', variable: 'AWS_CREDENTIALS')]) {
                        sh """
                            echo \${AWS_CREDENTIALS} | aws configure --profile temp-profile set aws_access_key_id aws_secret_access_key
                            \$(aws ecr get-login --no-include-email --region ap-southeast-2)
                          """                    
                       sh "aws ecr get-login-password --region ap-southeast-2 | docker login --username AWS --password-stdin 333920746455.dkr.ecr.ap-southeast-2.amazonaws.com"
                       sh "docker tag helm-repo:latest 333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo:${BUILD_NUMBER}"
                       sh "docker push 333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo:${BUILD_NUMBER}"
   
                }
            }
        }
        
        stage ("Helm package") {
            steps {
                    sh "helm package springboot"
                }
            }
                
        stage ("Helm install") {
            steps {
                    sh "helm upgrade myrelease-21 springboot-0.1.0.tgz"
                }
            }
    
        }
    }
}        
