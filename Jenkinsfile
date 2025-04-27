pipeline {
    agent any

    environment {
        JAVA_HOME = 'C:\\Program Files\\Java\\jdk-21'
        PATH = "${JAVA_HOME}\\bin;${env.PATH}"
        // Docker configuration
        COMPOSE_FILE = 'docker-compose.ci.yml'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                url: 'https://github.com/mystery5639/java-ddd-example.git',
                credentialsId: 'your-github-credentials' // Add if private repo
            }
        }

        stage('Start Containers') {
            steps {
                bat "docker-compose -f ${COMPOSE_FILE} up -d"  // Fixed command
            }
        }

        stage('Wait for Services') {
            steps {
                bat """
                    :loop_mysql
                    docker exec codely-java_ddd_example-mysql mysqladmin ping -uroot -p"" --silent
                    if %errorlevel% neq 0 (
                        echo Waiting for MySQL...
                        timeout /t 5 /nobreak > nul
                        goto loop_mysql
                    )
                    
                    :loop_es
                    curl -s http://localhost:9200/_cluster/health
                    if %errorlevel% neq 0 (
                        echo Waiting for Elasticsearch...
                        timeout /t 5 /nobreak > nul
                        goto loop_es
                    )
                """
            }
        }

        stage('Build & Test') {
            steps {
                bat """
                    gradlew build test ^
                    -Dspring.datasource.url=jdbc:mysql://localhost:3306/mooc ^
                    -Dspring.datasource.username=root ^
                    -Dspring.datasource.password= ^
                    -Delasticsearch.host=localhost
                """
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
            bat "docker-compose -f ${COMPOSE_FILE} down"
            cleanWs()
        }
    }
}
