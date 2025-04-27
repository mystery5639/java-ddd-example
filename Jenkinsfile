pipeline {
    agent any

    environment {
        JAVA_HOME = 'C:\\Program Files\\Java\\jdk-21'
        PATH = "${JAVA_HOME}\\bin;${env.PATH}"
        COMPOSE_FILE = 'docker-compose.ci.yml'
        ELASTICSEARCH_HOST = 'host.docker.internal'
        MYSQL_HOST = 'host.docker.internal'
        RABBITMQ_HOST = 'host.docker.internal'
    }

    stages {
        stage('Start Services') {
            steps {
                bat "docker-compose -f %COMPOSE_FILE% up -d"
            }
        }

        stage('Verify Services') {
            steps {
                script {
                    // Service-specific verification with detailed logging
                    verifyService('MySQL', "docker exec codely-java_ddd_example-mysql mysqladmin ping -uroot -p\"\" --silent", 60)
                    verifyService('Elasticsearch', "curl -s http://%ELASTICSEARCH_HOST%:9200/_cluster/health | find \"green\"", 60)
                    verifyService('RabbitMQ', "docker exec codely-java_ddd_example-rabbitmq rabbitmqctl await_startup", 30)
                }
            }
        }

        stage('Build & Test') {
            steps {
                bat """
                    gradlew build test ^
                    -Dspring.datasource.url=jdbc:mysql://%MYSQL_HOST%:3306/mooc ^
                    -Dspring.datasource.username=root ^
                    -Dspring.datasource.password= ^
                    -Delasticsearch.host=%ELASTICSEARCH_HOST%:9200 ^
                    -Drabbitmq.host=%RABBITMQ_HOST%
                """
            }
        }
    }

    post {
        always {
            bat "docker-compose -f %COMPOSE_FILE% down -v"
            cleanWs()
            archiveArtifacts artifacts: '**/build/reports/tests/test', allowEmptyArchive: true
        }
    }
}

// Custom verification method
def verifyService(String serviceName, String checkCommand, int timeoutSeconds) {
    def startTime = System.currentTimeMillis()
    def maxTime = startTime + (timeoutSeconds * 1000)
    
    while (System.currentTimeMillis() < maxTime) {
        try {
            // Run check command
            def status = bat(script: checkCommand, returnStatus: true)
            
            if (status == 0) {
                echo "${serviceName} is ready!"
                return
            }
            
            // Get service logs for debugging
            if (serviceName == 'MySQL') {
                bat "docker logs codely-java_ddd_example-mysql --tail 20 || echo \"Could not get ${serviceName} logs\""
            } else if (serviceName == 'Elasticsearch') {
                bat "docker logs codely-java_ddd_example-elasticsearch --tail 20 || echo \"Could not get ${serviceName} logs\""
            } else if (serviceName == 'RabbitMQ') {
                bat "docker logs codely-java_ddd_example-rabbitmq --tail 20 || echo \"Could not get ${serviceName} logs\""
            }
            
            // Wait before retrying
            sleep(time: 5, unit: 'SECONDS')
            echo "Waiting for ${serviceName}... (${(System.currentTimeMillis() - startTime)/1000}s elapsed)"
            
        } catch (Exception e) {
            echo "Error checking ${serviceName}: ${e.getMessage()}"
            sleep(time: 5, unit: 'SECONDS')
        }
    }
    
    error("${serviceName} failed to start within ${timeoutSeconds} seconds")
}
