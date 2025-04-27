pipeline {
    agent any

    environment {
        JAVA_HOME = 'C:\\Program Files\\Java\\jdk-21'
        PATH = "${JAVA_HOME}\\bin;${env.PATH}"
        COMPOSE_FILE = 'docker-compose.ci.yml'
    }

    stages {
        stage('Cleanup Previous Containers') {
            steps {
                bat """
                    docker-compose -f ${COMPOSE_FILE} down || echo "No containers to remove"
                    docker rm -f codely-java_ddd_example-mysql codely-java_ddd_example-elasticsearch codely-java_ddd_example-rabbitmq || echo "Containers not found"
                """
            }
        }

        stage('Start Containers') {
            steps {
                bat "docker-compose -f ${COMPOSE_FILE} up -d"
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
                    
                    :loop_rabbit
                    docker exec codely-java_ddd_example-rabbitmq rabbitmqctl await_startup
                    if %errorlevel% neq 0 (
                        echo Waiting for RabbitMQ...
                        timeout /t 5 /nobreak > nul
                        goto loop_rabbit
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
                    -Delasticsearch.host=localhost ^
                    -Drabbitmq.host=localhost
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
