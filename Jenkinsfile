pipeline {
    agent any
    
    environment {
        // New environment variables you requested
        JAVA_HOME = 'C:\\Program Files\\Java\\jdk-21'
        PATH = "${JAVA_HOME}\\bin;${env.PATH}"
        COMPOSE_FILE = 'docker-compose.ci.yml'
        
        // Existing environment variables
        COMPOSE_DOCKER_CLI_BUILD = '1'
        DOCKER_BUILDKIT = '1'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Setup Docker') {
            steps {
                sh '''
                    sudo apt-get update
                    sudo apt-get install -y docker-ce docker-ce-cli containerd.io
                    sudo systemctl start docker
                '''
            }
        }
        
        stage('Start Containers') {
            steps {
                // Now using COMPOSE_FILE environment variable
                sh 'docker compose -f $COMPOSE_FILE up -d'
            }
        }
        
        stage('Wait for Services') {
            steps {
                script {
                    timeout(time: 3, unit: 'MINUTES') {
                        waitUntil {
                            try {
                                sh 'docker exec codely-java_ddd_example-mysql mysqladmin ping --silent'
                                return true
                            } catch (Exception e) {
                                echo 'Waiting for MySQL...'
                                sleep 5
                                return false
                            }
                        }
                    }
                    
                    timeout(time: 2, unit: 'MINUTES') {
                        waitUntil {
                            try {
                                sh 'docker exec codely-java_ddd_example-rabbitmq rabbitmqctl await_startup'
                                return true
                            } catch (Exception e) {
                                echo 'Waiting for RabbitMQ...'
                                sleep 5
                                return false
                            }
                        }
                    }
                    
                    timeout(time: 3, unit: 'MINUTES') {
                        waitUntil {
                            try {
                                sh 'curl -s http://localhost:9200/_cluster/health'
                                return true
                            } catch (Exception e) {
                                echo 'Waiting for Elasticsearch...'
                                sleep 5
                                return false
                            }
                        }
                    }
                }
            }
        }
        
        stage('Code Formatting Check') {
            steps {
                sh 'docker exec codely-java_ddd_example-test_server ./gradlew spotlessCheck'
            }
        }
        
        stage('Build Project') {
            steps {
                sh 'docker exec codely-java_ddd_example-test_server ./gradlew build --warning-mode all'
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'docker exec codely-java_ddd_example-test_server ./gradlew test --warning-mode all'
            }
        }
    }
    
    post {
        always {
            // Using COMPOSE_FILE environment variable here too
            sh 'docker compose -f $COMPOSE_FILE down'
        }
    }
}
