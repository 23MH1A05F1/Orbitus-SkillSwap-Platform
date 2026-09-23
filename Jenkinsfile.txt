pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'gowthamibotsa870'
        BACKEND_IMAGE = "${DOCKERHUB_USER}/orbitus-skillswap-backend"
        FRONTEND_IMAGE = "${DOCKERHUB_USER}/orbitus-skillswap-frontend"
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/23MH1A05F1/Orbitus-SkillSwap-Platform.git'
            }
        }

        stage('SonarQube Analysis - Backend') {
            steps {
                dir('backend') {
                    withSonarQubeEnv('SonarQube') {
                        bat 'sonar-scanner -Dsonar.projectKey=orbitus-backend -Dsonar.sources=.'
                    }
                }
            }
        }

        stage('SonarQube Analysis - Frontend') {
            steps {
                dir('frontend') {
                    withSonarQubeEnv('SonarQube') {
                        bat 'sonar-scanner -Dsonar.projectKey=orbitus-frontend -Dsonar.sources=.'
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Backend Image') {
            steps {
                dir('backend') {
                    bat "docker build -t ${BACKEND_IMAGE}:${IMAGE_TAG} -t ${BACKEND_IMAGE}:latest ."
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir('frontend') {
                    bat "docker build -t ${FRONTEND_IMAGE}:${IMAGE_TAG} -t ${FRONTEND_IMAGE}:latest ."
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat 'echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'
                    bat "docker push ${BACKEND_IMAGE}:${IMAGE_TAG}"
                    bat "docker push ${BACKEND_IMAGE}:latest"
                    bat "docker push ${FRONTEND_IMAGE}:${IMAGE_TAG}"
                    bat "docker push ${FRONTEND_IMAGE}:latest"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully — images pushed to Docker Hub.'
        }
        failure {
            echo 'Pipeline failed — check the stage logs above.'
        }
    }
}