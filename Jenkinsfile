pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk21'
        // Remove the sonarQube line if it still causes issues
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKERHUB_USERNAME = 'ranjithjrdocker'  // Set this directly to your Docker Hub username
        DOCKER_IMAGE = "${DOCKERHUB_USERNAME}/spotify-app:latest"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/RanjithVr98/SonarQube-Project-Kastro.git'
            }
        }

        stage('Compile') {
            steps {
                bat "mvn compile"
            }
        }

        stage('Test') {
            steps {
                bat "mvn test"
            }
        }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    bat "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Ranjith -Dsonar.projectKey=RanjithKey -Dsonar.java.binaries=target"
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
        
        stage('Build') {
            steps {
                bat "mvn package"
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    // Building Docker Image
                    bat "docker build -t $DOCKER_IMAGE ."
                }
            }
        }

        stage('Docker Push to DockerHub') {
            steps { 
                     withCredentials([usernamePassword(credentialsId: 'Dockerhub', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                         bat 'echo %DOCKERHUB_PASSWORD%| docker login -u %DOCKERHUB_USERNAME% --password-stdin'
                         bat "docker push ${DOCKER_IMAGE}"
                    }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    // Stop any existing container with the same name
                    bat 'docker rm -f spotify-app || exit /b 0'
                    
                    // Running the container
                    bat "docker run -d --name spotify-app -p 5555:5555 ${DOCKER_IMAGE}"
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()  // Clean up the workspace after the build
        }
    }
}
