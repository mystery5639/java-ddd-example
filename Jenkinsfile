pipeline {
    agent any
    
    environment {  // Corrected spelling
        JAVA_HOME = 'C:\\Program Files\\Java\\jdk-21'  // Adjust path as needed
        PATH = "${JAVA_HOME}\\bin;${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/mystery5639/java-ddd-example.git'
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
