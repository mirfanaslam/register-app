pipeline {
    agent {label 'agent'}

    tools {
        jdk 'jdk17'
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
                git branch: 'main' , credentialsId: 'github', url: 'https://github.com/mirfanaslam/register-app.git'
            }
        }

        stage('Build App') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('sonarqube analysiz'){
            steps {
                script {
                    withSonarQubeEnv (credentialsId: 'jenkins-sonarqube-token')
                    sh "mvn sonar:sonar"
                }
            }
            
        }
    }
}
