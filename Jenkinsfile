pipeline{
    agent any

    stages{
        stage("sonar quality check"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew --status'
                        sh './gradlew cleanBuildCache'
                        sh './gradlew sonarqube --warning-mode all'
                    }
                }
            }
        }
    }
}