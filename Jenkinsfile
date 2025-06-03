pipeline {
    agent { label 'jenkins-agent' }
    tools { jdk 'java17'
            maven 'maven3'}
    environment {
            APP_NAME = "myapp-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "jawahartolearn"
            DOCKER_PASS = "dockerhub"
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    
    stages { 
        stage("cleanup Workspace") {
          steps{ cleanWs() }
        }

        stage("Checkout from SCM") {
          steps { 
          git url: 'https://github.com/jawaharnotes/registerapp',
              credentialsId: 'github',
              branch: 'main'  
          }
        }

        stage("Build App"){
          steps{ 
            sh "mvn clean package"
          }
        }

       stage("Test Application"){ 
         steps{
           sh "mvn test"
         }
       }

       stage("SonarQube Analysis"){
         steps{
           script {
              withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
              sh "mvn sonar:sonar"
                  }
               }
            }
         }

      stage("Quality Gates"){
          steps{
            script{
                waitForQualityGate abortPipeline: false,
                                   credentialsId: 'jenkins-sonarqube-token'
            }
          }
      }

      stage("Build and Push Docker Image") {
          steps{
              script {
                  docker.withRegistry('',DOCKER_PASS) {
                      def docker_image = docker.build ("${IMAGE_NAME}:${IMAGE_TAG}")
                      docker_image.push()
                      docker_image.push('latest')
                  }
              }
              
          }
      }

     stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/root/reports aquasec/trivy image jawahartolearn/myapp-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format json --output /root/reports/trivy-report.json')
		    archiveArtifacts artifacts: 'trivy-report.json'   
               }
           }
       }
    }
}
