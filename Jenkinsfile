pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yukeshh/devopsss"
        K8S_DEPLOYMENT = "k8s/doctor-app-deployment.yaml"
        K8S_SERVICE = "k8s/doctor-app-service.yaml"
        PROMETHEUS_DEPLOYMENT = "k8s/prometheus-deployment.yaml"
        PROMETHEUS_CONFIG = "k8s/prometheus-configmap.yaml"
        GRAFANA_DEPLOYMENT = "k8s/grafana-deployment.yaml"

        // 🛠️ Updated paths for macOS
        KUBECONFIG_PATH = "/Users/adithyan/.kube/config"
        MINIKUBE_HOME_PATH = "/Users/adithyan/.minikube"
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                echo "🧹 Cleaning workspace..."
                cleanWs()
            }
        }

        stage('Clone Repository') {
            steps {
                echo "📥 Cloning repository..."
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/yukeshxxd/chocolateshop',
                        credentialsId: 'github-credentials-id'
                    ]]
                ])
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker Image: ${DOCKER_IMAGE}"
                sh """
                    echo "🔨 Starting Docker Build..."
                    docker build -t ${DOCKER_IMAGE} .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "📤 Pushing Docker Image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credential', usernameVariable: 'DOCKER_HUB_USERNAME', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                    sh """
                        echo "🔐 Logging in to Docker Hub..."
                        echo "\$DOCKER_HUB_PASSWORD" | docker login -u "\$DOCKER_HUB_USERNAME" --password-stdin
                        echo "🚀 Pushing Docker Image: ${DOCKER_IMAGE}..."
                        docker push ${DOCKER_IMAGE}
                        echo "✅ Docker Image Push Successful!"
                    """
                }
            }
        }

        stage('Start Minikube & Deploy App') {
            steps {
                echo "🚀 Starting Minikube and Deploying App..."
                sh """
                    set -e

                    echo "🔧 Setting up Minikube Environment..."
                    export MINIKUBE_HOME=${MINIKUBE_HOME_PATH}
                    export KUBECONFIG=${KUBECONFIG_PATH}

                    echo "🧹 Cleaning old Minikube setup (if any)..."
                    minikube delete || true

                    echo "🔄 Starting Minikube with Docker driver..."
                    minikube start --driver=docker --force

                    echo "📦 Deploying Application to Kubernetes..."
                    kubectl apply -f ${K8S_DEPLOYMENT}
                    kubectl apply -f ${K8S_SERVICE}

                    echo "🔍 Checking Pod Status..."
                    kubectl get pods
                """
            }
        }

        stage('Deploy Monitoring Stack') {
            steps {
                echo "📊 Deploying Prometheus and Grafana..."
                sh """
                    export KUBECONFIG=${KUBECONFIG_PATH}

                    echo "📌 Applying Prometheus Config..."
                    kubectl apply -f ${PROMETHEUS_CONFIG}

                    echo "📌 Applying Prometheus Deployment..."
                    kubectl apply -f ${PROMETHEUS_DEPLOYMENT}

                    echo "📌 Applying Grafana Deployment..."
                    kubectl apply -f ${GRAFANA_DEPLOYMENT}
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "✅ Verifying Kubernetes Deployment..."
                sh """
                    export KUBECONFIG=${KUBECONFIG_PATH}
                    echo "🔍 Listing Pods..."
                    kubectl get pods
                """
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment Successful!"
        }
        failure {
            echo "❌ Deployment Failed! Check logs for details."
        }
    }
}
