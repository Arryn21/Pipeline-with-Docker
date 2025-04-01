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
                    image 'maven:3.9.4-eclipse-temurin-17'
                    args '--network ci_network -v /root/.m2:/root/.m2'
                }
            }
            environment {
                JAVA_HOME = '/opt/java/openjdk'
                PATH = "${JAVA_HOME}/bin:${PATH}"
            }
            steps {
                sh 'echo "Using Java:" && java -version'
                sh 'echo "Using Maven:" && mvn -v'
                sh 'echo "JAVA_HOME is $JAVA_HOME"'
                sh 'mvn clean compile'
            }
        }

        stage('Test with Java 11') {
            agent {
                docker {
                    image 'maven:3.8.6-eclipse-temurin-11'
                    args '--network ci_network -v /root/.m2:/root/.m2'
                }
            }
            steps {
                sh 'java -version'
                sh 'mvn -v'
                // Skip compilation to avoid Java 17 compatibility error
                sh 'mvn test -Dmaven.compiler.skip=true'
            }
        }

        stage('SonarQube Analysis with Java 8') {
            agent {
                docker {
                    image 'maven:3.9.1-eclipse-temurin-8'
                    args '--network ci_network -v /root/.m2:/root/.m2'
                }
            }
            steps {
                sh 'java -version'
                sh 'mvn -v'
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh 'mvn clean verify sonar:sonar'
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
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }
    }
}
