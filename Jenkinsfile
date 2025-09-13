pipeline{
    agent any
    
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    environment {
        APP_NAME = "devops-mega-project"
        RELEASE = "1.0.0"
        DOCKER_USER = "moizhussainsadhix"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
 //JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")

    }

   stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }

        }  
    stage("Checkout from SCM"){
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Devops1224789/Devops-Mega-project.git'
            }

        }


        stage("Build the application"){
            steps {
               sh "mvn clean package"
            }

        }

        stage("Test the application"){
            steps {
               sh "mvn test"
            }

        } 
        stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-token'){
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-token'
                }
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

        stage("Trivy Scan") {
            steps {
                script {
		   sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image mydevopsuser46/devops-mega-project:latest  --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
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