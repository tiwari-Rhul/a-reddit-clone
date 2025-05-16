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
		}
}
