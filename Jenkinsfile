pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        APP_NAME = "reddit-clone-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "8871255273"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages {
        stage ('Cleanup') {
            steps {
                cleanWs()
            }
        }
        stage ('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/madhursarangpure/a-reddit-clone.git'
            }
        }
        stage ('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('SonarQuber-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=reddit-clone-ci \
                        -Dsonar.projectKey=reddit-clone-ci'''
                }
            }
        }
        stage ('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token'
                }
            }
        }
        stage ('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage ('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
             }
        }

    }
}