pipeline {
    agent any

    environment {
        IMAGE_BACKEND  = "portfolio-api"
        IMAGE_FRONTEND = "portfolio-react"
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }

    stages {

        stage('Start MongoDB') {
            steps {
                sh '''
                    docker rm -f mongo-test || true
                    docker run -d \
                        --name mongo-test \
                        --network host \
                        mongo:7
                    sleep 8
                '''
            }
        }

        stage('Backend install and test') {
            steps {
                dir('portfolio-api') {
                    withEnv(['MONGO_URI=mongodb://localhost:27017/portfolioDB_test']) {
                        sh 'npm ci'
                        sh 'npm test -- --watchAll=false --forceExit'
                    }
                }
            }
        }

        stage('Frontend install and build') {
            steps {
                dir('React') {
                    sh 'npm ci'
                    sh 'npm run build'
                }
            }
        }

        stage('Archive build') {
            steps {
                archiveArtifacts artifacts: 'React/build/**', allowEmptyArchive: true
            }
        }

        stage('Docker build') {
            parallel {
                stage('Build image backend') {
                    steps {
                        dir('portfolio-api') {
                            sh "docker build -t ${IMAGE_BACKEND}:${IMAGE_TAG} ."
                            sh "docker tag ${IMAGE_BACKEND}:${IMAGE_TAG} ${IMAGE_BACKEND}:latest"
                        }
                    }
                }
                stage('Build image frontend') {
                    steps {
                        dir('React') {
                            sh "docker build -t ${IMAGE_FRONTEND}:${IMAGE_TAG} ."
                            sh "docker tag ${IMAGE_FRONTEND}:${IMAGE_TAG} ${IMAGE_FRONTEND}:latest"
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker compose down || true'
                sh 'docker compose up -d --build'
                sh 'sleep 15'
                sh 'docker compose ps'
            }
        }

        stage('Health check') {
            steps {
                sh 'curl -f http://localhost:5000/health || echo "Backend pas encore pret"'
                sh 'curl -f http://localhost:3000 || echo "Frontend pas encore pret"'
            }
        }
    }

    post {
        always {
            sh 'docker rm -f mongo-test || true'
            cleanWs()
        }
        success {
            echo "Build termine avec succes"
        }
        failure {
            echo "Build echoue - consulter les logs"
        }
    }
}
