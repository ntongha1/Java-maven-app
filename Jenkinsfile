pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u $DOCKER_USER --password-stdin
                            docker build -t java-maven-app:latest .
                            # docker push java-maven-app:latest  # Uncomment if you have a registry
                        '''
                    }
                }
            }
        }
    }
    
    post {
        failure {
            echo '❌ Pipeline failed! Check logs above.'
        }
        success {
            echo '✅ Pipeline succeeded!'
        }
    }
}
