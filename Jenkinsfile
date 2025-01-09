pipeline {
    agent any

    environment {
        NODE_VERSION = '18.14.0'
        FRONTEND_DIR = 'frontend'
        BACKEND_DIR = 'backend'
        S3_BUCKET = 'wanderlust-jyotirmoy'
        AWS_REGION = 'ap-south-1'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Correct. Checks out the 'main' branch of the specified Git repository.
                git branch: 'main', url: 'https://github.com/jyotirmoy44/wanderlust.git'
            }
        }

        stage('Setup Node.js') {
            steps {
                // Ensure nvm setup persists for the current and subsequent steps
                sh '''
                export NVM_DIR="$HOME/.nvm"
                curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
                [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
                nvm install ${NODE_VERSION}
                nvm alias default ${NODE_VERSION}
                nvm use default
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                // Correctly navigates to frontend and backend directories to install dependencies
                sh '''
                cd ${FRONTEND_DIR} && npm install --force
                cd ../${BACKEND_DIR} && npm install --force
                '''
            }
        }

        stage('Build Frontend') {
            steps {
                // Correct. Ensures build output is verified
                sh '''
                cd ${FRONTEND_DIR}
                npm run build || exit 1
                echo "Build command completed. Checking build directory..."
                ls -al build
                '''
            }
        }

        stage('Deploy Frontend') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-credentials-id', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    // Correct. Syncs build files to S3 and ensures it cleans old files.
                    sh '''
                    aws sts get-caller-identity
                    aws s3 sync ${FRONTEND_DIR}/build s3://${S3_BUCKET} --delete
                    '''
                }
            }
        }

        stage('Build and Deploy Backend') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                    cd ${BACKEND_DIR}
                    docker build -t devopsjyotirmoy/backend-app:latest . || exit 1
                    docker tag devopsjyotirmoy/backend-app:latest devopsjyotirmoy/backend-app:${BUILD_NUMBER}
                    echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                    docker push devopsjyotirmoy/backend-app:latest
                    docker push devopsjyotirmoy/backend-app:${BUILD_NUMBER}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline execution failed. Check logs.'
        }
    }
}
