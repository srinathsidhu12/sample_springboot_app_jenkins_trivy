pipeline {
    agent any
    environment {
        DOCKER_HUB_REPO = "srinathsidhu12/spring_boot_app_trivy"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    tools {
       maven 'Maven'
    }
    stages{
        stage('clone code'){
            steps{
                git 'https://github.com/srinathsidhu12/sample_springboot_app_jenkins_trivy.git'
            }
        }
        stage('sonar_analysis'){
            steps{
                withSonarQubeEnv('sonarserver') {
                   sh '''
                   mvn clean verify sonar:sonar' \
                   -Dsonar.projectKey=Jenkins-sample-springboot-app-pipeline \
                   -Dsonar.projectName="Jenkins-sample-springboot-app-pipeline"
                   '''
               }
            }
        }
        stage('Quality Gate'){
            steps{
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true, credentialsId: 'sonarqube_token'
                }       
            }
        }
        stage('Application_build'){
            steps{
                sh 'mvn clean package'
            }
        }
        stage('Build_docker_image'){
            steps{
                sh 'docker build -t $DOCKER_HUB_REPO:$IMAGE_TAG .'
            }
        }
        stage('Trivy_image_scan'){
            steps{
                sh 'trivy image --severity HIGH --exit-code 1 $DOCKER_HUB_REPO:$IMAGE_TAG'
            }
        }
        stage('Push Image'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'password', usernameVariable: 'username')]) {
                 sh """
                 echo "$password" | docker login -u "$username" --password-stdin
                 docker push $DOCKER_HUB_REPO:$Image_tag
                 """
             }
          }
        }
    }
}
