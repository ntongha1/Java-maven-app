pipeline {
    agent any

    tools {
        maven 'maven-3.9' // Make sure this is configured in Jenkins Global Tool Configuration
    }

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        // Retry pipeline step on agent loss or flaky issues (Jenkinsfile stability)
        timeout(time: 15, unit: 'MINUTES')
    }

    environment {
        DOCKER_IMAGE = 'ntongha1/demo-app:2.0'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    retry(2) {
                        git branch: 'jenkins-jobs', url: "https://github.com/ntongha1/Java-maven-app.git"
                    }
                    sh 'ls -la'
                }
            }
        }

        stage('Build Jar') {
            steps {
                script {
                    echo "🔧 Building the application..."
                    echo "📦 Running Maven package command"
                    retry(2) {
                        sh 'mvn clean package -B'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "🐳 Building Docker image..."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        retry(2) {
                            sh '''
                                docker build -t $DOCKER_IMAGE .
                                echo $PASS | docker login -u $USER --password-stdin
                                docker push $DOCKER_IMAGE
                            '''
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "🚀 Deploying the application..."
                    // Add actual deploy script/command here
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build and deployment completed successfully."
        }

        failure {
            echo "❌ Build or deployment failed. Please check logs."
        }

        always {
            echo "🧼 Cleaning up workspace..."
            cleanWs()
        }
    }
}
