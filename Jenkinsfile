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
                        docker tag ${registry}:${BUILD_NUMBER} 333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo:${BUILD_NUMBER}
                        docker push 333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo:${BUILD_NUMBER}
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
                withCredentials([gitUsernamePassword(credentialsId: 'Github_id', gitToolName: 'Default')]) {
                    withCredentials([usernameColonPassword(credentialsId: 'Github_id', variable: 'Github')]) {

                    sh '''
                        git config user.email "shan6101995@gmail.com"
                        git config user.name "shantanudatarkar"
                        BUILD_NUMBER=${BUILD_NUMBER}
                        sed -i "s|image: .*|image: shantanu/shantanu:${BUILD_NUMBER}|g" /home/shantanu/docker-spring-boot/my-helm-chart/values.yaml
                        git add --all
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
        }
     }
  }
}
          
