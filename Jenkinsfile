pipeline{
	agent any
	tools{
		jdk 'jdk17'
		nodejs 'node16'
	}
	environment{
		SCANNER_HOME = tool 'sonar-scanner'
		APP_NAME = "reddit-clone-app"
		RELEASE = "1.0.0"
		DOCKER_USER = "rahulkube"
		DOCKER_PASS = 'dockerhub'
		IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
		IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	}
	stages{
		stage("Clean Worksapce"){
			steps{
				cleanWs()
			}
		}
		stage("Checkout from git"){
			steps{
				git branch: 'main', url: "https://github.com/tiwari-Rhul/a-reddit-clone"
			
	                }
	         }
		stage("Sonarqube Analysis"){
			steps{
				withSonarQubeEnv('SonarQube-Server'){
						 sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Reddit-Clone-CI \
                                                -Dsonar.projectKey=Reddit-Clone-CI'''
				}
			}
		}

		stage("QualityGate"){
			steps{
				script{
					waitForQualityGate abortPipeline: false, credentialsId:'SonarQube-Token'
				}
			}
		}

		stage("Install Dependencies"){
			steps{
				sh "npm install"
			}
		}

		stage('TRIVY FS SCAN'){
			steps{
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
	                  sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image rahulkube/reddit-clone-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table > trivyimage.txt')
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
