pipeline {
    agent any

    environment {
        // 1. Use consistent networking
        NETWORK_NAME = 'ddd_network'
        ELASTICSEARCH_URL = 'http://elasticsearch:9200'  // Use Docker service names
        MYSQL_URL = 'jdbc:mysql://mysql:3306/mooc'
        RABBITMQ_HOST = 'rabbitmq'
    }

    stages {
        stage('Setup Network') {
            steps {
                bat """
                    docker network create %NETWORK_NAME% || echo "Network already exists"
                """
            }
        }

        stage('Start Services') {
            steps {
                // 2. Force rebuild and use custom network
                bat """
                    docker-compose -f docker-compose.ci.yml down -v --remove-orphans
                    docker-compose -f docker-compose.ci.yml build --no-cache
                    docker-compose -f docker-compose.ci.yml up -d
                """
            }
        }

        stage('Verify Services') {
            steps {
                script {
                    // 3. Container-native health checks
                    def checkService = { cmd, timeout ->
                        waitUntil(initialRecurrencePeriod: 5000) {
                            try {
                                bat(script: cmd, returnStatus: true) == 0
                            } catch(e) {
                                echo "Check failed: ${e.message}"
                                false
                            }
                        }
                    }

                    checkService('docker exec mysql mysqladmin ping -uroot -p"" --silent', 120)
                    checkService('curl -s http://elasticsearch:9200/_cluster/health | find "green"', 120)
                    checkService('docker exec rabbitmq rabbitmqctl await_startup', 60)
                }
            }
        }

        stage('Run Tests') {
            steps {
                // 4. Run tests INSIDE the test container
                bat """
                    docker exec -e "SPRING_PROFILES_ACTIVE=test" \\
                    test_container ./gradlew test \\
                    -Dspring.datasource.url=%MYSQL_URL% \\
                    -Dspring.datasource.username=root \\
                    -Dspring.datasource.password= \\
                    -Delasticsearch.host=elasticsearch \\
                    -Drabbitmq.host=%RABBITMQ_HOST%
                """
            }
        }
    }

    post {
        always {
            // 5. Capture diagnostics before cleanup
            bat """
                docker logs elasticsearch > elasticsearch.log 2>&1 || echo "No ES logs"
                docker-compose -f docker-compose.ci.yml down -v
                docker network rm %NETWORK_NAME% || echo "Network removal failed"
            """
            archiveArtifacts artifacts: '*.log', allowEmptyArchive: true
        }
    }
}
