pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonarqube-scanner'
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
                withSonarQubeEnv('SonarQuber-Server') {
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
        stage("Build & Push Docker Image") {
             steps {
                script {
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
        stage("Trivy Image Scan") {
            steps {
                script {
	              sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image 8871255273/reddit-clone-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table > trivyimage.txt')
                }
            }
        }
        stage ('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }

    }
}