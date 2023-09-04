pipeline {
    agent any

    environment {
        registry = "333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo"
        helmChartPath = "/var/lib/jenkins/workspace/Helm-pipeline/spring-boot/"
        imageName = "shantanu/shantanu"
        // Use the Jenkins build number dynamically
        buildNumber = "${env.BUILD_NUMBER}"
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
                    def imageFullName = "${registry}:${buildNumber}"
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
                        docker tag ${env.IMAGE_NAME} 333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo:${buildNumber}
                        docker push 333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo:${buildNumber}
                    """
                }
            }
        }

       stage("Update Deployment File") {
    steps {
        script {
            def filePath = "${helmChartPath}/values.yaml" // Adjust the path if needed
            def buildNumber = "${env.BUILD_NUMBER}"  // Enclose BUILD_NUMBER in double quotes

            sh """
                 git config user.email "shan6101995@gmail.com"
                 git config user.name "shantanudatarkar"
                 sed -i "s|tag: 'REPLACE_ME'|tag: '${buildNumber}'|" ${filePath}
                 git -C ${helmChartPath} add --all
                 git -C ${helmChartPath} commit -m "Update deployment image to version ${buildNumber}"
                 git -C ${helmChartPath} push https://${GITHUB_USER}:${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:master
                 cat ${filePath}  // Print the updated file
              """
                }
            }
        }
    }
}
