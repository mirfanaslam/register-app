pipeline {
    agent {label 'agent'}

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "mirfanaslam"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        
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
                    withSonarQubeEnv (credentialsId: 'jenkins-sonarqube-token'){
                    sh "mvn sonar:sonar"
                }
            }
            }
        }
        stage ('quality gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-toke'
                }
            }
        }
        stage ('Build and push'){
            steps{
                script{
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                }
            }
        }
    }
}
}
