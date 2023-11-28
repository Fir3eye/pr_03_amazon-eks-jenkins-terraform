pipeline {
    agent any 
    tools {
        jdk 'jdk11'
        maven 'maven3'
    }
    environment {
        APP_NAME = "image_name"
        RELEASE = "latest"
        DOCKER_USER = "fir3eye"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}" 
    }
    stages{
        stage("Clean WorkSpace"){
            steps {
                cleanWs()
            }
        }
        stage("Checkout SCM"){
            steps {
                git 'https://github.com/Fir3eye/pr_03_amazon-eks-jenkins-terraform.git'
            }
        }
        stage("Maven Compile"){
            steps {
                sh 'mvn clean compile'
            }
        }
        stage("SonarQube Analysis"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
        stage ('Build War file') {
            steps {
                sh 'mvn clean install package'
            }
        }  
        stage("Build Docker Image"){
            steps{
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        //sh "docker images --format '{{.Repository}}:{{.Tag}}' | grep ${IMAGE_NAME} | grep -v ${RELEASE}-${BUILD_NUMBER} | grep -v latest | xargs -I {} docker rmi {} || true" 
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                }
            }
        }
        stage("Push Docker Image"){
            steps{
                script {
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage ('Deploy to Container') {
            steps {
                sh 'docker run -d --name pet1 -p 8082:8080 fir3eye/image_name:latest'
            }
        }
    }
}