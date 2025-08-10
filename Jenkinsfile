pipeline {
    agent {
        docker {
            image 'maven:3.9'
            args '-v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker -v $HOME/.m2:/root/.m2'
            reuseNode true
        }
    }

    environment {
        DOCKER_IMAGE = 'ntongha1/demo-app:${env.BUILD_NUMBER}'
        DOCKER_REGISTRY = 'docker.io'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],  // Changed from jenkins-jobs to main
                    userRemoteConfigs: [[
                        url: 'https://github.com/ntongha1/Java-maven-app.git',
                        credentialsId: 'github-credentials'
                    ]]
                ])
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -B -DskipTests'
                archiveArtifacts 'target/*.jar'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test -B'
                junit 'target/surefire-reports/*.xml'
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-repo',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            docker build -t ${DOCKER_REGISTRY}/${DOCKER_IMAGE} .
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin ${DOCKER_REGISTRY}
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}
                        """
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh """
                        docker stop java-app || true
                        docker rm java-app || true
                        docker run -d \
                          --name java-app \
                          -p 8080:8080 \
                          ${DOCKER_REGISTRY}/${DOCKER_IMAGE}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline succeeded! Image pushed to: ${DOCKER_REGISTRY}/${DOCKER_IMAGE}"
        }
        failure {
            echo "❌ Pipeline failed! Check logs for details."
        }
        always {
            cleanWs()
        }
    }
}
