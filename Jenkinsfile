pipeline {
    agent any
    
    environment {
        GITHUB_REPO_URL = 'https://github.com/Preet018/learn-docker-complex-app.git'
        DOCKER_USERNAME = 'preet1018'
        IMAGE_TAG = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()

        ANSIBLE_INVENTORY = "Ansible/inventory.ini"
        ANSIBLE_PLAYBOOK  = "Ansible/deploy.yaml"
        KUBECONFIG = "/home/chand/.kube/config"
    }
    stages {
        stage('Build Test Image') {
            steps {
                sh "docker build -t ${DOCKER_USERNAME}/multi-client-test -f client/Dockerfile.dev ./client"
            }
        }
        stage('Run Tests') {
            steps {
                sh "docker run --rm -e CI=true ${DOCKER_USERNAME}/multi-client-test npm test"
            }
        }
        stage('Build Docker Images') {
            steps {
                script {
                    // Build images with version tag
                    sh "docker build -t ${DOCKER_USERNAME}/multi-client:${IMAGE_TAG} -f client/Dockerfile ./client"
                    // sh "docker build -t ${DOCKER_USERNAME}/multi-nginx:${IMAGE_TAG} -f nginx/Dockerfile ./nginx"
                    sh "docker build -t ${DOCKER_USERNAME}/multi-server:${IMAGE_TAG} -f server/Dockerfile ./server"
                    sh "docker build -t ${DOCKER_USERNAME}/multi-worker:${IMAGE_TAG} -f worker/Dockerfile ./worker"

                    // Build images with latest tag
                    sh "docker tag ${DOCKER_USERNAME}/multi-client:${IMAGE_TAG} ${DOCKER_USERNAME}/multi-client:latest"
                    // sh "docker tag ${DOCKER_USERNAME}/multi-nginx:${IMAGE_TAG} ${DOCKER_USERNAME}/multi-nginx:latest"
                    sh "docker tag ${DOCKER_USERNAME}/multi-server:${IMAGE_TAG} ${DOCKER_USERNAME}/multi-server:latest"
                    sh "docker tag ${DOCKER_USERNAME}/multi-worker:${IMAGE_TAG} ${DOCKER_USERNAME}/multi-worker:latest"
                }
            }
        }
        stage('Push Docker Images') {
            steps {
                script {
                    docker.withRegistry('', 'DockerHubCred') {
                        // Push versioned tags (required so Kubernetes can pull the SHA-tagged image during rollout)
                        sh "docker push ${DOCKER_USERNAME}/multi-client:${IMAGE_TAG}"
                        // sh "docker push ${DOCKER_USERNAME}/multi-nginx:${IMAGE_TAG}"
                        sh "docker push ${DOCKER_USERNAME}/multi-server:${IMAGE_TAG}"
                        sh "docker push ${DOCKER_USERNAME}/multi-worker:${IMAGE_TAG}"

                        // Push latest tags
                        sh "docker push ${DOCKER_USERNAME}/multi-client:latest"
                        // sh "docker push ${DOCKER_USERNAME}/multi-nginx:latest"
                        sh "docker push ${DOCKER_USERNAME}/multi-server:latest"
                        sh "docker push ${DOCKER_USERNAME}/multi-worker:latest"
                    }
                }
            }
        }
        stage ('Deployment') {
            steps {
                sh "kubectl apply -f k8s"
            }
        }
        stage ('Update images on each deployment') {
            steps {
                script {
                    sh "kubectl set image deployment/client-deployment client=${DOCKER_USERNAME}/multi-client:${IMAGE_TAG}"
                    sh "kubectl set image deployment/server-deployment server=${DOCKER_USERNAME}/multi-server:${IMAGE_TAG}"
                    sh "kubectl set image deployment/worker-deployment worker=${DOCKER_USERNAME}/multi-worker:${IMAGE_TAG}"
                    
                }
            }
        }
    }
}