pipeline {
    agent any
    tools {
        maven 'maven-3.9'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'jenkins-jobs']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/ntongha1/Java-maven-app.git'
                    ]]
                ])
            }
        }

        stage("Build Jar") {
            steps {
                echo "building the application for second webhook testing"
                echo "This is to test that the webhook integration works fine"
                sh 'mvn package'
            }
        }


        stage("Build Image") {
            steps {
                script {
                    try {
                        echo "=== BUILDING DOCKER IMAGE ==="
                        withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                            sh '''
                                docker build -t ntongha1/demo-app:2.0 . | tee docker-build.log
                                cat docker-build.log
                                echo $PASS | docker login -u $USER --password-stdin
                                docker push ntongha1/demo-app:2.0
                            '''
                        }
                    } catch (Exception e) {
                        echo "Docker build failed: ${e}"
                        sh 'cat docker-build.log || true'
                        error("Build failed") 
                    }
                }
            }
        }

        stage("Deploy") {
            steps {
                echo "deploying the application..."
                // Add your deployment commands here
            }
        }
    }
}