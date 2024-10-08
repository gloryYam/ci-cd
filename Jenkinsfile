pipeline {
    agent any

    environment {
        repository = "gloryyam/ci_cd"
        DOCKERHUB_CREDENTIALS = credentials('DockerHub-Credentials')
        IMAGE_TAG = ""
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                git branch: 'main', url : "https://github.com/gloryYam/ci-cd"
            }
        }

        stage('Test') {
            steps {
                script {
                    sh "docker --version"
                    sh "docker compose --version"
                }
            }
        }

        stage('Set Image Tag') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        IMAGE_TAG = "1.0.${BUILD_NUMBER}"
                    } else {
                        IMAGE_TAG = "0.0.${BUILD_NUMBER}"
                    }
                    echo "Image tag set to: ${IMAGE_TAG}"
                }
            }
        }

        stage('Building Image') {
            steps {
                script {
                    sh "docker build -t ${repository}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DockerHub-Credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                }
            }
        }

        stage('Deploy Image') {
            steps {
                sh "docker push ${repository}:${IMAGE_TAG}"
            }
        }

        stage('Clean up') {
            steps {
                sh "docker rmi ${repository}:${IMAGE_TAG}"
            }
        }
    }

    post {
        always {
            cleanWs(cleanWhenNotBuilt: false,
            deleteDirs: true,
            disableDeferredWipeout: true,
            notFailBuild: true,
            patterns: [[pattern: '.gitignore', type: 'INCLUDE'], [pattern: '.propsfile', type: 'EXCLUDE']])
        }

        success {
            echo 'Build and deployment successful'
        }
        failure{
            echo 'Build and deployment failed'
        }
    }
}
