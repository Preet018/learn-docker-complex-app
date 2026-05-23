pipeline {
    agent any
    
    environment {
        DOCKER_USERNAME = 'preet1018'

        ANSIBLE_INVENTORY = "Ansible/inventory.ini"
        ANSIBLE_PLAYBOOK  = "Ansible/deploy.yaml"
    }

    stages {
        stage('Build Test Image') {
            steps {
                sh '''
                docker build \
                -t preet1018/complex-app-test \
                -f ./client/Dockerfile.dev \
                ./client
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh 'docker run --rm -e CI=true preet1018/complex-app-test npm test'
            }
        }

        stage('Build Production Images') {
            steps {
                sh 'docker build -t preet1018/complex-app-client ./client'
                sh 'docker build -t preet1018/complex-app-nginx ./nginx'
                sh 'docker build -t preet1018/complex-app-server ./server'
                sh 'docker build -t preet1018/complex-app-worker ./worker'
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
    }
}