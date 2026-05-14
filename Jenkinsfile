pipeline {
    agent any

    environment {
        MONGO_URI_TEST  = 'mongodb://localhost:27017/portfolioDB_test'
        IMAGE_BACKEND   = "portfolio-api"
        IMAGE_FRONTEND  = "portfolio-react"
        IMAGE_TAG       = "${BUILD_NUMBER}"
    }

    stages {

        stage('Lint') {
            parallel {
                stage('Lint Backend') {
                    steps {
                        dir('portfolio-api') {
                            sh 'npm ci'
                            sh 'npx eslint . --ext .js || true'
                        }
                    }
                }
                stage('Lint Frontend') {
                    steps {
                        dir('React') {
                            sh 'npm ci'
                            sh 'npx eslint src/ --ext .js,.jsx,.ts,.tsx || true'
                        }
                    }
                }
            }
        }

        stage('Start MongoDB') {
            steps {
                sh '''
                    docker rm -f mongo-test || true
                    docker run -d \
                        --name mongo-test \
                        --network host \
                        mongo:7
                    sleep 5
                '''
            }
        }

        stage('Backend test') {
            steps {
                dir('portfolio-api') {
                    sh 'npm test -- --watchAll=false --forceExit'
                }
            }
        }

        stage('Frontend build') {
            steps {
                dir('React') {
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
                sh 'sleep 10'
                sh 'docker compose ps'
            }
        }

        stage('Health check') {
            steps {
                sh '''
                    echo "Vérification backend..."
                    curl -f http://localhost:5000/health || exit 1
                    echo "Vérification frontend..."
                    curl -f http://localhost:3000 || exit 1
                    echo "Tous les services sont operationnels"
                '''
            }
        }
    }

    post {
        always {
            // Nettoyage dans le contexte agent (node already provided by agent any)
            sh 'docker rm -f mongo-test || true'
            sh 'docker image prune -f || true'
            cleanWs()
        }
        success {
            echo "Build #${BUILD_NUMBER} termine avec succes"
        }
        failure {
            echo "Build #${BUILD_NUMBER} echoue - consulter les logs"
        }
    }
}