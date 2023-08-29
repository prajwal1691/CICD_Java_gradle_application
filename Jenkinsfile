pipeline {
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
        NEXUS_URL = "65.0.110.219:8081"
        DOCKER_HOSTED = "65.0.110.219:8083"
    } 
    stages{
        stage("sonar quality check") {
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube'
                    }

                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                }
            }
        }

        stage("Docker build and push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_password')]) {
                        sh '''
                          docker build -t ${DOCKER_HOSTED}/springapp:${VERSION} .
                          docker login -u admin -p $nexus_password ${DOCKER_HOSTED}
                          docker push ${DOCKER_HOSTED}/springapp:${VERSION}
                          docker rmi ${DOCKER_HOSTED}/springapp:${VERSION}
                        '''
                    }
                }
            }
        }

        stage("pushing the helm charts to nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_password')]) {
                          dir('kubernetes/') {
                             sh '''
                                 helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                 tar -czvf  myapp-${helmversion}.tgz myapp/
                                 curl -u admin:$nexus_password http://${NEXUS_URL}/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                          }
                    }
                }
            }
        }
    }                
}