pipeline {
    agent {
        docker {
            image 'maven:3.9-eclipse-temurin-21'
            args '-v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker'
        }
    }

    environment {
        DOCKER_IMAGE = 'ntongha1/demo-app:${env.BUILD_ID}'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -B -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test -B -DforkCount=1'
                junit '**/target/surefire-reports/*.xml'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-repo',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            docker build -t $DOCKER_IMAGE .
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push $DOCKER_IMAGE
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    docker stop java-app || true
                    docker rm java-app || true
                    docker run -d --name java-app -p 8080:8080 $DOCKER_IMAGE
                """
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
