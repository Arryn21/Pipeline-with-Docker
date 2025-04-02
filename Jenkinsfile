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
            steps {
                sh 'java -version'
                sh 'mvn -v'
                // Compile for Java 8 target to ensure compatibility
                sh 'mvn clean install -DskipTests -Dmaven.compiler.target=8 -Dmaven.compiler.source=8'
                stash name: 'built-artifacts', includes: 'target/**/*'
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
                unstash 'built-artifacts'
                sh 'java -version'
                sh 'mvn -v'
                sh 'mvn surefire:test -DskipCompile'
            }
        }

        stage('SonarQube Analysis Using Java 8') {
            agent {
                docker {
                    image 'maven:3.8.6-openjdk-8'
                    args '--network ci_network -v /root/.m2:/root/.m2'
                }
            }
            steps {
                unstash 'built-artifacts'
                sh 'java -version'
                sh 'mvn -v'
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv("${SONARQUBE_SERVER}") {
                        sh '''
                            mvn org.codehaus.mojo:sonar-maven-plugin:3.7.0.1746:sonar \
                            -Dsonar.projectKey=simple-java-app \
                            -Dsonar.host.url=http://sonarqube:9000 \
                            -Dsonar.login=$SONAR_TOKEN \
                            -DskipCompile
                        '''
                    }
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
