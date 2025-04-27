pipeline {
    agent any
    
    environment {
        // Set JAVA_HOME (adjust path to your Java installation)
        JAVA_HOME = 'C:\Program Files\Java\jdk-17.0.12'  // Windows path example
        PATH = "${JAVA_HOME}\\bin;${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/wxwwixx/java1.git'
            }
        }
        
        stage('Build') {
            steps { 
                bat 'gradlew clean build'
            }
        }
        
        stage('Test') {
            steps { 
                bat 'gradlew test' 
            }
        }
        
        stage('Deploy') {
            steps { 
                powershell 'java -jar build/libs/hello-world-java-V1.jar'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace'
            deleteDir()
        }
        success {
            echo 'Build succeeded!!!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
