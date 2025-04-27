pipeline {
    agent any

    environment {
        JAVA_HOME = 'C:\\Program Files\\Java\\jdk-21'
        PATH = "${JAVA_HOME}\\bin;${env.PATH}"
        COMPOSE_FILE = 'docker-compose.ci.yml'
        ELASTICSEARCH_HOST = 'host.docker.internal' // Changed from localhost
    }

    stages {
        stage('Cleanup') {
            steps {
                bat """
                    docker-compose -f %COMPOSE_FILE% down -v --remove-orphans || echo "Cleanup failed"
                """
            }
        }

        stage('Start Services') {
            steps {
                bat "docker-compose -f %COMPOSE_FILE% up -d"
            }
        }

        stage('Verify Services') {
            steps {
                script {
                    // More robust waiting with retries and timeouts
                    def servicesReady = false
                    def attempts = 0
                    
                    while(!servicesReady && attempts < 12) {
                        attempts++
                        try {
                            def mysqlStatus = bat(
                                script: 'docker exec codely-java_ddd_example-mysql mysqladmin ping -uroot -p"" --silent',
                                returnStatus: true
                            )
                            
                            def esStatus = bat(
                                script: 'curl -s -o nul -w "%{http_code}" http://%ELASTICSEARCH_HOST%:9200/_cluster/health',
                                returnStatus: true
                            )
                            
                            def rabbitStatus = bat(
                                script: 'docker exec codely-java_ddd_example-rabbitmq rabbitmqctl await_startup',
                                returnStatus: true
                            )
                            
                            if (mysqlStatus == 0 && esStatus == 0 && rabbitStatus == 0) {
                                servicesReady = true
                                echo "All services are ready!"
                            } else {
                                echo "Waiting for services... (attempt $attempts)"
                                sleep(time: 10, unit: 'SECONDS')
                            }
                        } catch (Exception e) {
                            echo "Service check failed, retrying... ${e.message}"
                            sleep(time: 10, unit: 'SECONDS')
                        }
                    }
                    
                    if (!servicesReady) {
                        error("Services failed to start within expected time")
                    }
                }
            }
        }

        stage('Build & Test') {
            steps {
                bat """
                    gradlew build test ^
                    -Dspring.datasource.url=jdbc:mysql://host.docker.internal:3306/mooc ^
                    -Dspring.datasource.username=root ^
                    -Dspring.datasource.password= ^
                    -Delasticsearch.host=%ELASTICSEARCH_HOST%:9200 ^
                    -Drabbitmq.host=host.docker.internal
                """
            }
        }
    }

    post {
        always {
            bat "docker-compose -f %COMPOSE_FILE% down -v"
            cleanWs()
        }
    }
}
