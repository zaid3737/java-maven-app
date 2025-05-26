pipeline {
    agent any
    
    tools {
        maven "Maven3"
    }
       
    stages {
	    stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Aseemakram19/java-maven-app.git'
            }
        }
        
        stage('Build WAR FILE') {
            steps {
                sh 'mvn clean package'
            }
        }
		stage("Docker Build & Push Image to DockerHub"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t javamavenapp ."
                       sh "docker tag javamavenapp aseemakram19/javamavenapp:latest "
                       sh "docker push aseemakram19/javamavenapp:latest "
                    }
                }
            }
        }           
        stage('Check Port available & delete container') {
            steps {
                script {
                    def portStatus = sh(script: 'netstat -tuln | grep ":9000"', returnStatus: true)
                    if (portStatus == 0) {
                        echo "Port 9000 is in use, stopping and removing the existing container"
                        sh 'docker stop javamavenapp_container'
                        sh 'docker rm javamavenapp_container'
                    } else {
                        echo "Port 9000 is available"
                    }
                }
            }
        }
        
        stage('Run Docker Container on 9000 port number') {
            steps {
                sh 'docker run -d -p 9000:8080 --name javamavenapp_container -t aseemakram19/javamavenapp:latest'
            }
        }
    }
    post {
    always {
        script {
            def buildStatus = currentBuild.currentResult
            def buildUser = currentBuild.getBuildCauses('hudson.model.Cause$UserIdCause')[0]?.userId ?: 'Github User'
            
            emailext (
                subject: "Pipeline ${buildStatus}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <p>This is a Jenkins amazon-prime-video CICD pipeline status.</p>
                    <p>Project: ${env.JOB_NAME}</p>
                    <p>Build Number: ${env.BUILD_NUMBER}</p>
                    <p>Build Status: ${buildStatus}</p>
                    <p>Started by: ${buildUser}</p>
                    <p>Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """,
                to: 'mohdaseemakram19@gmail.com',
                from: 'mohdaseemakram19@gmail.com',
                replyTo: 'mohdaseemakram19@gmail.com',
                mimeType: 'text/html'                
            )
           }
       }

    }
}
