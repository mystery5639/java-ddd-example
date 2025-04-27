pipeline {
    agent any

    environment {
        // Docker compose file (adjust path if needed)
        COMPOSE_FILE = 'docker-compose.ci.yml'
        JAVA_HOME = 'C:\\Program Files\\Java\\jdk-21' 
        PATH = "${JAVA_HOME}/bin:${env.PATH}:/usr/local/bin"
    }

    stages {
        stage('Setup Docker') {
            steps {
                script {
                    // Install Docker if not present (Linux agents)
                    if (isUnix()) {
                        sh '''
                            sudo apt-get update -y
                            sudo apt-get install -y docker-ce docker-ce-cli containerd.io
                            sudo systemctl start docker
                            sudo usermod -aG docker $USER
                        '''
                    }
                }
            }
        }

        stage('Start Containers') {
            steps {
                sh "docker-compose -f ${COMPOSE_FILE} up -d"
            }
        }

        stage('Wait for Services') {
            steps {
                sh '''
                    # Wait for MySQL
                    until docker exec codely-java_ddd_example-mysql mysqladmin ping --silent; do
                        echo "Waiting for MySQL..."
                        sleep 5
                    done

                    # Wait for RabbitMQ
                    until docker exec codely-java_ddd_example-rabbitmq rabbitmqctl await_startup; do
                        echo "Waiting for RabbitMQ..."
                        sleep 5
                    done

                    # Wait for Elasticsearch
                    until curl -s http://localhost:9200/_cluster/health; do
                        echo "Waiting for Elasticsearch..."
                        sleep 5
                    done
                '''
            }
        }

        stage('Build & Test') {
            steps {
                sh 'docker exec codely-java_ddd_example-test_server ./gradlew spotlessCheck'
                sh 'docker exec codely-java_ddd_example-test_server ./gradlew build --warning-mode all'
                sh 'docker exec codely-java_ddd_example-test_server ./gradlew test --warning-mode all'
            }
        }

        stage('Deploy') {
            agent {
                label 'windows' // Use Windows agent for PowerShell
            }
            steps {
                powershell 'java -jar build/libs/hello-world-java-V1.jar'
            }
        }
    }

    post {
        always {
            sh "docker-compose -f ${COMPOSE_FILE} down"
            cleanWs() // Clean workspace
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
