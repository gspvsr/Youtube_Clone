pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage(' git checkout') {
            steps {
                git 'https://github.com/gspvsr/Youtube_Clone.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Youtube-CICD \
                    -Dsonar.projectKey=Youtube-CICD'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Quality-Gate'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('TRIVY FS SCAN') {
             steps {
                sh "/usr/local/bin/trivy fs . > trivyfs.txt"
             }
        }
        stage("Dockr Build"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
                      sh "docker build -t gspvsr/youtube-clone:latest ."
                    } 
                }
            }
        }
        stage("Dockr push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){   
                      sh "docker push gspvsr/youtube-clone:latest "
                    } 
                }
            }
        }
        stage("TRIVY Image Scan"){
            steps{
                sh "trivy image gspvsr/youtube-clone:latest > trivyimage.txt" 
            }
        }       
    }

    post {
        always {
            emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'gspvsr@yahoo.com',                              
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }  
}
