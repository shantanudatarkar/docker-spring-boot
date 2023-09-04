pipeline {
    agent any

    environment {
        registry = "333920746455.dkr.ecr.ap-southeast-2.amazonaws.com/helm-repo"
        helmChartPath = "/home/shantanu/docker-spring-boot/my-helm-chart"
        imageName = "shantanu/shantanu"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/shantanudatarkar/docker-spring-boot.git']])
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
            steps {
                withCredentials([[
                    $class: 'UsernamePasswordMultiBinding',
                    usernameVariable: 'GIT_USERNAME',
                    passwordVariable: 'GIT_PASSWORD',
                    credentialsId: 'Github_id'
                ]]) {
                    script {
                        // Get the current build number
                        def BUILD_NUMBER = env.BUILD_NUMBER

                        // Configure Git user and email
                        sh """
                            git config user.email "shan6101995@gmail.com"
                            git config user.name "shantanudatarkar"
                        """

                        // Create the Helm chart directory if it doesn't exist
                        sh "sudo mkdir -p ${helmChartPath}"

                        // Update the image tag in values.yaml
                        writeFile(file: "${helmChartPath}/values.yaml", text: "image: ${imageName}:${BUILD_NUMBER}\n")

                        // Change to the Helm chart directory
                        // Add and commit changes
                        sh """
                            cd ${helmChartPath}
                            git add --all
                            git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                        """
                    }
                }
            }
        }
    }
}
