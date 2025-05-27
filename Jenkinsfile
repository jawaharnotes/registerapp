pipeline {
    agent { label 'jenkins-agent' }
    tools { jdk 'java17'
            maven 'maven3'}
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
              withSonarQubeEnv(
                  credentialsId: 'jenkins-sonarqube-token') {
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
    }
}
