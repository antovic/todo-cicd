pipeline {
    agent any 

    environment {
        DOCKER_IMAGE = 'todo-app'
        REPO_URL = 'https://github.com/WELF9I/todo-app.git'
    }

    tools {
        nodejs 'Node-18'  // Assurez-vous que ce nom correspond à votre configuration Jenkins
    }

    stages {
        stage('Cloner le dépôt') {
            steps {
                cleanWs()
                git branch: 'master',
                    url: "${env.REPO_URL}"
                
                script {
                    if (!fileExists('package.json')) {
                        error "Le clonage a échoué : package.json non trouvé"
                    }
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    // Installation de pnpm via npm
                    sh '''
                        curl -f https://get.pnpm.io/v6.js | node - add --global pnpm
                        pnpm install
                    '''
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    try {
                        sh 'pnpm test'
                    } catch (err) {
                        unstable('Tests failed')
                        error "Test stage failed: ${err}"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        sh 'docker build -t $DOCKER_IMAGE .'
                    } catch (err) {
                        error "Docker build failed: ${err}"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    try {
                        sh 'docker-compose up -d'
                    } catch (err) {
                        error "Deployment failed: ${err}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline exécuté avec succès!'
            script {
                sh 'echo "Application déployée avec succès sur http://localhost:80"'
            }
        }
        failure {
            echo 'Échec du pipeline'
            script {
                sh 'docker-compose down || true'
            }
        }
        always {
            script {
                sh 'docker system prune -f || true'
                archiveArtifacts artifacts: '**/log/*.log', allowEmptyArchive: true
            }
        }
    }
}
