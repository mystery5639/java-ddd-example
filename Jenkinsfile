pipeline {
    agent any
    environment {
        // Set JAVA_HOME (adjust path to your Java installation)
        JAVA_HOME = 'C:\\Program Files\\Java\\jdk-21'  // Windows path example
        PATH = "${JAVA_HOME}\\bin;${env.PATH}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/mystery5639/java-ddd-example.git'
            }
        }
        stage('Build') {
            steps { bat 'gradlew clean build'}
        }
        stage('Test') {
            steps { bat 'gradlew test'}
        }
        stage('Deploy') {
            steps { powershell 'java -jar build/libs/hello-world-java-V1.jar'}           
        }    
}

post {
        always {
            echo 'Cleaning up workspace'
            deleteDir() // Clean up the workspace after the build
        }
        success {
            echo 'Build succeeded!!!'
            // You could add notification steps here
        }
        failure {
            echo 'Build failed!'
            // You could add notification steps here
        }
    }
}
