pipeline {
    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-17'
          
        }
    }

    environment {
        SONAR_TOKEN = credentials('sonarcloud-token')
        SONAR_ORG = 'eric94310'
        SONAR_PROJECT_KEY = 'Eric94310_PayMyBuddy'
    }

    stages {

        stage('Tests automatisés') {
            steps {
                echo 'Exécution des tests unitaires et d’intégration...'
                sh 'chmod +x mvnw'
                sh './mvnw test'
            }
        }

        stage('Qualité du code - SonarCloud') {
            steps {
                echo 'Analyse du code avec SonarCloud...'
                sh """
                    ./mvnw sonar:sonar \
                    -Dsonar.organization=${SONAR_ORG} \
                    -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                    -Dsonar.host.url=https://sonarcloud.io \
                    -Dsonar.token=${SONAR_TOKEN}
                """
            }
        }
    }
}
