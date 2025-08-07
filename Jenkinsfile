pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = 'your-registry-url'
    }
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
                        sh """
                            docker login -u $DOCKER_USER -p $DOCKER_PASS $DOCKER_REGISTRY
                            docker build -t $DOCKER_REGISTRY/java-maven-app:${env.BUILD_ID} .
                            docker push $DOCKER_REGISTRY/java-maven-app:${env.BUILD_ID}
                        """
                    }
                }
            }
        }
    }
    post {
        failure {
            echo '❌ Pipeline failed! Investigate logs.'
        }
        success {
            echo '✅ Pipeline succeeded!'
        }
    }
}
