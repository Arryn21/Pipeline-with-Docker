pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "arrynrex/simple-java-app"
        SONARQUBE_SERVER = "SonarQube"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Arryn21/Pipeline-with-Docker.git'
            }
        }

        stage('Build with Java 17') {
            agent {
                docker {
                    image maven:3.9.4-eclipse-temurin-17
                    args '--network ci_network'
                }
            }
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test with Java 11') {
            agent {
                docker {
                    image 'openjdk:11-jdk'
                    args '--network ci_network'
                }
            }
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis with Java 8') {
            agent {
                docker {
                    image 'openjdk:8-jdk'
                    args '--network ci_network'
                }
            }
            tools {
                maven 'M3'  // This assumes Maven is installed in Jenkins under that label
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }
    }
}
