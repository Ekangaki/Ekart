pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Ekangaki/Ekart.git'
            }
        }

        stage('Maven Compile') {
            steps {
                sh "mvn clean compile"
            }
        }

        stage('Maven Test') {
            steps {
                sh "mvn test"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=africa-shopping-cart \
                        -Dsonar.projectName=africa-shopping-cart \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('Maven Package') {
            steps {
                sh "mvn clean package"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-credential', toolName: 'docker') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag shopping-cart ekangaki/shopping-cart:latest"
                    }
                }
            }
        }

        stage('Trivy Docker Image Scan') {
            steps {
                sh "trivy image --format table -o image.html ekangaki/shopping-cart:latest"
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-credential', toolName: 'docker') {
                        sh "docker push ekangaki/shopping-cart:latest"
                    }
                }
            }
        }

        stage('Deploy to Docker Container') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-credential', toolName: 'docker') {
                        sh "docker run -d --name africa-shopping -p 8070:8070 ekangaki/shopping-cart:latest"
                    }
                }
            }
        }
    }

 }
