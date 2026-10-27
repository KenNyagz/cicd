pipeline {
    agent any

    environment {
        // CHANGE THIS to your Docker Hub username
        IMAGE     = 'YOUR_DOCKERHUB_USER/devops-capstone'
        TAG       = "${env.BUILD_NUMBER}"
        CONTAINER = 'devops-app'
    }

    options {
        timeout(time: 15, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                // Builds only up to the "test" stage; fails if pytest fails
                sh 'docker build --target test -t ${IMAGE}:test .'
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build --target runtime -t ${IMAGE}:${TAG} -t ${IMAGE}:latest .'
            }
        }

        stage('Push Image') {
            steps {
                // Create a "Username with password" credential in Jenkins with ID: dockerhub-creds
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                  usernameVariable: 'DH_USER',
                                                  passwordVariable: 'DH_PASS')]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker push ${IMAGE}:${TAG}
                        docker push ${IMAGE}:latest
                    '''
                }
            }
        }

        stage('Deploy') {
            when { branch 'main' }   // Only deploy from main (needs a Multibranch Pipeline job)
            steps {
                sh '''
                    docker rm -f ${CONTAINER} || true
                    docker run -d --name ${CONTAINER} -p 5000:5000 \
                        --restart unless-stopped ${IMAGE}:${TAG}
                '''
            }
        }

        stage('Smoke Test') {
            when { branch 'main' }
            steps {
                sh '''
                    sleep 5
                    docker exec ${CONTAINER} python -c \
                      "import urllib.request; urllib.request.urlopen('http://localhost:5000/health')"
                '''
            }
        }
    }

    post {
        always  { sh 'docker logout || true' }
        success { echo "Build ${TAG} deployed successfully." }
        failure { echo "Build ${TAG} FAILED. Check the stage logs above." }
    }
}
