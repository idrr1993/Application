pipeline {
    agent any

    environment {
        NODE_VERSION = '20'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Node.js and Yarn') {
            steps {
                sh '''
                    curl -fsSL https://deb.nodesource.com/setup_${NODE_VERSION}.x | bash -
                    apt-get install -y nodejs
                    npm install --global yarn
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'yarn install --frozen-lockfile'
            }
        }

        stage('Build Application') {
            steps {
                sh 'yarn build'
            }
        }

        stage('Start Application and Wait') {
            steps {
                sh '''
                    yarn run dev &
                    for i in {1..10}; do
                        curl http://localhost:3000 && break
                        echo "Waiting for app..."
                        sleep 2
                    done
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t demo-crm:latest .'
            }
        }

        stage('Run E2E Tests with Docker Compose') {
            steps {
                script {
                    try {
                        sh '''
                            docker compose up -d
                            for i in {1..10}; do
                                curl http://localhost:80 && break
                                echo "Waiting for app on port 80..."
                                sleep 2
                            done
                        '''
                    } finally {
                        sh 'docker compose down'
                    }
                }
            }
        }
    }

    post {
        always {
            sh '''
                docker compose down || true
                docker system prune -f || true
            '''
            cleanWs()
        }
    }
}
