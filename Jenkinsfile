pipeline {
  agent {label 'agent'}
  tools {
    jdk 'jdk21'
    maven 'maven3'
  }
  stages {
    stage ("clean Workspace")
      steps {
        cleanWs()
            }
  }
   stage ("checkout from scm"){
     steps {
       git branch: 'main' , CredentialId: 'github', url: 'https://github.com/mirfanaslam/register-app.git' 
     }
   }
   stage ("build App"){
     steps {
       sh "mvn clean package"
     }
   }
}
