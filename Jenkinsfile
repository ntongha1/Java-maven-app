pipeline {
    agent any

    tools {
        maven 'maven-3.9'
        // REMOVED jdk declaration since we'll use container's JDK
    }

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        DOCKER_IMAGE = 'ntongha1/demo-app:${env.BUILD_ID}'
        // Set JAVA_HOME to container's JDK
        JAVA_HOME = '/opt/java/openjdk'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/jenkins-jobs']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/ntongha1/Java-maven-app.git',
                        credentialsId: 'github-credentials'
                    ]]
                ])
            }
        }

        stage('Build') {
            steps {
                sh '''
                    echo "Using Java:"
                    java -version
                    mvn clean package -B -DskipTests
                '''
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test -B -DforkCount=1'
                junit '**/target/surefire-reports/*.xml'
                archiveArtifacts 'target/*.jar'
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
                    // Use single quotes to avoid Groovy interpolation
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
                script {
                    sh """
                        docker stop java-app || true
                        docker rm java-app || true
                        docker run -d \\
                          --name java-app \\
                          -p 8080:8080 \\
                          $DOCKER_IMAGE
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build succeeded! Image: $DOCKER_IMAGE"
        }
        failure {
            echo "❌ Build failed! Check logs."
        }
        always {
            cleanWs()
        }
    }
}
