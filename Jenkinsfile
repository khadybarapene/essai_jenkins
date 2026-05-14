pipeline {
    agent any

    environment {
        MONGO_URI_TEST  = 'mongodb://localhost:27017/portfolioDB_test'
        IMAGE_BACKEND   = "portfolio-api"
        IMAGE_FRONTEND  = "portfolio-react"
        IMAGE_TAG       = "${BUILD_NUMBER}"
        DOCKER_USER     = credentials('dockerhub-credentials')
    }

    stages {

        // ─── 1. LINT ──────────────────────────────────────────────────────────
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

        // ─── 2. TESTS ─────────────────────────────────────────────────────────
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

        // ─── 3. BUILD ─────────────────────────────────────────────────────────
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

        // ─── 4. DOCKER BUILD ──────────────────────────────────────────────────
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

        // ─── 5. PUSH REGISTRY ─────────────────────────────────────────────────
        stage('Push to registry') {
            when { branch 'main' }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push $DOCKER_USER/${IMAGE_BACKEND}:${IMAGE_TAG}"
                    sh "docker push $DOCKER_USER/${IMAGE_BACKEND}:latest"
                    sh "docker push $DOCKER_USER/${IMAGE_FRONTEND}:${IMAGE_TAG}"
                    sh "docker push $DOCKER_USER/${IMAGE_FRONTEND}:latest"
                }
            }
        }

        // ─── 6. DÉPLOIEMENT ───────────────────────────────────────────────────
        stage('Deploy') {
            when { branch 'main' }
            steps {
                sh 'docker compose down || true'
                sh 'docker compose up -d --build'
                sh 'sleep 10' // laisser le temps aux healthchecks
                sh 'docker compose ps'
            }
        }

        // ─── 7. HEALTH CHECK POST-DÉPLOIEMENT ─────────────────────────────────
        stage('Health check') {
            when { branch 'main' }
            steps {
                sh '''
                    echo "Vérification backend..."
                    curl -f http://localhost:5000/health || exit 1

                    echo "Vérification frontend..."
                    curl -f http://localhost:3000 || exit 1

                    echo "✅ Tous les services sont opérationnels"
                '''
            }
        }
    }

    post {
        always {
            sh 'docker rm -f mongo-test || true'
            sh 'docker image prune -f || true'
            cleanWs()
        }
        success {
            mail(
                to: 'ton-email@example.com',
                subject: "✅ Build #${BUILD_NUMBER} réussi — ${JOB_NAME}",
                body: """
                    Pipeline terminé avec succès.

                    Job      : ${JOB_NAME}
                    Build    : #${BUILD_NUMBER}
                    Branche  : ${GIT_BRANCH}
                    Durée    : ${currentBuild.durationString}
                    Lien     : ${BUILD_URL}
                """
            )
        }
        failure {
            mail(
                to: 'ton-email@example.com',
                subject: "❌ Build #${BUILD_NUMBER} échoué — ${JOB_NAME}",
                body: """
                    Le pipeline a échoué.

                    Job      : ${JOB_NAME}
                    Build    : #${BUILD_NUMBER}
                    Branche  : ${GIT_BRANCH}
                    Logs     : ${BUILD_URL}console
                """
            )
        }
    }
}