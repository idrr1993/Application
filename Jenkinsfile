pipeline {
    agent any

    environment {
        NODE_VERSION = '20'
        GIT_COMMIT_SHORT = ''
        IMAGE_TAG = ''
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'CloneOption', noTags: false, depth: 0, shallow: false]],
                    userRemoteConfigs: [[url: 'https://github.com/idrr1993/Application.git']]
                ])
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

        stage('Test Docker CLI') {
            steps {
                sh 'docker --version'
            }
        }

        stage('Generate Git Tag') {
            steps {
                script {
                    sh 'git rev-parse --short HEAD > GIT_SHA.txt || echo "0000000" > GIT_SHA.txt'
                    env.GIT_COMMIT_SHORT = readFile('GIT_SHA.txt').trim()
                    env.IMAGE_TAG = "demo-crm:${env.GIT_COMMIT_SHORT}"
                    echo "Using image tag: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image with tag: ${env.IMAGE_TAG}"
                    sh "docker build -t ${env.IMAGE_TAG} ."
                    sh "docker tag ${env.IMAGE_TAG} demo-crm:latest"
                }
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
