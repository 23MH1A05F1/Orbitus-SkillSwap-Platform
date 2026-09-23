pipeline {

    agent any

    environment {
        DOCKERHUB_USERNAME = 'gowthamibotsa870'

        BACKEND_IMAGE = 'gowthamibotsa870/orbitus-backend'
        FRONTEND_IMAGE = 'gowthamibotsa870/orbitus-frontend'
    }

    stages {

        // =========================================================
        // 1. CHECKOUT
        // =========================================================
        stage('Checkout') {
            steps {
                echo 'Checking out Orbitus SkillSwap project...'

                bat '''
                    echo ========================================
                    echo ORBITUS SKILLSWAP
                    echo ========================================

                    echo.
                    echo Current Workspace:
                    echo %CD%

                    echo.
                    echo Project Files:
                    dir
                '''
            }
        }


        // =========================================================
        // 2. VERIFY PROJECT STRUCTURE
        // =========================================================
        stage('Verify Project') {
            steps {
                echo 'Checking project structure...'

                bat '''
                    echo ========================================
                    echo VERIFY PROJECT STRUCTURE
                    echo ========================================

                    echo.
                    echo Checking Backend...

                    if exist "backend" (
                        echo Backend folder FOUND
                    ) else (
                        echo ERROR: backend folder NOT FOUND
                        exit /b 1
                    )

                    echo.
                    echo Checking Frontend...

                    if exist "frontend" (
                        echo Frontend folder FOUND
                    ) else (
                        echo ERROR: frontend folder NOT FOUND
                        exit /b 1
                    )

                    echo.
                    echo Checking Backend Dockerfile...

                    if exist "backend\\Dockerfile" (
                        echo Backend Dockerfile FOUND
                    ) else (
                        echo ERROR: backend\\Dockerfile NOT FOUND
                        exit /b 1
                    )

                    echo.
                    echo Checking Frontend Dockerfile...

                    if exist "frontend\\Dockerfile" (
                        echo Frontend Dockerfile FOUND
                    ) else (
                        echo ERROR: frontend\\Dockerfile NOT FOUND
                        exit /b 1
                    )

                    echo.
                    echo Project structure verification PASSED
                '''
            }
        }


        // =========================================================
        // 3. CHECK DOCKER
        // =========================================================
        stage('Check Docker') {
            steps {
                echo 'Checking Docker installation...'

                bat '''
                    echo ========================================
                    echo DOCKER CHECK
                    echo ========================================

                    docker --version

                    echo.
                    echo Docker Compose:
                    docker compose version

                    echo.
                    echo Docker Info:
                    docker info > nul

                    if %ERRORLEVEL% EQU 0 (
                        echo Docker is running successfully
                    ) else (
                        echo ERROR: Docker is not running
                        exit /b 1
                    )
                '''
            }
        }


        // =========================================================
        // 4. BUILD BACKEND IMAGE
        // =========================================================
        stage('Build Backend Image') {
            steps {
                echo 'Building Orbitus Backend Docker image...'

                dir('backend') {

                    bat '''
                        echo ========================================
                        echo BUILDING BACKEND IMAGE
                        echo ========================================

                        docker build ^
                        -t %BACKEND_IMAGE%:latest ^
                        -t %BACKEND_IMAGE%:%BUILD_NUMBER% ^
                        .

                        if %ERRORLEVEL% NEQ 0 (
                            echo Backend Docker build FAILED
                            exit /b 1
                        )

                        echo.
                        echo Backend Docker image built successfully
                    '''
                }
            }
        }


        // =========================================================
        // 5. BUILD FRONTEND IMAGE
        // =========================================================
        stage('Build Frontend Image') {
            steps {
                echo 'Building Orbitus Frontend Docker image...'

                dir('frontend') {

                    bat '''
                        echo ========================================
                        echo BUILDING FRONTEND IMAGE
                        echo ========================================

                        docker build ^
                        -t %FRONTEND_IMAGE%:latest ^
                        -t %FRONTEND_IMAGE%:%BUILD_NUMBER% ^
                        .

                        if %ERRORLEVEL% NEQ 0 (
                            echo Frontend Docker build FAILED
                            exit /b 1
                        )

                        echo.
                        echo Frontend Docker image built successfully
                    '''
                }
            }
        }


        // =========================================================
        // 6. SHOW DOCKER IMAGES
        // =========================================================
        stage('Verify Docker Images') {
            steps {
                echo 'Verifying Docker images...'

                bat '''
                    echo ========================================
                    echo ORBITUS DOCKER IMAGES
                    echo ========================================

                    docker images | findstr /I "orbitus-backend orbitus-frontend"
                '''
            }
        }


        // =========================================================
        // 7. LOGIN TO DOCKER HUB
        // =========================================================
        stage('Docker Hub Login') {
            steps {

                echo 'Logging into Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    bat '''
                        echo ========================================
                        echo DOCKER HUB LOGIN
                        echo ========================================

                        echo %DOCKER_PASSWORD% | docker login ^
                        -u %DOCKER_USERNAME% ^
                        --password-stdin

                        if %ERRORLEVEL% NEQ 0 (
                            echo Docker Hub login FAILED
                            exit /b 1
                        )

                        echo.
                        echo Docker Hub login successful
                    '''
                }
            }
        }


        // =========================================================
        // 8. PUSH BACKEND IMAGE
        // =========================================================
        stage('Push Backend Image') {
            steps {

                echo 'Pushing backend image to Docker Hub...'

                bat '''
                    echo ========================================
                    echo PUSH BACKEND IMAGE
                    echo ========================================

                    docker push %BACKEND_IMAGE%:latest

                    if %ERRORLEVEL% NEQ 0 (
                        echo Backend latest image push FAILED
                        exit /b 1
                    )

                    docker push %BACKEND_IMAGE%:%BUILD_NUMBER%

                    if %ERRORLEVEL% NEQ 0 (
                        echo Backend version image push FAILED
                        exit /b 1
                    )

                    echo.
                    echo Backend images pushed successfully
                '''
            }
        }


        // =========================================================
        // 9. PUSH FRONTEND IMAGE
        // =========================================================
        stage('Push Frontend Image') {
            steps {

                echo 'Pushing frontend image to Docker Hub...'

                bat '''
                    echo ========================================
                    echo PUSH FRONTEND IMAGE
                    echo ========================================

                    docker push %FRONTEND_IMAGE%:latest

                    if %ERRORLEVEL% NEQ 0 (
                        echo Frontend latest image push FAILED
                        exit /b 1
                    )

                    docker push %FRONTEND_IMAGE%:%BUILD_NUMBER%

                    if %ERRORLEVEL% NEQ 0 (
                        echo Frontend version image push FAILED
                        exit /b 1
                    )

                    echo.
                    echo Frontend images pushed successfully
                '''
            }
        }


        // =========================================================
        // 10. FINAL VERIFICATION
        // =========================================================
        stage('Final Verification') {
            steps {

                echo 'Performing final verification...'

                bat '''
                    echo ========================================
                    echo ORBITUS CI/CD FINAL VERIFICATION
                    echo ========================================

                    echo.
                    echo Backend Image:
                    docker images %BACKEND_IMAGE%

                    echo.
                    echo Frontend Image:
                    docker images %FRONTEND_IMAGE%

                    echo.
                    echo ========================================
                    echo BUILD NUMBER: %BUILD_NUMBER%
                    echo ========================================

                    echo.
                    echo Docker Hub repositories:
                    echo %BACKEND_IMAGE%
                    echo %FRONTEND_IMAGE%

                    echo.
                    echo CI/CD PIPELINE COMPLETED SUCCESSFULLY
                '''
            }
        }
    }


    // =============================================================
    // POST ACTIONS
    // =============================================================
    post {

        success {
            echo '''
            ================================================
            ORBITUS CI/CD PIPELINE SUCCESS
            ================================================

            GitHub Checkout       : SUCCESS
            Project Verification  : SUCCESS
            Docker Check          : SUCCESS
            Backend Build         : SUCCESS
            Frontend Build        : SUCCESS
            Docker Hub Login      : SUCCESS
            Backend Push          : SUCCESS
            Frontend Push         : SUCCESS

            ================================================
            '''
        }

        failure {
            echo '''
            ================================================
            ORBITUS CI/CD PIPELINE FAILED
            ================================================

            Check the failed stage in the Jenkins console.

            ================================================
            '''
        }

        always {
            echo "Build Number: ${env.BUILD_NUMBER}"
            echo "Workspace: ${env.WORKSPACE}"
        }
    }
}
