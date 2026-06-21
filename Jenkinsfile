pipeline {

    agent none

    environment {
        SONAR_TOKEN = credentials('sonarcloud-token')
        SONAR_ORG = 'eric94310'
        SONAR_PROJECT_KEY = 'Eric94310_PayMyBuddy'

        DOCKERHUB_USERNAME = 'docker94310'
        DOCKERHUB_TOKEN = credentials('jenkins-token')

        IMAGE_NAME = "${DOCKERHUB_USERNAME}/paymybuddy"
        IMAGE_TAG = "${BUILD_NUMBER}"

        STAGING_HOST = 'IP_PUBLIQUE_OU_DNS_AWS'
        STAGING_USER = 'ubuntu'
    }

    stages {

        stage('Tests automatisés') {
            agent {
                docker {
                    image 'maven:3.9.6-eclipse-temurin-17'
                }
            }
            steps {
                sh '''
                    unset MAVEN_CONFIG
                    chmod +x mvnw
                    ./mvnw test
                '''
            }
        }

        stage('Qualité du code - SonarCloud') {
            agent {
                docker {
                    image 'maven:3.9.6-eclipse-temurin-17'
                }
            }
            steps {
                sh '''
                    unset MAVEN_CONFIG
                    chmod +x mvnw
                    ./mvnw sonar:sonar \
                    -Dsonar.organization=$SONAR_ORG \
                    -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                    -Dsonar.host.url=https://sonarcloud.io \
                    -Dsonar.token=$SONAR_TOKEN
                '''
            }
        }

        stage('Compilation et Packaging') {
            agent {
                docker {
                    image 'maven:3.9.6-eclipse-temurin-17'
                }
            }
            steps {
                sh '''
                    unset MAVEN_CONFIG
                    chmod +x mvnw
                    ./mvnw clean package -DskipTests
                '''
            }
        }

        stage('Build image Docker') {
            agent {
                docker {
                    image 'docker:24-cli'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                sh '''
                    docker build \
                    -t $IMAGE_NAME:$IMAGE_TAG \
                    -t $IMAGE_NAME:latest .
                '''
            }
        }

        stage('Push image DockerHub') {
            agent {
                docker {
                    image 'docker:24-cli'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                sh '''
                    echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    docker push $IMAGE_NAME:latest
                '''
            }
        }

        stage('Staging') {
            agent any
            steps {
                echo 'Déploiement en environnement de staging...'

                sshagent(credentials: ['aws-staging-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no $STAGING_USER@$STAGING_HOST "
                            docker pull $IMAGE_NAME:latest &&
                            docker stop paymybuddy-staging || true &&
                            docker rm paymybuddy-staging || true &&
                            docker run -d \
                                --name paymybuddy-staging \
                                -p 8080:8080 \
                                $IMAGE_NAME:latest
                        "
                    '''
                }
            }
        }
    }
}
