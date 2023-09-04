pipeline {
    agent any

    environment {
        registry = "333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo"
        helmChartPath = "/var/lib/jenkins/workspace/Helm-pipeline/spring-boot/"
        imageName = "shantanu/shantanu"
        buildNumber = "${BUILD_NUMBER}" // Store BUILD_NUMBER in an environment variable
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
                    // Build the Docker image with a unique tag
                    def imageFullName = "${registry}:${env.buildNumber}"
                    docker.build(imageFullName)
                    env.IMAGE_NAME = imageFullName
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
                        docker tag ${env.IMAGE_NAME} 333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo:${env.buildNumber}
                        docker push 333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo:${env.buildNumber}
                    """
                }
            }
        }

        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "docker-spring-boot"
                GIT_USER_NAME = "shantanudatarkar"
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'Github_id', usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_TOKEN')]) {

                   sh '''
                       git config user.email "shan6101995@gmail.com"
                       git config user.name "shantanudatarkar"
                       sed -i "s|tag: 'REPLACE_ME'|tag: '${env.buildNumber}'|" ${helmChartPath}/values.yaml
                       git -C ${helmChartPath} add --all
                       git -C ${helmChartPath} commit -m "Update deployment image to version ${env.buildNumber}"
                       git -C ${helmChartPath} push https://${GITHUB_USER}:${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:master
                      '''
                }
            }
        }
     }
}
