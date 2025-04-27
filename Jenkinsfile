pipeline {
    agent any
    
    environment {
        // Java configuration
        JAVA_HOME = 'C:\\Program Files\\Java\\jdk-21'
        PATH = "${JAVA_HOME}\\bin;${env.PATH}"
        
        // Docker compose configuration
        COMPOSE_FILE = 'docker-compose.ci.yml'
        
        // Service URLs (using container names as hosts)
        DB_URL = 'jdbc:mysql://codely-java_ddd_example-mysql:3306/mooc'
        ES_HOST = 'codely-java_ddd_example-elasticsearch'
    }

    stages {
        stage('Setup Docker') {
            steps {
                script {
                    // Install Docker on Linux agents
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
                script {
                    // For Windows agents
                    if (isUnix()) {
                        sh """
                            docker exec codely-java_ddd_example-test_server \
                            ./gradlew build test \
                            -Dspring.datasource.url=${DB_URL} \
                            -Dspring.datasource.username=root \
                            -Dspring.datasource.password='' \
                            -Delasticsearch.host=${ES_HOST}
                        """
                    } else {
                        bat """
                            gradlew build test ^
                            -Dspring.datasource.url=${DB_URL} ^
                            -Dspring.datasource.username=root ^
                            -Dspring.datasource.password= ^
                            -Delasticsearch.host=${ES_HOST}
                        """
                    }
                }
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
            // Clean up containers
            sh "docker-compose -f ${COMPOSE_FILE} down"
            cleanWs()
        }
    }
}
