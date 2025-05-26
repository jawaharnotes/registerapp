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
    }
}
