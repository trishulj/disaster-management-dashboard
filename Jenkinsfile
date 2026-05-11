pipeline {
    agent any

    tools {
        nodejs 'NodeJS-20'
    }

    environment {
        DEPLOY_DIR = '/var/www/disaster-dashboard'
        APP_NAME   = 'disaster-dashboard'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Cloning latest code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing backend dependencies...'
                sh 'cd backend && npm install'
            }
        }

        stage('Run Simulation') {
            steps {
                echo 'Running simulation briefly for demo...'
                sh '''
                    cd backend
                    timeout 10 node simulate.js || true
                '''
            }
        }

        stage('Build / Copy Frontend') {
            steps {
                echo 'Copying frontend files...'
                sh 'echo "No build step needed for static HTML"'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying to server...'
                sh '''
                    # Clear old deploy and copy fresh files
                    rm -rf ${DEPLOY_DIR}/*
                    cp -r backend  ${DEPLOY_DIR}/backend
                    cp -r frontend ${DEPLOY_DIR}/frontend

                    # Copy .env file
                    cp backend/..env ${DEPLOY_DIR}/backend/.env || true

                    cd ${DEPLOY_DIR}/backend

                    # Start or restart the app using PM2
                    if pm2 describe ${APP_NAME} > /dev/null 2>&1; then
                        pm2 restart ${APP_NAME}
                        echo "App restarted"
                    else
                        pm2 start server.js --name ${APP_NAME}
                        pm2 save
                        echo "App started fresh"
                    fi
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline SUCCESS — app is live!'
        }
        failure {
            echo 'Pipeline FAILED — check logs above.'
        }
    }
}
