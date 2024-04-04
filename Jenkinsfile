pipeline {
    agent { node { label 'Agent-1' } }
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        NODE_OPTIONS = "--max-old-space-size=4096"
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
                sh export NODE_OPTIONS="--max-old-space-size=4096"
                sh "npm install"
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "/usr/bin/trivy fs . > trivyfs.txt"
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
        stage('Deploy to Kubernets'){
            steps{
                script{
                    dir('Kubernetes') {
                      withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubernetes', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                      sh 'kubectl delete --all pods'
                      sh 'kubectl apply -f deployment.yml'
                      sh 'kubectl apply -f service.yml'
                      }   
                    }
                }
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
