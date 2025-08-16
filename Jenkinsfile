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
		    JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
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
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/root/reports aquasec/trivy image jawahartolearn/myapp-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL --format json --output /root/reports/trivy-report.json')
		    archiveArtifacts artifacts: 'trivy-report.json'   
               }
           }
       }

    stage("cleanup Artifacts") {
	    steps{
		script {
             sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
		     sh "docker rmi ${IMAGE_NAME}:latest"
		}
	    }
         }

	 stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-13-235-82-34.ap-south-1.compute.amazonaws.com:8080/job/user-registration-cd/buildWithParameters?token=gitops-token'"
                }
            }
		
    }
	}

	 post {
    	always {
        	emailext(
            	to: "jawaharr1393@gmail.com",
            	from: "jawahar.tolearn@gmail.com",
            	subject: "Test from pipeline - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            	body: "This is a test email sent from Jenkins pipeline.",
            	mimeType: 'text/plain'
        )
    }
}

}
