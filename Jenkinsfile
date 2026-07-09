pipeline {
    agent {label 'agent'}

    tools {
        jdk 'jdk21'
        maven 'maven3'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main' , CredentialId: 'github', url: 'https://github.com/mirfanaslam/register-app.git'
            }
        }

        stage('Build App') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}
