pipeline {
    agent any

    environment {
        registry = "333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo"
        helmChartPath = "/var/lib/jenkins/workspace/Helm-pipeline/spring-boot"
        newImageTag = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/master']],
                    userRemoteConfigs: [[url: 'https://github.com/shantanudatarkar/docker-spring-boot.git']]])
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
                    def imageFullName = "${registry}:${newImageTag}"
                    docker.build(imageFullName)
                    env.IMAGE_NAME = imageFullName
                }
            }
        }

        stage("Push to ECR") {
            steps {
                withCredentials([
                    [
                        $class: 'AmazonWebServicesCredentialsBinding',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',
                        credentialsId: 'aws_cred'
                    ]
                ]) {
                    sh """
                        aws ecr get-login-password --region ap-southeast-2 | docker login --username AWS --password-stdin 333920746455.dkr.ecr.ap-southeast-2.amazonaws.com
                        docker tag ${env.IMAGE_NAME} 333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo:${newImageTag}
                        docker push 333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo:${newImageTag}
                    """
                }
            }
        }

        stage('Update Helm Chart') {
            steps {
                script {
                    sh "sed -i 's|tag: \"\"|tag: \"${newImageTag}\"|' ${helmChartPath}/values.yaml"
                }
            }
        }
    stage('Update Deployment File') {
    environment {
        GIT_REPO_NAME = "docker-spring-boot"
        GIT_USER_NAME = "shantanudatarkar"
    }
    steps {
        withCredentials([gitUsernamePassword(credentialsId: 'Github_id', gitToolName: 'Default')]) {
            withCredentials([usernameColonPassword(credentialsId: 'Github_id', variable: 'Github')]) {
                script {
                    sh """
                        git config user.email "shan6101995@gmail.com"
                        git config user.name "shantanudatarkar"
                        git add .
                        git commit -m "Update deployment image to version \${BUILD_NUMBER}"
                        git push https://\${GITHUB_TOKEN}@github.com/\${GIT_USER_NAME}/\${GIT_REPO_NAME} HEAD:master
                    """
                }
            }
        }
    }
}
    }
}
