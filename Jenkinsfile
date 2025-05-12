pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'JAVA17'
        maven 'M3'
    }

    environment {
	APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "paultofunmi"
        DOCKER_PASS = credentials('dockerhub')
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"	    
    }
   
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/paultofunmi/register-app'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }

       stage("SonarQube Analysis"){
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
       }
       
       stage("Quality Gate"){
            steps {
                script {
                    waitForQualityGate abortPipeline: true, credentialsId: 'jenkins-sonarqube-token'
                }
            }
       }

       stage("Build & Push Docker Image") {
            steps {
                script {
	            withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
	                sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
	
	                def dockerImage = docker.build("${IMAGE_NAME}")
	                dockerImage.push("${IMAGE_TAG}")
	                dockerImage.push("latest")
	            }
	        }
            }
       }	    
    }
}
