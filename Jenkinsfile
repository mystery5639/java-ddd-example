pipeline {
    agent {
        
    }
    
    environment {
        JAVA_HOME = 'C:\\Program Files\\Java\\jdk-21'
        PATH = "${JAVA_HOME}\\bin;${env.PATH}"
        // Windows Docker paths
        DOCKER_COMPOSE = 'docker-compose'
        COMPOSE_FILE = 'docker-compose.ci.yml'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/mystery5639/java-ddd-example.git'
            }
        }

        stage('Start Containers') {
            steps {
                bat "docker-compose.ci.yml up -d"
            }
        }

        stage('Wait for Services') {
            steps {
                bat """
                    :loop_mysql
                    ${DOCKER_COMPOSE} exec -T mysql mysqladmin ping --silent
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
            bat "${DOCKER_COMPOSE} -f ${COMPOSE_FILE} down"
            cleanWs()
        }
    }
}
