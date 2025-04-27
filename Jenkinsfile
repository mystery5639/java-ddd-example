pipeline {
    agent any
    
    environment {
        // Set JAVA_HOME (adjust path to your Java installation)
        JAVA_HOME = 'C:\graalvm-jdk-21_windows-x64_bin\graalvm-jdk-21.0.6+8.1'  // Windows path example
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
