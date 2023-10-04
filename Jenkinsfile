pipeline {
    agent any
    tools{
         jdk  'jdk11'
         maven 'maven3'
    }
    environment{
        
        SCANNER_HOME= tool 'sonar-scanner'
        VERSION = "${env.BUILD_ID}"
    }

    stages {
        stage('Git Chekout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: 'ad3ac41a-6f0f-4f40-a2e4-1e6803226597', url: 'https://github.com/MBM-mery/Ekart.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Sonarqube"){

            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Shopping-Cart \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Shopping-Cart '''
                }

                      }           
                }
        
    }
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'ebd2ac05-2956-4aae-b65f-5a346cdfc7d8', toolName: 'docker') {
   
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag  shopping-cart meryemcherkaoui/shopping-cart:latest"
                        sh "docker push meryemcherkaoui/shopping-cart:latest"
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'ebd2ac05-2956-4aae-b65f-5a346cdfc7d8', toolName: 'docker') {
   
                        sh "docker run -d --name projectstageee-projectstageee -p 8073:8073 meryemcherkaoui/shopping-cart:latest"
                        
                    }
                }
            }
        }
}
  post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "meryem.cherkaouidakkaki@usmba.ac.ma";  
		}
	} 
}
