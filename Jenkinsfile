
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    tools {
        maven 'maven3'
        jdk 'jdk21'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKERHUB_USERNAME = 'ranjithjrdocker'
        DOCKER_IMAGE = "${DOCKERHUB_USERNAME}/spotify-app:latest"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/RanjithVr98/SonarQube-Project-Kastro.git'
            }
        }

        stage('Build and Test') {
            steps {
                bat 'mvn clean verify'
            }
        }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    bat '''
                        "%SCANNER_HOME%\\bin\\sonar-scanner.bat" ^
                        -Dsonar.projectName=Ranjith ^
                        -Dsonar.projectKey=RanjithKey ^
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate(
                            abortPipeline: true
                        )
                        echo "Quality Gate Status: ${qg.status}"
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE% .'
            }
        }

        stage('Docker Push to DockerHub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'Dockerhub',
                        usernameVariable: 'REGISTRY_USER',
                        passwordVariable: 'REGISTRY_PASSWORD'
                    )
                ]) {
                    bat '''
                        @echo off
                        echo %REGISTRY_PASSWORD% | docker login --username %REGISTRY_USER% --password-stdin
                        if errorlevel 1 exit /b 1

                        docker push %DOCKER_IMAGE%
                        if errorlevel 1 exit /b 1
                    '''
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                bat '''
                    docker rm -f spotify-app 2>nul
                    docker run -d --name spotify-app -p 5555:5555 %DOCKER_IMAGE%
                    if errorlevel 1 exit /b 1
                '''
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}
