pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'
        DOCKER_IMAGE = 'cithit/minterjt'                                                                    //<------change this
        IMAGE_TAG = "lab35-build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/srxxm49mps-png/225-lab3-5.git'                                          //<------change this
        KUBECONFIG = credentials('minterjt')                                                         //<------change this
    }

    stages {
        stage('Checkout') {
            steps {
                sh 'test "$(kubectl get nodes -o jsonpath=\'{.items[0].metadata.name}\')" = "minterjt-fl26-pool1-sfjhj-t2bf2"'
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: "${GITHUB_URL}"]]])
            }
        }

        stage('Configure Assigned LoadBalancer IP') {
            steps {
                sh 'kubectl apply -f metallb-production.yaml'
            }
        }

        stage('Lint HTML') {
            steps {
                sh 'npm install htmlhint --save-dev'
                sh 'npx htmlhint *.html'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}", "-f Dockerfile.build .")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Dev Environment') {
            steps {
                script {
                    // Set up Kubernetes configuration using the specified KUBECONFIG
                    // Update deployment-dev.yaml to use the new image tag
                    sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-dev.yaml"
                    sh "kubectl apply -f deployment-dev.yaml"
                    sh "kubectl rollout status deployment/minterjt-lab35-dev --timeout=180s"
                }
            }
        }

       stage('Deploy to Prod Environment') {
            steps {
                script {
                    // Set up Kubernetes configuration using the specified KUBECONFIG
                    //sh "ls -la"
                    sh "kubectl annotate service/minterjt-lab34-prod-service metallb.universe.tf/allow-shared-ip=minterjt-labs --overwrite"
                    sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-prod.yaml"
                    sh "kubectl apply -f deployment-prod.yaml"
                    sh "kubectl rollout status deployment/minterjt-lab35-prod --timeout=180s"
                    sh "kubectl wait --for=jsonpath='{.status.loadBalancer.ingress[0].ip}'=10.48.229.34 service/minterjt-lab35-prod-service --timeout=120s"
                    sh 'curl --fail --retry 10 --retry-delay 3 --retry-connrefused http://10.48.229.34:8081/ | grep "Lab3-5 Works"'
                }
            }
        }

        stage('Check Kubernetes Cluster') {
            steps {
                script {
                    sh "kubectl get all"
                }
            }
        }
    }
    post {

        success {
            slackSend color: "good", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
        unstable {
            slackSend color: "warning", message: "Build Unstable: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
        failure {
            slackSend color: "danger", message: "Build Failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
    }
}

