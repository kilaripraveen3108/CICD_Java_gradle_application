pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            agent {
              docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'Sonar-Token') {
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
        stage("docker build and push"){
            steps{
               scripts{
                 withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                     sh'''
                     docker build -t 34.171.30.155:8085/springapp/${VERSION} .
                     docker login -u admin -p $docker_password 34.171.30.155:8085
                     docker push 34.171.30.155:8085/springapp/${VERSION}
                     docker rmi 34.171.30.155:8085/springapp/${VERSION}
                     '''
               } 
            }
        }  
    }
}
