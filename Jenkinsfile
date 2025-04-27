pipeline {
    agent any

    environment {
        COMPOSE_FILE = 'docker-compose.ci.yml'
        NETWORK_NAME = 'ddd_network'
    }

    stages {
        stage('Prepare Network') {
            steps {
                script {
                    // 1. Safely handle existing network
                    def networkStatus = bat(
                        script: 'docker network inspect %NETWORK_NAME% > nul 2>&1 && echo EXISTS || echo MISSING',
                        returnStdout: true
                    ).trim()
                    
                    if (networkStatus == 'EXISTS') {
                        bat 'docker network rm %NETWORK_NAME%'
                    }
                    bat 'docker network create --label com.docker.compose.network=default %NETWORK_NAME%'
                }
            }
        }

        stage('Start Services') {
            steps {
                // 2. Force clean start with proper network
                bat """
                    docker-compose -f %COMPOSE_FILE% down -v --remove-orphans
                    docker-compose -f %COMPOSE_FILE% build --no-cache
                    docker-compose -f %COMPOSE_FILE% up -d
                """
            }
        }

        stage('Verify Services') {
            steps {
                script {
                    // 3. Container-native checks with retries
                    def services = [
                        [name: 'MySQL', cmd: 'docker exec mysql mysqladmin ping -uroot -p"" --silent'],
                        [name: 'Elasticsearch', cmd: 'curl -s http://elasticsearch:9200/_cluster/health | find "green"'],
                        [name: 'RabbitMQ', cmd: 'docker exec rabbitmq rabbitmqctl await_startup']
                    ]
                    
                    services.each { svc ->
                        retry(3) {
                            timeout(time: 2, unit: 'MINUTES') {
                                waitUntil {
                                    try {
                                        bat(script: svc.cmd, returnStatus: true) == 0
                                    } catch(e) {
                                        echo "${svc.name} check failed: ${e.message}"
                                        bat "docker logs ${svc.name.toLowerCase()} --tail 50"
                                        sleep 10
                                        false
                                    }
                                }
                                echo "${svc.name} verified!"
                            }
                        }
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                bat """
                    docker exec -e "SPRING_PROFILES_ACTIVE=test" \\
                    test_container ./gradlew test \\
                    -Dspring.datasource.url=jdbc:mysql://mysql:3306/mooc \\
                    -Dspring.datasource.username=root \\
                    -Dspring.datasource.password= \\
                    -Delasticsearch.host=elasticsearch \\
                    -Drabbitmq.host=rabbitmq
                """
            }
        }
    }

    post {
        always {
            // 4. Preserve logs even if pipeline fails
            script {
                try {
                    bat """
                        docker-compose -f %COMPOSE_FILE% logs --no-color > all_logs.txt
                        docker inspect %NETWORK_NAME% > network_info.json
                    """
                    archiveArtifacts artifacts: '*.txt,*.json', allowEmptyArchive: true
                } finally {
                    bat """
                        docker-compose -f %COMPOSE_FILE% down -v
                        docker network rm %NETWORK_NAME% || echo "Network removal failed"
                    """
                }
            }
        }
    }
}
