pipeline {
    agent any
    
    environment {
        DOCKER_USERNAME = 'preet1018'

        // Elastic Beanstalk configuration
        AWS_REGION = 'ap-south-1'
        EB_APP = 'learn-docker-complex-app'
        EB_ENV = 'learn-docker-complex-app-env'
        BUCKET_NAME = 'elasticbeanstalk-ap-south-1-147856894694'
        BUCKET_PATH = 'docker-multi'
    }

    stages {

        stage('Build Test Image') {
            steps {
                sh '''
                docker build \
                -t ${DOCKER_USERNAME}/complex-app-test \
                -f ./client/Dockerfile.dev \
                ./client
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                docker run --rm \
                -e CI=true \
                ${DOCKER_USERNAME}/complex-app-test \
                npm test
                '''
            }
        }

        stage('Build Production Images') {
            steps {
                sh "docker build -t ${DOCKER_USERNAME}/complex-app-client ./client"
                sh "docker build -t ${DOCKER_USERNAME}/complex-app-nginx ./nginx"
                sh "docker build -t ${DOCKER_USERNAME}/complex-app-server ./server"
                sh "docker build -t ${DOCKER_USERNAME}/complex-app-worker ./worker"
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    docker.withRegistry('', 'DockerHubCred') {
                        sh "docker push ${DOCKER_USERNAME}/complex-app-client:latest"
                        sh "docker push ${DOCKER_USERNAME}/complex-app-nginx:latest"
                        sh "docker push ${DOCKER_USERNAME}/complex-app-server:latest"
                        sh "docker push ${DOCKER_USERNAME}/complex-app-worker:latest"
                    }
                }
            }
        }

        stage('Deploy to Elastic Beanstalk') {
            when { branch 'master' }
            steps {
                withCredentials([
    string(credentialsId: 'AWS_ACCESS_KEY', variable: 'AWS_ACCESS_KEY_ID'),
    string(credentialsId: 'AWS_SECRET_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
]) {

                    sh '''
                    zip -r deploy.zip . \
                        -x "*.git*" \
                        -x "node_modules/*"

                    aws s3 cp deploy.zip \
                    s3://$BUCKET_NAME/$BUCKET_PATH/deploy-$BUILD_NUMBER.zip \
                    --region $AWS_REGION

                    aws elasticbeanstalk create-application-version \
                    --application-name $EB_APP \
                    --version-label deploy-$BUILD_NUMBER \
                    --source-bundle S3Bucket="$BUCKET_NAME",S3Key="$BUCKET_PATH/deploy-$BUILD_NUMBER.zip" \
                    --region $AWS_REGION

                    aws elasticbeanstalk update-environment \
                    --environment-name $EB_ENV \
                    --version-label deploy-$BUILD_NUMBER \
                    --region $AWS_REGION
                    '''
                }
            }
        }
    }
}